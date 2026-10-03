# dotagents

[APM](https://microsoft.github.io/apm/) plugin for [oh-my-openagent (omo)](https://github.com/code-yeongyu/oh-my-openagent) skills.

## Skills

| Skill | 分類 | Description |
|-------|------|-------------|
| `create-skill` | **omo** | Interactive skill scaffolder — gathers requirements, generates SKILL.md, and auto-fixes via Oracle review |
| `git-commit` | **omo** | Structured commit workflow with context gathering, message drafting, and hook handling |
| `missing-tools` | **generic** | Missing tool use powered by nix systems |
| `sandbox-extra` | **omo** | Resolves file-write/file-access failures under the opencode/senpi bwrap sandbox via `sandbox-extra.sh` mount configuration |
| `worktree-pr` | **generic** | Work in project-local worktrees, create a PR, and merge and clean up after user approval and CI success |

### 分類の詳細

- **omo**: oh-my-openagent (omo) 周辺環境のツール、設定パス、sandbox などに依存するスキルです。OpenCode / Senpi (omo-native) での利用を想定できますが、Senpi との完全互換を示す分類ではありません。
  - `create-skill`: 現在の本文は OpenCode の Oracle / `task()` API と設定ディレクトリ検出に依存します。Senpi で使うにはツール呼び出しとパスの調整が必要です。
  - `git-commit`: コミット手順は汎用ですが、GUIDE.md を `~/.config/opencode/skill/git-commit/` の固定パスから読み込み、omo の `git-master` スキルとの併用を前提としています。omo-native にも `git-master` はありますが、Senpi では GUIDE の参照パスを調整する必要があります。
  - `sandbox-extra`: 手順は OpenCode / Senpi の bwrap sandbox に対応していますが、GUIDE.md の参照先は `~/.config/opencode/skill/sandbox-extra/` に固定されています。Senpi で使うには参照パスを調整する必要があります。
- **generic**: 特定のエージェント環境に依存せず、他環境でも流用可能なスキルです。
  - `missing-tools`: nix / direnv 環境が前提となるだけで、特定のエージェントには依存しません。
  - `worktree-pr`: Git とホスティングの CLI / API を使い、`.worktrees` 内での作業から PR 作成、許可後のマージと後片付けまで進めます。特定のエージェントや追加スキルには依存しません。

Nix 利用者は `import ./nix/skills.nix` で skill ID → 分類の属性セットを取得できます。分類に応じた配布先は利用側で決めます。

> **Note:** These skills are designed primarily for oh-my-openagent (omo) and its agent system. See the classification above for portability.

## Usage

Add this plugin as a dependency in your project's `apm.yml`:

```sh
apm install turtton/dotagents
```

Skills will be deployed to `.agents/skills/` (when using `agent-skills` target).

## License

MIT
