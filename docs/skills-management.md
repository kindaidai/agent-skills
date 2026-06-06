# Skill 管理ドキュメント

`gh skill`（GitHub CLI の preview 機能）でグローバル skill を管理するための運用記録。

## 目的

`~/.claude/skills`（および共有実体の `~/.agents/skills`）に手動配置されていた skill を、
すべて `gh skill` で **更新追従できる状態（tracking metadata 付き）** に移行する。

`gh skill` は GitHub リポジトリ経由で install/update/publish する仕組みで、
各 skill の `SKILL.md` frontmatter に `metadata.github-*`（repo / ref / tree-sha）を注入し、
`gh skill update` がそれを使って差分検知する。

## 管理方針（upstream か 自前リポか）

判断基準は **「信頼できる canonical な原作者リポが gh skill 公開しているか」**。

- **upstream 追従**: 公式／原作者のリポがある場合は、自前にコピーを持たず元リポから install。
  `gh skill update` で本家の改善を取り込める。
- **自前リポ（`kindaidai/agent-skills`）**: canonical が未公開で第三者 fork しか無い、
  または独自 skill の場合。見知らぬ fork に依存するより自前で持つ方が安全。

> search でヒットしても同名 fork が中身別物のことがある（例: baseline-ui は
> 「UI slop 防止」版と「animation 検証」版が混在）。ヒット＝管理可ではなく、
> **中身が一致する原作者リポか**を確認すること。

## skill 一覧（グローバル全12）

### upstream 追従（公式リポ）

| skill | 管理元 |
|---|---|
| browser-use | `browser-use/browser-use`（公式・`references/` 付きの完全版） |
| difit | `yoshiko-pg/difit`（difit ツール作者） |
| modern-web-guidance | `GoogleChrome/modern-web-guidance`（Google Chrome 公式） |
| terraform-style-guide | `hashicorp/agent-skills`（HashiCorp 公式） |

### 自前リポ `kindaidai/agent-skills`

| skill | 自前管理の理由 |
|---|---|
| baseline-ui | ibelick 原典が gh 未公開、fork は別物多数 |
| frontend-design | Anthropic 公式が gh 未公開、ヒットはミラーのみ |
| web-design-guidelines | Vercel 公式が gh 未公開、ヒットはミラーのみ |
| vercel-react-best-practices | Vercel 原典が gh 未公開、ミラーのみ |
| terraform-module-library | 原作者(wshobson)は marketplace ミラー経由のみ |
| find-skills | canonical 無し |
| tmux-relay | 独自 skill（upstream なし） |
| code-review | 独自 skill（CodeRabbit ベース）、canonical 無し |

## ディレクトリ構成

```
~/.claude/skills/        Claude Code が読む場所
  ├─ baseline-ui/        ← 実体（自前リポ install 版）
  ├─ browser-use/        ← 実体（公式 install 版）
  ├─ difit  → ../../.agents/skills/difit   （symlink）
  └─ ...
~/.agents/skills/        複数エージェント（Cursor/Codex 等）共有の実体
```

- symlink 系の skill は実体が `~/.agents/skills` にあるため、symlink を壊さないよう
  `gh skill install --dir ~/.agents/skills` で共有実体側に install している。
- `~/.claude/skills` の実体 skill は `--scope user` で install。

## 運用

```bash
# まとめて更新（公式 + 自前リポの両方）
gh skill update --all

# 管理状態の確認（未管理があれば "no GitHub metadata" と出る）
gh skill update --dir ~/.claude/skills --dry-run
gh skill update --dir ~/.agents/skills --dry-run
```

### 自前 skill を変更したとき

```bash
cd ~/develop/agent-skills
# skills/<name>/ を編集
git add -A && git commit -m "..." && git push
gh skill publish --tag vX.Y.Z          # バリデーション + リリース作成
gh skill update --all                   # ローカルに反映
```

### skill を追加するとき

1. `skills/<name>/SKILL.md`（必要なら追加ファイル）を置く
2. `gh skill publish --tag vX.Y.Z`
3. `gh skill install kindaidai/agent-skills <name> --agent claude-code --scope user`
   （共有実体に入れる場合は `--dir ~/.agents/skills`）

## リリース履歴

| tag | 内容 |
|---|---|
| v0.1.0 | 初期5つ（baseline-ui, browser-use, frontend-design, web-design-guidelines, tmux-relay） |
| v0.2.0 | browser-use を削除（公式 `browser-use/browser-use` upstream 管理へ移行） |
| v0.3.0 | terraform-module-library, vercel-react-best-practices, find-skills を追加 |
| v0.4.0 | code-review（CodeRabbit）を追加 |

## 補足・残課題

- `~/.agents/skills` にあった `source-command-*`（migrated command の自動生成ラッパー、
  本体 skill と冗長）は削除済み。
- **プロジェクトスコープ** `dopa-global-frontend/.claude/skills/`（create-pr, fix-pr-review,
  post-review, pre-pr-qa 等）は project 管理のため対象外（そのまま）。
- リポの中身は agent 非依存（[Agent Skills 仕様](https://agentskills.io/specification)）。
  `--agent` を変えれば Claude Code 以外（Cursor/Codex/Copilot 等）にも install 可能。
