---
title: "project: Infra documentation automation"
date: 2026-04-21T12:17:23+02:00
hero: /images/posts/awx.png
description: CICD documentation automation
theme: Toha
menu:
  sidebar:
    name: CICD documentation automation
    identifier: documentation_automation
    parent: cat-automation
    weight: 402
---

**From zero to automated infrastructure documentation:**  
A Complete Journey with Ansible, Jinja2, AWX, and GitLab CI/CD
Learning how to automate infrastructure documentation using Ansible, Jinja2 templates, AWX, and GitLab CI/CD.
From first playbook to production-grade multi-role reporting pipeline.

**Keywords**  
_Ansible automation · Jinja2 · AWX Tower · GitLab CI/CD · Infrastructure as Code · DevOps · Kubernetes · Infrastructure documentation_

# **1\. Introduction**

Infrastructure documentation is one of the most consistently neglected areas in modern engineering organisations. Teams invest heavily in automation, observability, and deployment pipelines, yet the documentation describing what is actually running in production frequently lags weeks or months behind reality. When an engineer needs to understand the state of a server at 2 AM during an incident, a stale Confluence page or an out-of-date spreadsheet is worse than no documentation at all: it creates false confidence.

This article documents a real-world journey: starting from a single Ansible playbook that collects a hostname and writes a Markdown file, and progressing through iterative engineering decisions to arrive at a production-grade, multi-role automated documentation pipeline. Every problem encountered along the way is examined in detail: DNS resolution inside Kubernetes, Jinja2 template errors, CoreDNS configuration, CI/CD pipeline design.

**Why this matters:** Infrastructure as Code (IaC) has fundamentally changed how teams manage servers and services. Tools like Ansible, Terraform, and Kubernetes make it possible to define and reproduce environments programmatically. However, the human-readable documentation layer (the layer that explains what exists, why it exists, and what state it is in) is rarely treated with the same engineering mindset. This project addresses that gap by treating documentation as a first-class deliverable, generated automatically alongside the infrastructure itself.

> This is not a tutorial about a fictional environment. Every step, every error, and every fix described in this article occurred in a real homelab environment running AWX on K3s, with a self-hosted GitLab instance, Pi-hole for DNS, and Proxmox as the hypervisor platform.

# **2\. Conceptual foundation**

## **2.1 Core technologies**

Before examining the implementation, it is worth establishing a precise understanding of each technology in the stack and how they interact.

| **Technology**   | **Role in this pipeline** |
|------------------|---------------------------|
| **Ansible**      | An agentless automation engine that connects to managed hosts over SSH, executes tasks defined in YAML playbooks, and collects system facts via its fact-gathering module. |
| **Jinja2**       | A Python templating engine that renders text output by substituting variables and executing control flow logic (loops, conditionals) within template files. Ansible uses Jinja2 natively for both playbook variable interpolation and the template module. |
| **AWX**          | The open-source upstream project for Red Hat Ansible Automation Platform. AWX provides a web UI, REST API, RBAC, credential management, and job scheduling on top of Ansible. It runs as a containerised application on Kubernetes. |
| **GitLab CI/CD** | A pipeline automation system integrated into GitLab. On each git push, GitLab reads _.gitlab-ci.yml_ and executes defined stages in isolated containers. Used here to validate, sync, and deploy the Ansible project. |
| **MkDocs**       | A static site generator that converts Markdown files into a navigable HTML documentation website. The _docs/_ directory structure maps directly to the navigation tree. |
| **K3s**          | A lightweight, certified Kubernetes distribution optimised for resource-constrained environments. AWX runs on K3s in this environment, which introduces Kubernetes-specific networking considerations such as CoreDNS. |
| **Pi-hole**      | A network-wide DNS resolver. In this environment it also serves as the authoritative DNS server for the internal domain _siemforge.xyz_, including the GitLab instance. |

## **2.2 The relationship between Ansible and Jinja2**

Understanding the execution model of Ansible is essential for writing correct templates. The following sequence occurs when Ansible processes a template: task.

