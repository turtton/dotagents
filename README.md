# dotagents

[APM](https://microsoft.github.io/apm/) plugin for [OhMyOpenCode (omo)](https://github.com/anomalyco/opencode) agent skills.

## Skills

| Skill | Description |
|-------|-------------|
| `git-commit` | Structured commit workflow with context gathering, message drafting, and hook handling |
| `final-review` | Post-implementation review cycle with oracle agents and iterative feedback |

> **Note:** These skills are designed exclusively for OhMyOpenCode (omo) and its agent system. They may not work correctly with other AI coding assistants.

## Usage

Add this plugin as a dependency in your project's `apm.yml`:

```sh
apm install turtton/dotagents
```

Skills will be deployed to `.agents/skills/` (when using `agent-skills` target).

## License

MIT
