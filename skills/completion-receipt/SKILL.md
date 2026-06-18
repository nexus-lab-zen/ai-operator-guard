---
name: completion-receipt
description: Use before an AI agent writes "done", "fixed", or "complete" — verify physical evidence exists in five places and that the same failure has not recurred, so a completion claim is not made when there is actually no artifact.
---

# Completion Receipt

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) が「完了しました」 と書く前に、 物理的な証拠が揃ってるか確認する仕組み。 「完了の話」 を AI agent 単独で残さない form = 「実際には成果物がない」 という最大の運用失敗の class への対策。

「完了 = 成功裏に動いた」 と短絡されると、 owner が「翌日見たら何も起きてなかった」 という silent failure の典型に直結する。 この skill は「完了」 の意味を物理的に再定義する。

## いつ使う

- AI agent が「完了」 「done」 「fixed」 と書く前 = 必須
- 長い chain の終了時 (= 多 commit + 多 board update 後) = 特に重要
- owner に「次の動きは ○○」 と書く前 = 必須 (= 「完了」 の意味を持って次に進むため)

## form (= 5 ヶ所物理 evidence + 同型再発検出 + AI agent 単独の完了話なし)

### 5 ヶ所物理 evidence

「完了」 と書く時、 次の 5 ヶ所すべてに物理 evidence があることを確認:

#### 1. Source-of-Truth (= 正本となる決定 file)

- 仕様 / 設計 / 決定の最終形が書かれた file (= spec / current_decisions / owner-decision 等)
- 完了の場合 = 更新があり、 「完了状態」 が文章として残ってる
- 空 / 未更新 = 「完了」 と書かない、 「進行中」 に止める

#### 2. Agent Bus (= 通信 / 連携 file)

- 別の AI agent / co-worker / owner との通信記録 (= 共有 board / chat outbox 等)
- 完了の場合 = 関連 board file が起稿 / response received
- 空 = 「内輪の完了」 = 別の lane から見えない、 「完了」 と書かない

#### 3. Home (= 自分の status surface)

- 自分の現状を書いた file (= 自分の status / 持ち lane 一覧 / 今やってる軸の記述)
- 完了の場合 = 更新があり、 完了 entry が残ってる
- 空 / 未更新 = 「自分の中で完了」 = 次セッションで再開不可、 「完了」 と書かない

#### 4. Dashboard (= 数字 / 指標 / 数量化 file)

- 数字 / lane / ledger 等の数量化 file (= 売上 / 公開件数 / 完了件数 等)
- 完了の場合 = 関連数字が update されてる、 または「数字変更なし」 と明示
- 「数字を持たない完了」 = boundary を明示して許容、 ただし黙ったまま素通りは NG

#### 5. board / handoff (= 引き継ぎ可能性確認)

- 次セッション / 別 agent / owner が引き継ぎできる form で記録された file (= handoff skill 軸の form)
- 完了の場合 = 「再開のため何を読むか」 「完了の物理証拠」 「次の 1 件」 が記述されてる
- 空 = 「自分しか分からない完了」 = silent failure の前段階、 「完了」 と書かない

### 同型再発検出

加えて、 完了の前に「同型の問題が再発してないか」 を物理的に検出:

- 過去 7 日の board / commit 履歴を grep
- 同 lane の同型 fire (= 同じ問題で同じ対策を 2 回以上) を検出
- 検出されたら = 内部 review board 起稿必須 (= 「process repair / offer reframe / route variation / smaller experiment / one-screen owner packet」 の 5 軸のうち 1 件以上 articulate)
- 検出 0 件 = 完了の記述 OK

### AI agent 単独の完了話なし

最後に、 「完了」 の話を AI agent 単独で残さない:

- 完了判定 = owner / peer の独立 review 経由
- AI agent 単独で「直った」 「完成」 「完了」 と残すことは NG
- 実運用の振り返り = 1 晩の見直しで 11 件の同型再発が見つかり、 AI agent 単独の「直った」 話が同じ form の再発を生んでいた

例外 = owner が standing authorization で「自走範囲内の完了は AI agent 判定 OK」 と明示してる場合 (= 例: 軽い chain の commit + board update level)。 ただしこの場合も 5 ヶ所物理 evidence は必須。

## form 例 (= 架空の汎用例: Product X template chain の receipt)

```markdown
# 完了 receipt: Product X template chain

## 5 ヶ所物理 evidence

1. Source-of-Truth = internal_spec.md form 更新済 (= commit 済)
2. Agent Bus = `~/team-ops/board/2026-01-15_agent_a_agent_b_product_x_handoff.md` (= 起稿済) + Agent B substantive response received
3. Home = agent_a_status update 別 sit (= owner chat 戻り中、 重い update は避ける)
4. Dashboard = 売上 0 / 内部体制変更のみ、 数字変更なし
5. board / handoff = handoff skill の 5 段に従う form 内包済 (= 「次の 1 件」 あり)

## 同型再発検出

- 過去 7 日 board grep = 同型 fire 1 件検出 (= 確認過剰 ask の既知 form、 物理対策 land 済)
- 検出 1 件、 ただし物理対策 land 済 + 同 lane の continuation じゃない (= 別 chain で発火、 反復じゃない)
- 内部 review board 起稿不要、 ただし完了の記述時に該当事項を併記

## AI agent 単独の完了話なし

- 本 chain の「完了判定」 = owner の review 経由
- AI agent 単独で「template 4 件完成 + 商品化準備完了」 とは残さない
- 「中心 4 件 land done = template directory に 4 件起稿済」 のみ、 「商品化準備完了」 「公開準備完了」 は owner 判定後
```

## boundary

- 5 ヶ所の記述 (= file path / form) は project ごとに config 可能、 ただし 5 ヶ所すべての存在は必須
- 「完了の話」 を AI agent 単独で残さない軸 = 強制 (= 例外は owner 明示 standing authorization 時のみ)
