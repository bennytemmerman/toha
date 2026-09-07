---
title: "CICD pipeline Homepage"
date: 2026-09-7T12:00:15+02:00
hero: /images/posts/cicd.png
description: Pipeline to organize Homepage
theme: Toha
draft: true
menu:
  sidebar:
    name: CICD homepage pipeline
    identifier: cicd_homepagePipeline_post
    parent: cat-linux
    weight: 306
---

# GitLab CI/CD Deployment for Homepage

## Overview

This report documents the setup of a GitLab CI/CD pipeline for a self-hosted Homepage instance.

The goal was simple:

> Push configuration changes to the `main` branch and automatically synchronize them to the production Homepage server.

Final architecture:

```text
Local workstation
      |
      | git push
      v
GitLab
siemforge/homepage
      |
      | CI pipeline
      v
GitLab Runner
host 27
      |
      | SSH + rsync
      v
192.168.1.20
homepage-deploy
      |
      v
/opt/homepage/
      |
      v
Homepage Docker container
```

The completed setup removes manual file copying while retaining Git history, repeatable deployments and an auditable CI pipeline.

---

## 1. Environment

### GitLab

Self-hosted GitLab:

```text
https://gitlab.siemforge.xyz
```

Project:

```text
siemforge/homepage
```

Default branch:

```text
main
```

The GitLab Runner is running on host `27` and uses the Docker executor.

### Homepage host

Homepage runs on:

```text
192.168.1.20
```

Configuration directory:

```text
/opt/homepage
```

Container:

```text
homepage
```

Docker bind mount:

```text
/opt/homepage -> /app/config
```

Because Homepage reads its configuration from this bind mount, synchronized changes are picked up by the running instance automatically.

---

# 2. Why GitLab CI/CD?

Manual deployment works, but it introduces several problems:

- Changes can be forgotten or deployed inconsistently.
- Production can drift away from Git.
- It is difficult to know exactly which version is running.
- Manual SSH/file transfers are repetitive.
- There is no consistent deployment process.

GitLab CI/CD provides:

- Version control
- Repeatable deployments
- Automatic deployment after `git push`
- Deployment history through GitLab pipelines
- Dedicated deployment credentials
- Separation between source control and production
- Easy rollback through Git

The resulting workflow is:

```text
Edit
  ↓
git commit
  ↓
git push
  ↓
GitLab CI
  ↓
rsync
  ↓
Homepage
```

---

# 3. Repository structure

The repository contains Homepage configuration such as:

```text
bookmarks.yaml
custom.css
custom.js
docker.yaml
kubernetes.yaml
proxmox.yaml
services.yaml
settings.yaml
widgets.yaml
```

It also contains:

```text
.gitlab-ci.yml
README.md
```

The latter files are useful in GitLab but do not need to be deployed to `/opt/homepage`.

The `.git` directory contains Git metadata and should likewise remain in GitLab rather than production.

Runtime-generated files such as `logs/` are also not source-controlled.

---

# 4. Dedicated deployment account

A dedicated Linux account was used:

```text
homepage-deploy
```

This is preferable to using the personal `benny` account for CI/CD.

The deployment process only needs to write Homepage configuration. It does not require administrator privileges.

The intended permission model is:

```text
benny
  └── configuration ownership / administration

homepage-deploy
  └── automated deployment

homepage
  └── shared filesystem group
```

This follows the principle of least privilege:

> Automated jobs should have only the permissions required to perform their task.

Importantly, the pipeline does not require `sudo`.

---

# 5. Dedicated SSH key

A dedicated Ed25519 SSH key was generated on the GitLab Runner host:

```bash
ssh-keygen -t ed25519   -f ~/.ssh/homepage_deploy   -C "gitlab-ci-homepage"
```

The public key was installed on the Homepage server:

```bash
ssh-copy-id   -i ~/.ssh/homepage_deploy.pub   homepage-deploy@192.168.1.20
```

The private key remains on the CI side as a GitLab CI/CD secret and is never committed to the repository.

