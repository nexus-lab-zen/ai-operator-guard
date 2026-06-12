# Completion Receipt Template v0 (= AI Operator Guard vertical slice 3 件目、 既 form 汎用化)

generated_at: 2026-06-09 08:33 JST
status: v0 draft (= 公開判断は owner GO 経由)
origin:
- AI Operator Guard internal_spec_v0.md § 3.1 vertical slice 中心 4 件 3 件目
- nokaze 内 form (= 「直った」 の新定義 = 5 ヶ所物理 evidence + 同型再発検出なし) を AI agent 運用一般向けに汎用化
- 内部 origin = 「完了 narrative を AI agent 単独で書かない」 + 「物理 evidence 5 ヶ所再生成」 = 短期間に 11 件の同型再発が出た自社実例への根本対策

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) が「完了しました」 と書く前に、 物理的な証拠が揃ってるか確認する仕組み。 「完了 narrative」 を AI agent 単独で persist しない form = 「実際には成果物がない」 という最大の運用失敗の class への対策。

「完了 = 成功裏に動いた」 と短絡されると、 owner が「翌日見たら何も起きてなかった」 という silent failure の典型に直結する。 本 template は「完了」 の意味を物理的に再定義する。

## いつ使う

- AI agent が「完了」 「done」 「fixed」 と articulate する前 = 必須
- 長い chain の終了時 (= 多 commit + 多 board update 後) = 特に重要
- owner に「次の動きは ○○」 と articulate する前 = 必須 (= 「完了」 の意味で持って次に進むため)

## form (= 5 ヶ所物理 evidence + 同型再発検出 + AI agent 単独完了 narrative なし)

### 5 ヶ所物理 evidence

「完了」 と書く時、 次の 5 ヶ所すべてに物理 evidence があることを確認:

#### 1. Source-of-Truth (= canonical な決定 file)

- 仕様 / 設計 / 決定の最終形が書かれた file (= spec / current_decisions / owner-decision 等)
- 完了の場合 = articulate update があり、 「完了状態」 が文章として残ってる
- 空 / 未更新 = 「完了」 と書かない、 「進行中」 articulate に止める

#### 2. Agent Bus (= 通信 / 連携 file)

- 別の AI agent / co-worker / owner との通信記録 (= 共有 board / chat outbox 等)
- 完了の場合 = 関連 board file が起稿 / response received
- 空 = 「内輪の完了」 = 別の lane から見えない、 「完了」 と書かない

#### 3. Home (= 自分の status surface)

- 自分の現状を articulate した file (= 自分の status / 持ち lane 一覧 / 今やってる軸の articulate)
- 完了の場合 = articulate update があり、 完了 entry が残ってる
- 空 / 未更新 = 「自分の中で完了」 = 次セッションで再開不可、 「完了」 と書かない

#### 4. Dashboard (= 数字 / 指標 / 数量化 file)

- 数字 / lane / ledger 等の数量化 file (= 売上 / 公開件数 / 完了件数 等)
- 完了の場合 = 関連数字が update されてる、 または「数字変更なし」 と articulate
- 「数字を持たない完了」 = boundary articulate して許容、 ただし articulate なしの sneak は NG

#### 5. board / handoff (= 引き継ぎ可能性確認)

- 次セッション / 別 agent / owner が引き継ぎできる form で記録された file (= handoff_template_v0 軸の form)
- 完了の場合 = 「再開のため何を読むか」 「完了の物理証拠」 「次の 1 件」 が articulate
- 空 = 「自分しか分からない完了」 = silent failure pre-stage、 「完了」 と書かない

### 同型再発検出

加えて、 完了 articulate の前に「同型の問題が再発してないか」 を物理的に検出:

