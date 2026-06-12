# 7-Signal Drift Check Template v0 (= AI Operator Guard vertical slice 後段 4 件 2 件目)

generated_at: 2026-06-09 12:30 JST
status: v0 draft (= 公開判断は owner GO 経由)
origin:
- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 2 件目
- nokaze 内 form (= startup sweep script の Ambiguity Gate 7 signals self-check) を AI agent 運用一般向けに汎用化
- 内部 origin = 内部の ambiguity / soft binder 設計 note の 5 categories + Soft Binder 2 軸の articulate

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) が owner 指示を受領した後、 着手する前に「指示の曖昧度 + 自分の理解度」 を 7 軸で物理的に確認する self-check。

「曖昧な指示を silently に解釈して動く」 → 「動いた結果が owner の期待と違う」 → 「やり直し」 という最大の運用失敗の class への対策。 7-signal drift check は「着手前の minimum 確認」 として hook script + chat output 前の自己確認の両方で fire。

## いつ使う

- owner 指示を受領した後の最初の応答 = 必須
- 自走で動く判断する前 = 必須
- 重い chain を fire する前 = 特に重要

skip 可能 = 短い受領のみ (= 「OK」 「了解」 のみ) + 直前の指示と同じ context の連続応答。

## form (= 5 + 2 = 7 signals self-check)

### Case A: 意味が未解決 = 5 categories の曖昧度確認

#### Signal 1: product audience (= 誰向けか)

- 商品 / 機能 / output の対象 audience が明確か
- 「自分達」 「内部」 「外部」 「特定 persona」 のどれか曖昧 = 1 件
- 自社使用実績 = 「商品を作る agent と商品を使う agent が分離してる」 という振り返りのような分離検出に有効

#### Signal 2: expected behavior (= 振る舞いの仕様)

- 期待される動きが明確か
- 「適切に処理」 「うまくやる」 等の vague articulate = 1 件
- 具体的な input / output / 副作用が明確 = clear

#### Signal 3: safety level (= 安全境界)

- 軸の boundary (= red / yellow / green) が明確か
- 「これは red gate に触れるか」 「自走範囲内か」 「公開接点か」 が曖昧 = 1 件
- standing authorization + red gate list を base に明確な軸 = clear

#### Signal 4: implementation target (= どこに implement するか)

- 修正 / 起稿先 file path / module / repo が明確か
- 「適当な所」 「どこか」 等の vague articulate = 1 件
- 具体的な file path / module 軸 = clear

#### Signal 5: missing "why" (= 動機 / 価値)

- なぜそれをするか (= 動機 / 価値) が articulate されてるか
- 「何のためか分からない」 「言われたからやる」 軸 = 1 件
- 価値 / 影響 / 目的 articulate あり = clear

### Case A 判定:

- 5 signals のうち 1 件以上「曖昧」 = silently に解釈せず、 必ず articulate して owner に確認:
  - mode: ambiguity_gate (= mode declaration template 軸)
  - interpreted: 自分の解釈 articulate
  - held: 保留した軸 articulate
  - 2-3 候補 + recommended (= 安全な時のみ、 候補列挙 default 振り戻し回避)

### Case B: 意味が解決 + Green path 複数 = Soft Binder 2 軸

5 categories 全部 clear + ただし default の選び方の自由度がある場合:

#### Signal 6: reversibility (= 戻しやすさ)

- 1 つ default 選んでも cheap に reverse 可能か
- file create + 後で削除 = high reversibility
- 公開 publish + 取り消し困難 = low reversibility
- API call + side effect あり = case by case

#### Signal 7: evidence confidence (= 証拠の確信度)

- default 選びの根拠の強さ
- 過去の同型 chain で fire した経験あり = high confidence
- 新規 chain + 推測軸 = low confidence

### Case B 判定:

- reversibility high + evidence confidence high = soft_binder mode で default fire (= owner 確認 skip 軸)
- reversibility low or evidence confidence low = ambiguity_gate mode で owner 確認

### Case C: Red 触れる可能性 = tripwire_hold

5-7 のいずれの軸でも、 red gate (= 価格 / 契約 / payment / 法務 / 顧客接触 / publish / アカウント変更 / 認証情報) に触れる可能性ある = tripwire_hold mode で動かない articulate のみ。

### Case D: 中継のみ = relay_only

別 agent / peer の articulate を owner に渡すだけ + 自分の判断含めない軸 = relay_only mode。

### Case E: 自走範囲内 = executive_action

5 signals clear + Case A/B/C/D いずれでもない + standing authorization 範囲内 = executive_action mode で自分の判断で動く。

## form 例 (= 架空の汎用例: Product X chain start の self-check)

```
着手前 7-signal drift check:
1. product audience = clear (= 「Product X の担当移管 + 仕様更新」 2 軸、 audience 明確)
2. expected behavior = clear (= 「Agent B に正式 ACK 板 land + template 4 件 chain fire」 form 明確)
3. safety level = clear (= red gate なし、 standing authorization 範囲内、 公開 / 価格 / 契約なし)
4. implementation target = clear (= `~/team-ops/board/` + `your-repo/products/product-x/templates/`)
5. missing "why" = clear (= 担当の構造化と移管 chain の完成軸)

Case B 確認:
6. reversibility = high (= 全部 internal file edit、 削除 / revert 可能)
7. evidence confidence = high (= 過去の同型 chain で fire 済 + owner 「進めて OK」 明示)

判定 = executive_action mode (= 自走範囲内 + standing authorization respect)
```

## boundary

- 本 template = internal design draft、 公開 / 価格 / 契約なし
- 7 signals の category articulate は固定、 各 signal の判定 criteria は project ごとに config 可能
- 「すべて曖昧」 軸の振り戻し既定 (= 過剰 ask) と 「すべて clear」 軸の振り戻し既定 (= 過剰 silent execution) の両方を避ける + 5 + 2 軸の物理 form で minimum 確認

## v0 → v1 への進化軸

- 最初の 1-2 週間 actual 採用後 = 7 signals の使い分けで「skip しがちな軸」 + 「過剰に articulate しがちな軸」 検証
- AI agent 別 (= Claude Code / Codex / Gemini CLI) の指示形式の適応性検証
- 5 categories の追加 / 削除 candidate 検証 (= 例: temporal urgency 軸の追加、 ただし最初は 5+2 で固定)
- Soft Binder の 2 軸を 3 軸に拡張 candidate 検証 (= cost / time / resource 軸、 ただし最初は reversibility + confidence で固定)

## Related

- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 2 件目
- nokaze 内実装 form = startup sweep script の 7 signals self-check section (= 自社運用で採用中)
- mode_declaration_template_v0.md = vertical slice 1 件目、 5 mode (= ambiguity_gate / soft_binder / tripwire_hold / relay_only / executive_action) と本 template の Case A/B/C/D/E の完全互換軸
- auto_ack_rule_template_v0.md = 後段 4 件 1 件目、 「relay_only mode = 自動受領」 軸との連動
- 内部の ambiguity / soft binder 設計 note (= origin 起点、 5 categories + Soft Binder articulate の base)
- 自社の再発記録 (= 「確認過剰 ask」 の繰り返し) = 本 template が対策 form として連動
