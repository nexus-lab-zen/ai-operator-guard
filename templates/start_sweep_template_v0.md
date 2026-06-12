# Start Sweep Template v0 (= AI Operator Guard vertical slice 後段 4 件 最終件、 chain 完成)

generated_at: 2026-06-09 13:35 JST
status: v0 draft (= 公開判断は owner GO 経由)
origin:
- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 最終件
- nokaze 内 form (= startup sweep script、 11 section 構成) を AI agent 運用一般向けに汎用化
- 内部 origin = 「セッション開始時に board / status / 別 lane surface を読まずに動き出す」 → 「前 session の続きを把握せず別 work を fire」 同型再発の対策

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) のセッション開始時 (= 新規 session 立ち上がり + 自走 wake fire + chat 戻り直後) に、 状態把握のために必ず読むものを物理化する hook script form。

「セッション開始時に状態が飛ぶ」 「前 session の続きを把握せず新規 chain fire」 「別 lane の動きを skip して 1 人 lane で動く」 という最大の運用失敗の class への対策。 物理 hook で「必ず read する」 軸を強制する。

## いつ使う

- 新規 session 立ち上がり時 = 必須 (= SessionStart hook で fire)
- 自走 wake fire 直後 = 必須 (= wake 直後の物理 audit 手順と組合せ運用)
- 長期 quiet (= 3-6 時間以上) 後の再開時 = 必須
- owner chat 戻り直後 = 任意 (= chat lane priority、 sweep より対話)

## form (= 11 必読 section の汎用化)

### Section 1: timestamp + 起動 trigger articulate

```
Sweep — <timestamp> (= 起動 trigger articulate)
```

「いつ + 何経由」 を articulate、 後で「直近 sweep N 時間前」 軸の判定材料軸。

### Section 2: project / repo side state

project 配下の状態確認:

- git status (= uncommitted / untracked 軸)
- 直近 N 件 commit history
- residual TODO / WIP 軸

「自分が触ってる project の現状」 を seconds 級で grasp。

### Section 3: 今日の 1 件 decision template

セッション内で「今日動かす 1 件」 を articulate するための template:

- 7 日以内のまだ手をつけてない軸を 1 件
- 今日中に完了可能 + 過大じゃない軸
- owner / peer に articulate 可能な軸

「件数評価じゃなく value 評価で進む」 軸 (= 経営者視点の節目 check と整合)。

### Section 4: memory-lint

memory file の整合性確認:

- canonical index file (= MEMORY.md 等) の読み戻し
- 直近 N 日の新規 entry 確認
- duplicate / contradiction の軽 detection

「自分の memory 軸の状態把握」、 重い audit じゃなく軽 lint。

### Section 5: 外部 metrics fetch (= 任意、 必要な project のみ)

外部 service の metrics 取得 (= 売上 / 公開件数 / 顧客数 等):

- platform API call (= 公開 endpoint で取得可能な軸)
- 過大 fetch にしない (= 軽 fire 軸)

数字を持たない project は skip 可能。

### Section 6: board audit step (= board 配下の自分宛 message read)

共有 board / channel の自分宛 message 確認:

- 自分宛 + 自分起稿 + response_required: yes の file 一覧
- 直近 24-48h の新規 message
- 自動受領 file の除外 (= auto_ack_rule_template_v0 軸)

board の read miss 防止 = 同日に届いた board を読まずに別 work を fire した同型再発への対策 (= 自社の再発記録で物理化済)。

### Section 7: Controlled Wake / Task Queue

自走の wake queue 状態確認:

- 既に仕込まれてる wake / task の一覧
- 残作業の count
- 自走 chain の cadence 軸の articulate

### Section 8: External Outcome Accounting (= 「進んだ」 軸の数字軸)

外部の動き / 売上の数字軸 articulate:

- world_movement (= 公開接点での動き)
- revenue_signal (= 売上 signal)
- gap articulate (= 「世界の動きなし」 「売上なし」 軸の明示)
- 「進んだ claim」 の許可軸 articulate (= progress_claim_allowed=true/false)

