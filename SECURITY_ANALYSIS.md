# 🔒 Security Report

**Date:** 2026-07-17 | **Stack:** Unknown / Unknown

## Summary

Security posture is concerning with 3 high-severity vulnerabilities centered on containers running as root and an exposed debug server. The Delve debugger listening on 0.0.0.0:2345 with --accept-multiclient is critical to address immediately as it allows arbitrary remote debugging access. Multiple Dockerfiles lack proper privilege separation and install unnecessary packages (openssh-server, nginx, sudo) that expand the attack surface.

## Severity Breakdown

| Severity | Total | Auto-Fixed |
|----------|-------|------------|
| 🟠 high | 3 | 3 |
| 🟡 medium | 9 | 9 |
| 🟢 low | 3 | 2 |

## Findings

### 🟠 High

**1. Container Running as Root User** — `Dockerfile.tilt:1`
```
FROM docker.io/library/golang:1.26.5@sha256:63f132d58c1f589f0dcda584933a9bb44bfda1150f1506377f5a902f34d86033
```
The Dockerfile does not specify a non-root USER, meaning the container will run as root by default. This violates the principle of least privilege and increases the attack surface if the container is compromised. ✅ Auto-fixed

**2. Debug Server Exposed on All Interfaces** — `Dockerfile.tilt:62`
```
ENTRYPOINT ["/usr/bin/tini", "-s", "--", "dlv", "exec", "--continue", "--accept-multiclient", "--headless", "--listen=:2345", "--api-version=2"]
```
The Delve debugger is configured to listen on all interfaces (0.0.0.0:2345) with --accept-multiclient, allowing any network client to connect and control the debugger. This is extremely dangerous if exposed beyond development environments. ✅ Auto-fixed

**3. Container Running as Root User** — `Dockerfile.ui.tilt:1`
```
FROM docker.io/library/node:24.17.0@sha256:032e78d7e54e352129831743737e3a83171d9cc5b5896f411649c597ce0b11ea
```
The Dockerfile does not specify a non-root user, meaning the container will run as root by default. This violates the principle of least privilege and increases the attack surface if the container is compromised. ✅ Auto-fixed

### 🟡 Medium

**1. Running as root user in builder stage** — `Dockerfile:8`
```
FROM docker.io/library/golang:1.26.5@sha256:63f132d58c1f589f0dcda584933a9bb44bfda1150f1506377f5a902f34d86033 AS builder
```
The builder stage runs as root by default without dropping privileges. While build stages are less critical, running as root can lead to privilege escalation if the build process is compromised. ✅ Auto-fixed

**2. Unnecessary packages installed - openssh-server** — `Dockerfile:17`
```
openssh-server \
```
Installing openssh-server in a container is a security risk as it opens up remote access capabilities that are typically unnecessary and increase the attack surface. ✅ Auto-fixed

**3. Unnecessary packages installed - nginx** — `Dockerfile:18`
```
nginx \
```
Installing nginx in a builder image is unnecessary and increases the attack surface. Web servers should not be in build containers. ✅ Auto-fixed

**4. Sudo installed in container** — `Dockerfile:25`
```
sudo \
```
Installing sudo in a container is generally unnecessary and can be used for privilege escalation attacks. ✅ Auto-fixed

**5. USER root directive without switching back** — `Dockerfile:42`
```
USER root
```
The argocd-base stage explicitly sets USER root for operations. While it does switch to a non-root user later, the explicit USER root is unnecessary as it's the default. ✅ Auto-fixed

**6. Insecure Package Installation Without Verification** — `Dockerfile.tilt:11`
```
RUN apt-get update && apt-get install --no-install-recommends -y \
```
The apt-get install command does not clean up package lists after installation, and the package cache remains in the image. While not a direct vulnerability, combined with running as root, this increases attack surface. ✅ Auto-fixed

**7. World-Writable Directories Created** — `Dockerfile.tilt:46`
```
RUN mkdir -p tls && \
```
Directories are created with default permissions which may be overly permissive, especially for sensitive directories like 'tls' and 'gpg' that will contain cryptographic material. ✅ Auto-fixed

**8. Untagged Base Image Reference** — `Dockerfile.dev:4`
```
FROM argocd-base
```
The base image 'argocd-base' is referenced without a specific tag or digest, which can lead to unpredictable builds and potential security issues if the base image is compromised or updated with vulnerabilities. ✅ Auto-fixed

**9. Running as Root User for Operations** — `Dockerfile.dev:7`
```
USER root
```
The Dockerfile switches to root user to perform operations. While it does switch back to a non-root user (999) at the end, running any operations as root increases the attack surface during build time. ✅ Auto-fixed

### 🟢 Low

**1. Build configuration - CGO_ENABLED=0 is good security practice** — `.goreleaser.yaml`
Build configuration - CGO_ENABLED=0 is good security practice ⚠️ Manual review

**2. World-readable GPG keys directory permissions** — `Dockerfile:77`
```
chmod 0700 gpg/keys
```
While 0700 is set for gpg/keys, the parent directories may have overly permissive settings. The gpg/source directory doesn't have restricted permissions. ✅ Auto-fixed

**3. Binary Copied Without Integrity Verification** — `Dockerfile.dev:5`
```
COPY argocd /usr/local/bin/
```
The argocd binary is copied without any checksum or signature verification, which could allow a tampered binary to be included in the image. ✅ Auto-fixed

## Action Plan

*Generated by Agnixa Recon 2.0 Brain — 2026-07-17T12:36:49.325Z*
