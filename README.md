# dotagents

[APM](https://microsoft.github.io/apm/) plugin for [oh-my-openagent (omo)](https://github.com/code-yeongyu/oh-my-openagent) skills.

## Skills

| Skill | 分類 | Description |
|-------|------|-------------|
| `create-skill` | **omo** | Skill scaffolder for OpenCode / Senpi — gathers requirements, generates SKILL.md, and fixes issues through an available reviewer |
| `git-commit` | **omo** | Structured commit workflow with context gathering, message drafting, and hook handling |
| `missing-tools` | **generic** | Missing tool use powered by nix systems |
| `sandbox-extra` | **omo** | Resolves file-write/file-access failures under the opencode/senpi bwrap sandbox via `sandbox-extra.sh` mount configuration |
| `worktree-pr` | **generic** | Work in project-local worktrees, create a PR, and merge and clean up after user approval and CI success |

### 分類の詳細

- **omo**: oh-my-openagent (omo) 周辺環境で使うスキルです。OpenCode / Senpi (OMO Native) の公開ツールと実際の設定に合わせて動作します。分類自体は、個々の runtime でのロードや実行を保証するものではありません。
  - `create-skill`: 実行中の harness の設定から配置先を決め、利用可能なレビュー役と task API を使います。Native に Oracle や OpenCode と同じ API があるとは仮定しません。
  - `git-commit`: ロードされた SKILL.md と同じディレクトリの GUIDE.md を読み込みます。`git-master` は利用可能で、広い Git 操作に必要な場合に併用します。
  - `sandbox-extra`: ロードされた SKILL.md と同じディレクトリの GUIDE.md を読み込み、OpenCode / Senpi の `sandbox-extra.sh` を扱います。この設定を読み込む bwrap wrapper が前提です。
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
