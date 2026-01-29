# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ansible-based infrastructure-as-code project for managing virtual machines in OpenShift Virtualization (KubeVirt). Automates VM creation, ISO attachment, and information retrieval. In-code comments are written in Polish.

## Running Playbooks

Playbooks are numbered and intended to run in sequence:

```bash
# Step 1: Create VMs (Fedora image)
ansible-playbook 01create.yml
# Step 1 alt: Create VMs (blank disks, no auto-start)
ansible-playbook 01create2.yml

# Step 2: Attach ISO to VMs
ansible-playbook 02iso.yml
# Step 2 alt: Attach ISO with explicit boot order, auto-starts VMs
ansible-playbook 02iso2.yaml

# Step 3: Retrieve VM info (detailed)
ansible-playbook 03vm_info.yaml
# Step 3 alt: Retrieve VM interfaces/MAC addresses only
ansible-playbook 03vminfo.yaml
```

No build system, test suite, linter, or CI/CD pipeline exists.

## Architecture

The project consists of three functional pairs of playbooks (each pair has a standard and an alternative variant):

- **01create / 01create2** — VM creation. Creates N VMs with generated names (`{base}-1`, `{base}-2`, ...), sanitized to RFC1123. `01create` uses a Fedora container disk image; `01create2` creates blank disks and does not start VMs.
- **02iso / 02iso2** — ISO attachment. Attaches a PVC-backed ISO as a CDROM volume. `02iso2` additionally configures explicit boot order (ISO first) and auto-starts VMs.
- **03vm_info / 03vminfo** — Information retrieval. Queries VM specs/status from the cluster. `03vm_info` returns full details (CPU, RAM, disks, interfaces); `03vminfo` returns only interfaces and MAC addresses.

Each playbook independently rebuilds the VM name list and sanitizes names (idempotent, can be run standalone).

**Centralized configuration** lives in `group_vars/vms.yml` — VM count, CPU, memory, disk size, image URL, SSH key, namespace, kubeconfig path, and ISO PVC name.

## Key Dependencies

- **Ansible collections**: `kubevirt.core` (v2.2.3) and `kubernetes.core` (v6.2.0), pre-packaged in `ccoolection.tar.gz` for offline use.
- **ansible.cfg** configures an offline-first collection path (`./collections` and a local filesystem galaxy server).

## Conventions

- Playbook filenames follow `[step_number][action].yml` pattern; alternative variants have a `2` suffix.
- VMs use dual NICs: default pod network (masquerade) + VLAN214 bridge.
- Cloud-init configures SSH keys and a `clouduser` account with sudo access.
