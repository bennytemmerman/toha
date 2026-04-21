---
title: "Incident report: AWX failure"
date: 2026-04-06T17:17:23+02:00
hero: /images/posts/awx.png
description: AWX failure
theme: Toha
menu:
  sidebar:
    name: AWX failure
    identifier: awxfail
    parent: cat-automation
    weight: 401
---
# 1. Incident summary
This report documents a networking failure in a self-hosted AWX deployment running on a single-node k3s cluster. All Ansible job templates, including trivial tasks like the ping module, would hang silently for 30–90 seconds before being automatically cancelled or erroring. The root cause was a mismatch between the iptables backend used by k3s (legacy) and the system's active packet filter (nf_tables), which caused all inter-pod traffic to be silently dropped.

| Field	| details |
| --- | --- |
| Environment	| single-node k3s on Ubuntu (Proxmox VM) |
| AWX version	| AWX Operator on k3s, PostgreSQL 15 |
| CNI plugin	| Flannel (VXLAN, MTU 1450) |
| Pod subnet	| 10.42.0.0/16 (node subnet: 10.42.1.0/24) |
| Symptom	| All AWX jobs hang then cancel/error, ping module times out |
| Root cause	| iptables-nft active on host, k3s wrote rules via iptables-legacy |
| Resolution	| Switch system alternative to iptables-legacy, restart k3s |

# 2. Observed symptoms
## 2.1  AWX job behaviour
Every job, regardless of complexity, followed an identical failure pattern:
- Job transitions through waiting → pre_run → preparing_playbook → running_playbook → work_unit_id_assigned in under one second (AWX itself was healthy).
- No Ansible output was ever produced, the job container started but produced zero events.
- After 30–90 seconds, AWX received an internal cancel signal and terminated the job with rc=1 or rc=None.
- The pattern was 100% reproducible across all job templates and inventory hosts.

## 2.2  Pod networking
- New pods spawned by AWX jobs (Execution Environment containers) were stuck in ContainerCreating with no IP assigned, even though scheduling succeeded.
- The k3s node could not ping pod IPs within its own cni0 bridge subnet, returning Destination Host Unreachable despite a valid route entry.
- The metrics-server pod at 10.42.1.52 was unreachable: _dial tcp 10.42.1.52:10250: connect: no route to host_
- k3s logs showed repeated 502 Bad Gateway errors proxying to pod IPs on the cluster subnet.

# 3. Diagnostic process
## 3.1  Verify pod health
First, confirm that AWX pods themselves are healthy and not crash-looping:
```bash
kubectl get pods -n awx
```
```bash
kubectl logs -n awx <awx-task-pod> -c awx-task --tail=100
```
All four AWX pods (operator, postgres, task, web) showed Running with no crash loops. The task logs showed jobs entering running_playbook state normally — ruling out AWX application issues.

## 3.2  Test execution environment image pull
AWX runs jobs inside container images (Execution Environments). A pull failure would cause identical symptoms:
```bash
kubectl run ee-test --image=quay.io/ansible/awx-ee:latest \
  --restart=Never -n awx -- sleep 5
kubectl describe pod ee-test -n awx
kubectl delete pod ee-test -n awx
```
The image pulled successfully (414 ms from local cache). However, the test pod was also stuck in ContainerCreating with no IP — pointing firmly at a CNI/networking issue rather than a registry problem.

## 3.3  Check CNI and node network interfaces
```bash
ip addr show
ip route show
ip link show type vxlan
ip link show cni0
```
Both flannel.1 (VXLAN, MTU 1450) and cni0 (bridge, 10.42.1.1/24) existed and appeared up. Multiple veth pairs were bridged into cni0. Routes for 10.42.1.0/24 were present. Structurally, the network looked correct.

## 3.4  Ping a pod IP directly
```bash
ip route get 10.42.1.52
ping -c2 10.42.1.52
```
The route resolved to cni0, but ping returned Destination Host Unreachable with 100% packet loss. This confirmed that traffic destined for pod IPs was being dropped at the host, despite correct routing entries. This is the definitive indicator of this failure mode.

