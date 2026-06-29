---
name: create-skill
description: "対話型スキル作成コマンド。スキル名・説明・動作内容をヒアリングし、SKILL.md を生成・配置し、Oracle レビューによる自動修正まで行う。"
---

# /create-skill — スキル作成コマンド

このスキルは、OpenCode 環境向けの新しいスキルを作成します。
ユーザーから必要な情報を対話的にヒアリングし、SKILL.md を生成して適切な場所に配置し、
Oracle サブエージェントによる品質レビューと自動修正まで行います。

**環境適応**: opencode の設定ディレクトリや dotagents リポジトリのパスは
実行時に自動検出するため、どんな環境でも動作します。

## 使い方

### 引数付き起動（クイックスタート）

引数でスキル名と説明を指定すると、その情報を初期値として対話が始まります：

```text
skill(name="create-skill", user_message="pdf-converter PDFファイルを画像に変換するスキル")
```

### 引数なし起動（フル対話）

すべての情報を対話的にヒアリングします：

```text
skill(name="create-skill")
```

---

## [1/5] スキル情報の収集

### 1a. 引数の解析

`$ARGUMENTS` または `user_message` が空でない場合、解析を試みます：

- 最初の単語 → スキル名（仮）
- 残り → スキル説明（仮）

これらを初期値として使いますが、後続の質問で確認・修正できます。

### 1b. 対話的ヒアリング

以下の情報を `question` ツールで順に収集します：

**① スキル名**
```text
kebab-case（ハイフン区切り）で入力。例: pdf-converter, code-review, data-analyzer
```

**② スキルの説明（1行）**
```text
SKILL.md の frontmatter `description:` に記載する短い説明。
例: 「PDFファイルを画像に変換し、OCR処理を行うスキル」
```

**③ スキルの目的・動作内容（詳細）**
```text
スキルが何をするのか、どのようなワークフローか。マークダウン本文に記載する内容。
例:
- 入力PDFを受け取りページごとに画像に変換
- tesseract で OCR 処理
- 結果を JSON で出力
```

**④ 使用するツール・アクション**
```text
スキル内で使用するツールの種類。複数選択可：
- task() によるサブエージェントの活用
- bash コマンドの実行
- ファイル操作（write / edit / read）
- webfetch による外部リソース取得
- websearch による情報検索
- context7 によるドキュメント参照
- その他（自由記述）
```

**⑤ スキルの配置場所**
後述の `1c. 環境パスの自動検出` で検出したパスを基に、以下の選択肢から選択：
```text
a) ${OPENCODE_SKILL_DIR}/<name>/ のみ（opencode グローバルスキル）
b) ${DOTAGENTS_SKILL_DIR}/<name>/ のみ（APM プラグイン用）
c) ${PROJECT_SKILL_DIR}/<name>/ のみ（プロジェクト固有スキル）
d) グローバル + プロジェクトの両方（推奨）
e) カスタムパス
```

**⑥ その他特記事項**
```text
依存関係、注意事項、制約などがあれば。
```

### 1c. 環境パスの自動検出

上記のヒアリングと並行して（またはヒアリング前に）、以下のパスを自動検出します。
検出結果は変数 `OPENCODE_SKILL_DIR`、`DOTAGENTS_SKILL_DIR`、`PROJECT_SKILL_DIR` に保持し、
以降のフェーズで使用します。どの層も見つからなくてもエラーにせず、ユーザーに尋ねます。

**検出手順:**

スキルには **3つの設置層** があります。目的に応じて適切な層を選びます。

| 層 | 役割 | 検出パス（優先順） |
|----|------|-------------------|
| **Global** | 全プロジェクトで使う汎用スキル | `$XDG_CONFIG_HOME/opencode/skill/` → `~/.config/opencode/skill/` → `~/.config/opencode-go/skill/` |
| **Plugin** | APM で配布・管理するスキル | `ghq` → `~/.ghr/github.com/*/dotagents/` → `find` 探索 → `~/dotagents/` |
| **Project-local** | 特定プロジェクト専用スキル | **カレントディレクトリの `.agents/skills/`**（実行時に確定） |

```text
# opencode グローバルスキルディレクトリの検出（優先順位順）
1. $XDG_CONFIG_HOME/opencode/skill/   （XDG 準拠、最優先）
2. ~/.config/opencode/skill/           （汎用 opencode）
3. ~/.config/opencode-go/skill/        （opencode-go 特殊プロファイル）
4. $OPENCODE_CONFIG_DIR/skill/ 環境変数が設定されていればそれも候補
→ 見つかった最初の既存ディレクトリを使用
→ どれも存在しない場合は ~/.config/opencode/skill/ をデフォルトとする
```

