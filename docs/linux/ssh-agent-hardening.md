# SSH Agent Hardening (Linux Workstations)

## Overview

This document describes a hardened approach to using `ssh-agent` on Linux workstations that access infrastructure, servers, or cloud resources.

The goal is to **reduce SSH private key exposure** while maintaining a usable workflow for administrators and power users.

This guidance applies to **user workstations**, not servers.

---

## Threat Model

### Primary Risks
- SSH private keys remaining loaded indefinitely
- Unauthorized processes accessing the SSH agent socket
- Credential reuse across long-lived desktop sessions
- Agent forwarding abuse
- Silent key usage without user awareness

### Assumptions
- The workstation is used interactively
- SSH keys are passphrase-protected
- The user may access sensitive infrastructure
- Physical access is not guaranteed to be trusted

---

## Security Objectives

- Ensure SSH keys are **encrypted at rest**
- Limit the **lifetime** of loaded keys
- Restrict **who can access the agent**
- Avoid exposing SSH credentials to untrusted processes
- Preserve usability for daily administrative work

---

## Baseline Requirements

Before proceeding, ensure:

- SSH keys are **Ed25519** or stronger
- All private keys are **passphrase-protected**
- `openssh` is installed and up to date

Verify key types:
```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub

Agent Strategy
Key Principles

    One agent per user session

    Agent socket scoped to the user runtime directory

    Explicit key loading (no silent auto-add)

    No persistent agent across reboots

Agent Initialization (Wayland / Sway / Minimal Desktops)

On lightweight or non-GNOME desktops, the SSH agent should be started explicitly and scoped to the session.
Recommended Configuration

Add the following to your compositor or session startup file
(e.g. ~/.config/sway/config or equivalent):

# --- SSH Agent ---
exec_always --no-startup-id ssh-agent -a $XDG_RUNTIME_DIR/ssh-agent.sock
set $SSH_AUTH_SOCK $XDG_RUNTIME_DIR/ssh-agent.sock

This ensures:

    The socket is created in a user-only directory

    The agent is tied to the active session

    The agent is destroyed on logout

Key Loading Policy
Manual Key Loading (Recommended)

Keys should be added only when needed:

ssh-add ~/.ssh/id_ed25519

Advantages:

    Prevents long-lived credential exposure

    Makes key usage explicit

    Reduces silent abuse

Verify Loaded Keys

ssh-add -l

Key Lifetime Restrictions

To reduce risk from unattended sessions, set a key timeout:

ssh-add -t 1h ~/.ssh/id_ed25519

After expiration, the key is automatically unloaded.

This is strongly recommended for:

    Laptops

    Shared environments

    Admin workstations

SSH Configuration Hardening

Edit ~/.ssh/config:

Host *
    AddKeysToAgent no
    ForwardAgent no
    IdentitiesOnly yes

Rationale

    Prevents automatic key loading

    Blocks agent forwarding by default

    Ensures only explicit keys are used

Agent forwarding should only be enabled per-host, never globally.
Optional: GNOME Keyring Integration

On GNOME-based systems, gnome-keyring may manage the SSH agent.
Tradeoffs

Pros

    Automatic key unlock

    Integrated session handling

Cons

    Keys may remain loaded longer than intended

    Reduced visibility into agent state

If using GNOME Keyring:

    Verify SSH_AUTH_SOCK points to the keyring agent

    Ensure keys still have passphrases

    Avoid automatic login without disk encryption

Validation and Testing

Confirm agent socket location:

echo $SSH_AUTH_SOCK

Confirm agent accessibility:

ssh-add -l

Test GitHub authentication:

ssh -T git@github.com

Expected result:

    Authentication succeeds

    No password prompt

    Passphrase requested only when key is first loaded

Operational Best Practices

    Lock the screen when stepping away

    Reboot after long admin sessions

    Avoid reusing keys across unrelated systems

    Never copy private keys between machines

    Use separate keys for:

        Git hosting

        Production servers

        Backup systems

Common Failure Modes
Issue	Cause	Mitigation
Agent accessible after logout	Agent started globally	Scope to session
Key always loaded	Auto-add enabled	Disable AddKeysToAgent
Silent key usage	Agent forwarding	Disable ForwardAgent
Repeated passphrase prompts	No agent	Ensure agent is running
Summary

A hardened SSH agent configuration:

    Protects private keys

    Limits credential exposure

    Preserves administrator usability

    Aligns with real-world threat models

This configuration is appropriate for:

    IT staff

    MSP technicians

    Homelab administrators

    Security-conscious workstation users

References

    man ssh-agent

    man ssh-add

    OpenSSH Security Guidelines

