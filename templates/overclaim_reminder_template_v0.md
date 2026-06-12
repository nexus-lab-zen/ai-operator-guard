# Overclaim Reminder Template v0 (= AI Operator Guard vertical slice 後段 4 件 3 件目)

generated_at: 2026-06-09 13:05 JST
status: v0 draft (= 公開判断は owner GO 経由)
origin:
- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 3 件目
- nokaze 内 form (= 「自分で使ってない商品を売ると書く違反を止める 形にする 道具」、 物理対策 5 件) を AI agent 運用一般向けに汎用化
- 内部 origin = 自分で使ってない商品に「販売開始」 と書く同型を 1.5 ヶ月で 2 回繰り返した自社実例

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) が商品 / 機能 / output に対して「販売開始」 「公開」 「価格」 「証明済」 「全 user に効く」 等の過大な claim articulate を持つ前に、 物理的に確認する rule / 確認 list。

「未完成段階での過大な claim」 → 「事実と articulate のズレ」 → 「owner / 顧客 / 関係者の混同」 という最大の運用失敗の class への対策。 この同型は「気をつける」 という普段のクセでは再発する evidence (= 自社で 1.5 ヶ月に 2 回再発した実例)、 物理対策が必須。

## いつ使う

- 商品 / 機能 / output の articulate を書く前 = 必須
- 「販売開始」 「公開」 「価格」 「証明済」 「ローンチ」 「production-ready」 「全 user に効く」 等の keyword を出す前 = 必須
- owner に「商品の状況」 articulate を持つ前 = 必須

## form (= 過大 claim keyword list + 確認 5 軸 + 物理対策 5 件)

### 過大 claim keyword list (= articulate 前検出 軸)

以下の keyword を出す前に確認軸 fire:

#### Tier 1: 状態 articulate (= 「未完成 vs 完成」 軸)

- 「販売開始」 「ローンチ」 「launching」 「launched」
- 「公開」 「publish」 「published」 「release」 「released」
- 「商品化完了」 「production-ready」 「商品準備完了」

#### Tier 2: 効果 articulate (= 「未検証 vs 検証済」 軸)

- 「効果あり」 「効く」 「証明済」 「validated」 「proven」
- 「顧客価値証明」 「customer value confirmed」
- 「全 user に効く」 「all users benefit」

#### Tier 3: 評価 articulate (= 過剰な評価軸)

- 「次世代」 「革新的」 「突破」 「急成長」
- 「全自動」 「完全防止」 「100%」
- 「業界初」 「業界トップ」

### 確認 5 軸 (= articulate 前の self-check)

各 Tier の keyword を出す前に、 以下 5 軸を確認:

#### 軸 1: 自分で使った証拠あり?

- AI agent / project / team で actual に使った record があるか
- 「設計」 「draft」 「starter」 段階は使った証拠じゃない
- 「自分で 1 回以上使い終わって + 失敗 / 成功の articulate あり」 がminimum

#### 軸 2: 検証期間あり?

- 「使い始めて 1 日以下」 = Tier 1 keyword 不可
- 「使い始めて 1 週間以下」 = Tier 2 keyword 不可
- 「使い始めて 1 ヶ月以下」 = Tier 3 keyword 不可

#### 軸 3: 顧客 / 外部 user の検証あり?

- AI agent / team の内部使用実績 = 内部 evidence
- 外部 user / 顧客の検証 = 外部 evidence
- Tier 1-2 keyword は外部 evidence が必須

#### 軸 4: 物理的な公開先あり?

- Tier 1 keyword (= 公開 / 販売開始) は actual な公開先 (= URL / marketplace / store) が必要
- 「予定」 「準備中」 段階で Tier 1 keyword 不可

#### 軸 5: owner / peer の独立確認あり?

- AI agent 単独で「直った」 「完成」 「効果あり」 narrative は persist しない
- owner / peer の独立 review / 確認経由で articulate

### 物理対策 5 件 (= hook script + workflow 物理化)

#### 対策 1: dogfood preflight script

- 商品の articulate を持つ前に必ず動かす script
- 証拠 file の存在確認 + 最終更新日 確認
- 軽い fire (= shell script で 30 秒以内)