```text
# dotagents（APM プラグインリポジトリ）の検出
1. ghq list --full-path | grep dotagents  （ghq 管理下）
2. ls ~/.ghr/github.com/*/dotagents/     （ghr 管理下）
3. find ~ -maxdepth 6 -name "dotagents" -type d 2>/dev/null
   （全探索、重いので最終手段）
4. ~/dotagents/                            （シンプルな fallback）
→ 見つかった最初の既存ディレクトリを使用
→ どれも見つからない場合はユーザーに直接パスを尋ねる
```

```text
# プロジェクト固有スキルディレクトリの検出
# Global や Plugin と違い、これは「今いるプロジェクトのルート」に依存する
1. .agents/skills/   （カレントディレクトリ直下、APM デプロイ先）
2. .opencode/skill/  （opencode プロジェクト設定）
→ 両方存在する場合はユーザーに選ばせる
→ どちらも存在しない場合は .agents/skills/ を作成することを提案
```

**bash での実装イメージ:**
```bash
# opencode グローバルスキルディレクトリ
OPENCODE_SKILL_DIR=""
for dir in \
  "${XDG_CONFIG_HOME:-$HOME/.config}/opencode/skill" \
  "$HOME/.config/opencode/skill" \
  "$HOME/.config/opencode-go/skill"; do
  [ -d "$dir" ] && { OPENCODE_SKILL_DIR="$dir"; break; }
done
# 環境変数 $OPENCODE_CONFIG_DIR が設定されていればチェック
[ -z "$OPENCODE_SKILL_DIR" ] && [ -n "$OPENCODE_CONFIG_DIR" ] && [ -d "$OPENCODE_CONFIG_DIR/skill" ] && {
  OPENCODE_SKILL_DIR="$OPENCODE_CONFIG_DIR/skill"
}
: "${OPENCODE_SKILL_DIR:=$HOME/.config/opencode/skill}"

# dotagents スキルディレクトリ（ghq を最優先）
DOTAGENTS_SKILL_DIR=""
which ghq >/dev/null 2>&1 && {
  d=$(ghq list --full-path 2>/dev/null | grep dotagents | head -1)
  [ -n "$d" ] && [ -d "$d/skills" ] && DOTAGENTS_SKILL_DIR="$d/skills"
}
# ghq で見つからなければ ghr 管理下を確認
[ -z "$DOTAGENTS_SKILL_DIR" ] && {
  for dir in $HOME/.ghr/github.com/*/dotagents; do
    [ -d "$dir/skills" ] && { DOTAGENTS_SKILL_DIR="$dir/skills"; break; }
  done
}
# それでも見つからなければ探索 → ~/dotagents/ fallback
[ -z "$DOTAGENTS_SKILL_DIR" ] && {
  d=$(find "$HOME" -maxdepth 6 -name "dotagents" -type d 2>/dev/null | head -1)
  [ -n "$d" ] && [ -d "$d/skills" ] && DOTAGENTS_SKILL_DIR="$d/skills"
}
[ -z "$DOTAGENTS_SKILL_DIR" ] && [ -d "$HOME/dotagents/skills" ] && {
  DOTAGENTS_SKILL_DIR="$HOME/dotagents/skills"
}

# プロジェクト固有スキルディレクトリ（git ルート基準）
PROJECT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || echo "$PWD")
if [ -d "$PROJECT_ROOT/.agents/skills" ]; then
  PROJECT_SKILL_DIR="$PROJECT_ROOT/.agents/skills"
elif [ -d "$PROJECT_ROOT/.opencode/skill" ]; then
  PROJECT_SKILL_DIR="$PROJECT_ROOT/.opencode/skill"
else
  PROJECT_SKILL_DIR="$PROJECT_ROOT/.agents/skills"
fi
```

**検出結果の利用**: 検出したパスを `question` の選択肢のデフォルト表示に使います。
ユーザーが「カスタムパス」を選んだ場合は、入力されたパスを優先します。

---

## [2/5] テンプレートの選択とカスタマイズ

収集した情報に基づいて、SKILL.md の構造を組み立てます。

### 基本テンプレート構造

```yaml
---
name: <skill-name>
description: "<skill-description>"
---

# /<skill-name> — <skill-title>

<skill-purpose>

## 使い方

<usage-instructions>

---

## [1/N] <step-title>

<step-description>

## [2/N] <step-title>

...
```

### 既存スキルとの一貫性ルール

- YAML frontmatter は `---` で囲む（`missing-tools` スタイル）
- セクション見出しは `##`、サブセクションは `###`
- コードブロックの言語指定は省略しない
- 日本語で記述する
- 必要な情報は `question` ツールで収集し、決め打ちしない

