---
name: auto-ack-rule
description: Use when an AI agent receives a reply from a peer or another lane, before treating it as handled — distinguish an automatic acknowledgement from a substantive response by filename and content patterns, so an "ack received" is not mistaken for the content actually being taken up.
---

# Auto-ACK Rule

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) の通信 / 連携の流れの中で、 自動的に生成される受領 file (= 「ACK」 「自動受領」 「automated response」 等) と、 実質的な返答 (= substantive response) を物理的に区別する rule。

「自動受領 = 完了」 短絡が起きると、 内容の評価 / 取り込み / 判断が skip されて、 「動いてる風で何も進んでない」 という最大の運用失敗の class に直結する。 この skill はこの短絡への対策の rule。

## いつ使う

- AI agent が peer / co-worker / 別の AI lane からの返答を受け取った時 = 必須
- 「未返事」 / 「完了」 と書く前 = 必須
- 自走 chain の中で 「peer の返事 received = chain 完了」 と書く前 = 必須

## form (= 3 軸の区別 rule)

### 軸 1: 自動受領 (= auto_ack) の検出

file 名 + 中身 pattern で自動的に判定可能な軸:

#### file 名 pattern

- `*auto_ack*` (= 「auto_ack」 keyword 含む)
- `*automated_response*` (= 「automated」 keyword)
- `*acknowledgement_only*` / `*receipt_only*` (= 受領のみ articulate)

#### frontmatter / body pattern

- frontmatter の `type` field に `auto_ack` / `automated` / `acknowledgement` 等の値
- 中身の最初の 1-2 段で 「受領」 「acknowledged」 「received」 のみ (= 評価 / 判断なし)
- 「response_required: no」 + 「priority: low」 + 中身 1-2 段の short receipt

= 上記のいずれかに該当 = 自動受領、 「完了 / 評価 / 取り込み」 の判定材料じゃない。

### 軸 2: 実質的な返答 (= substantive response) の検出

自動受領じゃない、 実質的な返答軸:

#### file 名 pattern

- `*substantive_response*` (= 明示)
- `*review_response*` (= review 軸の返答)
- `*evaluation*` / `*assessment*` (= 評価軸)
- 中身の長さ ≥ 100-200 行 + 評価記述あり

#### frontmatter / body pattern

- frontmatter の `type` field に `substantive_response` / `review` / `evaluation` 等の値
- 中身に「評価 = X」 「採用 / 不採用 = Y」 「修正案 = Z」 の記述
- 「response_required: yes」 (= 返答が必要な軸)
- decision / judgement の記述あり

= 上記のいずれかに該当 = 実質的な返答、 「完了 / 評価 / 取り込み」 の判定材料軸。

### 軸 3: 区別の物理化 (= hook / script で発火)

stop hook / start sweep / wake hook で:

1. file 名 pattern を grep して自動受領を除外
2. 実質的な返答を「未返事 / 取り込み未済」 として count
3. count > 0 + 24-48h 経過 = 警告 surface

reference 実装 (= 概念的):

```bash
# 自動受領を除外 + 実質的な返答のみ count
SUBSTANTIVE_BOARD=$(find "$BOARD" -name "*.md" -newer "$LAST_CHECK" \
  | grep -v "auto_ack\|automated_response\|acknowledgement_only\|receipt_only")

UNREPLIED=0
for f in $SUBSTANTIVE_BOARD; do
  TO=$(grep -m1 "^to:" "$f" | sed 's/^to: *//')
  RESPONSE_REQUIRED=$(grep -m1 "^response_required:" "$f" | sed 's/^response_required: *//')
  if [[ "$TO" == "$MY_NAME" && "$RESPONSE_REQUIRED" == "yes" ]]; then
    if ! grep -q "$(basename $f)" "$BOARD"/*_response_*.md 2>/dev/null; then
      UNREPLIED=$((UNREPLIED + 1))
    fi
  fi
done

[[ $UNREPLIED -gt 0 ]] && echo "[warn] substantive unreplied: $UNREPLIED 件" >&2
```

## 由来 (= 実運用の振り返り、 内部使用実績)

実際の運用での振り返り (= 固有名 / 日付は伏せて一般化):

- 「自動受領 = 完了」 短絡が 1 朝に 4 回再発した実例 = 受け取り側の物理 audit 手順を起こした起点
- peer agent の実質的な返答への「軽い ACK + 詳細別 sit」 を 5 日間放置した実例 = 本 rule の典型例
- turn end hook の auto_ack file 除外 logic を base に、 別 lane surface 未言及検出 + skill 未言及検出を追加実装

= 本 rule は「自社環境で踏んだ失敗 → 物理対策」、 「全 user に効く」 ものではない。 「失敗の class が実在する」 という事実 + 「物理対策の汎用化」 が main で、 内部使用実績はその補足の証拠。

## boundary

- 検出 pattern (= file 名 / frontmatter / body) は project ごとに config 可能、 ただし「自動受領 vs 実質的な返答」 の区別軸は固定
- 「実質的な返答 = 必ず長い」 を仮定しない (= 短い実質返答も実在、 中身の量で判定しない、 form + 中身の評価記述で判定)
