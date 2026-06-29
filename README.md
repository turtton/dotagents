# dotagents

[APM](https://microsoft.github.io/apm/) plugin for [oh-my-openagent (omo)](https://github.com/code-yeongyu/oh-my-openagent) skills.

## Skills

| Skill | Description |
|-------|-------------|
| `create-skill` | Interactive skill scaffolder — gathers requirements, generates SKILL.md, and auto-fixes via Oracle review |
| `git-commit` | Structured commit workflow with context gathering, message drafting, and hook handling |
| `missing-tools` | Missing tool use powered by nix systems |

> **Note:** These skills are designed exclusively for oh-my-openagent (omo) and its agent system. They may not work correctly with other AI coding assistants.

## Usage

Add this plugin as a dependency in your project's `apm.yml`:

```sh
apm install turtton/dotagents
```

Skills will be deployed to `.agents/skills/` (when using `agent-skills` target).

## License

MIT
