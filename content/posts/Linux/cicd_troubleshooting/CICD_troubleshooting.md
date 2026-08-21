---
title: "CICD troubleshooting"
date: 2026-08-20T12:00:15+02:00
hero: /images/posts/cicd.png
description: Chasing ghosts
theme: Toha
menu:
  sidebar:
    name: CICD troubleshooting
    identifier: cicd_troubleshooting_post
    parent: cat-linux
    weight: 304
---
# Chasing ghosts: Debugging a GitLab CI → AWX sync pipeline

*How a five-line curl command turned into a full afternoon of troubleshooting.*

## The goal

Simple enough on paper: after pushing changes to an Ansible inventory hostfile in GitLab, automatically trigger AWX to:

1. Sync the project (pull the latest code from GitLab)
2. Wait for that sync to finish successfully
3. Only then trigger the inventory source sync, so AWX picks up the updated hostfile

The starting point was a minimal .gitlab-ci.yml job that just fired a single curl POST at AWX's project update endpoint.
No waiting, no chaining, no error handling. Good enough to prove the token and URL worked, not good enough for a real pipeline.

What followed was a chain of failures that each looked unrelated, but were really just successive layers of the same onion. 
Here's the full trail, in case it saves someone else the same afternoon.

## 1: YAML won't parse the script block

**Symptom:**
```
jobs:sync_awx:script config should be a string or a nested array of strings up to 10 levels deep
```

The first thing was to check the bash logic inside the "script:-block". But this is a **YAML parsing error**, not a shell error. It means GitLab couldn't even interpret the "script:-key" as a valid string/array before a single command ran.

**Fix:** Move the multi-line logic out of the YAML entirely and into a proper shell script file (scripts/awx-sync.sh), called from a one-line "script:-block:"

```yaml
script:
  - chmod +x ./scripts/awx-sync.sh
  - ./scripts/awx-sync.sh
```

**Lesson:** if the error mentions YAML structure ("should be a string..."), the problem is in the .gitlab-ci.yml-file itself. Don't go debugging the script content until the pipeline editor's linter (or glab ci lint) gives the YAML a clean bill of health.

## 2: Classic CRLF trap

**Symptom:**
```
bash: ./scripts/awx-sync.sh: /bin/sh^M: bad interpreter: No such file or directory
```

That `^M` is a carriage return sitting right after the shebang line. The script had been touched by a Windows editor at some point, saving it with `\r\n` line endings instead of `\n`. The kernel then tries to resolve an interpreter literally named `/bin/sh\r`, which of course doesn't exist.

**Fix:**
```bash
dos2unix scripts/awx-sync.sh
# or, without dos2unix installed:
sed -i 's/\r$//' scripts/awx-sync.sh
```
Or use the "Select End of Line Sequence" in VScode and change to LF.

**The permanent fix** — add a `.gitattributes` file so Git normalizes line endings on every checkout, regardless of which OS or editor touches the file next:

```
*.sh text eol=lf
.gitlab-ci.yml text eol=lf
```

Followed by a one-time repo-wide normalization: 
```
git add --renormalize .
```

CRLF issues are sneaky, specifically because the resulting errors never mention line endings. They show up as "file not found," "bad interpreter," or cryptic YAML parse failures. Any time an error looks like it *shouldn't* be possible given the file clearly exists and is clearly valid, checking for `\r` characters is a cheap first move.

## 3: Missing jq in CI image

**Symptom:**
```
./scripts/awx-sync.sh: 33: jq: not found
```

I  had the following container image (`curlimages/curl:latest`) configured in my pipeline, that only ships `curl`, not `jq`. So switch to an image that already bundles both.

```yaml
image: badouralix/curl-jq:latest
```

## 4: Is it even running in a container?

Before chasing the `jq` fix further, it was worth confirming a more fundamental assumption: was the job actually running inside the specified Docker image at all?

GitLab Runners can be configured with different executors: `docker`, `kubernetes`, `shell`... The shell executor silently ignores the "image:-key" entirely and runs jobs directly on the runner host's own filesystem. On a self-hosted GitLab instance, it's entirely possible a runner got registered with the shell executor early on for convenience.

**How to check:** every job log prints a line near the top:
```
Preparing the "docker" executor
```
Or `"shell"`, or `"kubernetes"`. That one line answers the question definitively, far more reliably than guessing from symptoms.

Reconfiguring the gitlab runner to have Docker as the executor and providing the right image, fixed the missing jq package and allowed the script to continue.

## 5: 301 moved permanently (from openresty)

**Symptom:**
```
Raw response: <html><head><title>301 Moved Permanently</title></head>...
<center>openresty</center>
HTTP status: 301
```

Once jq had something to parse, it got stuck on HTML instead of JSON, meaning the request never actually reached AWX's API. The openresty signature gave it away: this was the reverse proxy (Nginx Proxy Manager) intercepting the request, not AWX itself.

