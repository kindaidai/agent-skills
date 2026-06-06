---
name: tmux-relay
description: |
  tmux経由で他のペインで動作しているClaude Codeにメッセージを送信し、応答を取得する。
  複数のClaude Codeインスタンス間でアドホックな連携を実現する。
---

# tmux-relay

tmux のペイン間通信を使って、他の Claude Code インスタンスにメッセージを送り、応答を取得するスキル。

## 使い方

- `/tmux-relay` — 利用可能なペインを一覧表示し、対話的にメッセージを送信
- `/tmux-relay <pane> <message>` — 指定ペインにメッセージを直接送信

## 前提条件

- tmux セッション内で Claude Code が動作していること
- 送信先のペインで別の Claude Code が起動していること

## 手順

### Step 1: ペインの発見

以下のコマンドで利用可能なペインを一覧表示する:

```bash
tmux list-panes -a -F '#{session_name}:#{window_index}.#{pane_index} | #{pane_current_command} | #{pane_current_path} | #{pane_width}x#{pane_height}'
```

- 自分自身のペインを特定するために `tmux display-message -p '#{session_name}:#{window_index}.#{pane_index}'` を実行し、結果を除外する
- `pane_current_command` が `node` や `claude` を含むペインが Claude Code の候補

### Step 2: メッセージの送信

対象ペインにメッセージを送信する:

```bash
tmux send-keys -t <target_pane> "<message>" Enter
```

**注意事項:**
- メッセージ内のシングルクォート、ダブルクォート、特殊文字は適切にエスケープすること
- 長いメッセージは改行を含めず1行で送ること
- `Enter` は文字列の外に置くこと（`tmux send-keys -t <pane> "message" Enter`）

### Step 3: 応答の取得

送信後、相手の Claude Code が処理を完了するまで待機し、出力を取得する:

```bash
# 直近の出力を取得（100行分）
tmux capture-pane -t <target_pane> -p -S -100
```

**応答待ちのポーリング方法:**

1. 送信直後に 5 秒待機する
2. `tmux capture-pane` で出力を取得する
3. 出力の末尾に Claude Code のプロンプト記号（`>` や `$`）が表示されていれば処理完了
4. まだ処理中であれば、さらに 5 秒待機して再取得する
5. 最大 120 秒（24回）まで繰り返す

```bash
# ポーリングの例
sleep 5
tmux capture-pane -t <target_pane> -p -S -100 | tail -5
```

相手の処理完了を判定する目安:
- 末尾に入力プロンプトが表示されている
- 出力が前回と変わっていない（2回連続で同じ出力）

### Step 4: 結果の報告

取得した応答をユーザーに報告する。相手の出力をそのまま転載するのではなく、要約して伝える。

## 使用例

### 例1: 隣の Claude Code に質問する

```
ユーザー: 「隣のClaudeにバックエンドのAPI仕様を聞いて」

1. tmux list-panes で対象ペインを発見
2. tmux send-keys -t main:2.1 "このプロジェクトのユーザーAPIのエンドポイント一覧を教えて" Enter
3. 応答をポーリングで待機
4. 結果を要約してユーザーに報告
```

### 例2: 別の Claude Code に作業を依頼する

```
ユーザー: 「隣のClaudeにテストを書いてもらって」

1. tmux list-panes で対象ペインを発見
2. tmux send-keys -t main:2.1 "src/utils/format.ts のユニットテストを書いて" Enter
3. 応答をポーリングで待機
4. 完了を確認してユーザーに報告
```

## 制約事項

- tmux セッション外では動作しない
- 相手のペインで Claude Code が起動していない場合、メッセージはシェルに送信される（注意）
- 長時間かかる処理は 120 秒のタイムアウトで打ち切られる
- 相手のコンテキストウィンドウの残量は確認できない