This gives the deployment its own identity:

```text
GitLab CI
   |
   | homepage_deploy SSH key
   v
homepage-deploy@192.168.1.20
```

If the deployment credential ever needs to be rotated, it can be replaced without affecting the user's normal SSH account.

---

# 6. Testing SSH independently

Before troubleshooting rsync or GitLab, SSH was tested directly:

```bash
ssh -i ~/.ssh/homepage_deploy     homepage-deploy@192.168.1.20     'echo SSH OK'
```

Successful output:

```text
SSH OK
```

confirmed that:

- The server was reachable.
- SSH was available.
- The deployment account existed.
- The public key was installed.
- The private key was valid.
- Authentication worked.

This was an important troubleshooting step.

A useful general rule is:

> Test the lowest layer independently before debugging the layer above it.

---

# 7. SSH host-key verification

SSH authentication and SSH host verification solve different problems.

The private key answers:

```text
Who am I?
```

The `known_hosts` file answers:

```text
Is this really the server I intended to connect to?
```

The production host's keys were collected with:

```bash
ssh-keyscan -H 192.168.1.20
```

and saved as:

```text
~/.ssh/homepage_known_hosts
```

The `-H` option hashes the host information stored in the file.

This was then supplied to GitLab CI as:

```text
SSH_KNOWN_HOSTS
```

The pipeline copies it to:

```text
~/.ssh/known_hosts
```

This is much safer than disabling host verification with:

```text
-o StrictHostKeyChecking=no
```

The pipeline can therefore verify the production host before sending configuration to it.

---

# 8. GitLab CI/CD variables

Two GitLab CI/CD variables are required:

```text
SSH_PRIVATE_KEY
SSH_KNOWN_HOSTS
```

Both are configured as **File** variables.

At the time of setup:

```text
Type: File
Protected: OFF
```

The variables were not protected because `main` was not configured as a protected branch.

## SSH_PRIVATE_KEY

Contains the complete private Ed25519 key.

It must never be committed to Git.

## SSH_KNOWN_HOSTS

Contains the output from:

```bash
ssh-keyscan -H 192.168.1.20
```

It tells the CI runner which SSH host key to trust.

---

# 9. Important lesson: File variables

One of the most important GitLab CI/CD details encountered during this setup was the difference between normal variables and File variables.

A normal variable exposes its value directly.

A File variable exposes the **path to a temporary file** containing the value.

Therefore:

```bash
cp "$SSH_PRIVATE_KEY" ~/.ssh/homepage_deploy
```

is correct.

This is incorrect for a File variable:

```bash
echo "$SSH_PRIVATE_KEY" > ~/.ssh/homepage_deploy
```

Likewise:

```bash
cp "$SSH_KNOWN_HOSTS" ~/.ssh/known_hosts
```

is correct.

This distinction was directly relevant to one of the failed pipeline runs.

---

# 10. Final `.gitlab-ci.yml`

The final working pipeline is:

```yaml
stages:
  - deploy

deploy_homepage:
  stage: deploy

  image: alpine:latest

  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

  before_script:
    - apk add --no-cache openssh-client rsync
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh

    - cp "$SSH_PRIVATE_KEY" ~/.ssh/homepage_deploy
    - chmod 600 ~/.ssh/homepage_deploy

    - cp "$SSH_KNOWN_HOSTS" ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts

  script:
    - rsync -rv --no-times --no-perms
        --exclude='.git/'
        --exclude='.gitlab-ci.yml'
        --exclude='README.md'
        --exclude='logs/'
        -e "ssh -i ~/.ssh/homepage_deploy"
        ./ homepage-deploy@192.168.1.20:/opt/homepage/
```

## Pipeline stages

There is currently one stage:

```yaml
stages:
  - deploy
```