- 過去 7 日の board / commit 履歴を grep
- 同 lane の同型 fire (= 同じ問題で同じ対策を 2 回以上) を検出
- 検出されたら = 内部 review board 起稿必須 (= 「process repair / offer reframe / route variation / smaller experiment / one-screen owner packet」 の 5 軸のうち 1 件以上 articulate)
- 検出 0 件 = 完了 articulate OK

### AI agent 単独完了 narrative なし

最後に、 「完了」 narrative を AI agent 単独で persist しない:

- 完了判定 = owner / peer の独立 review 経由
- AI agent 単独で「直った」 「完成」 「完了」 と persist することは NG
- 自社使用実績 = 1 晩の見直しで 11 件の同型再発が見つかり、 AI agent 単独の「直った」 narrative が同じ form の再発を生んでいた

例外 = owner が standing authorization で「自走範囲内の完了は AI agent 判定 OK」 と明示してる場合 (= 例: 軽い chain の commit + board update level)。 ただしこの場合も 5 ヶ所物理 evidence は必須。

## form 例 (= 架空の汎用例: Product X template chain の receipt)

```markdown
# 完了 receipt: Product X template chain

## 5 ヶ所物理 evidence

1. Source-of-Truth = internal_spec_v0.md § 3.1 form 更新済 (= commit 済)
2. Agent Bus = `~/team-ops/board/2026-01-15_agent_a_agent_b_product_x_handoff.md` (= 起稿済) + Agent B substantive response received
3. Home = agent_a_status update 別 sit (= owner chat 戻り中、 重い update は避ける)
4. Dashboard = 売上 0 / 内部体制変更のみ、 数字変更なし articulate
5. board / handoff = handoff_template_v0.md の 5 段 articulate に従う form 内包済 (= 「次の 1 件」 articulate あり)

## 同型再発検出

- 過去 7 日 board grep = 同型 fire 1 件検出 (= 確認過剰 ask の既知 form、 物理対策 land 済)
- 検出 1 件、 ただし物理対策 land 済 + 同 lane の continuation じゃない (= 別 chain で発火、 反復じゃない)
- 内部 review board 起稿不要、 ただし完了 articulate 時に該当 articulate 必須

## AI agent 単独完了 narrative なし

- 本 chain の「完了判定」 = owner の review 経由
- AI agent 単独で「template 4 件完成 + 商品化準備完了」 narrative は persist しない
- 「中心 4 件 land done = template directory に 4 件起稿済」 articulate のみ、 「商品化準備完了」 「公開準備完了」 は owner 判定後
```

## boundary

- 本 template = internal design draft、 公開 / 価格 / 契約なし
- 5 ヶ所の articulate (= file path / form) は project ごとに config 可能、 ただし 5 ヶ所すべての存在は必須
- 「完了 narrative」 を AI agent 単独で persist しない axis = 強制 (= 例外は owner 明示 standing authorization 時のみ)

## v0 → v1 への進化軸

- 最初の 5-10 件 actual receipt fire 後 = 5 ヶ所の articulate で「skip しがちな所」 検証 (= 自社使用実績では Dashboard skip が多い)
- 同型再発検出の精度検証 = 過去 N 日の範囲、 grep pattern の調整
- AI agent 別 (= Claude Code / Codex / Gemini CLI) の 5 ヶ所適用性検証
- owner 標準 standing authorization form との接続軸 articulate

## Related

- AI Operator Guard internal_spec_v0.md § 3.1 vertical slice 3 件目
- nokaze 内 form = 「直った」 の新定義 (= 完了判定の固定、 自社運用で採用中)
- handoff_template_v0.md = vertical slice 4 件目、 5 段の「完了の物理証拠 5 ヶ所」 と本 template の 5 ヶ所が完全互換
- mode_declaration_template_v0.md = vertical slice 1 件目、 mode 宣言で「自走範囲内の完了」 articulate との連動軸
- stop_finalization_template_v0.md = vertical slice 2 件目、 Layer 1-2 (= 未処理 packet / 未返事 peer) と本 template の Agent Bus 軸の連動
