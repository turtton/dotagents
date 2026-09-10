---
name: missing-tools
description: Resolves missing CLI tools. Use when a command is unavailable, a shell reports command not found, or a tool must be run without installing it globally.
---

# Missing Tools

Use this workflow when a command is unavailable in the current shell.

## Priority Order

1. Try the current project's direnv environment:

   ```sh
   direnv exec . <command>
   ```

2. Use `nix run` for tools from nixpkgs:

   ```sh
   nix run nixpkgs#<package> -- <args>
   ```

   When the command may fetch from GitHub, also use the `nix-github-rate-limit` skill.

   If you don't know the package name, search with `nix-index`:

   ```sh
   # One-time database build:
   nix-index

   # Search for the command:
   nix-locate <command>
   # Then: nix run nixpkgs#<package> -- <args>
   ```

3. Use `nix shell` as the last resort:

   ```sh
   nix shell nixpkgs#<package> --command <command>
   ```

   When the command may fetch from GitHub, also use the `nix-github-rate-limit` skill.

## Notes

- Never install missing tools globally. Do not use commands such as `npm install -g`, `npm i -g`, `pnpm add -g`, `yarn global add`, `bun add -g`, `uv tool install`, `brew install`, or language-specific global installers to resolve a missing command.
- Prefer `direnv exec .` first because project-local dev shells often already provide the right tool version and environment variables.
- `nix run` works without a TTY and doesn't require an interactive picker.