The deployment job only runs for `main`:

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
```

This means development branches do not automatically deploy to production.

---

# 11. What happens inside the pipeline?

The runner starts an Alpine Linux container:

```yaml
image: alpine:latest
```

It then installs:

```text
openssh-client
rsync
```

The SSH directory is created:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

The private key is copied from the GitLab File variable:

```bash
cp "$SSH_PRIVATE_KEY" ~/.ssh/homepage_deploy
chmod 600 ~/.ssh/homepage_deploy
```

The known-hosts file is installed:

```bash
cp "$SSH_KNOWN_HOSTS" ~/.ssh/known_hosts
chmod 644 ~/.ssh/known_hosts
```

Finally, rsync connects to the production host:

```text
homepage-deploy@192.168.1.20
```

and synchronizes the repository contents to:

```text
/opt/homepage/
```

---

# 12. Why rsync?

`rsync` is a good fit for this type of deployment.

Advantages include:

- Recursive synchronization
- Only changed files need to be transferred
- SSH transport
- Simple CI integration
- Clear output
- No additional deployment daemon
- Well understood and widely available

The deployment command is:

```bash
rsync -rv --no-times --no-perms   --exclude='.git/'   --exclude='.gitlab-ci.yml'   --exclude='README.md'   --exclude='logs/'   -e "ssh -i ~/.ssh/homepage_deploy"   ./ homepage-deploy@192.168.1.20:/opt/homepage/
```

---

# 13. Why `-rv` instead of `-av`?

The initial deployment used:

```bash
rsync -av
```

Archive mode preserves additional filesystem metadata, including:

- Permissions
- Timestamps
- Recursive directory structure
- Other attributes

For this deployment, preserving those attributes was unnecessary.

The first failure was:

```text
rsync: [generator] failed to set times on "/opt/homepage/.": Operation not permitted
```

The files themselves were being transferred successfully. The problem was that the deployment account could not modify directory timestamps.

The command was changed to:

```bash
rsync -rv --no-times
```

That removed the timestamp problem.

A second error then appeared:

```text
rsync: [generator] failed to set permissions on "/opt/homepage/.": Operation not permitted
```

The final solution was:

```bash
rsync -rv --no-times --no-perms
```

This copies the configuration without unnecessarily trying to preserve destination timestamps and permissions.

---

# 14. Excluded files

The final pipeline excludes:

```text
.git/
.gitlab-ci.yml
README.md
logs/
```

## `.git/`

Git history belongs in GitLab.

There is no reason for production to contain the repository's `.git` metadata.

## `.gitlab-ci.yml`

This controls GitLab CI/CD and is not required by Homepage.

## `README.md`

Documentation belongs in the repository rather than the runtime configuration directory.

## `logs/`

Logs are runtime data and should not be controlled by the deployment pipeline.

---

# 15. Why existing `.git` and `logs` were not automatically removed

Adding a directory to rsync's exclusion list does not remove an existing directory from the destination.

Therefore, after the exclusions were added, existing directories still remained on the production server.

They were removed manually:

```bash
sudo rm -rf /opt/homepage/.git /opt/homepage/logs
```

The exclusions then ensure they are not synchronized again.

---

# 16. Why `--delete` was deliberately avoided

A tempting option is:

```text
--delete
```

This would make the destination more closely match the Git repository.

However, it also means files that exist on the production server but not in Git could be deleted.

That is undesirable when runtime data exists under the same directory.

For example:

```text
/opt/homepage/
├── Git-managed configuration
└── runtime-generated files
```

With `--delete`, runtime files could potentially be removed.

Therefore the current deployment intentionally uses a safer, non-destructive synchronization model.

If an exact mirror is desired later, production should first be reorganized so that Git-managed configuration and runtime data live in separate directories.

---

# 17. Troubleshooting journey

The final solution was reached through several distinct failures.

Each failure represented a different layer of the deployment stack.

---

## 17.1 Host key verification failure

Initial CI error:

```text
Host key verification failed.
```

This meant SSH was being invoked, but the CI environment did not know the production host's key.

Solution:

```bash
ssh-keyscan -H 192.168.1.20
```

The result was stored in `SSH_KNOWN_HOSTS` and copied into:

```text
~/.ssh/known_hosts
```

---

## 17.2 Empty private-key File variable

The next failure was:

```text
cp: can't stat '': No such file or directory
```

This showed that the expected GitLab variable was empty or unavailable to the job.

The CI/CD variable configuration was checked and corrected.

The key point was ensuring:

```text
SSH_PRIVATE_KEY
Type: File
```

and that the variable was available to the pipeline's branch.

Because `main` was not protected, the variable could not be configured as Protected while still being available to this pipeline.

---

## 17.3 `libcrypto` private-key error

The next failure was:

```text
Load key "/root/.ssh/homepage_deploy": error in libcrypto: unsupported
```

This indicated that OpenSSH could not parse the private key supplied to the CI job.

The dedicated key was recreated:

```bash
rm -f ~/.ssh/homepage_deploy ~/.ssh/homepage_deploy.pub