- **Fact gathering:** ansible.builtin.setup runs on the target host and populates the hostvars dictionary with hundreds of system facts: kernel version, network interfaces, memory, CPU architecture, and more.
- **Template rendering:** The .j2 file is read by the Ansible controller (the AWX runner pod), the Jinja2 engine substitutes all {{ variable }} expressions using the collected facts, and the rendered content is written to the destination path on the target host.
- **Variable scope:** Variables are scoped per host. In a multi-host playbook, hostvars\[hostname\] provides access to another host's facts from within a different play, a critical pattern used later when the MkDocs play needs to reference facts gathered from the testservers play.

> A common misconception is that the template file is sent to the target host for rendering. It is not. **Jinja2 rendering happens on the Ansible controller**, and the rendered output is pushed to the target. This is why the template: task can reference localhost-side facts and why delegate_to: localhost works for URI lookups during a remote play.

## **2.3 Hostname vs. FQDN**

This distinction causes confusion frequently enough to warrant explicit clarification:

| **Variable** | **Meaning** |
| -------------------- | -------------------------- |
| **ansible_hostname** | The short hostname as returned by the hostname -s command. Example: 30test. This is what appears in shell prompts and is typically used for file naming. |
| **ansible_fqdn** | The Fully Qualified Domain Name, the complete DNS name including domain suffix. Example: 30test.siemforge.xyz. This is the name used for DNS resolution and TLS certificate subjects. |

  
On a correctly configured server these will differ. On a minimal installation where the hostname has not been set to include the domain, they may be identical. The playbook collects both because each is useful in different contexts: the short hostname is used for filenames and display, the FQDN for network-relevant documentation.

# **3\. From first template to production pipeline**

## **3.1 Phase 1 - The minimal viable playbook**

The project began with the simplest possible goal: connect to a test server, get its hostname, write a Markdown file, push it to the MkDocs server, and rebuild the documentation site. This forced engagement with the full pipeline without the complexity of advanced fact gathering: AWX project configuration, inventory setup, credential management, and the Jinja2 template mechanism.

The initial playbook structure used two plays: one targeting the testserver group to gather facts and render the template, and a second targeting the MkDocs host to receive the file and trigger the build. The critical design decision was using Ansible's fetch:-module to pull the rendered Markdown file back to the AWX runner after rendering, then using copy: in the second play to push it to the MkDocs server. This pattern is necessary because the two hosts cannot communicate directly with each other through Ansible - all file transfer flows through the AWX controller.

```bash
# Simplified initial flow

Play 1 (testserver):
gather_facts → render template → fetch .md to AWX runner

Play 2 (31.mkdocs):
copy .md from AWX runner → mkdocs build
```

## **3.2 Phase 2 - The first Jinja2 error: ansible_interfaces**

The first non-trivial template error occurred immediately when attempting to render network interface information. The initial template used this pattern:
```bash
{% for iface, details in ansible_interfaces | sort %}
```

This fails with _ValueError: too many values to unpack (expected 2)_ because ansible*interfaces is a list of strings (interface names), not a dictionary. The correct approach is to iterate the list of names and look up each interface's details separately using the ansible_<ifacename&gt> fact:

```bash
{% for iface in ansible_interfaces | sort %}
{% set iface_data = hostvars\[inventory_hostname\]\['ansible_' + iface\] | default({}) %}
{% if iface_data.ipv4 is defined %}
| {{ iface }} | {{ iface_data.ipv4.address }} | {{ iface_data.ipv4.netmask }} |
{% endif %}
{% endfor %}
```

This error illustrates a fundamental principle of Ansible fact structure: facts are not uniformly typed. Some are strings, some are lists, some are dictionaries, and some are nested combinations of all three. Before writing a Jinja2 loop, it is worth verifying the type of the variable being iterated using the debug: module in a test playbook.

## **3.3 Phase 3 - Inventory architecture and the multi-inventory question**

A practical question arose early. There were separate AWX inventories for different server groups (inv_test, inv_mkdocs, inv_all). An AWX Job Template accepts exactly one inventory. Can a playbook that targets multiple host groups work with a single inventory that contains other unrelated hosts?

