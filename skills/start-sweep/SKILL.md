---
name: start-sweep
description: Use at the start of a session, right after an autonomous wake, or after a long quiet period — read the board, status files, and other lanes' surfaces before starting new work, so the agent does not lose the state of the previous session and start unrelated work.
---

# Start Sweep

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) のセッション開始時 (= 新規 session 立ち上がり + 自走 wake 直後 + chat 戻り直後) に、 状態把握のために必ず読むものを物理化する hook 的な form。

「セッション開始時に状態が飛ぶ」 「前 session の続きを把握せず新規 chain を発火」 「別 lane の動きを skip して 1 人 lane で動く」 という最大の運用失敗の class への対策。 物理的な確認手順で「必ず read する」 軸を作る。

## いつ使う

- 新規 session 立ち上がり時 = 必須 (= SessionStart hook で発火)
- 自走 wake 直後 = 必須 (= wake 直後の物理 audit 手順と組み合わせ運用)
- 長期 quiet (= 3-6 時間以上) 後の再開時 = 必須
- owner chat 戻り直後 = 任意 (= chat lane を優先、 sweep より対話)

## form (= 11 必読 section)

### Section 1: timestamp + 起動 trigger

```
Sweep — <timestamp> (= 起動 trigger)
```

「いつ + 何経由」 を書く、 後で「直近 sweep N 時間前」 を判定する材料。

### Section 2: project / repo side state

project 配下の状態確認:

- git status (= uncommitted / untracked 軸)
- 直近 N 件の commit history
- 残った TODO / WIP 軸

「自分が触ってる project の現状」 を数秒で把握。

### Section 3: 今日の 1 件 decision

セッション内で「今日動かす 1 件」 を決めるための枠:

- 7 日以内のまだ手をつけてない軸を 1 件
- 今日中に完了可能 + 過大じゃない軸
- owner / peer に説明可能な軸

「件数評価じゃなく value 評価で進む」 軸 (= 経営者視点の節目 check と整合)。

### Section 4: memory-lint

memory file の整合性確認:

- 正本の索引 file (= MEMORY.md 等) の読み戻し
- 直近 N 日の新規 entry 確認
- 重複 / 矛盾の軽い検出

「自分の memory 軸の状態把握」、 重い audit じゃなく軽い lint。

### Section 5: 外部 metrics fetch (= 任意、 必要な project のみ)

外部 service の metrics 取得 (= 売上 / 公開件数 / 顧客数 等):

- platform API call (= 公開 endpoint で取得可能な軸)
- 過大な fetch にしない (= 軽い fire 軸)

数字を持たない project は省略可能。

### Section 6: board audit step (= board 配下の自分宛 message を読む)

共有 board / channel の自分宛 message 確認:

- 自分宛 + 自分起稿 + response_required: yes の file 一覧
- 直近 24-48h の新規 message
- 自動受領 file の除外 (= auto-ack-rule skill 軸)

board の read miss 防止 = 同日に届いた board を読まずに別 work を発火した同型再発への対策。

### Section 7: Controlled Wake / Task Queue

自走の wake queue 状態確認:

- 既に仕込まれてる wake / task の一覧
- 残作業の count
- 自走 chain の cadence

### Section 8: External Outcome Accounting (= 「進んだ」 軸の数字)

外部の動き / 売上の数字軸:

- world_movement (= 公開接点での動き)
- revenue_signal (= 売上 signal)
- gap の明示 (= 「世界の動きなし」 「売上なし」 軸の明示)
- 「進んだ claim」 の許可軸 (= progress_claim_allowed=true/false)

「内部 commit を売上と数えない軸」 + 「世界の動き = 0 なら progress_claim_allowed = false」 軸の物理化。

### Section 9: External Surface Read (= 別 AI agent の hint 読み込み)

別 AI agent / 別 lane の出してる「次の動き」 surface 読み込み:

- 別 lane の status file 軸 (= 例: 別 agent の preflight / review surface file、 file path を config で指定)
- 別 lane の「top_action」 「next_behavior」 「next_min_experiment」
- 自分の判断に取り込む軸

これは「自分以外の AI agent が hint を出してるが、 自分が拾ってない」 構造的な disconnect の対策。

### Section 10: Ambiguity Gate 7 signals self-check (= seven-signal-drift-check 軸)

着手前の 7 signals self-check (= seven-signal-drift-check skill 軸):

- Case A 5 categories (= product audience / expected behavior / safety level / implementation target / missing why)
- Case B 2 軸 (= reversibility / evidence confidence)
- 判定 = ambiguity_gate / soft_binder / tripwire_hold / relay_only / executive_action

mode declaration skill との連動で「sweep 完了 = mode 確定 + 着手 OK」 軸。

### Section 11: Sweep 完了

sweep の終了:

- 読んだ section count + 検出された警告 count
- 次の動き (= 今日の 1 件 + 自走 wake 仕込み)
- 「Sweep 完了 = 着手 OK」

## hook 化サンプル (= SessionStart hook への接続)

セッション開始時に sweep を自動で促したい場合、 SessionStart hook から軽い script を発火する形にできる。 「進行板がどこか」 は環境ごとに違うので、 file path / pattern は自分の環境に合わせて config する (= 推測のままだと空振りする)。

```bash
#!/usr/bin/env bash
# start_sweep_hook (= SessionStart 接続の reference 実装、 軽い fire)

set -uo pipefail

SHARED="$HOME/team-ops"     # = 共有 board の path、 project ごとに config
MY_NAME="agent_a"           # = 自分の agent 名、 project ごとに config

echo "Sweep — $(date '+%Y-%m-%d %H:%M') (= SessionStart)" >&2

# Section 6: 自分宛 + 未返事の board を surface (= 自動受領は除外)
PENDING=$(find "$SHARED/board" -name "*_to_${MY_NAME}_*.md" 2>/dev/null \
  | grep -v "auto_ack\|automated_response\|receipt_only" | wc -l)
[[ $PENDING -gt 0 ]] && echo "[sweep] 自分宛 board: $PENDING 件 (= 読んでから着手)" >&2

# Section 9: 別 lane の hint surface を 1 行 surface
grep -h "next_min_experiment\|next_behavior\|top_action" \
  "$SHARED/external_surface/"*.md 2>/dev/null | head -1 \
  | sed 's/^/[sweep] 別 lane hint: /' >&2

echo "[sweep] sweep を確認してから今日の 1 件を決める" >&2
exit 0
```

## boundary

- 11 section の中身は project ごとに config 可能、 ただし「必読軸」 は固定
- 重い sweep にしない (= 30 秒 - 2 分の最小、 owner chat 戻り直後は省略可)
- sweep 結果は記述のみ、 外部 action の triggering なし
