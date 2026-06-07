______________________________________________________________________

## name: final-review description: "End-of-work review cycle skill. Run two baseline rounds of oracle agent review, re-evaluate findings, ask user decisions if needed, apply fixes, then confirm completion. Triggers: 'final review', 'end of work review', 'review cycle'."

# Final Review Cycle

This skill defines the end-of-work review procedure. The full procedure is in a separate file.

**IMPORTANT**: Before proceeding, read the full guide using the Read tool:

```
Read file: ~/.config/opencode/skill/final-review/GUIDE.md
```

Then follow the procedure described in GUIDE.md exactly.

## Quick Reference (use GUIDE.md for details)

1. **Review Cycle 1** — Fire oracle agents, re-evaluate findings, apply fixes
1. **Review Cycle 2** — Repeat with fresh perspective
## Review Scope

- **Covers**: Changes in current session, immediate interfaces affected
- **Does NOT cover**: Unrelated code, pre-existing issues, broad refactoring debates

For complete procedure, diagnostics handling, cycle report format, and all rules → **Read GUIDE.md**.