「内部 commit を売上と数えない軸」 + 「世界の動き = 0 なら progress_claim_allowed = false」 軸の物理化。

### Section 9: External Surface Read (= 別 AI agent の hint 読み込み)

別 AI agent / 別 lane の出してる「次の動き」 surface 読み込み:

- 別 lane の status file 軸 (= 例: 別 agent の preflight / review surface file、 file path を config で指定)
- 別 lane の「top_action」 「next_behavior」 「next_min_experiment」 articulate
- 自分の judgement に取り込む軸の articulate

これは「自分以外の AI agent が hint を出してるが、 自分が拾ってない」 構造的 disconnect の対策。

### Section 10: Ambiguity Gate 7 signals self-check (= 7-signal drift check 軸)

着手前の 7 signals self-check (= 7_signal_drift_check_template_v0 軸):

- Case A 5 categories (= product audience / expected behavior / safety level / implementation target / missing why)
- Case B 2 軸 (= reversibility / evidence confidence)
- 判定 = ambiguity_gate / soft_binder / tripwire_hold / relay_only / executive_action

mode declaration template との連動で「sweep 完了 = mode 確定 + 着手 OK」 軸。

### Section 11: Sweep 完了 articulate

sweep の終了 articulate:

- 読んだ section count + 検出された警告 count
- 次の動き articulate (= 今日の 1 件 + 自走 wake 仕込み)
- 「Sweep 完了 = 着手 OK 軸」 articulate

## boundary

- 本 template = internal design draft、 公開 / 価格 / 契約なし
- 11 section の中身は project ごとに config 可能、 ただし「必読軸」 軸は固定
- 重い sweep にしない (= 30 秒 - 2 分の minimum、 owner chat 戻り直後の skip 軸 OK)
- sweep 結果は articulate のみ、 外部 action triggering なし

## v0 → v1 への進化軸

- 最初の 1-2 週間 actual 採用後 = 各 section の「skip しがちな軸」 + 「重い軸」 検証
- AI agent 別 (= Claude Code / Codex / Gemini CLI) の hook 接続軸 検証
- section 追加 / 削除 candidate (= 例: temporal urgency check、 ただし最初は 11 section で固定)
- 別 AI agent surface (= section 9) との接続 form の標準化軸

## Related

- AI Operator Guard internal_spec_v0.md § 3.2 後段 4 件 最終件
- nokaze 内実装 form = startup sweep script (= 11 section 構成、 自社運用で採用中)
- mode_declaration_template_v0.md = vertical slice 1 件目、 section 10 で 5 mode 連動
- stop_finalization_template_v0.md = vertical slice 2 件目、 turn end の物理 hook と本 template の session start の対対称軸
- auto_ack_rule_template_v0.md = 後段 1 件目、 section 6 board audit で auto_ack 除外連動
- seven_signal_drift_check_template_v0.md = 後段 2 件目、 section 10 で完全互換
- overclaim_reminder_template_v0.md = 後段 3 件目、 section 5 + section 8 で数字 articulate 軸との連動
- handoff_template_v0.md = vertical slice 4 件目、 「読むもの」 軸と本 template の section 2 + 6 軸の連動

## vertical slice chain 完成 articulate

本 template の land = AI Operator Guard vertical slice 中心 4 件 + 後段 4 件 = 8 件 chain 完成:

- 中心 4 件 = mode_declaration / stop_finalization / completion_receipt / handoff
- 後段 4 件 = auto_ack_rule / seven_signal_drift_check / overclaim_reminder / start_sweep (= 本 template)

次の phase = owner GO + 別 sit fire:

- README + 商品 description 起稿 (= internal_spec_v0 + 8 件 template の external-facing 軸)
- private repo の prototype 実装
- public publish / marketplace / 価格 = owner 別 turn GO

= 完了 narrative は AI agent 単独で persist しない (= completion_receipt_template_v0 軸 respect)、 「template directory 8 件 land done」 articulate のみ、 「商品化準備完了」 「公開準備完了」 は owner 判定経由。
