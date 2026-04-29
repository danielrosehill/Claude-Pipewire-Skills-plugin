---
name: delete-preference-rule
description: Remove a Wireplumber preference rule — by filename, by matched device name, or by description. Restarts Wireplumber so the change takes effect immediately. Use when a rule is no longer wanted or is causing wrong-device routing.
---

# Delete Preference Rule

Cleanly remove a previously-installed Wireplumber rule.

## Steps

1. Call `list-preference-rules` if the user hasn't named a specific file.
2. Confirm with the user which rule to delete — show the file path and a one-line summary of what it does. Do **not** delete silently.
3. Remove the file:
   ```bash
   rm ~/.config/wireplumber/wireplumber.conf.d/<file>.conf
   ```
4. Restart Wireplumber:
   ```bash
   systemctl --user restart wireplumber
   ```
5. Verify with `wpctl status` — the previously-pinned device should no longer be forced default.

## Safety

- Never `rm -rf` the whole `wireplumber.conf.d/` directory.
- Never touch `/etc/wireplumber/` files without an explicit confirm — those are system-level and may have been installed by a package.
- If unsure whether a rule is custom or distro-shipped, check `dpkg -S <path>` (Debian/Ubuntu) — if it's owned by a package, leave it alone and tell the user.

## Optional: archive instead of delete

If the user might want the rule back, rename rather than remove:

```bash
mv ~/.config/wireplumber/wireplumber.conf.d/<file>.conf{,.disabled}
```

Wireplumber only loads `.conf` files, so the `.disabled` suffix shelves it without losing the content.
