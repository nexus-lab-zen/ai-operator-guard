# Auto-ACK Rule Template v0 (= AI Operator Guard vertical slice 後段 4 件 1 件目)

generated_at: 2026-06-09 11:55 JST
status: v0 draft (= 公開判断は owner GO 経由)
origin:
- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 1 件目
- nokaze 内 form (= turn end hook script の auto_ack file 除外 logic) を AI agent 運用一般向けに汎用化
- 内部 origin = 「ACK は complete ではない」 admit + 「軽 ACK + 別 sit」 数日放置の同型再発の振り返り

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) の通信 / 連携の流れの中で、 自動的に生成される受領 file (= 「ACK」 「自動受領」 「automated response」 等) と、 実質的な返答 (= substantive response) を物理的に区別する rule。

「自動受領 = 完了」 短絡が起きると、 内容の評価 / 取り込み / 判断が skip されて、 「動いてる風で何も進んでない」 という最大の運用失敗の class に直結する。 本 template はこの短絡への対策の rule articulate。

## いつ使う

- AI agent が peer / co-worker / 別の AI lane からの返答を受け取った時 = 必須
- 「未返事」 / 「完了」 articulate を持つ前 = 必須
- 自走 chain の中で 「peer の返事 received = chain 完了」 と articulate する前 = 必須

## form (= 3 軸の区別 rule)

### 軸 1: 自動受領 (= auto_ack) の detection

file 名 + 中身 pattern で自動的に判定可能な軸:

#### file 名 pattern

- `*auto_ack*` (= 「auto_ack」 keyword 含む)
- `*automated_response*` (= 「automated」 keyword)
- `*acknowledgement_only*` / `*receipt_only*` (= 受領のみ articulate)

#### frontmatter / body pattern

- frontmatter の `type` field に `auto_ack` / `automated` / `acknowledgement` 等の値
- 中身の最初の 1-2 段で 「受領」 「acknowledged」 「received」 のみ articulate (= 評価 / 判断なし)
- 「response_required: no」 + 「priority: low」 + 中身 1-2 段の short receipt

= 上記のいずれかに該当 = 自動受領、 「完了 / 評価 / 取り込み」 軸の判定材料じゃない。

### 軸 2: 実質的な返答 (= substantive response) の detection

自動受領じゃない、 実質的な返答軸:

#### file 名 pattern

- `*substantive_response*` (= articulate 明示)
- `*review_response*` (= review 軸の返答)
- `*evaluation*` / `*assessment*` (= 評価軸)
- 中身の長さ ≥ 100-200 行 + 評価 articulate あり

#### frontmatter / body pattern

- frontmatter の `type` field に `substantive_response` / `review` / `evaluation` 等の値
- 中身に「評価 = X」 「採用 / 不採用 = Y」 「修正案 = Z」 articulate
- 「response_required: yes」 (= 返答が必要な軸)
- decision / judgement articulate あり

= 上記のいずれかに該当 = 実質的な返答、 「完了 / 評価 / 取り込み」 軸の判定材料軸。

### 軸 3: 区別の物理化 (= hook / script で fire)

stop hook / start sweep / wake hook で:

1. file 名 pattern を grep して自動受領を除外
2. 実質的な返答を「未返事 / 取り込み未済」 として count
3. count > 0 + 24-48h 経過 = warn surface

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

## 自社使用実績 (= evidence、 「appendix-level proof」 軸)

nokaze 環境での実際の運用 (= 固有名 / 日付は伏せて一般化):

- 「自動受領 = 完了」 短絡が 1 朝に 4 回再発した実例 = 受け取り側の物理 audit 手順の起稿 origin
- peer agent の substantive response への「軽 ACK + 詳細別 sit」 を 5 日間放置した実例 = 本 rule の典型 sample
- turn end hook の auto_ack file 除外 logic を base に、 別 lane surface 未言及検出 + skill 未言及検出を追加実装

= 本 rule は「自社環境で踏んだ失敗 → 物理対策」、 「全 user に効く」 軸じゃない、 「失敗の class が実在する」 evidence + 「物理対策の汎用化軸」 が main、 自社使用実績は appendix-level の証拠軸。

## boundary

- 本 template = internal design draft、 公開 / 価格 / 契約なし
- detection pattern (= file 名 / frontmatter / body) は project ごとに config 可能、 ただし「自動受領 vs 実質的な返答」 の区別軸は固定
- 「実質的な返答 = 必ず substantive」 を仮定しない (= short receipt な実質返答も実在、 中身の量で判定しない、 form + 中身の評価 articulate で判定)

## v0 → v1 への進化軸

- 最初の 1-2 週間 actual 採用後 = false positive (= auto_ack 判定したが actual には実質返答) + false negative (= 実質返答判定したが actual には auto_ack) の検証
- AI agent 別 (= Claude Code / Codex / Gemini CLI) の receipt form 適応性検証
- 「短い実質返答 (= 「OK、 詳細 X 軸」 1 段 form)」 と 「auto_ack」 の境界判定の精度検証

## Related

- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 1 件目
- nokaze 内実装 form = turn end hook script の auto_ack file 除外 + 未返事 count logic (= 自社運用で採用中)
- mode_declaration_template_v0.md = vertical slice 1 件目、 「mode 宣言 = relay_only」 軸との連動 (= 自動受領 ≒ relay_only mode + 実質的な返答 ≒ executive_action mode)
- stop_finalization_template_v0.md = vertical slice 2 件目、 Layer 2 「未返事 peer 検出」 と本 template の軸 3 の連動
- completion_receipt_template_v0.md = vertical slice 3 件目、 「完了の物理証拠 5 ヶ所」 の「Agent Bus」 軸と本 template の軸 2 の連動
- 自社の再発記録 (= 「受領と内容取り込みの分離」 の繰り返し) = 本 rule の原因軸
