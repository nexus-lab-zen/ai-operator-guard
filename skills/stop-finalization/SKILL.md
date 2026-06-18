---
name: stop-finalization
description: Use before an AI agent ends its turn — physically check for unanswered messages, unreplied peers, overclaiming wording, unread hints from other lanes, and missing follow-up scheduling, so "I responded" is not mistaken for "the work is finished."
---

# Stop Finalization

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) が応答を終わらせる時、 「終わっていい状態か」 を物理的に確認する hook。 「気をつける」 という普段のクセに頼らず、 物理的な検出で「pause + 次の動きへの言及なし」 「未処理の peer 返事あり」 「自走 chain 仕掛けなし」 を警告として surface する。

「応答終了 = 完了」 という短絡が起きると、 owner が「実際に何が起きたか分からない」 という最大の運用失敗の class に直結する。 この skill はこの短絡への対策。

## いつ使う

- AI agent の turn end 時 = 必須 (= hook として shell script で発火)
- 長い chain の応答終了時 = 特に重要 (= 多 commit + 多 board update 後の「終わりよう」 確認)
- 自走 loop tick の応答終了時 = 必須 (= 次 wake 仕込み確認軸)

## form (= 5 layer の警告)

実装 = shell script (= 例: bash) で stdin から応答 input を受け取り、 stderr に警告を出す。 応答自体は止めない (= 警告のみ、 block しない)。

### Layer 1: 未処理 packet 検出

owner / peer から届いてる board / message のうち、 未返事 + 未対応のものを count。 0 件超過 = 警告 surface。

検出範囲:
- 共有 board directory の自分宛 file (= `<from>_<to>_<topic>.md` form の to が自分)
- response_required field が yes な未返事 file
- 直近 24-48 時間以内の peer からの substantive response

除外範囲:
- 自動受領 file (= 「auto_ack」 / 「automated」 keyword 含む)
- 自分起稿の board (= 自分の起稿物への自分の返事は不要)

### Layer 2: 未返事 peer 検出

peer (= 別の AI agent / co-worker / 自動化 lane) からの board response のうち、 substantive 軸 (= 自動 ACK じゃない) で 24 時間以上未返事のものを count。

これは Layer 1 の subset ではなく、 「peer の response の評価軸」 = 「自動 ACK と実質的な返答の区別」 を物理化する layer。

実運用の振り返り = 「peer の実質的な返答を数日間放置」 という同型が複数回再発し、 物理検出が必須だった。

### Layer 3: 文体検出

応答出力中の文体違反軸を count。 「警告のみ + block しない」 form。

検出範囲:
- 数字盛りの言い方 (= 「販売開始」 「公開」 「全自動」 等の未完成段階での言い方)
- 候補列挙 form (= 「A/B/C/D どれにする?」)
- 過剰な言い方 (= 「直ちに動かす?」 着手前の言い方)
- 比較以外で表を使った場合
- 過剰な外国語混入 (= 1 段落に 5 件以上の英単語、 owner の言語が日本語等の場合)

注意 = 言い換え候補リストを毎回 echo するのは逆効果 (= 「監督じゃなく作業者の文体」 を補強する)、 件数 + 1 行警告のみにして自分の修正力に渡す form。

### Layer 4: 外部 surface 言及確認 (= 別 lane 連携由来)

別 AI agent / 別 lane が出してる「次の小実験」 「次の動き」 surface (= 例: 別 lane の status file の next_min_experiment field) を、 自分の応答が言及してるか確認。 言及なし = 警告 surface。

これは「自分以外の AI agent が hint を出してるが、 自分が拾ってない」 という構造的な disconnect の物理化。 実運用で言うと、 別 lane の agent が毎時 hint を出してるのに自分の lane が読まない、 という同型が複数日続き、 owner の指摘で発覚してから物理検出に切り替えた。

検出範囲:
- 別 lane の status surface (= file path を config で指定)
- 別 lane の「次の動き」 articulate (= 例: top_action / next_behavior / next_min_experiment)

### Layer 5: skill 言及確認 + wake 仕込み確認

