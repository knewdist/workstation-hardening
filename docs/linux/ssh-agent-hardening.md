# SSH Agent Hardening (Arch Linux + Sway)

## Goal
Make Git/SSH use your SSH key **without re-entering the passphrase every time**, while keeping the key encrypted on disk and only unlocked in-memory for your session.

This doc covers:
- A simple `ssh-agent` approach (recommended baseline)
- Optional: systemd user agent
- Checks + troubleshooting
- Security notes

---

## Baseline Recommendation
Use **`ssh-agent`** and load your GitHub key once per login/session using `ssh-add`.

This avoids storing any GitHub token/password for Git operations and keeps your private key encrypted at rest.

---

## Prereqs
- You already have an SSH key (example): `~/.ssh/id_ed25519_github`
- Your GitHub account has the **public key** added (`~/.ssh/id_ed25519_github.pub`)
- Your git remote should use SSH (not HTTPS):
  - `git@github.com:knewdist/<repo>.git`

Check:
```bash
git remote -v


## Step 1 — Fix Key Permissions

SSH will refuse keys with loose permissions.

chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519_github
chmod 644 ~/.ssh/id_ed25519_github.pub


## Step 2 — Add a GitHub Host Block ####

Edit ~/.ssh/config:

Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
This forces GitHub to use the correct key (and prevents it from trying other keys).


## Step 3 — Start ssh-agent automatically in Sway

Add this to ~/.config/sway/config (near the top is fine):

# --- SSH Agent ---
exec_always --no-startup-id sh -lc '
  if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)" >/dev/null
  fi
'

# Why this works

Ensures an agent exists for your session

Avoids manually running eval "$(ssh-agent -s)" every boot

Keeps keys in memory only

Note: This creates the agent dynamically. If you want a fixed socket path, see the “Systemd user agent” option later.

Reload Sway:

swaymsg reload

