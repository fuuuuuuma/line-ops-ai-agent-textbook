# 🎁 LINE Harness運用Skills集（Claude Code用）

LINE HarnessのMCPサーバーが接続された状態のClaude Codeで、よくある運用タスクを自然言語1つで済ませるためのSkillです。

**受け取り方**: 下のコードブロック右上のコピーボタンでコピーするか、このファイルをそのままダウンロードしてください。

**使い方**: コピーした内容を `.claude/skills/line-harness-ops/SKILL.md` として保存してください。LINE HarnessのMCPサーバーを接続したClaude Codeに「友だち追加時のシナリオを作って」のように話しかけると、このSkillが該当するツールを実行します。

```text
---
name: line-harness-ops
description: LINE HarnessのMCPサーバーが接続されたClaude Codeで、シナリオ作成・セグメント配信・クリック解析・広告連携設定など、よくあるLINE運用タスクを自然言語の指示から実行する。「シナリオ作って」「配信して」「クリック数教えて」「広告連携を確認して」等の依頼で使う。
---

# LINE Harness運用Skills集

LINE Harness MCPサーバーの各ツールを、目的別の会話パターンに対応させたガイドです。ユーザーの依頼文から該当パターンを判断し、対応するMCPツールを呼び出してください。

**重要な前提**: 以下に登場するツール名・パラメータ名は、本資料の執筆環境で実際にLINE Harness MCPサーバーに接続して確認した2026年7月時点（v0.14.1相当）の仕様です。ただしLINE Harnessは開発速度が速く、バージョンアップで変わる可能性があります。**必ず実行前にMCPのツール一覧（tools/list等）で正式名称・パラメータを確認してから使ってください。** 一致しない場合はツール一覧を優先し、本Skillの記述を鵜呑みにしないでください。

## パターン1: 友だち追加時の自動シナリオを作る
依頼例: 「友だち追加した人に3日間のウェルカムシナリオを流して」
→ `create_scenario`（name, triggerType: friend_add, steps: [{delay, type, content}, ...]）で、ユーザーが指定した本数・間隔のstepsを組み立てる。delayは「0m」（即時）「30m」（30分後）「24h」（24時間後）のような形式で指定する。typeは"text"または"flex"。

## パターン2: タグに応じたシナリオを作る
依頼例: 「VIPタグを付けた人にフォローアップシナリオを流して」
→ 先に `manage_tags`（action: list）でタグの存在を確認し、無ければ `manage_tags`（action: create, tagName, tagColor） → `create_scenario`（name, triggerType: tag_added, triggerTagId: 該当タグID, steps: [...]）

## パターン3: セグメント配信をする
依頼例: 「先月申込んだ人にセミナーの案内を配信して」
→ `list_crm_objects`（objectType: tags）等で該当タグ/条件を確認したうえで、以下のいずれかで実行する。
  - 全員/タグ単位の配信は `broadcast`（targetType: all または tag, targetTagId, messageType, messageContent）で即時送信、または `manage_broadcasts`（action: create_draft → 内容確認 → action: send）で下書きしてから送る
  - 条件を組み合わせたセグメント配信は `broadcast`（targetType: segment, segmentConditions: {operator, rules:[...]}）、または `manage_broadcasts`（action: send_to_segment, segmentConditions）※segment配信には下書き保存が無く即時送信になる点に注意
  - **配信は取り消せないため、実際に送信する前に、対象人数・メッセージ内容を必ずユーザーに見せて確認を取る**

## パターン4: クリック解析・コンバージョンレポートをまとめる
依頼例: 「今週のトラッキングリンクのクリック数を教えて」
→ `list_crm_objects`（objectType: tracked_links）で対象リンクのIDを特定し、`get_link_clicks`（linkId）で集計。広告連携の成果を聞かれたら`manage_ad_platforms`（action: list）でplatformIdを確認→`get_conversion_logs`（platformId）で送信ログを確認

## パターン5: 広告プラットフォーム連携をセットアップする
依頼例: 「Meta広告のコンバージョン計測を設定して」
→ `manage_ad_platforms`（action: create, name: meta, config: {pixel_id, access_token}）。X/Google/TikTokの場合はconfigの中身が変わる（例: Googleは{customer_id, conversion_action_id, oauth_token}）ので、実行前にツールのパラメータ説明を確認する。アクセストークン等の秘匿情報は必ずユーザー本人から直接受け取り、会話ログや生成物に平文で残さない

## パターン6: リッチメニューを作る
依頼例: 「メニューに問い合わせボタンを追加したい」
→ `create_rich_menu`（name, areas: JSON文字列で[{bounds:{x,y,width,height}, action:{type, uri/text/data}}]）で組み立てる。画像はimageData（base64）で渡すか、既存メニューの一覧確認は`manage_rich_menus`（action: list）で行う。

## 共通の注意事項
- **配信・広告連携・友だち情報の変更など「取り消せない/外部に影響する」操作の前には、必ず内容をユーザーに見せて実行の承認を得る**
- 存在しないタグID・シナリオID・アカウントIDを推測で使わない。必ず`list`系ツール（`list_crm_objects`, `manage_tags`(list), `list_friends`等）で実在確認してから操作する
- 個別の友だちの実データ（氏名・会話内容等）を資料や生成物に転記しない
- **本Skillに書かれたツール名・パラメータ名は執筆時点の実接続確認に基づく例であり、絶対の仕様書ではない。実行前に必ずツール一覧で正式名称・必須パラメータを確認すること**
```

**ポイント**: MCPツールの正確な仕様（パラメータ名等）はLINE Harnessのバージョンによって変わることがあります。実行前にツール一覧を確認してください。