The answer is yes, without any special configuration. Ansible's host targeting is additive and scoped: when a play specifies hosts: testservers, Ansible queries the inventory for hosts matching that group or pattern and operates only on those hosts. All other hosts in the inventory are ignored for that play. This means a large shared inventory (inv_all) containing dozens of servers can safely be used with a targeted playbook, only the explicitly addressed hosts are contacted.

> Security note: While using a shared inventory is operationally convenient, it is worth ensuring that the AWX credential attached to the job template has SSH access only to the hosts it needs. A single machine credential with access to all hosts in a large inventory is a broader attack surface than role-scoped credentials. This is particularly relevant in shared AWX environments with multiple teams.

## **3.4 Phase 4 - Dynamic file naming and the hostvars pattern**

As the playbook evolved to handle multiple hosts in the testservers group, file naming became a challenge. The initial approach hardcoded the hostname in the copy: task destination path. The correct approach uses Ansible's hostvars to dynamically construct the path at runtime:

```bash
- name: Copy markdown files organized by group
copy:
src: "/tmp/fetched\_{{ hostvars\[item\]\['ansible_hostname'\] }}\_report.md"
dest: >-
{{
docs_base + '/' +
(hostvars\[item\]\['group_names'\]
| difference(\['all'\])
| first
| default('ungrouped'))
\+ '/' +
hostvars\[item\]\['ansible_hostname'\] + '\_report.md'
}}
loop: "{{ groups\['all'\] | difference(\['31.mkdocs'\]) }}"
```

The group_names variable is populated per host by Ansible and contains a list of all groups that host belongs to. The difference(\['all'\]) filter removes the implicit 'all' group that every host belongs to, leaving only the meaningful group memberships. The first filter takes the primary group, and default('ungrouped') provides a fallback for hosts with no explicit group membership.

## **3.5 Phase 5 - The DNS resolution problem in K3s**

This was the most operationally interesting problem in the entire journey. The symptom was intermittent: AWX project syncs against the internal GitLab instance would fail with a DNS resolution error, despite the AWX host itself being able to resolve the domain.