The cause: AWX_URL was set to http://... instead of https://..., and the proxy was redirecting HTTP to HTTPS with a 301 before the request could reach the backend.

**Fix:** update the `AWX_URL` CI/CD variable to use `https://`.

**Why not just add -L to follow the redirect?**  
Because `curl -L` on a `POST` request will, by default, downgrade to a `GET` on redirect (unless `--post301` is also specified), silently turning a "trigger project sync" call into a harmless no-op. Fixing the URL directly avoids the redirect entirely.

## 6: 404 not found (wrong variable name)

Fixing the protocol got a real response from AWX's server, but still a 404. Adding a debug line to print the exact URL being called:

```bash
echo "Calling: $AWX_URL/api/v2/projects/$PROJECT_ID/update/"
```

immediately revealed the issue: the GitLab CI/CD variable name didn't match what the script expected, so `$PROJECT_ID` was substituting as empty, producing a malformed URL that AWX correctly reported as not found. While AI can speed up building scripts, you still need to know the environment and how the pieces are connected. The script was correct, but the old var names where different from the ones created in the AI script.

**Lesson:** whenever a URL-based API call 404s unexpectedly, print the fully-interpolated URL before debugging anything else. It takes ten seconds and eliminates an entire class of "which variable is actually empty" guesswork.

## 7: Treating 202 as a failure

Once the URL was correct, the project sync call actually succeeded, AWX returned a full JSON payload describing the queued job. But the script still reported failure:

```
AWX API call failed with status 202
```

This one was self-inflicted in the script logic: it was checking for HTTP `201 Created`, but AWX's async job-trigger endpoints correctly return `202 Accepted`. The semantically correct code for "request accepted, processing queued," as opposed to "resource created synchronously." Fix was a one-character change in the status check.

**Lesson:** for any API that queues asynchronous work (which describes most CI/CD-adjacent automation), check the actual documented response code rather than assuming `200`/`201`. `202` is common and easy to overlook if you're pattern-matching from more typical REST CRUD behavior.

## 8: Inventory source ID ≠ inventory ID

With the project sync working end-to-end, the exact same `404` pattern reappeared for the inventory sync call. This time the cause was conceptual rather than a typo: an AWX Inventory and an AWX Inventory Source are different objects with different IDs.

- The **Inventory** is the top-level container (what you'd select when launching a job template).
- The **Inventory Source** is a child object under that inventory's Sources-tab. It's the thing that actually knows how to sync (e.g., from a GitLab-hosted file, a cloud provider, etc.), and it has its own separate numeric ID.

The `/api/v2/inventory_sources/<id>/update/` endpoint needs the *source's* ID, not the inventory's. Grabbing the ID from the inventory's own URL bar (an easy, intuitive mistake) will always 404.

**Fix:** navigate into the specific source under the inventory's Sources-tab to get the correct ID, or query it directly:
```bash
curl -sk "$AWX_URL/api/v2/inventories/<INVENTORY_ID>/inventory_sources/" \
  -H "Authorization: Bearer $AWX_TOKEN" | jq '.results[] | {id, name}'
```

## The final working pipeline

**`.gitlab-ci.yml`**
```yaml
stages:
  - deploy

variables:
  POLL_INTERVAL: "5"
  POLL_TIMEOUT: "300"

sync_awx:
  stage: deploy
  image: badouralix/curl-jq:latest
  script:
    - chmod +x ./scripts/awx-sync.sh
    - ./scripts/awx-sync.sh
  only:
    - main
```

scripts/awx-sync.sh triggers the project update, polls "/api/v2/project_updates/<id>/" until `status` is `successful`, then repeats the same pattern against "/api/v2/inventory_sources/<id>/update/" and "/api/v2/inventory_updates/<id>/". Every API call checks for `HTTP 202`, prints the raw response body on failure, and the whole script uses `set -e` so any unexpected failure stops the pipeline immediately rather than limping forward with a stale inventory.

![GitlabRunnerJobSuccess](images/posts/gitlabRunner.png)

## Takeaways

A few patterns that were worth internalizing, beyond the specific fixes:

- **Read the error literally before assuming.**  
A YAML structure error is not a bash error. A jq parse error means the previous step returned something unexpected, not that jq is broken.
- **CRLF line endings cause confusing failures.**  
The fix is cheap; the .gitattributes prevention is even cheaper. Worth doing on every repo touched by more than one OS.
- **Print before debugging.**  
Half of these roadblocks resolved in under a minute once the actual interpolated URL, HTTP status, or raw response body was printed to the job log instead of assumed.
- **Async APIs don't always return 200/201.**  
Check documented status codes for job-triggering endpoints specifically "202 Accepted" is common and easy to overlook.
- **Don't assume the ID you see is the ID you need.**  
AWX's inventory vs. inventory-source distinction is a good reminder that adjacent-looking objects in a UI can have entirely separate identities in the API.

None of these individually were hard once isolated, but stacked one after another, each masking the next, they made for a genuinely tricky debugging session. Hopefully this saves someone else a step or two.