---

## [3/5] ファイルの生成と配置

### TodoWrite で進捗管理

```js
todowrite([{
  content: "create-skill/[1/5]: スキル情報と環境パスを収集 - expect 6項目の回答",
  status: "completed",
  priority: "high"
}, {
  content: "create-skill/[2/5]: テンプレートを選択し SKILL.md 本文を生成 - expect 生成完了",
  status: "in_progress",
  priority: "high"
}, {
  content: "create-skill/[3/5]: SKILL.md を配置先に書き込み - expect ファイル作成完了",
  status: "pending",
  priority: "high"
}, {
  content: "create-skill/[4/5]: Oracle レビューと自動修正 - expect 問題なし",
  status: "pending",
  priority: "medium"
}, {
  content: "create-skill/[5/5]: 最終検証と完了報告 - expect ユーザーに完了通知",
  status: "pending",
  priority: "medium"
}])
```

### SKILL.md の生成

`write` ツールで SKILL.md を生成します：

1. 収集した情報をもとに frontmatter + 本文を組み立てる
2. 生成内容を一度ユーザーに提示し、修正があれば `edit` で調整する
3. ユーザーが OK したら次の配置フェーズに進む

### ファイルの配置

選択された配置先にファイルを書き込みます：

| 配置先 | パス |
|--------|------|
| opencode グローバルスキル | `${OPENCODE_SKILL_DIR}/<name>/SKILL.md` |
| dotagents プラグイン | `${DOTAGENTS_SKILL_DIR}/<name>/SKILL.md` |
| プロジェクト固有スキル | `${PROJECT_SKILL_DIR}/<name>/SKILL.md` |

配置手順:

1. 各配置先のディレクトリが存在しない場合、`mkdir -p <dir>` で作成する
2. `write` ツールで SKILL.md を書き込む
3. 配置後に `ls <path>/SKILL.md` で実在を確認する
4. ユーザーがプロジェクト固有を選んだ場合のみ、`.gitignore` に `!.agents/skills/<name>/` の追記を提案する

---

## [4/5] レビューと自動修正

このフェーズでは、生成した SKILL.md を Oracle サブエージェントにレビューさせ、指摘事項を自動修正します。
問題がなくなるまで最大3回までループします。

### TodoWrite 更新

```js
todowrite([{
  content: "create-skill/[1/5]: スキル情報と環境パスを収集 - completed",
  status: "completed",
  priority: "high"
}, {
  content: "create-skill/[2/5]: テンプレート選択と生成 - completed",
  status: "completed",
  priority: "high"
}, {
  content: "create-skill/[3/5]: ファイル配置 - completed",
  status: "completed",
  priority: "high"
}, {
  content: "create-skill/[4/5]: Oracle レビューと自動修正 - in_progress",
  status: "in_progress",
  priority: "high"
}, {
  content: "create-skill/[5/5]: 最終検証と完了 - pending",
  status: "pending",
  priority: "medium"
}])
```

### 4a. Oracle レビューの実行

`task()` で Oracle サブエージェントを起動し、生成した SKILL.md をレビューさせます：

```text
task(
  subagent_type="oracle",
  prompt="""\
以下の SKILL.md をレビューし、問題点を列挙してください。
改善点がある場合は「修正が必要」と明示し、具体的な修正案も示してください。

## レビュー基準

1. **YAML frontmatter**:
   - `---` で正しく囲まれているか
   - `name:` が kebab-case で、実際のファイル名と一致しているか
   - `description:` が ' または " で囲まれ、内容が適切か

2. **マークダウン品質**:
   - セクション見出しが `##`、サブセクションが `###` で統一されているか
   - コードブロックに言語指定があるか（\`\`\`sh, \`\`\`js など）
   - セクション区切りに `---` が適切に使われているか

3. **ワークフローの完全性**:
   - 各フェーズに具体的なアクションが記述されているか
   - 使用するツール名が正確に記載されているか（`question`, `todowrite`, `write`, `edit`, `task()` など）
   - エラーハンドリングが考慮されているか

4. **実行可能性**:
   - この SKILL.md を読んだエージェントが迷わず実行できる粒度か
   - 「適切に」「必要に応じて」などの曖昧な指示がなく、具体的な条件が書かれているか

5. **既存スキルとの一貫性**:
   - 日本語で記述されているか
   - このリポジトリの既存スキル（git-commit, missing-tools）のスタイルに合っているか

## レビュー対象ファイル

<生成した SKILL.md の内容をここに埋め込む>
"""
)
```

**出力形式**: Oracle は以下の形式でレビュー結果を返す：

