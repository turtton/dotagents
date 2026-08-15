# Sandbox Extra Guide

Resolve file-write and file-access failures caused by the agent sandbox
(bwrap) using the `sandbox-extra.sh` mechanism.

## When to Use

Trigger this skill when the agent hits a file-write or file-access failure that
is caused by sandbox isolation, not by real permissions. Typical symptoms:

- `EROFS` (read-only filesystem) on a path outside the repo
- `ENOENT` on a path that clearly exists on the host
- `EPERM` / `EACCES` on a directory the host user can write
- A tool that needs a socket, config, or data dir that is not mounted
- `systemctl --user` or D-Bus failures because the session bus is not reachable

The sandbox bind-mounts a curated set of paths (config, data, cache, state
dirs, credentials, container sockets, Wayland socket, D-Bus bus, etc.). Before
adding an extra mount, verify the failing path is not already covered.

## Procedure

### Step 1 — Confirm the failure is sandbox-related

Reproduce the failure and capture the exact error:

```sh
touch /some/host/path/test 2>&1
# Read-only file system
```

If the path is not under the repo root and not one of the standard mounted
paths, it is a sandbox gap.

### Step 2 — Choose the extra file path

The sandbox sources a project-local extension file at startup:

| Tool     | Extra file path                          |
|----------|------------------------------------------|
| opencode | `<repo-root>/.opencode/sandbox-extra.sh`  |
| senpi    | `<repo-root>/.senpi/sandbox-extra.sh`     |

Determine which tool you are running under and use the corresponding path.

### Step 3 — Write the sandbox-extra.sh

Create the extra file at the repo root with the needed `BWRAP_ARGS+=` entries.
Choose the mount type carefully:

| Mount flag       | Use case                                        |
|------------------|-------------------------------------------------|
| `--bind`         | Read-write access (config/data dirs the agent must modify) |
| `--ro-bind`      | Read-only access (config, credentials, toolchains) |
| `--ro-bind-try`  | Read-only, ignore if the source is missing        |

Common patterns:

```bash
# Read-write project-specific cache or data dir
BWRAP_ARGS+=(--bind "${HOME}/.cache/my-tool" "${HOME}/.cache/my-tool")

# Read-only config dir
BWRAP_ARGS+=(--ro-bind "${HOME}/.config/my-tool" "${HOME}/.config/my-tool")

# A socket the tool needs
BWRAP_ARGS+=(--bind "/run/user/$(id -u)/my-socket" "/run/user/$(id -u)/my-socket")
```

Rules:

- Normally use identical source and destination paths; bwrap allows
  different paths, but identical paths are the convention for exposing the
  same host location.
- Prefer `--ro-bind` unless the agent genuinely needs to write.
- Do not mount `$HOME` wholesale — that defeats the sandbox's isolation.

### Step 4 — Verify the mount is correct

```sh
bash -n .opencode/sandbox-extra.sh   # or .senpi/
```

Confirm the source path exists on the host (unless the entry uses
`--ro-bind-try`, which tolerates a missing source) and the destination matches
the source.

### Step 5 — Recommend a restart

The sandbox is constructed at process launch. The change only takes effect on
the next start of opencode/senpi. Tell the user:

> The sandbox config change requires a restart of {opencode|senpi} to take
> effect. Restart the agent session and retry the failing operation.

Do not attempt to hot-reload the sandbox — there is no live mechanism.

## Notes

- `sandbox-extra.sh` is sourced, not executed, so it must be valid bash and
  must only append to `BWRAP_ARGS` (or set environment variables via
  `BWRAP_ARGS+=(--setenv ...)`).
- The file is evaluated inside the sandbox-building shell, which runs with the
  user's normal environment — `$HOME`, `$USER`, `$XDG_RUNTIME_DIR`, etc. are
  available.