## 3.5  Inspect iptables rules and backend
```bash
iptables --version
update-alternatives --list iptables
iptables -L FORWARD -n | head -20
iptables -t nat -L -n | grep -E 'KUBE|CNI|flannel' | head -20
ufw status
nft list ruleset | grep -c rule
```
This step revealed the root cause. Key findings:
- iptables reported version: _iptables v1.8.10 (nf_tables)_
- Both backends were installed: /usr/sbin/iptables-legacy and /usr/sbin/iptables-nft
- The nft list ruleset returned 0 rules — k3s rules were invisible to nftables
- Massive warnings: XT target MASQUERADE not found, XT target DNAT not found, nft backend could not interpret legacy extension targets
- The FORWARD chain had ACCEPT policy and correct KUBE-* rules visible via iptables-nft, but those rules were not being enforced by the kernel because the kernel was applying the nftables ruleset (empty), not the legacy iptables layer.

# 4. Root cause analysis
## 4.1  The iptables / nftables split-brain problem
Modern Linux systems provide two independent packet filtering frameworks:
| Backend	| How it works	| Used by |
| --- | --- |
| iptables-legacy	| Writes directly to kernel netfilter tables	| k3s, older Kubernetes distributions |
| iptables-nft	| Translates iptables rules to nftables; nftables enforces them	| Newer Ubuntu/Debian systems (default) |

When k3s starts, it installs its pod networking rules (KUBE-FORWARD, FLANNEL-FWD, MASQUERADE, DNAT, etc.) using whichever iptables binary the system provides. In this case, k3s wrote its rules via the legacy backend — but the system's active alternative was iptables-nft. This meant:
- k3s rules were written to the legacy netfilter tables
- The kernel was enforcing the nftables ruleset, which was empty
- Pod traffic was forwarded to the cni0 bridge, found no matching FORWARD rules, and was silently dropped
- From the outside, everything looked correct: interfaces up, routes present, pods scheduled

## 4.2  Why it happened
This failure mode is triggered by a system update to iptables that switches the default alternative from iptables-legacy to iptables-nft, after k3s was already installed and running. The switch can happen silently during routine package upgrades. Because k3s caches its setup and the interfaces all look healthy, there is no obvious error logged, only the downstream effect of pods being unreachable.

# 5. Resolution
## 5.1  Fix: Switch iptables alternative to legacy
The fix is to align the system's iptables alternative with what k3s expects, then restart k3s to rebuild all rules cleanly:
### Step 1: Switch to legacy iptables backend
```bash
update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```

### Step 2: Restart k3s (will briefly restart all pods)
```bash
systemctl restart k3s
```

### Step 3: Wait ~30 seconds, then verify
```bash
kubectl get pods -n awx
ping -c2 10.42.1.1          # cni0 gateway — must return 0% loss
kubectl get pods -A | grep -v Running   # should be empty
```
  
> ✓ Result: After the restart, all pods returned to Running, the cni0 gateway became reachable (0% packet loss), and AWX jobs completed successfully.

## 5.2  Alternative fix: Use k3s bundled binaries
A more permanent solution that avoids any dependency on the system iptables version is to configure k3s to use its own statically-linked binaries:
```yaml
# /etc/rancher/k3s/config.yaml
prefer-bundled-bin: true
```
With this option, k3s ignores the system iptables entirely and uses its own bundled copy, making it immune to future system package updates that change the iptables alternative.

# 6. Warning signs & prevention
## 6.1  Early warning indicators
Watch for any combination of these signals, they indicate the iptables problem before it causes full outages:
- iptables warnings in logs: XT target MASQUERADE not found, XT target DNAT not found
- k3s logs: no route to host when connecting to pod IPs in the cluster subnet
- Pods stuck in ContainerCreating with no IP assigned despite scheduling succeeding
- New pod IPs unreachable from the node: ping <pod-ip> returns Destination Host Unreachable
- Metrics-server returning 502/503 via the k3s API server proxy
- AWX jobs that go directly from running_playbook to cancellation with no output in between
- nft list ruleset | grep -c rule returns 0 while iptables shows KUBE-* chains

