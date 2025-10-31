# prompts/digest.prompt.md
# 目的
あなたはアシスタントとして、過去24時間に Gmail で「自分宛」に届いたメールを収集し、緊急度×重要度（Eisenhowerマトリクス）で分類したタスクダイジェストをSlackの本人DMへ送ります。


# 前提
- タイムゾーンは Asia/Tokyo（JST）。
- 「過去24時間」は実行時刻(JST)基準。
- Slackの投稿先は事前に用意した本人DMチャンネル（環境変数 SLACK_CHANNEL_IDS で1つ指定）。


# 手順
1. Gmail検索:
- Gmail MCP を使い、以下の検索条件で受信メールを取得。
- クエリ例: `to:me newer_than:1d -category:promotions -category:social`
- 取得する属性: 送信者名/メール、受信日時、件名、本文の先頭 400 文字、スレッドURL/Message-ID。
2. タスク抽出:
- 各メールから次のアクション（ToDo）を日本語で1行に要約。なければ「情報のみ」。
- 期限/締切・依頼主・合意事項が本文にあれば抽出。
3. 優先度判定:
- 緊急度(Urgency) と 重要度(Impact) をそれぞれ High/Medium/Low の3段階で推定。
- ルール例:
- 期限が本日〜明日: Urgency=High
- 取引・顧客影響/経営層/法務・障害: Impact=High
- ニュース/共有のみ: Impact=Low
- Eisenhower マトリクス区分を付与: [A]重要かつ緊急, [B]重要だが緊急でない, [C]緊急だが重要でない, [D]どちらでもない。
4. 整形:
- 見出しに今日の日付 (YYYY-MM-DD JST) と「Gmailダイジェスト」。
- 各区分(A→D)の順に、最大 10 件/区分まで。
- 各行フォーマット:
- `• [H/M/L|H/M/L] 期限:YYYY-MM-DD | 件名 — 送信者 (要約) <Gmailリンク>`
5. 送信:
- Slack MCP の `slack_post_message` を用い、本人DMチャンネルに投稿。
- 末尾に「/snooze 24h」などのSlackワークフローヒントは入れない。


5. 送信:
- **User Token 最小構成**のため、Slack MCP で次の順序で実行。
1) `auth.test` で `user_id` を取得。
2) `conversations.open` を `users:[user_id]` で呼び、DM チャンネル ID を得る。
3) `chat.postMessage`（MCP 側の `slack_post_message` 相当）で当該チャンネルに投稿。


# 出力ルール
- 投稿は日本語。簡潔で、箇条書き中心。
- 機密保護のため本文は要約のみ。固有の秘密情報は伏せ字にしてよい。
- 該当メールが0件なら「対象なし」とだけ投稿。


# 付記
- 可能なら重複スレッドは1件に集約し件名に (n通) を付与。