```text
## 問題なし
または
## 問題あり（N件）
1. [severity: high/medium/low] [問題の説明]
   → 修正案: [具体的な修正方法]
2. ...
```

### 4b. 自動修正

**問題がない場合:**
次のフェーズ [5/5] に進む。

**問題がある場合:**
Oracle の指摘に基づいて `edit` ツールで修正を適用する：

1. Oracle の出力から問題リストを読み取る
2. 各問題に対して `edit` で該当箇所を修正
3. 修正後、`todowrite` で試行回数をカウントする

```js
todowrite([...<既存>...,
  {
    content: "create-skill/[4/5]: 自動修正 試行 N/3 回目 - expect 修正適用完了",
    status: "completed",
    priority: "high"
  },
  {
    content: "create-skill/[4/5]: 再レビュー 試行 N/3 回目 - expect 問題なし判定",
    status: "in_progress",
    priority: "high"
  }
])
```

### 4c. 再レビューループ

1. 修正後、再度 Oracle レビューを実行する（初回の `task()` が返した `ses_...` を `task_id` に渡して継続）:
   ```text
   task(task_id="ses_<初回のsession_id>", prompt="修正しました。再レビューしてください。")
   ```
2. 「問題なし」ならループ終了
3. 「問題あり」かつ試行回数 < 3回 → 4b に戻る
4. **3回試行しても問題が解決しない場合** → ユーザーに報告し、残存問題を一覧表示して手動修正を依頼する

### ループ終了条件

- Oracle が「問題なし」と判定 → 成功
- 3回試行しても問題が解決しない → ユーザーに引き継ぎ

---

## [5/5] 検証と完了

### 最終確認

以下のチェックを行います：

- [ ] SKILL.md が正しいパスに存在するか → `glob` で確認
- [ ] frontmatter の name / description が正しいか → `read` で確認
- [ ] マークダウンの構文問題がないか → 未クローズのコードブロックがないか目視確認
- [ ] （推奨）`skill(name="<name>")` でエラーなくロード可能か確認

### 完了サマリー

レビュー結果を含めて以下の形式で報告します：

```text
✓ スキルの作成が完了しました

スキル名: <name>
説明: <description>

作成されたファイル: （実際に作成したものだけ表示）
  - ${OPENCODE_SKILL_DIR}/<name>/SKILL.md    （グローバル）
  - ${DOTAGENTS_SKILL_DIR}/<name>/SKILL.md     （プラグイン）
  - ${PROJECT_SKILL_DIR}/<name>/SKILL.md       （プロジェクト固有）

レビュー結果: {問題なし / N件の問題を自動修正}
{自動修正した問題がある場合はその内容}
{3回試行しても解決しなかった問題がある場合はその一覧}

次のステップ:
  - skill(name="<name>") でスキルをロードして使用開始
  - ユーザーが望めば GUIDE.md などのサポートファイルを追加
```

### エラーハンドリング

- **ファイル書き込み失敗**: エラーメッセージを表示し、別のパスを提案
- **既存ファイルとの競合**: 上書き確認をユーザーに問い合わせる
- **不正なスキル名**: 有効な kebab-case 形式を提案して再入力を促す
- **Oracle レビュー失敗**: 初回なら `task_id` を引き継いで最大2回リトライ、それでも失敗したらスキップしてユーザーに報告
- **3回の自動修正で解決しない問題**: 残存問題を一覧表示し、ユーザーに手動修正を依頼

---

## 補足: 生成する SKILL.md の品質基準

以下の基準は Oracle レビューでも使用されます。生成時にこれらの基準を満たすことが期待されます。

### フォーマット
1. **YAML frontmatter** は必ず `---` で囲む
2. **description** は ' または " で囲む（シングルクォート優先）
3. **コマンド名** は `/create-skill` のように `/` から始める
4. **セクション区切り** は `---` で視覚的に区切る
5. **コードブロック** は言語指定必須（` ```sh ` など）
6. **日本語** で記述する
7. **この環境の既存スキル**（git-commit, missing-tools）のスタイルに合わせる
8. **依存スキル** があれば冒頭で明記する
9. **TodoWrite** の `content` には `[場所/機能名] 動詞句 - expect 結果` の形式を使う

### レビュー品質基準（Oracle 評価項目）
1. **実行可能性**: 記述された手順がエージェントに迷いなく実行可能な粒度か
2. **具体性**: 「適切に」「必要に応じて」などの曖昧表現がなく、条件が明確か
3. **完全性**: エラーハンドリング、エッジケースの考慮がされているか
4. **ツール正確性**: 使用するツール名（`question`, `todowrite`, `task()` など）が正確か
5. **一貫性**: 既存スキルのスタイル・用語法と矛盾していないか
