---
name: seven-signal-drift-check
description: Use after an AI agent receives an instruction and before acting on it — run a seven-signal self-check on how ambiguous the instruction is and how confident the agent is, so a vague instruction is not silently interpreted and acted on in a way the owner did not intend.
---

# 7-Signal Drift Check

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) が owner の指示を受領した後、 着手する前に「指示の曖昧度 + 自分の理解度」 を 7 軸で物理的に確認する self-check。

「曖昧な指示を黙ったまま解釈して動く」 → 「動いた結果が owner の期待と違う」 → 「やり直し」 という最大の運用失敗の class への対策。 7-signal drift check は「着手前の最小確認」 として hook script + chat 出力前の自己確認の両方で発火。

## いつ使う

- owner の指示を受領した後の最初の応答 = 必須
- 自走で動く判断をする前 = 必須
- 重い chain を発火する前 = 特に重要

省略可能 = 短い受領のみ (= 「OK」 「了解」 のみ) + 直前の指示と同じ context の連続応答。

## form (= 5 + 2 = 7 signals self-check)

### Case A: 意味が未解決 = 5 categories の曖昧度確認

#### Signal 1: product audience (= 誰向けか)

- 商品 / 機能 / output の対象 audience が明確か
- 「自分達」 「内部」 「外部」 「特定 persona」 のどれか曖昧 = 1 件
- 実運用の振り返り = 「商品を作る agent と商品を使う agent が分離してる」 ような分離の検出に有効

#### Signal 2: expected behavior (= 振る舞いの仕様)

- 期待される動きが明確か
- 「適切に処理」 「うまくやる」 等の曖昧な言い方 = 1 件
- 具体的な input / output / 副作用が明確 = clear

#### Signal 3: safety level (= 安全境界)

- 軸の boundary (= red / yellow / green) が明確か
- 「これは red gate に触れるか」 「自走範囲内か」 「公開接点か」 が曖昧 = 1 件
- standing authorization + red gate list を base に明確な軸 = clear

#### Signal 4: implementation target (= どこに implement するか)

- 修正 / 起稿先の file path / module / repo が明確か
- 「適当な所」 「どこか」 等の曖昧な言い方 = 1 件
- 具体的な file path / module 軸 = clear

#### Signal 5: missing "why" (= 動機 / 価値)

- なぜそれをするか (= 動機 / 価値) が示されてるか
- 「何のためか分からない」 「言われたからやる」 軸 = 1 件
- 価値 / 影響 / 目的の記述あり = clear

### Case A 判定:

- 5 signals のうち 1 件以上「曖昧」 = 黙ったまま解釈せず、 必ず明示して owner に確認:
  - mode: ambiguity_gate (= mode declaration skill 軸)
  - interpreted: 自分の解釈
  - held: 保留した軸
  - 2-3 候補 + recommended (= 安全な時のみ、 候補列挙 default への振り戻し回避)

### Case B: 意味が解決 + Green path 複数 = Soft Binder 2 軸

5 categories 全部 clear + ただし default の選び方の自由度がある場合:

#### Signal 6: reversibility (= 戻しやすさ)

- 1 つ default を選んでも安く reverse 可能か
- file create + 後で削除 = high reversibility
- 公開 publish + 取り消し困難 = low reversibility
- API call + side effect あり = case by case

#### Signal 7: evidence confidence (= 証拠の確信度)

- default 選びの根拠の強さ
- 過去の同型 chain で発火した経験あり = high confidence
- 新規 chain + 推測軸 = low confidence

### Case B 判定:

- reversibility high + evidence confidence high = soft_binder mode で default 発火 (= owner 確認を省略)
- reversibility low or evidence confidence low = ambiguity_gate mode で owner 確認

### Case C: Red 触れる可能性 = tripwire_hold

5-7 のいずれの軸でも、 red gate (= 価格 / 契約 / payment / 法務 / 顧客接触 / publish / アカウント変更 / 認証情報) に触れる可能性ある = tripwire_hold mode で動かない、 記述のみ。

### Case D: 中継のみ = relay_only

別 agent / peer の記述を owner に渡すだけ + 自分の判断を含めない軸 = relay_only mode。

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
7. evidence confidence = high (= 過去の同型 chain で発火済 + owner 「進めて OK」 明示)

判定 = executive_action mode (= 自走範囲内 + standing authorization respect)
```

## boundary

- 7 signals の category は固定、 各 signal の判定 criteria は project ごとに config 可能
- 「すべて曖昧」 軸の振り戻し既定 (= 過剰な確認) と 「すべて clear」 軸の振り戻し既定 (= 過剰な silent execution) の両方を避ける + 5 + 2 軸の物理 form で最小確認
