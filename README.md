# Homelab Ansible

Example Ansible layout for a small Linux lab. It includes:

- a chrony role
- a generic systemd timer role
- a guarded patching playbook that reports first, applies only when enabled, and reboots only when a reboot is required *and* you opt in

The inventory uses `example.com` names and TEST-NET-1 addresses from RFC 5737 (`192.0.2.0/24`). It is not a real network.

## Requirements

Ansible 2.15 or later. Target hosts are expected to be Red Hat family (RHEL, Fedora, and similar) or Debian family (Debian, Ubuntu). Privilege escalation is used by the chrony, timer, and patch playbooks.

## Layout

- `inventory/` — example hosts, group vars, and one host-var file
- `playbooks/` — `ping.yml`, `chrony.yml`, `timer.yml`, `patch.yml`, `site.yml`
- `roles/chrony` — install chrony and deploy a simple configuration
- `roles/systemd_timer` — install a oneshot service plus a calendar timer
- `roles/guarded_patch` — pending-update report, optional apply, optional reboot

## Usage

From this directory:

    ansible-playbook playbooks/ping.yml
    ansible-playbook playbooks/chrony.yml
    ansible-playbook playbooks/timer.yml

Patching is report-only unless you turn the gates on. Start with a dry run:

    ansible-playbook playbooks/patch.yml --check --diff

To install updates but leave the host running if a reboot is needed:

    ansible-playbook playbooks/patch.yml -e patch_apply=true

To install updates and reboot only when the role detects that a reboot is required:

    ansible-playbook playbooks/patch.yml -e patch_apply=true -e patch_reboot=true

If more packages are pending than `patch_max_packages` (default 50), the apply step fails unless you also pass `-e patch_allow_large_batch=true`.

`playbooks/site.yml` applies the chrony and timer roles together.

## Notes

Override `timer_exec_start` and `timer_on_calendar` in group or host vars so the timer command matches the host's package manager. The example command is a DNF check-only run.

This repository does not ship a live inventory, secrets, or a connection to any real machine.