> The root cause is architectural. AWX job execution does not happen on the host operating system, it happens inside Kubernetes pods managed by K3s. These pods use CoreDNS as their DNS resolver (not the host's systemd-resolved or /etc/resolv.conf). 

The host could reach Pi-hole and resolve internal domains but the pods could not, because CoreDNS was forwarding to the host's stub resolver (127.0.0.53) which in turn was not reliably forwarding internal domain queries to Pi-hole.

The diagnostic path:
- **Step 1:** Confirm the domain resolves from the AWX host: 
```bash
dig gitlab.siemforge.xyz @192.168.1.6
```
This worked correctly.
- **Step 2:** Test resolution from inside a Kubernetes pod: 
```bash
kubectl run dnstest --image=busybox --restart=Never --rm -it -- nslookup gitlab.siemforge.xyz
```
This returned NXDOMAIN.
- **Step 3:** Test direct Pi-hole query from the pod: 
```bash
nslookup gitlab.siemforge.xyz 192.168.1.6
```
This resolved correctly, confirming the issue was CoreDNS forwarding, not Pi-hole's records.

The final CoreDNS forward directive was:
```bash
forward . 192.168.1.6
```
Not 
```bash
forward . 192.168.1.6 1.1.1.1
```
The reason for removing 1.1.1.1 as a fallback was deliberate: CoreDNS's forward plugin uses a health-checking and load-balancing algorithm across multiple upstreams. If Pi-hole is the authoritative source for internal domains like siemforge.xyz, having 1.1.1.1 as a second upstream creates a race condition, CoreDNS may forward a query to Cloudflare instead of Pi-hole depending on which upstream responds faster or which is currently marked healthy. Cloudflare will return NXDOMAIN for internal domains, and CoreDNS may use that response. Removing 1.1.1.1 ensures all DNS queries go exclusively to Pi-hole, which itself has upstream resolvers configured (typically 1.1.1.1 or 8.8.8.8) for external domains. Pi-hole becomes the single source of truth for all DNS, handling both internal and external resolution.

### The TCP DNS configuration
Standard DNS operates over UDP on port 53. UDP is connectionless and has a payload limit of 512 bytes for traditional DNS responses (4096 bytes with EDNS0 extensions). For most simple A record lookups this is sufficient. However, certain query types (DNSSEC responses, large TXT records, zone transfers, and some responses with many answer records) exceed this limit and require TCP fallback.

Pi-hole, as a local resolver, sometimes responds faster and more reliably over TCP, and CoreDNS's forward plugin supports explicit TCP forcing which eliminates the UDP-to-TCP fallback entirely.

The CoreDNS forward plugin supports a force_tcp option:
```bash
forward . 192.168.1.6 {
    force_tcp
}
```
With force_tcp, CoreDNS sends all upstream DNS queries to Pi-hole over TCP rather than UDP. 

The advantages in a local network context:
- **Reliability**: TCP is connection-oriented. The three-way handshake confirms the upstream is reachable before a query is sent. UDP queries can be silently dropped with no immediate feedback to CoreDNS.
- **No truncation**: TCP has no payload size limit for DNS. Responses that would be truncated over UDP and require a retry are delivered in one shot.
- **Predictable behaviour**: Eliminates the TC (truncation) bit handling and UDP-to-TCP retry logic, simplifying the resolution path.
Local network overhead is negligible: The argument against force_tcp in internet-facing resolvers is latency, TCP handshake adds round-trip time. On a local network with sub-millisecond latency between the K3s node and Pi-hole, this overhead is immeasurable.

The full updated CoreDNS Corefile block:
```bash
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
      pods insecure
      fallthrough in-addr.arpa ip6.arpa
    }
    hosts /etc/coredns/NodeHosts {
      ttl 60
      reload 15s
      fallthrough
    }
    prometheus :9153
    forward . 192.168.1.6 {
        force_tcp
    }
    cache 30
    loop
    reload
    loadbalance
    import /etc/coredns/custom/*.override
}
import /etc/coredns/custom/*.server
```

> AWX job execution involves many DNS lookups in rapid succession, connecting to managed hosts, resolving GitLab for project syncs, and any uri: tasks in playbooks that call external APIs like endoflife.date. Under high query load, UDP-based DNS can experience dropped packets on even well-provisioned local networks. A failed DNS lookup in the middle of an Ansible play causes a task failure that appears unrelated to DNS at first glance. Forcing TCP for all upstream queries to Pi-hole removes this failure mode entirely for the K3s pod network.

## **3.6 Phase 6 - Enriching the template: OS lifecycle, system metrics, and storage**

With the basic pipeline working, the template was progressively enriched with operationally valuable data. Each addition required corresponding playbook tasks to collect the data before the template could render it.

### **OS End-of-Life Information**

The endoflife.date API provides machine-readable EOL data for virtually every Linux distribution. Querying this API from Ansible using the uri: module with delegate_to: localhost provides EOL dates for both Ubuntu (queried by full version) and Debian (queried by major version). The days-remaining calculation is performed in Jinja2:
```bash
eol_days: >-
{{
((eol_raw.json.eol | to_datetime('%Y-%m-%d')) -
(ansible_date_time.date | to_datetime('%Y-%m-%d'))).days
}}
```
The template then renders a colour-coded status indicator: red for already EOL, amber for within 180 days of EOL, and green for healthy. This makes the MkDocs output immediately actionable without requiring the reader to cross-reference external sources.

### **Process and Memory Snapshots**

CPU and memory data are point-in-time snapshots collected via _free -m_ and _top -bn1_. The top 10 processes by CPU usage are captured via _ps aux --sort=-%cpu_. These are explicitly labelled as snapshots in the generated documentation, which is important for accuracy, a process list from a playbook run at 3 AM on a Wednesday is not representative of typical load.

### **Storage and LVM**

Storage information uses three complementary commands: _lsblk -J_ for block device hierarchy in JSON format (enabling structured parsing in Jinja2), _df -h_ for filesystem usage, and the LVM trio _pvs , vgs , lvs_ for volume group information. The LVM commands use become: true since they require root, and are gated behind a which lvm check so they are skipped gracefully on non-LVM hosts.

## **3.7 Phase 7 - Role-Based Template Splitting**

As the template grew to cover OS lifecycle, network interfaces, open ports, storage, LVM, process lists, and inode usage, a single template became inappropriate. A Proxmox hypervisor requires LVM details and VM inventory. A K3s node requires cluster membership and pod counts. A Portainer host requires container status. None of these are relevant to the others.

The solution was to split the single playbook into multiple plays, each targeting a specific group and referencing a group-specific Jinja2 template. Shared fact-gathering logic was extracted into reusable task files in a tasks/ directory, imported via import_tasks: into each play. This eliminated duplication while maintaining the ability to customise what each role's template renders.

The resulting structure:
```bash
playbooks/
  server_reports.yml # One play per role group
tasks/
  common_facts.yml # Hostname, IP, memory, CPU
  eol_facts.yml # endoflife.date API queries
  storage_facts.yml # lsblk, df
  lvm_facts.yml # pvs, vgs, lvs
  network_facts.yml # Interface enumeration
  process_facts.yml # ps, top
  k3s_facts.yml # kubectl node status
  docker_facts.yml # docker ps, version
templates/
  proxmox.md.j2
  k3s.md.j2
  portainer.md.j2
  ubuntu.md.j2
  debian.md.j2 # Proxmox hosts use this
  rocky.md.j2
  oracle.md.j2
  generic.md.j2 # Ungrouped fallback
  overview.md.j2 # Cross-host index page
```
> The import_tasks: directive (as opposed to include_tasks: ) is used deliberately here because import_tasks performs static inclusion at parse time, making the task list predictable and fully visible to ansible-lint during CI validation. Dynamic includes with include_tasks would complicate linting and syntax checking.

## **3.8 Phase 8 - The infrastructure overview page**

A final play generates an overview index page on the MkDocs server by rendering overview.md.j2 on the 31.mkdocs host. This template loops over all known groups using the groups variable (which is populated from the AWX inventory) and generates a table per group, with each row linking to the individual host's report.

The overview page serves as the entry point to the documentation site, providing immediate visibility into the entire infrastructure: which hosts are running which OS, which are approaching EOL, and direct links to detailed reports. This transforms the MkDocs site from a collection of individual files into a coherent infrastructure knowledge base.

# **4\. The CI/CD pipeline**

## **4.1 Pipeline design**

The GitLab CI/CD pipeline adds a critical safety layer between a git push and AWX execution. Without it, a syntax error in a playbook or a malformed Jinja2 template would only be discovered when AWX ran the job against live infrastructure. With the pipeline, such errors are caught in isolated containers before any code reaches AWX.

The pipeline has three stages:

| **Stage** | **Purpose** |
| ------------ | ------------------------- |
| **validate** | Runs ansible-playbook --syntax-check and ansible-lint against the playbook. Separately, renders all .j2 templates using j2cli with a synthetic test vars JSON to confirm they parse without errors. |
| **sync** | Calls the AWX REST API to trigger a project sync (git pull). Polls the AWX API every 5 seconds until the sync status is either successful or failed, with a 2-minute timeout. |
| **deploy** | Calls the AWX REST API to launch the job template. Only runs if the sync stage succeeded. |

> The validate stage is the most important. It catches two distinct failure modes: structural errors (invalid YAML, undefined variables, incorrect module arguments) caught by ansible-lint, and rendering errors (Jinja2 syntax errors, missing filters, incorrect variable types) caught by j2cli rendering against known test data.

## **4.2 Secrets management in CI/CD**

The pipeline requires an AWX API token to trigger syncs and launches. This token must never appear in the .gitlab-ci.yml file or any committed file. It is stored as a masked GitLab CI/CD variable (AWX_TOKEN), which means it is injected into the pipeline environment at runtime and never appears in job logs.

> Security principle: Treat AWX API tokens with the same sensitivity as SSH private keys. They grant the ability to run arbitrary playbooks against your infrastructure. Use tokens scoped to Write access only where needed, rotate them on a schedule, and store them exclusively in secret management systems, never in version control.

Additional security practices applied in this pipeline:
- The AWX token is scoped to a dedicated service account with permissions limited to syncing the specific project and launching the specific job template.
- The GitLab repository containing the playbooks is private.
- The AWX project is configured with a dedicated read-only deploy key for GitLab access, not a personal access token tied to a user account.
- Template validation uses synthetic test data with no real hostnames, IPs, or credentials.

# **5\. Security considerations**

## **5.1 Ansible credential management**

AWX stores SSH credentials in an encrypted credential store. The machine credential used in this project provides SSH access to all managed hosts. Several hardening practices apply:

- **Principle of least privilege:** The Ansible user should have sudo access scoped only to the specific commands required (LVM queries, service checks). A _NOPASSWD: ALL_ sudo rule is a significant security risk in production environments.
- **SSH key hygiene:** Use ed25519 keys rather than RSA. Rotate credentials when team membership changes. Store private keys only in AWX's credential vault, never in the repository.
- **Become usage:** The become: true tasks (LVM queries) execute with root privileges. These are scoped to read-only commands but the privilege escalation itself should be audited.

## **5.2 The generated documentation security surface**

The Markdown reports generated by this pipeline contain detailed system information: kernel versions, open ports, running processes, network interface addresses, and storage layout. This information has legitimate value for operations teams but also constitutes a comprehensive reconnaissance report if it falls into the wrong hands.

- Ensure the MkDocs site is not publicly accessible. It should be behind authentication (OAuth, LDAP, or at minimum HTTP basic auth) or accessible only on the internal network.
- Consider whether the process list and open port data should be included in the generated reports, or whether that level of detail should be restricted to higher-privilege views.
- The overview page linking all reports is particularly sensitive. It provides a complete inventory of the infrastructure in one place.

## **5.3 Jinja2 template injection**

Jinja2 templates are evaluated with the full Jinja2 engine, including the ability to call Python functions. If any variable rendered into a template is sourced from user input or an untrusted external API without sanitisation, template injection attacks are possible. In this pipeline, all variables originate from **Ansible-gathered system facts** on trusted internal hosts, which significantly limits this risk. However, the endoflife.date API responses are external, they should be treated as untrusted input and not passed to Jinja2 filters that evaluate code.

# **6\. Extending beyond Ansible: Bash and j2cli**

Not every environment has Ansible or AWX available. Embedded systems, minimal container images, or legacy servers may only have a POSIX shell and Python available. The j2cli tool (installable via pip3 install j2cli) provides standalone Jinja2 rendering from the command line, accepting a template file and a JSON or YAML variables file as input.

A bash script can collect system facts, write them to a JSON file, and invoke j2cli to render the same template used by the Ansible pipeline:
```bash
# !/bin/bash
HOSTNAME=\$(hostname -s)
IP=\$(hostname -I | awk '{print \$1}')
MEM_TOTAL=\$(free -m | awk 'NR==2{print \$2}')
cat > /tmp/facts.json <<EOF
{
"server_hostname": "\$HOSTNAME",
"server_ip": "\$IP",
"ansible_memtotal_mb": \$MEM_TOTAL
}
EOF

j2 /path/to/templates/generic.md.j2 /tmp/facts.json > /tmp/\${HOSTNAME}\_report.md
```

Because j2cli uses the same CPython Jinja2 library as Ansible, templates are fully portable between the two execution modes. The only adjustment required is that Ansible-specific filters (such as to_datetime) may not be available in pure j2cli, those sections of a template should use the default() filter to degrade gracefully.

# **7\. Real-world use cases and production considerations**

## **7.1 Scheduling reports**

In production, server reports should be generated on a schedule rather than triggered manually. AWX supports scheduled job template execution via its Schedules feature. A reasonable cadence for infrastructure reports is daily, frequent enough to capture meaningful change, infrequent enough to avoid unnecessary load on managed hosts.

For environments with a large number of hosts, consider whether all hosts need to be reported on every run, or whether a rolling schedule (different groups on different days) would be more appropriate. Ansible's serial: keyword can be used to limit the number of hosts processed in parallel within a play.

## **7.2 Integrating with change management**

The generated reports provide a before-and-after snapshot capability. If reports are committed to a Git repository (the MkDocs source can itself be a Git repository), git diff between two report versions provides a detailed change log: which packages changed version, which ports opened or closed, which processes appeared or disappeared. This is valuable audit trail data for change management processes.

## **7.3 Scaling the pipeline**

As the number of managed hosts grows, several performance considerations apply:

- **Parallelism:** Ansible's default forks value is 5, meaning it processes 5 hosts concurrently. This can be increased in AWX job template settings or via ansible.cfg for environments with many hosts.
- **API rate limiting:** The endoflife.date API queries are delegated to localhost (the AWX runner). With 50 hosts, this means 50 sequential API calls. Consider caching the API response in a set_fact at the play level and reusing it across hosts of the same distribution.
- **MkDocs build time:** MkDocs build time scales with the number of pages. For large environments, consider whether the full site needs to be rebuilt on every run or whether incremental updates are sufficient.
- **File management:** The purge-and-recreate pattern (removing the entire group directory before copying fresh reports) is safe and simple but means the MkDocs site is temporarily incomplete during a run. For high-availability documentation requirements, consider a blue-green swap pattern: write to a staging directory, then atomically rename it.

# **8\. Pros, cons, and when not to use this approach**

## **8.1 Advantages**

- **Zero configuration drift:** Documentation is generated from the live system state, not from human memory. 
- **Reusability:** The task files, templates, and CI/CD pipeline are reusable across environments. Adding a new server group requires creating a template and adding a play, the infrastructure is already in place.
- **Auditability:** Every report generation is a tracked AWX job with a complete execution log, including which hosts were contacted, which tasks ran, and what changed.
- **No agent required:** Ansible's agentless architecture means this pipeline works on any SSH-accessible host without installing additional software.

## **8.2 Limitations**

- **Point-in-time accuracy:** Reports are accurate as of the moment of collection. Process lists, port states, and memory usage reflect a snapshot. For continuous monitoring, purpose-built tools (Prometheus, Zabbix, Datadog) are more appropriate.
- **Fact gathering overhead:** Ansible's gather_facts module performs a comprehensive system survey on each run. On a large fleet, this adds meaningful execution time. Custom fact gathering (gather_subset) can limit this to only required fact categories.
- **Template maintenance:** As the number of role-specific templates grows, keeping them consistent requires discipline. A change to the common data model (adding a new fact field) requires updating multiple templates.
- **Not a replacement for monitoring:** This pipeline generates documentation, not alerts. It should complement, not replace proper monitoring and observability tooling.

# **9\. Common mistakes and how to avoid them**

| **Mistake** | **Correct approach** |
| ------------------- | ------------------------------------------- |
| **Iterating ansible_interfaces as a dict** | ansible*interfaces is a list of names. Access details via hostvars\[inventory_hostname\]\['ansible*' + iface\]. |
| **Hardcoding hostnames in the second play** | Use hostvars\[item\]\['ansible_hostname'\] in loops. Hardcoded names break immediately when the playbook is extended to new hosts. |
| **Fixing DNS on the host, not in CoreDNS** | AWX jobs run in K3s pods. Host DNS configuration is irrelevant to pod DNS. Always edit the CoreDNS ConfigMap. |
| **Storing AWX tokens in .gitlab-ci.yml** | Use masked GitLab CI/CD variables. Tokens in committed files are a serious credential leak risk. |
| **Running LVM commands without become: true** | pvs, vgs, and lvs require root. Without become: true these tasks will fail silently or with a misleading permissions error. |
| **Not validating templates in CI** | A broken Jinja2 template is only discovered when AWX runs the job. Add j2cli rendering to the validate stage to catch errors before they reach production. |
| **Using include_tasks instead of import_tasks for CI** | ansible-lint cannot analyse dynamically included task files. Use import_tasks for task files that should be linted and syntax-checked. |

# **10\. Frequently asked questions**

### **Can this pipeline work without AWX, using plain Ansible?**

Yes. AWX is a wrapper around Ansible. The playbooks, templates, and task files work identically with ansible-playbook run from the command line. AWX adds scheduling, RBAC, credential management, and the REST API that enables CI/CD integration. For a single engineer on a homelab, ansible-playbook is sufficient.

### **How do I handle hosts that are sometimes offline?**

Add ignore_unreachable: true at the play level and use the ignore_errors: true option on the fetch and copy tasks. This allows the pipeline to complete a partial run (generating reports for reachable hosts) without failing entirely on unreachable ones. The PLAY RECAP will show unreachable hosts explicitly.

### **What is the difference between import_tasks and include_tasks?**

import_tasks performs static inclusion at parse time, the task list is fully known before execution begins. include_tasks performs dynamic inclusion at runtime, which allows conditional inclusion but prevents static analysis tools like ansible-lint from inspecting the included tasks. For reusable task files that should be linted, import_tasks is the correct choice.

### **Can the same Jinja2 templates be used with Terraform or other tools?**

Terraform has its own templatefile() function using HCL syntax, which is not compatible with Jinja2. However, j2cli can be called from a Terraform local-exec provisioner or a shell script in a Terraform null_resource, enabling Jinja2 template rendering as part of a Terraform workflow. This is a less common but valid pattern for generating documentation from Terraform state.

### **How should the MkDocs site be secured?**

MkDocs generates a static site, so security is implemented at the web server or reverse proxy layer. Nginx with HTTP basic authentication or OAuth2 Proxy provides authentication. For internal documentation, restricting access by network CIDR (internal-only) is often sufficient. Avoid exposing the MkDocs site to the public internet given the sensitive system information it contains.

# **11\. Summary**

Every server and system in an organisation has a story: what operating system it runs, how old that software is, whether it is still receiving security updates, how much memory it uses, and what services it provides. Traditionally, keeping this information accurate required engineers to manually update documentation, a process that is time-consuming and almost always falls behind reality.

This project automates that documentation process entirely. A scheduled automation job connects to every server in the environment, collects the current state of that server, and generates a formatted web page documenting what it found. The result is a documentation website that is always accurate, always current, and requires no manual effort to maintain.

The business benefits are concrete:

- Incident response is faster because engineers have accurate, current information about affected systems.
- Security audits are easier because outdated operating systems (those no longer receiving security patches) are immediately visible on the overview page.
- Onboarding is improved because new team members have a reliable reference for the infrastructure they will be working with.
- Compliance requirements that mandate documented system inventories are met automatically rather than through periodic manual surveys.

# **12\. Conclusion**

What began as a simple exercise, collect a hostname and write a Markdown file, evolved into a production-grade automated documentation pipeline covering multi-role server reporting, CI/CD validation, CoreDNS configuration, OS lifecycle tracking, and a cross-infrastructure overview page. Each step of this evolution was driven by a real operational need and a real problem encountered in a live environment.

Several principles emerged consistently throughout this journey:

- **Understand the execution model.** Knowing that Jinja2 rendering happens on the Ansible controller, not the target host, explains why certain variable access patterns work and others do not. Knowing that AWX runs in Kubernetes pods, not on the host OS, explains why host DNS configuration is irrelevant to AWX's network access.
- **Validate before you execute.** The CI/CD pipeline's validate stage is not overhead, it is the mechanism that prevents broken code from running against live infrastructure. The cost of a failed ansible-lint run in a GitLab CI container is zero. The cost of a failed playbook run against 50 production servers is significant.
- **Design for extension.** The task file architecture (import_tasks from a shared tasks/ directory) makes it trivial to add new roles, new fact-gathering capabilities, or new templates without restructuring the playbook. Good structure early prevents technical debt later.
- **Documentation is infrastructure.** Generated documentation should be treated with the same engineering rigour as the infrastructure it describes, versioned in Git, validated in CI/CD, and deployed via automation.

The logical next steps from this point include integrating the generated reports with a change detection system (alerting when a host's OS moves within 180 days of EOL), adding Ansible roles for automated remediation (applying available updates to hosts flagged as outdated), and expanding the bash/j2cli path to cover environments without Ansible access.

The pipeline described in this article is operational today in a real homelab environment. It is not a theoretical exercise. Every error, every fix, and every design decision documented here reflects something that was encountered, diagnosed, and resolved in practice.

**SIEMForge**

_Automate everything. Document everything. Trust nothing undocumented._