## 6.2  Health check commands
Run these commands after any system update or k3s restart to verify pod networking is functional:
### 1. Confirm iptables backend is legacy
```bash
iptables --version
```
__Expected: iptables v1.x.x (legacy)__

### 2. Verify cni0 gateway is reachable
```bash
ping -c2 10.42.1.1
```
__Expected: 0% packet loss__

### 3. Check all pods are Running
```bash
kubectl get pods -A | grep -v Running
```
__Expected: empty output__

### 4. Confirm update-alternatives is pinned
```bash
update-alternatives --display iptables
```
__Expected: link currently points to /usr/sbin/iptables-legacy__

### 5. Verify FORWARD chain has k3s rules
```bash
iptables -L FORWARD -n | grep -E 'KUBE|FLANNEL'
```
__Expected: multiple matching rules__

## 6.3  When to suspect this issue
Jobs hang silently with no output	iptables backend mismatch, pods can't communicate
Worked before, broke after apt upgrade, iptables alternative switched by package update
Interfaces look fine but pings fail,	Rules written to wrong netfilter layer
ContainerCreating stuck on new pods only, CNI can't set up veth pairs due to missing FORWARD rules
All jobs fail, AWX pod logs show no errors, Problem is below AWX, in k3s/CNI layer
metrics-server / kubectl top fails, Pod-to-node routing broken

# 7. Quick Reference
## 7.1  Full Diagnostic Runbook
### === STEP 1: Check AWX pods ===
```bash
kubectl get pods -n awx
kubectl logs -n awx <awx-task-pod> -c awx-task --tail=100
```

### === STEP 2: Test EE image pull ===
```bash
kubectl run ee-test --image=quay.io/ansible/awx-ee:latest \
  --restart=Never -n awx -- sleep 10
kubectl describe pod ee-test -n awx   # look for ContainerCreating stuck
kubectl delete pod ee-test -n awx
```

### === STEP 3: Check CNI interfaces ===
```bash
ip addr show | grep -E 'cni0|flannel'
ip route show
```

### === STEP 4: Ping a pod IP ===
Get any pod IP first:
```bash
kubectl get pods -n awx -o wide
ping -c2 <pod-ip>
```
If 'Destination Host Unreachable' -> iptables mismatch

### === STEP 5: Check iptables backend ===
```bash
iptables --version          # must say (legacy), not (nf_tables)
nft list ruleset | grep -c rule   # if 0, k3s rules invisible
iptables -L FORWARD -n | grep KUBE
```

### === STEP 6: Fix ===
```bash
update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
systemctl restart k3s
```

## 7.2  Permanent prevention
### Option A: Pin iptables-legacy system-wide
```bash
update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```

### Option B: Let k3s use its own bundled binaries
Add to /etc/rancher/k3s/config.yaml:
```yaml
prefer-bundled-bin: true
```

### Verify after next system update:
```bash
iptables --version   # must still say (legacy)
ping -c2 10.42.1.1  # must be 0% loss
```

# 8. Lessons learned
- CNI issues present as application-level failures, AWX job timeouts are an indirect symptom of a low-level networking problem. Always check pod-level connectivity before debugging the application.
- The iptables-nft vs iptables-legacy split is a common and silent failure mode on modern Ubuntu/Debian systems. Any Kubernetes distribution that uses legacy iptables rules can break this way after a system update.
- A key diagnostic clue: routes exist but pings fail. If ip route get <pod-ip> resolves correctly but ping returns Destination Host Unreachable, the problem is in the firewall layer, not routing.
- Running nft list ruleset | grep -c rule returning 0 while iptables shows active KUBE-* chains is a definitive indicator of the split-brain condition.
- Lock in prefer-bundled-bin: true in k3s config to future-proof the installation against OS package updates.