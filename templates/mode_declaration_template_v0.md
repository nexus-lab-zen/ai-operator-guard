# Mode Declaration Template v0 (= AI Operator Guard vertical slice 1 件目、 既 form 汎用化)

generated_at: 2026-06-09 07:35 JST
status: v0 draft (= 公開判断は owner GO 経由)
origin:
- AI Operator Guard internal_spec_v0.md § 3.1 vertical slice 中心 4 件 1 件目
- nokaze 内 form (= chat 出力 rule の mode 宣言 1 行 form) を AI agent 運用一般向けに汎用化
- 内部 origin = 「商品を作る側と商品を使う側が分離してる」 という自社使用実績の振り返り

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) が応答を返す時、 「今どんな判断 mode で返してるか」 を 1 行で宣言する。 受け手 (= 人間 owner / 別の AI agent) が「これは判断? 確認? 保留? 中継?」 を区別できるようにする。

「mode 宣言なし」 の応答が混ざると、 受け手が「軽い確認のつもりで投げたら自走で動かれてた」 「重い判断のつもりで聞いたら中継だけ返ってきた」 という混同を起こす。 mode declaration はこの混同への対策。

## いつ使う

- owner からの指示を受領した後の最初の応答 = 必須
- 連続会話の途中 = mode が切り替わった時のみ
- 短い応答 (= 1-2 行の受領のみ) = 任意
- 自走 loop の tick = 任意 (= 大半が maintenance のため)

## form (= 1 行宣言)

応答の冒頭か末尾に次の 1 行:

```
mode: <ambiguity_gate | soft_binder | tripwire_hold | relay_only | executive_action> | interpreted: <X> | held: <Y> | boundary: <Z>
```

## 5 つの mode

### 1. ambiguity_gate

owner の指示が曖昧で複数解釈ある時、 自分の解釈を articulate して owner の確認を求める時。

候補列挙 (= 「A/B/C/D どれにする?」) の代わりに、 「私はこう解釈した、 違うなら教えて」 form を使う。 candidate list 投げは owner の time を消費するし、 自分が経営者として判断していない signal を送る。

例:
```
mode: ambiguity_gate | interpreted: 「進めて OK」 = 仕様 3 軸全部正式化 + peer agent に正式 ACK 板起稿 + 実装に入る | held: なし | boundary: 公開 / 価格は別 turn GO
```

### 2. soft_binder

候補が複数あって 1 つを default に選んだ時。 「私は X を default に選んだ、 違うなら言って」 form。

ambiguity_gate との違い = ambiguity_gate は owner の解釈確認を求めるが、 soft_binder は自分が default を選んだことを articulate して proceed する form。

例:
```
mode: soft_binder | interpreted: 残 template 3 件の順番、 mode → stop → completion を default に選んだ | held: 関連 docs の補章 起稿は別 sit | boundary: なし
```

### 3. tripwire_hold

red gate (= 価格 / 契約 / payment / 顧客実績 / 法務 / アカウント変更 / 認証情報 / 公開 / 炎上対応) に触れる可能性ある時、 動かずに articulate のみする mode。

例:
```
mode: tripwire_hold | interpreted: marketplace publish の依頼を received | held: owner 明示 GO 必要 | boundary: 価格 / 公開接点は触らない
```

### 4. relay_only

中継のみ (= 別 agent / peer の response を owner に渡す等)、 自分の判断を含めない時。

例:
```
mode: relay_only | interpreted: peer agent の response の中身を owner に届ける | held: 評価は別 turn | boundary: 中継のみ、 修正なし
```

### 5. executive_action

自走範囲内で自分の判断で動く時 (= owner の standing authorization 範囲内、 既決の自走 OK list 内)。

例:
```
mode: executive_action | interpreted: 無料の通常発信は自走 OK の範囲内、 公開記事 1 件 publish 進める | held: 価格関連は別 turn | boundary: 既存接点のみ
```

## 4 field の articulate

### interpreted (= 必須)

owner の指示をどう解釈したか、 1 行で。 候補列挙ではなく「私はこう解釈した」 と書く。

### held (= 必須、 なければ「なし」)

保留してる判断軸、 1 行で。 「全部解釈した」 と書くのは避ける、 必ず保留した軸を articulate する。 何も保留してないなら「なし」 と書く。

### boundary (= 必須)

動かない範囲、 1 行で。 red gate 軸 / 自走 default の境界 / 「ここは owner ask 必要」 軸を明示。

### reason (= 任意)

mode の選び根拠、 1 行で。 mode の選びが非自明な時のみ書く。

## いつ skip 可能

- 単純な 1-2 行の受領 (= 「OK」 「了解」 「ありがとう」 のみ)
- 自走 loop tick の応答 (= maintenance 軸の articulate)
- 直前の応答と mode が同じ + 状況変化なしの連続応答

skip すべきでない = owner の指示を受領した直後の最初の応答 + 自走で動く前の articulate + 自分の判断を articulate する応答。

## 試運転と効果測定

採用後 1 週間 = stop hook で「mode declaration なしの応答」 を警告として surface。 1 週間後に効果測定 (= 採用率 / owner の混同 fire 件数) + 採用継続 / form 修正の判断。

## boundary

- 本 template = internal design draft、 公開 / 価格 / 契約なし
- form 自体 (= 1 行宣言 + 5 mode + 4 field) は固定、 mode の中身 articulate は project ごとに調整可能
- mode 宣言は応答の form、 owner 側に強制ではない (= owner は普通の文章で返してくれる前提)

## v0 → v1 への進化軸

- 最初の 1 週間 actual 採用後 = mode の使い分けで「混ざる組」 や「使いにくい組」 検証
- AI agent 別 (= Claude Code / Codex / Gemini CLI) の form 適応性検証
- 5 mode を 4 mode / 6 mode に増減する candidate 検証 (= 最初は 5 mode で固定)
- 別の field 追加 candidate 検証 (= 例: confidence_level、 ただし最初は 4 field で固定)

## Related

- AI Operator Guard internal_spec_v0.md § 3.1 vertical slice 1 件目
- nokaze 内実装 form = chat 出力 rule の mode 宣言 1 行 form (= 自社運用で採用中)
- handoff_template_v0.md = vertical slice 4 件目、 5 段の articulate 内の「人間判断に戻す境界」 と本 template の boundary field の互換性軸
- stop_finalization_template_v0.md = vertical slice 2 件目、 mode declaration なしの応答を警告 surface する物理化
