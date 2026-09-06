# dotagents

[APM](https://microsoft.github.io/apm/) plugin for [oh-my-openagent (omo)](https://github.com/code-yeongyu/oh-my-openagent) skills.

## Skills

| Skill | 分類 | Description |
|-------|------|-------------|
| `create-skill` | **opencode 専用** | Interactive skill scaffolder — gathers requirements, generates SKILL.md, and auto-fixes via Oracle review |
| `git-commit` | **汎用**(opencode 依存あり) | Structured commit workflow with context gathering, message drafting, and hook handling |
| `missing-tools` | **汎用** | Missing tool use powered by nix systems |

### 分類の詳細

- **opencode 専用**: opencode / oh-my-openagent (omo) の機能(`task()` サブエージェント、Oracle、opencode 設定ディレクトリなど)に直接依存しており、他の AI コーディングアシスタントでは動作しません。
- **汎用**: スキルの手順自体はエージェント非依存で、他環境でも流用可能です。
  - `git-commit`: コミット手順は汎用ですが、GUIDE.md を `~/.config/opencode/skill/git-commit/` の固定パスから読み込み、omo の `git-master` スキルとの併用を前提としているため、他環境で使うにはパス調整が必要です。
  - `missing-tools`: nix / direnv 環境が前提となるだけで、特定のエージェントには依存しません。

> **Note:** These skills are designed primarily for oh-my-openagent (omo) and its agent system. See the classification above for portability.

## Usage

Add this plugin as a dependency in your project's `apm.yml`:

```sh
apm install turtton/dotagents
```

Skills will be deployed to `.agents/skills/` (when using `agent-skills` target).

## License

MIT
