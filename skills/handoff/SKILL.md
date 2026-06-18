---
name: handoff
description: Use before ending a session, when the owner steps away, or when work passes to another agent or instance — write what to read, the physical evidence of completion, where human judgement is required, and the single next action, so the next session can resume without re-asking.
---

# Handoff

## 目的

AI agent (= Claude Code / Codex / Gemini CLI 等) の現セッションが終わる時、 次のセッション or 別の agent or owner が「再確認 0 件」 で再開できる状態を作る。 「セッション再開時に状態が飛ぶ」 「完了したと言われたが、 実際には成果物がない」 「自動化が止まっても silent failure」 という運用の失敗の class への対策。

## いつ使う

以下 3 場面で turn end 前に必須:

1. **長い chain の turn end** = 3 件以上の commit + board + spec update を 1 turn で動かした時
2. **owner が席を立つと明示した時** = 「今日はここまで」 「明日朝で」 「しばらく帰れない」 を受領した直後
3. **別 agent / 別 instance への bridge** = chat lane が切り替わる時、 wake hook で別 session に渡る時

省略条件 = owner と direct chat 中 + 1 turn 1 commit 以下の軽い fire + 緊急対応中。

## form (= 4 段の記述)

### 1. これは何 (= 1-2 行)

- 商品 / 領域 / lane の記述 (= 1 段で「何の作業の引き継ぎか」)
- 例: 「Product X 担当移管 chain の引き継ぎ」 「Product Y 実装 chain の bridge」

### 2. 読むもの (= 引き継ぎ前の必須 read list、 3-5 件)

新セッション / 別 agent が「現状把握」 のために最初に読む file path:

- 正本となる決定 (= owner-decision / current_decisions / standing authorization)
- 直近の board file (= 最新 3-5 件、 自分宛 / 自分起稿 / 自分関連)
- status surface (= 各 agent の status file)
- 進行中の spec / draft file (= 1 件、 最重要のもの)

「これ全部読まないと次の判断ができない」 という基準で選ぶ、 「読むと役立つ」 まで含めて 7 件超過 = 多すぎ。

### 3. 完了の物理証拠 (= 「直った」 の定義に従う 5 ヶ所)

「完了」 と書く時、 次の 5 ヶ所に物理 evidence があることを確認:

1. **Source-of-Truth** = 正本 file (= spec / current_decisions / owner-decision) の更新
2. **Agent Bus** = board file の起稿 / response received
3. **Home** = 各 agent の status file の更新
4. **Dashboard** = 数字 / lane / ledger の更新
5. **board** = 引き継ぎ board file (= 本 skill 経由の起稿そのもの)

5 ヶ所のいずれかが空 = 「完了」 と書かない、 「進行中」 か「ACK 段階」 に止める。

加えて、 同型の再発を見直し (= 過去 7 日 board grep で同 lane の同型 fire なし) で確認:

- 「同型 fire 2 回以上見つけた」 = 内部レビュー board 起稿必須
- 「同型 fire 0 件」 = 完了の記述 OK

### 4. 人間判断に戻す境界 (= owner 確認が必要な軸)

次の動きの中で、 owner の明示判断が必要な軸を記述:

- owner 一声 4 件 (= 価格 / 個人情報を含む公開 / 初回 account 変更 / 炎上 risk) に該当する軸
- 北極星 (= 売上 / 介入の頻度) を再評価する軸の判断
- 担当配分の変更 / 商品境界の変更 / 体制の変更

「確認なしで自走 OK」 と「確認必須」 の境界を引き継ぎ時に明示し、 次セッションの確認過剰 (= 確認過剰の同型再発) を防ぐ。

### 5. 次の 1 件 (= 引き継ぎ後の最初の動き、 1 行)

「再開直後の最初の 1 件」 を 1 行で書く。 「N 件残ってる、 どれから?」 ではなく「最初は X」 を明示。

「X が終わったら Y」 という chain は 2-3 件まで、 5 件以上の長い chain は spec / status に書いて引き継ぎ form には含めない。

## form 例 (= 架空の汎用例: Product X 担当移管 chain の引き継ぎ)

```markdown
# 引き継ぎ: Product X 担当移管 chain (= 朝 land)

## 1. これは何
Product X の担当 (= DRI) を Agent B に正式移管、 Agent A は Product Y を別商品ラインで担当。 owner GO 取得済、 Agent B に正式 ACK 板 land 済。

## 2. 読むもの
- ~/team-ops/owner-decisions/standing_authorization_v0.md
- ~/team-ops/board/2026-01-15_agent_a_agent_b_product_x_handoff.md
- ~/team-ops/status/agent_a_status.md
- ~/team-ops/status/agent_b_status.md
- your-repo/products/product-x/internal_spec.md (= 進行中の最重要 spec)

## 3. 完了の物理証拠
- Source-of-Truth = internal_spec.md (= 担当移管後の form 更新済、 commit 済)
- Agent Bus = 移管の board file land 済 + Agent B の ACK received
- Home = agent_a_status 更新 別 sit (= owner chat 戻り直後、 軽い fire のみ)
- Dashboard = 数字変更なし (= 売上 0 / 内部体制変更のみ)
- board = 本 引き継ぎ board file の起稿そのもの
- 同型 fire 見直し = 過去 7 日 board grep で同 lane の同型 fire 0 件、 完了の記述 OK

## 4. 人間判断に戻す境界
- Product X の public publish / marketplace / 価格 = owner 別 turn GO 必須
- template prototype 実装 = 自走 OK (= standing authorization 範囲内、 owner GO 取得済)
- Agent B に追加の調整が必要な軸 = 商品境界の interface 設計 (= 後日別 sit)

## 5. 次の 1 件
templates/ 配下に既存 form の汎用化版 3 件 起稿 (= mode declaration / stop finalization / completion receipt)、 順番は handoff land 後 → mode declaration → stop finalization → completion receipt の chain。
```

## boundary

- form (= 5 段の記述) のみ固定、 場面で「何を read するか」 は project ごとに調整
- 「過剰 board 起稿」 への振り戻し防止 = 1 turn 1 件以下の軽い chain は引き継ぎ form を省略