応答内で物理 audit skill / 節目 skill (= 例: 物理 audit / 節目の判断確認 skill) の言及があるか確認。 200 文字超過の応答で言及なし = 警告 surface。

加えて、 残作業ありの応答終了時 = 自走 chain (= 次 wake 仕込み) が済んでるか確認。 仕込みなし + 残作業あり = 警告 surface (= 「1 度しか動かない form」 防止)。

除外範囲:
- owner が「今日はここまで」 / 「明日朝で OK」 と明示した時
- 緊急対応中 (= short return 連続)

## form 例 (= 概念的な hook 実装)

下記は SessionStart / Stop hook に接続する時の参考実装。 検出する file path / pattern は利用者ごとに違うので、 そのままではなく自分の環境に合わせて config する。

```bash
#!/usr/bin/env bash
# stop_finalization_hook (= reference 実装)

set -uo pipefail

INPUT=$(cat)
MY_NAME="agent_a"          # = 自分の agent 名、 project ごとに config
SHARED="$HOME/team-ops"    # = 共有 board の path、 project ごとに config

# stop_hook_active flag 確認 (= 無限 loop 防止)
ACTIVE=$(echo "$INPUT" | jq -r '.stop_hook_active // false')
if [[ "$ACTIVE" == "true" ]]; then
    exit 0
fi

# Layer 1: 未処理 packet 検出
PENDING=$(find "$SHARED/board" -name "*_to_${MY_NAME}_*.md" -newer "$SHARED/.last_read" 2>/dev/null | wc -l)
[[ $PENDING -gt 0 ]] && echo "[warn] pending packet: $PENDING 件" >&2

# Layer 2: 未返事 peer 検出
UNREPLIED=$(grep -l "response_required: yes" "$SHARED/board/"*.md 2>/dev/null | grep -v "auto_ack" | wc -l)
[[ $UNREPLIED -gt 0 ]] && echo "[warn] unreplied peer board: $UNREPLIED 件" >&2

# Layer 3: 文体検出 (= 例: 数字盛り / 候補列挙)
LAST_OUTPUT=$(echo "$INPUT" | jq -r '.last_message // ""')
if echo "$LAST_OUTPUT" | grep -qiE "(販売開始|launching|どれにする|直ちに動か)"; then
    echo "[warn] 数字盛り / 候補列挙 / 着手前の言い方 を検出" >&2
fi

# Layer 4: 外部 surface 言及確認 (= 別 lane 連携由来)
EXTERNAL_HINT=$(grep -h "next_min_experiment\|next_behavior" "$SHARED/external_surface/"*.md 2>/dev/null | head -1)
if [[ -n "$EXTERNAL_HINT" ]] && ! echo "$LAST_OUTPUT" | grep -qiE "(hint|小実験|next.behavior)"; then
    echo "[warn] 外部 surface hint 言及なし: $EXTERNAL_HINT" >&2
fi

# Layer 5: skill 言及確認 + wake 仕込み確認
OUTPUT_LEN=${#LAST_OUTPUT}
if [[ $OUTPUT_LEN -gt 200 ]]; then
    if ! echo "$LAST_OUTPUT" | grep -qiE "(物理.audit|節目.skill|物理.見直し)"; then
        echo "[warn] skill 言及なし (= 物理 audit / 節目 skill)" >&2
    fi
fi

# 残作業ありの応答終了 = 自走 chain 仕掛け確認
if [[ $PENDING -gt 0 || $UNREPLIED -gt 0 ]]; then
    if ! echo "$LAST_OUTPUT" | grep -qiE "(schedule.?wakeup|次 wake|wake 仕込み)"; then
        echo "[warn] 残作業あり + 自走 chain 仕掛けなし、 1 度しか動かない form" >&2
    fi
fi

exit 2  # = 警告あり、 応答自体は止めない
```

## boundary

- form (= 5 layer の警告) は固定、 検出範囲 (= file path / pattern) は project config で調整可能
- 警告のみ、 block しない (= 「萎縮 default」 への振り戻し防止)
- 言い換え候補リストを毎回 echo しない (= 「作業者の文体」 補強への対策)
