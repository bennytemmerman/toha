---
title: "CICD pipeline AWX inventory"
date: 2026-08-20T12:00:15+02:00
hero: /images/posts/cicd.png
description: Pipeline between gitlab and AWX inventory sync
theme: Toha
menu:
  sidebar:
    name: CICD inventory pipeline
    identifier: cicd_inventoryPipeline_post
    parent: cat-linux
    weight: 305
---

# Closing the loop: A GitOps sync pipeline between GitLab and AWX

*An architectural assessment of a small pipeline solving a real operational problem.*

## Summary

This post documents a CI/CD pipeline that closes a gap between "infrastructure as code" and "infrastructure that's actually running": a GitLab-hosted Ansible inventory file that updates AWX automatically, in the correct order, without a human clicking sync buttons.

It's a small piece of automation, one pipeline, one shell script, two AWX API calls. It's a useful case study because the design decisions behind it generalize well beyond this specific stack: idempotency, ordering guarantees, failure handling, and the discipline of not letting "it works" substitute for "it's correct."

## The problem being solved

AWX (the upstream open-source project behind Red Hat Ansible Automation Platform's controller) manages two things relevant here:

- **Projects**  
A synced copy of a Git repository containing playbooks, roles, and in this case, inventory source files.
- **Inventory Sources**  
A definition of where AWX should pull dynamic inventory data from, and how often. In this environment we have a source file living in that same Git-backed project, defining hosts and host variables.

**The problem:**  
these two objects are decoupled in AWX. Updating the Git project does *not* automatically re-trigger a re-read of the inventory source. Left alone, that means:

1. A GitOps-minded engineer edits hosts.yml in GitLab, adds a host, commits, pushes.
2. AWX's Project still points at the old commit until someone manually clicks "Sync" on the project.
3. Even after that sync, the Inventory Source itself needs a *separate* manual sync to actually re-parse the updated file and reflect the new host in AWX's inventory.
4. Until both of those manual steps happen, any job template referencing that inventory runs against stale data, silently.

That silent staleness is the real risk. Nothing errors. Nothing warns you. Automation just quietly targets the wrong host list until someone remembers to click two buttons in the right order.

## The case for automating this

There's a reasonable counter-argument here worth stating honestly: **is this worth automating at all**, versus just training the team to click "Sync" twice after inventory changes?

For a single administrator on a small environment, arguably not. The manual click-twice workflow is a five-second habit. The case for automation strengthens specifically because of three properties of this environment:

- **Multi-tenant scale.**  
Manual sync steps that are "acceptable" for one inventory become an operational tax that scales linearly with tenant count, and linearly-scaling manual toil is exactly the kind of thing that should be automated early.
- **Ordering matters and is easy to get backwards.**  
Syncing the inventory source *before* the project sync completes means it reads against the previous commit, a subtle bug that produces a plausible-looking but wrong result, which is worse than an obvious failure. Humans reliably get this backwards under time pressure, a pipeline doesn't.
- **GitOps as a stated principle, not just a buzzword.**  
If the source of truth for inventory is "whatever's in Git," then reality (AWX's live state) should follow Git automatically. A manual sync step is a crack in that principle, it means Git can silently drift out of sync with the system that acts on it, for an arbitrary length of time, with no signal that drift has occurred.

The architectural bet here is that a small amount of automation complexity (one pipeline, ~80 lines of shell) is worth paying to eliminate an entire class of "did someone forget to sync" incidents.

## Decision points

A few decisions were made explicitly, with alternatives considered and rejected. Documenting the "why not" is often more valuable than documenting the "what," since it's the part that doesn't survive in the code itself.

### 1: Poll for job completion vs. fire-and-forget

**Chosen:** poll `project_updates/<id>/` and `inventory_updates/<id>/` until `status` reaches a terminal state, before proceeding to the next step.

**Rejected alternative:** fire both update triggers back-to-back with no wait, relying on AWX's internal job queue to serialize them correctly.

**Why:** AWX's API returns `202 Accepted` immediately on trigger, the HTTP response tells you the job was *queued*, not that it *finished*. Firing the inventory sync immediately after triggering the project sync risks a race condition: if the inventory sync starts (or even just reads project files) before the project update has actually completed pulling the latest commit, it could sync against stale data indistinguishable from a "successful" run. Polling trades a small amount of pipeline runtime (typically single-digit seconds in this environment) for a hard ordering guarantee. For a fleet-scale operation, that trade is clearly worth it.

### 2: Shell script in the repo vs. inline YAML

**Chosen:** a versioned `scripts/awx-sync.sh`, invoked from a minimal `.gitlab-ci.yml`.

**Rejected alternative:** the entire polling/curl logic embedded directly in the `script:` block of the YAML file.

**Why:** YAML multi-line block scalars are fragile under real-world editing conditions, indentation drift, line-ending inconsistency, and copy-paste from non-plain-text sources all produce parse errors that are hard to diagnose from the CI job log alone. Externalizing logic into an actual shell script means it can be linted (`shellcheck`), tested locally (`./scripts/awx-sync.sh` with exported env vars), and diffed cleanly in merge requests, none of which is true of logic buried in YAML.

### 3: Bash/curl vs. a purpose-built AWX CLI or SDK

**Chosen:** raw `curl` + `jq` against the AWX REST API.

**Rejected alternative:** `awx` CLI tool (the official AWX/Tower command-line client) or the `ansible.controller` Ansible collection modules.

**Why:** the CLI and collection approaches are arguably more idiomatic and offer better structured error handling out of the box. They were set aside here specifically to minimize the CI image's dependency surface, `curl` and `jq` are near-universally available or trivially installable in any minimal image, whereas the AWX CLI or Ansible collection requires a Python environment with additional packages, which adds image size, install time, and version-compatibility surface area to maintain. For a two-endpoint interaction, raw REST calls were judged the lower-maintenance option. **This is a decision I'd revisit** if the pipeline's scope grows beyond these two calls, at that point, the CLI's built-in job-wait functionality (`awx project_update --monitor`) would eliminate the custom polling logic entirely and is worth the added dependency.

### 4: Polling interval and timeout values

**Chosen:** 5-second poll interval, 300-second timeout, both exposed as pipeline variables rather than hardcoded.

**Why configurable:** project sync duration is a function of repo size, role/collection install steps, and network conditions to the Git remote. Values that will differ across projects and will drift over time as a given project grows. Hardcoding either value ties pipeline reliability to today's conditions. Exposing them as `variables:` in `.gitlab-ci.yml` means tuning them is a one-line change, not a script edit, and different projects reusing this pattern can override them per-job without touching the shared script.

### 5: Fail closed, not fail open

**Chosen:** any non-terminal-success status (`failed`, `error`, `canceled`, or timeout) causes the pipeline job to exit non-zero and *does not* proceed to the inventory sync step.

**Why:** the alternative, logging a warning but continuing anyway, optimizes for pipeline "greenness" over correctness. A failed project sync followed by an inventory sync that proceeds anyway produces the exact silent-staleness problem this pipeline exists to prevent, just moved one step later. A red pipeline that clearly states "project sync failed" is strictly more useful than a green one that hides a partial failure.

## Architecture and flow

![GitlabSyncFlow](images/posts/GitlabSyncFlow.png)

**Trigger:**  
a push to `main` on the GitLab repository. Scoped via `only: [main]` so feature-branch commits don't fire syncs against live AWX state.

**Step 1 - Project update.**  
`POST /api/v2/projects/<id>/update/` re-clones the Git repository AWX has configured as this project's source, pulling the latest commit on the configured branch. Response: `202 Accepted` with a `project_update` job ID.

**Step 2 - Poll project update status.**  
`GET /api/v2/project_updates/<job_id>/` repeatedly, checking the `status` field, until it reaches `successful` (proceed) or `failed`/`error`/`canceled` (abort) or the configured timeout elapses (abort).

**Step 3 - Inventory source update.**  
Only reached after Step 2 succeeds. `POST /api/v2/inventory_sources/<id>/update/` tells AWX to re-parse the inventory source which, because it lives in the project just synced in Step 1, now reflects the newly pushed hostfile content.

**Step 4 - Poll inventory update status.**  
Same polling pattern as Step 2, against `/api/v2/inventory_updates/<job_id>/`.

**Terminal state:**  
the CI job exits 0 only if both updates reached `successful`. Any other outcome exits non-zero, failing the pipeline visibly.

### Key commands

Triggering a project sync:
```bash
curl -sk -X POST "$AWX_URL/api/v2/projects/$PROJECT_ID/update/" \
  -H "Authorization: Bearer $AWX_TOKEN" \
  -H "Content-Type: application/json"
```

Polling job status:
```bash
curl -sk -X GET "$AWX_URL/api/v2/project_updates/$JOB_ID/" \
  -H "Authorization: Bearer $AWX_TOKEN" | jq -r '.status'
```

Discovering the correct Inventory Source ID (a distinct object from the Inventory itself):
```bash
curl -sk "$AWX_URL/api/v2/inventories/$INVENTORY_ID/inventory_sources/" \
  -H "Authorization: Bearer $AWX_TOKEN" | jq '.results[] | {id, name}'
```

Configuring a host-level enable/disable toggle so AWX's GUI reflects an `ansible_host_enabled` variable in the hostfile, set on the Inventory Source itself, not inferred from the file:
- **Enabled Variable:** `ansible_host_enabled`
- **Enabled Value:** `true`

![GitlabInventoryHostVar](images/posts/gitlab_inventoryHostvar.png)

## Operational considerations

**Trigger timing.**  
Push-to-main is the right trigger for this use case. Inventory changes are infrequent and deliberate, not continuous, so a push-based trigger avoids both the complexity and the wasted job runs of polling-based or scheduled sync approaches.

**Idempotency.** Both AWX endpoints are safe to call repeatedly. An update on an already-current project or inventory source simply completes quickly with no changes, rather than erroring. This matters because it means the pipeline is safe to re-run manually (e.g., after a transient network failure) without needing to reason about whether state was already correct.

**Failure recovery.**  
If the pipeline fails at the polling-timeout stage specifically (as opposed to an explicit `failed` status), that's a signal worth treating differently from a hard failure. It may mean the job is simply slow (a large project, a slow Git remote) rather than broken, and the correct operator response is checking the job directly in the AWX UI before assuming a bug.

**Secrets handling.** `AWX_URL` and `AWX_TOKEN` are stored as protected/masked GitLab CI/CD variables, not committed to the repository. `PROJECT_ID` and `INVENTORY_SOURCE_ID` are treated as configuration, not secrets. They're stable, low-sensitivity identifiers, though scoping them as CI/CD variables rather than hardcoding still centralizes them for easier updates if IDs change.

## What I'd do differently at scale

Being honest about the pipeline's current limitations, as I would in an architecture review:

- **No retry/backoff on transient network failures.**  
A single dropped connection to AWX mid-poll currently fails the whole job. A production-grade version should distinguish "AWX said no" from "the network hiccuped" and retry the latter a bounded number of times.
- **No parallelization for multi-inventory-source projects.**  
If a single project backs multiple inventory sources, this pipeline's linear one-project-then-one-inventory-source pattern would need to become one-project-then-N-inventory-sources-in-parallel, with aggregated success/failure reporting.
- **No pipeline-level locking.**  
Two rapid successive pushes to `main` could trigger overlapping pipeline runs racing against the same AWX objects. GitLab's `resource_group:` feature would serialize this correctly and is a low-effort addition worth making before this pattern is copied across many more projects.
- **Credentials are a single shared token.**  
At current scale, one AWX service-account token for this pipeline is reasonable. At higher tenant counts, scoping tokens per-project (least privilege, easier revocation blast-radius) becomes worth the added credential-management overhead.
- **CLI/collection migration threshold.**  
As noted in decision 3, if this pipeline's scope grows beyond two endpoints, migrating to the `ansible.controller` collection or AWX CLI would eliminate custom polling logic and reduce the maintenance surface of hand-rolled HTTP status handling.

## Lessons learned

- **Async APIs need to be treated as async in the pipeline design, not just the API contract.**  
A `202 Accepted` response is an invitation to poll, not a completion signal. Designing the *ordering guarantee* was the actual engineering problem here, not the HTTP calls themselves.
- **Decoupled systems drift silently unless something actively closes the loop.**  
AWX's Project and Inventory Source being separate objects is a reasonable design choice on AWX's part, but it pushes the responsibility for keeping them in sync onto whoever operates the system. Automating that responsibility away is a small investment that removes an entire failure mode.
- **The value of a pipeline like this isn't the code, it's the guarantee.**  
Anyone can write a curl command that triggers a sync. The engineering is in making the *ordering, failure handling, and idempotency* correct enough that the pipeline can be trusted to run unattended, indefinitely, without a human double-checking its work.
- **Small automation projects are good places to practice architectural discipline.**  
Documenting rejected alternatives (Decision 3, CLI vs. raw REST) matters just as much on an 80-line script as it would on a much larger system. The habit of writing down *why not* is what makes a design defensible later, when requirements change and someone has to decide whether the original trade-off still holds.

## Closing

This pipeline is intentionally small, but it was designed the way a much larger one would be: with explicit trade-offs, a fail-closed failure mode, and an honest list of what it doesn't yet handle. That's the difference between a script that happens to work and a piece of infrastructure someone else can safely build on.