```bash
# 概念的な実装
dogfood_preflight.sh <product>
# = check: <workspace>/dogfood/<product>/*.md が ≥ 1 件あるか + 直近 N 日以内に update あるか
```

#### 対策 2: pre commit public docs audit

- 公開 docs / 商品文章を commit する前に audit
- 過大 claim keyword の grep + 自社使用実績 link の存在確認
- pre-commit hook で物理化

#### 対策 3: 商品管理 file の「自分で使った状態」 欄

- 商品ごとの管理 file (= entry list) に「自分で使った状態」 欄を必須
- form = 「未着手 / 進行中 / 完了」 + 最後の証拠の日付
- 「未着手 / 進行中」 状態で Tier 1 keyword の articulate を禁止

#### 対策 4: dogfood/ directory form 固定

- 自社使用実績の record file を固定 directory に集約 (= `<workspace>/dogfood/<product>/<date>.md`)
- 1 回使うごとに 1 file
- form の articulate を README で固定

#### 対策 5: drift detection rule (= 本 template)

- 過大 claim keyword の articulate を検出するルール
- chat output 前 + commit 前 + board 起稿前で fire
- 検出時 = warn + articulate dial 推奨

## form 例 (= 架空の汎用例: Product X template chain land 時の self-check)

```
過大 claim self-check (= Product X template chain land articulate 時):

Tier 1 keyword check:
- 「商品化完了」 articulate なし = OK
- 「販売開始」 articulate なし = OK
- 「公開」 articulate なし (= 「reviewable state」 + 「公開は owner 別 turn GO 軸」 articulate) = OK

Tier 2 keyword check:
- 「証明済」 articulate なし (= 「dogfood evidence は appendix-level proof」 articulate) = OK
- 「全 user に効く」 articulate なし (= 「自社環境固有の使用実績」 articulate) = OK

Tier 3 keyword check:
- 「次世代」 「革新的」 articulate なし = OK
- 「全自動」 articulate なし (= 「警告のみ + block しない」 articulate) = OK

確認 5 軸:
1. 自分で使った証拠 = あり (= 約 2 ヶ月の内部実装 + commit chain)
2. 検証期間 = ≥ 2 ヶ月
3. 外部 user 検証 = なし (= 内部使用実績のみ articulate 済、 Tier 1 keyword 出してない)
4. 物理的な公開先 = なし (= private repo のみ、 owner GO 待ち)
5. owner / peer 独立確認 = 進行中 (= peer agent の codesign + owner GO、 template review はこれから)

判定 = articulate 「reviewable state + appendix-level proof + owner GO 待ち」 form = 過大 claim なし、 OK
```

## boundary

- 本 template = internal design draft、 公開 / 価格 / 契約なし
- keyword list は base、 project / 商品 ごとに追加 candidate 可能
- 「全部 NG」 軸の振り戻し既定 (= 過剰萎縮) を避ける + 「全部 OK」 軸の振り戻し既定 (= 過剰過大 claim) を避ける + 確認 5 軸 + 物理対策 5 件で minimum line

## v0 → v1 への進化軸

- 最初の 1-2 週間 actual 採用後 = false positive (= 正当な articulate を NG 判定) + false negative (= 過大 claim を OK 判定) 検証
- keyword list の追加 / 削除 candidate (= 例: 「AI-driven」 「next-gen」 等の業界 trend keyword)
- AI agent 別 (= Claude Code / Codex / Gemini CLI) の articulate 傾向検証
- Tier 1-2-3 の境界 dial (= 業界 / 顧客層に応じた厳しさ調整)

## Related

- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 3 件目
- nokaze 内実装 form = 内部 rule 文書の dogfood 違反対策の段 (= 物理対策 5 件、 自社運用で採用中)
- nokaze 内 memory = dogfood 違反の振り返り記録 (= 内部 memory file、 本 package には含めない)
- mode_declaration_template_v0.md = vertical slice 1 件目、 「mode: tripwire_hold」 = 過大 claim 触れる可能性ある時の articulate なし軸との連動
- completion_receipt_template_v0.md = vertical slice 3 件目、 「AI agent 単独完了 narrative なし」 軸と本 template の確認軸 5 の連動
- 自社の再発記録 (= 「未完成段階での販売開始 articulate」 の繰り返し) = 本 rule の原因軸