ssh-keygen -t ed25519   -f ~/.ssh/homepage_deploy   -C "gitlab-ci-homepage"
```

The public key was reinstalled:

```bash
ssh-copy-id   -i ~/.ssh/homepage_deploy.pub   homepage-deploy@192.168.1.20
```

SSH was then tested independently before returning to CI.

---

## 17.4 rsync code 23: timestamps

Once SSH worked, rsync transferred the configuration but exited with:

```text
rsync error: some files/attrs were not transferred (code 23)
```

The actual cause was:

```text
failed to set times
```

Solution:

```bash
--no-times
```

---

## 17.5 rsync code 23: permissions

After fixing timestamps, rsync reported:

```text
failed to set permissions
```

Solution:

```bash
--no-perms
```

The final rsync options became:

```bash
-rv --no-times --no-perms
```

The deployment then completed successfully.

---

# 18. Final result

The finished system now behaves as intended:

```text
Local configuration change
          |
          v
git commit
          |
          v
git push origin main
          |
          v
GitLab CI pipeline
          |
          v
GitLab Runner
          |
          v
SSH authentication
          |
          v
rsync
          |
          v
192.168.1.20:/opt/homepage/
          |
          v
Homepage container
```

Changes pushed to `main` are automatically synchronized to the production server, and the running Homepage instance picks up the updated configuration.

No manual deployment step is required.

---

# 19. Security assessment

The current implementation is already substantially better than manually deploying with a personal account.

Good security decisions include:

- Dedicated deployment account
- Dedicated SSH key
- No password authentication required by CI
- No `sudo` in the deployment pipeline
- SSH host-key verification
- Private key stored outside Git
- Runtime logs excluded from deployment
- Git metadata excluded from production
- CI/CD deployment triggered from a specific branch

There are still several improvements worth considering.

---

# 20. Protect `main`

The most important improvement would be protecting the `main` branch.

Currently:

```text
main
```

is not protected.

A more secure production workflow would be:

```text
Feature branch
      |
      v
Merge Request
      |
      v
Review
      |
      v
Protected main
      |
      v
Production deployment
```

Once `main` is protected, the CI/CD SSH variables can also be marked:

```text
Protected
```

This creates an additional security boundary.

Production credentials would only be available to trusted pipelines.

---

# 21. Use environment-scoped variables

A future configuration could define:

```text
production
```

as a GitLab environment.

The SSH variables could then be scoped specifically to production.

This becomes especially useful if staging is added:

```text
develop
   |
   v
staging

main
   |
   v
production
```

Each environment can have its own credentials and target server.

---

# 22. Add staging

A natural next step is a staging Homepage instance.

The pipeline could become:

```text
Feature branch
      |
      v
Validation
      |
      v
Staging
      |
      v
Testing
      |
      v
Merge to main
      |
      v
Production
```

This prevents configuration mistakes from immediately reaching the production instance.

---

# 23. Pin container and CI versions

The current pipeline uses:

```yaml
image: alpine:latest
```

and the Homepage container uses:

```text
ghcr.io/gethomepage/homepage:latest
```

Using `latest` is convenient, but it reduces reproducibility.

A more controlled setup would pin specific versions.

For example:

```yaml
image: alpine:<version>
```

and:

```text
ghcr.io/gethomepage/homepage:<version>
```

This separates application upgrades from configuration deployments.

For even stronger reproducibility, container images can be pinned by digest.

---

# 24. Add validation before deployment

The current pipeline goes directly from Git commit to deployment.

A future version could validate configuration first:

```text
Git push
    |
    v
