---
name: sandbox-extra
description: "MUST USE when a file-write or file-access failure occurs in a specific directory under opencode/senpi sandbox (bwrap). Reads GUIDE.md for the sandbox-extra.sh configuration procedure and user-facing restart recommendation. Triggers: 'sandbox', 'sandbox-extra', 'bwrap', 'EROFS', 'read-only filesystem', 'file write failed', 'cannot write to', 'permission denied (sandbox)', 'directory not writable', 'sandbox config'."
---

# Sandbox Extra

This skill resolves file-write and file-access failures caused by the opencode/senpi
bwrap sandbox. The full procedure is in a separate file.

**IMPORTANT**: Before proceeding, read the full guide using the Read tool:

```
Read file: ~/.config/opencode/skill/sandbox-extra/GUIDE.md
```

Then follow the procedure described in GUIDE.md exactly.

## Quick Reference (use GUIDE.md for details)

1. **Identify the failure** — confirm the error is a sandbox mount issue (EROFS,
   ENOENT, permission denied on a path outside the repo)
2. **Create sandbox-extra.sh** — `.opencode/sandbox-extra.sh` for opencode or
   `.senpi/sandbox-extra.sh` for senpi, with `BWRAP_ARGS+=` mount entries
3. **Verify the mount** — `bash -n` and confirm the path and mode
4. **Recommend a restart** — the sandbox must be restarted for the change to
   take effect
