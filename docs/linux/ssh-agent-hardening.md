SSH Agent Hardening (Sway / Wayland)
Goal

Use an SSH agent so you enter your key passphrase once per session

Avoid leaving decrypted keys laying around

Keep permissions tight

Prefer per-host keys (e.g., id_ed25519_github) and explicit config

1) File permissions (must be correct)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519_github
chmod 644 ~/.ssh/id_ed25519_github.pub


If you use other keys, apply the same pattern.

2) Use a dedicated GitHub key in ~/.ssh/config

Create / edit:

vim ~/.ssh/config


Example:

Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
  AddKeysToAgent yes


Notes:

IdentitiesOnly yes prevents SSH from trying every key it can find.

AddKeysToAgent yes helps auto-add once your agent is available.

3) Start an agent in Sway (recommended socket method)

In your Sway config (commonly ~/.config/sway/config), add:

# --- SSH Agent (per-user, socket in XDG runtime dir) ---
exec_always --no-startup-id ssh-agent -a $XDG_RUNTIME_DIR/ssh-agent.sock
set $SSH_AUTH_SOCK $XDG_RUNTIME_DIR/ssh-agent.sock


Reload sway:

swaymsg reload


Confirm the socket exists:

ls -la "$XDG_RUNTIME_DIR/ssh-agent.sock"

4) Add your GitHub key to the agent
ssh-add ~/.ssh/id_ed25519_github


Verify loaded keys:

ssh-add -l

5) Test GitHub auth
ssh -T git@github.com


Expected: a success message (you may still be prompted the first time per session when adding the key).

6) Make sure your shell is using the same agent socket

If you open new terminals and it “forgets” the agent, export the socket in your shell startup.

For bash: ~/.bashrc

export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.sock"


Then:

source ~/.bashrc

7) Security hardening checklist
✅ Use a passphrase

Make sure the GitHub private key is protected:

ssh-keygen -p -f ~/.ssh/id_ed25519_github

✅ Don’t leave keys world-readable

Re-check:

stat -c "%a %n" ~/.ssh ~/.ssh/id_ed25519_github ~/.ssh/config
✅ Prefer separate keys for separate purposes

id_ed25519_github → GitHub only

id_ed25519_server_backup → servers

Don’t reuse the same private key everywhere.

✅ Consider agent lifetime limits (optional)

Load key and auto-expire after 1 hour:

ssh-add -t 1h ~/.ssh/id_ed25519_github

8) Troubleshooting

“Still asks me for passphrase every time”

Most common causes:

The agent isn’t running

Your shell isn’t pointing at the right SSH_AUTH_SOCK

Key isn’t added to agent

Quick diag:

echo "$SSH_AUTH_SOCK"
ssh-add -l
ps aux | grep ssh-agent | grep -v grep

“Permission denied (publickey)”

Force SSH to use the GitHub key and show debug:
ssh -vvv -i ~/.ssh/id_ed25519_github git@github.com

Look for which key it actually tries.