YAML validation
    |
    +---- failure
    |
    v
SSH setup
    |
    v
rsync
```

This prevents obvious syntax errors from reaching production.

Homepage-specific validation should be added where practical.

---

# 25. Add a post-deployment health check

The pipeline currently verifies that rsync completes.

It does not yet verify that Homepage is healthy after the deployment.

A stronger pipeline would do:

```text
rsync
  |
  v
Health check
  |
  +---- failure --> pipeline fails
  |
  v
Deployment successful
```

The health check could verify that the Homepage HTTP endpoint responds successfully.

This changes the meaning of a successful pipeline from:

```text
Files were copied
```

to:

```text
Files were copied and Homepage is responding
```

---

# 26. Improve SSH command hardening

The current SSH invocation is functional:

```bash
-e "ssh -i ~/.ssh/homepage_deploy"
```

A more explicit hardened configuration could use options such as:

```text
-o IdentitiesOnly=yes
-o BatchMode=yes
-o StrictHostKeyChecking=yes
```

For example:

```bash
ssh -i ~/.ssh/homepage_deploy     -o IdentitiesOnly=yes     -o BatchMode=yes     -o StrictHostKeyChecking=yes
```

This makes the security expectations explicit:

- Use the intended identity.
- Never ask for interactive input.
- Require host-key verification.

---

# 27. Restrict the deployment account further

The `homepage-deploy` account could potentially be hardened further at the SSH level.

The authorized key can use restrictions such as:

```text
no-agent-forwarding
no-port-forwarding
no-X11-forwarding
no-user-rc
```

These should be introduced carefully because rsync uses SSH to execute a remote rsync process.

The objective is to reduce the impact if the CI private key is ever compromised.

---

# 28. Keep secrets out of the repository

Homepage configuration should be reviewed for:

- API tokens
- Passwords
- Service credentials
- Private keys
- Other sensitive values

These should not be committed directly into Git.

Better options include:

- GitLab CI/CD variables
- Environment variables
- Docker secrets
- A dedicated secrets manager
- Another existing infrastructure secret-management solution

Remember that removing a secret from the current file does not remove it from Git history.

---

# 29. Deployment concurrency

If multiple commits are pushed quickly, two deployment jobs could potentially overlap.

A future GitLab configuration could use a deployment resource lock such as:

```yaml
resource_group: homepage-production
```

This ensures that production deployments are serialized.

For a small homelab this may not be necessary, but it is useful as the deployment process grows.

---

# 30. Rollback

Git now provides a natural rollback mechanism.

If a configuration change breaks Homepage:

```text
Bad commit
    |
    v
Problem detected
    |
    v
git revert
    |
    v
Push
    |
    v
GitLab CI
    |
    v
Previous configuration restored
```

This is one of the biggest benefits of moving the configuration into Git.

A future improvement could add a dedicated rollback job, but ordinary Git reverts are already effective.

---

# 31. Recommended mature pipeline

The current pipeline is intentionally simple.

A more advanced implementation could eventually look like:

```text
                 Git push
                    |
                    v
             GitLab CI pipeline
                    |
             +------+------+
             |             |
             v             v
        YAML validation   Linting
             |             |
             +------+------+
                    |
                    v
             Staging deploy
                    |
                    v
              Health check
                    |
                    v
              Merge to main
                    |
                    v
           Production deploy
                    |
                    v
              Health check
                    |
                    v
              Deployment OK
```

This provides a sensible evolution path without adding unnecessary complexity immediately.

---

# 32. Troubleshooting methodology

The most useful lesson from this project was to troubleshoot each layer separately.

The effective sequence was:

```text
1. Network connectivity
       |
       v
2. SSH availability
       |
       v
3. SSH host-key verification
       |
       v
4. SSH authentication
       |
       v
5. Remote filesystem permissions
       |
       v
6. rsync behavior
       |
       v
7. GitLab CI variable handling
       |
       v
8. Complete pipeline
```

Examples:

```text
Host key verification failed
```

is primarily an SSH trust problem.

```text
Load key ... error in libcrypto
```

is a private-key formatting/loading problem.

```text
failed to set permissions
```

is a filesystem/rsync attribute problem.

Separating these layers makes CI/CD troubleshooting considerably easier.

---

# 33. Useful commands

## Test SSH

```bash
ssh -i ~/.ssh/homepage_deploy     homepage-deploy@192.168.1.20     'echo SSH OK'
```

## Generate known_hosts

```bash
ssh-keyscan -H 192.168.1.20 > ~/.ssh/homepage_known_hosts
```

## Verify the public key from the private key

```bash
ssh-keygen -y -f ~/.ssh/homepage_deploy
```

## Check the deployment directory

```bash
ls -la /opt/homepage
```

## Test deployment-user write access

```bash
sudo -u homepage-deploy touch /opt/homepage/.ci-test
sudo -u homepage-deploy rm /opt/homepage/.ci-test
```

## Manual rsync test

```bash
rsync -rv --no-times --no-perms   --exclude='.git/'   --exclude='.gitlab-ci.yml'   --exclude='README.md'   --exclude='logs/'   -e "ssh -i ~/.ssh/homepage_deploy"   ./ homepage-deploy@192.168.1.20:/opt/homepage/
```

---

# 34. Final architecture

```text
┌──────────────────────────┐
│ Local workstation        │
│                          │
│ Homepage YAML/CSS/JS     │
└────────────┬─────────────┘
             │
             │ git push
             ▼
┌──────────────────────────┐
│ Self-hosted GitLab       │
│                          │
│ siemforge/homepage       │
└────────────┬─────────────┘
             │
             │ CI/CD
             ▼
┌──────────────────────────┐
│ GitLab Runner            │
│ host 27                  │
│ Docker executor          │
│                          │
│ SSH + rsync              │
└────────────┬─────────────┘
             │
             │ SSH
             │ homepage-deploy
             ▼
┌──────────────────────────┐
│ 192.168.1.20             │
│                          │
│ /opt/homepage/           │
└────────────┬─────────────┘
             │
             │ Docker bind mount
             ▼
┌──────────────────────────┐
│ Homepage container       │
│                          │
│ /app/config              │
└──────────────────────────┘
```

The operational workflow is now:

```text
git push
   ↓
GitLab CI
   ↓
SSH
   ↓
rsync
   ↓
Homepage
```

---

# 35. Conclusion

The completed pipeline provides a clean and practical CI/CD deployment mechanism for a self-hosted Homepage instance.

The most important design decisions were:

1. **Git is the source of truth.**
2. **`main` triggers the production deployment.**
3. **A dedicated `homepage-deploy` account is used instead of a personal account.**
4. **A dedicated Ed25519 SSH key is used for automation.**
5. **SSH host keys are explicitly verified using `known_hosts`.**
6. **Private keys and host keys are stored as GitLab File variables.**
7. **rsync handles the configuration synchronization.**
8. **Git metadata, CI configuration, documentation and runtime logs are excluded.**
9. **Filesystem timestamps and permissions are not unnecessarily preserved.**
10. **`--delete` is intentionally avoided until runtime and Git-managed data are completely separated.**

The final result is exactly the desired automation:

```text
Change Homepage configuration
        ↓
Commit
        ↓
Push to main
        ↓
GitLab CI pipeline
        ↓
SSH authentication
        ↓
rsync deployment
        ↓
Homepage automatically uses the updated configuration
```

This turns Homepage configuration management from a manual process into a small, reproducible infrastructure-as-code workflow.

For a homelab, this is already a very solid foundation. The next logical improvements are protected `main`, staging, validation, health checks, pinned versions and tighter SSH restrictions.
