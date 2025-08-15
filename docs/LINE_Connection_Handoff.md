# LINE Connection Handoff
バージョン: 1.0 / 作成日: 2025-08-15 / 作成者: Claude
対象環境: 本番 / 検証

---

## 0. この文書の目的
- 現行の「LINE公式アカウント接続」を**固定値/変数名を変えず**に再現するための、唯一の参照文書。
- 新規プロジェクトに持ち込むべき**設定・手順・注意点**を一箇所に集約。
- すべての重要設定に**根拠ファイル(code_refs)**を付与。

---

## 1. 接続サマリ（一目で把握）
- チャネル: N/A（LINE_BASIC_ID未設定）
- Webhook: ローカルURL `http://localhost:5001/webhook`
- 署名検証: skippable（flag=SKIP_WEBHOOK_SIGNATURE_VERIFICATION）
- 送受信API: reply / push / multicast / broadcast
- リッチメニュー: default_menu=menu_a / ID一覧: richmenu-fa2687411d4f851c0a6dff090b145c78, richmenu-75b22a410b122ca8960cf272b36127c3 / 画像サイズ差分あり
- データ保存: JSON（files: admin/line_users.json, admin/line_messages.json, admin/accounts.json, admin/line_broadcast_history.json）
- ランタイム: Python/FastAPI/uvicorn/Flask

---

## 2. 環境変数と値（マスク済み）
> 値は末尾4文字のみ表示。`.env.template`と1:1対応させること。

| 変数名 | 値(末尾4) | 用途 | 定義場所 | code_refs |
|---|---|---|---|---|
| LINE_CHANNEL_ACCESS_TOKEN | ilFU | Message API 認証 | .env / admin/.env | .env:2, admin/.env:2 |
| LINE_CHANNEL_SECRET | cd1d | 署名検証 | .env / admin/.env | .env:3, admin/.env:3 |
| SKIP_WEBHOOK_SIGNATURE_VERIFICATION | true | 検証スキップ | admin/.env | admin/.env:6 |
| WEBHOOK_PATH | /webhook | 受信パス | ハードコード | webhook_server.py:111 |
| WEBHOOK_EXTERNAL_URL | N/A | 本番URL | 未設定 | N/A |
| APP_PORT | 5001 | ローカル起動 | ハードコード | webhook_server.py:287 |

**運用ルール**  
- 本番では `SKIP_WEBHOOK_SIGNATURE_VERIFICATION=false` を必須。  
- `.env` と `admin/.env` に差異がある場合は、**二重管理の整合**をこの文書に従う。

---

## 3. エンドポイント & ルーティング
### 3.1 Webhook受信
- URL: http://localhost:5001/webhook
- method: POST  
- 署名検証: 有（スキップ可能・フラグ名=SKIP_WEBHOOK_SIGNATURE_VERIFICATION）（code_refs: webhook_server.py:97-109）
- 成功フロー（ASCII図）:
```
Client(LINE) -> /webhook -> Verify X-Line-Signature -> Handler -> reply/push
```

### 3.2 管理/健全性
- `/status` (GET): BOT情報・デフォルトメニュー取得 / code_refs: webhook_server.py:247-251
- `/health` (GET): ヘルスチェック・LINE API疎通確認 / code_refs: webhook_server.py:253-262

---

## 4. メッセージング仕様
- 使用API: reply / push / multicast / broadcast
- 実装位置: webhook_server.py:177-245, admin/app/line_client.py:170-269, admin/app/routers/line_broadcast.py:29-143
- レート制限: 未実装
- 再送戦略: 未実装
- サンプル（擬似コードまたは関数名）:
  - reply: `send_reply_message(reply_token, messages)`（code_refs: webhook_server.py:225-241）
  - push: `send_push_message(user_id, messages)`（code_refs: admin/app/line_client.py:194-211）
  - multicast: `send_multicast_message(user_ids, messages)`（code_refs: admin/app/line_client.py:213-230）
  - broadcast: `send_broadcast_message(messages)`（code_refs: admin/app/line_client.py:232-249）

---

## 5. リッチメニュー
- デフォルト: menu_a (ID: richmenu-fa2687411d4f851c0a6dff090b145c78)
- 一覧:
  - menu_a: ID=richmenu-fa2687411d4f851c0a6dff090b145c78, サイズ=2500x1686, 画像=menu_a_true.png
  - menu_b: ID=richmenu-75b22a410b122ca8960cf272b36127c3, サイズ=2500x1686, 画像=menu_b_true.png
- 切り替えトリガー（code_refs: menu_switch_config.json:1-16）:
  - メニューA: ["A", "a", "メニューA", "メニューa"]
  - メニューB: ["B", "b", "メニューB", "メニューb"]
  - トグル: ["チェンジ", "切り替え", "change"]
- 画像サイズ差分: 2500x843と2500x1686の両方存在（現在は2500x1686使用）

---

## 6. データ保存（JSON or DB）
- タイプ: json_files
- ファイル群（JSONの場合）:
  - users: admin/line_users.json（主キー: id）
  - messages: admin/line_messages.json（主キー: id）
  - accounts: admin/accounts.json
  - broadcast_history: admin/line_broadcast_history.json
- スキーマのcode_refs: admin/app/routers/line_friends.py:37-50, admin/app/routers/chat.py:20-40

---

## 7. 使用ライブラリ & 依存
- 言語: Python 3.9以上（推定）
- Webフレーム: FastAPI / Flask (code_refs: admin/main.py:7, webhook_server.py:13)
- HTTPライブラリ: requests (code_refs: webhook_server.py:14, admin/app/line_client.py:5)
- 環境変数ローダー: python-dotenv (code_refs: webhook_server.py:15, admin/main.py:9)
- LINE SDK使用: なし

---

## 8. セットアップ手順（移行時）
1. **LINE Developers設定**
   - チャネル作成 -> アクセストークン/シークレット取得
   - Webhook URL設定（本番では外部URLが必要）
   - 応答メッセージOFF / あいさつメッセージOFF

2. **環境変数設定**
   - `.env.template` -> `.env` にコピー
   - 必須値を埋める（特にLINE_CHANNEL_ACCESS_TOKEN, LINE_CHANNEL_SECRET）
   - 本番では必ず `SKIP_WEBHOOK_SIGNATURE_VERIFICATION=false`

3. **リッチメニュー移行**
   - ID再作成が必要（新規アカウントの場合）
   - 画像ファイル: menu_a.png, menu_b.png, menu_a_2500x1686.png, menu_b_2500x1686.png
   - 設定ファイル更新: rich_menu_ids.json

4. **JSONデータ移行**（該当する場合）
   - バックアップ: `tar -czf line_data_backup.tar.gz admin/*.json`
   - リストア: `tar -xzf line_data_backup.tar.gz`

5. **起動確認**
   - `python webhook_server.py`（Webhookサーバー）
   - `cd admin && python -m uvicorn main:app --reload --port 8000`（管理API）
   - `/health` で疎通確認
   - LINEアプリからテストメッセージ送信

---

## 9. トラブルシューティング
- **署名検証エラー**: SKIP_WEBHOOK_SIGNATURE_VERIFICATION=true でバイパス（開発のみ）
- **404エラー（画像アップ）**: api-data.line.me を使う（api.line.meは404）（code_refs: admin/app/line_client.py:89）
- **ポート競合**: lsof -i :5001 で確認、必要に応じてポート変更
- **メニューが表示されない**: 5-10分待つ / アプリ再起動 / relink_default_menu.py実行

---

## 10. 差分メモ（発見した不整合）
- **.envとadmin/.envの二重管理**: 同じLINE_CHANNEL_ACCESS_TOKENとLINE_CHANNEL_SECRETが両ファイルに存在。admin/.envにのみSKIP_WEBHOOK_SIGNATURE_VERIFICATIONが追加
- **リッチメニュー画像サイズの不一致**: rich_menu_ids.jsonでは2500x1686と記載されているが、2500x843の画像も存在
- **WEBHOOK_EXTERNAL_URLの未設定問題**: 環境変数として定義されておらず、本番デプロイ時に手動設定が必要
- **start_all_complete.pyの不整合**: account_api.pyとchat_api.pyを起動しようとするが、実際はmain_8002.pyとmain.pyが該当ファイル

---

## 11. 機能の不足リスト（本番考慮）
- [ ] LIFF / LINE Login
- [ ] Flex Message
- [ ] レート制限対応
- [ ] 再送/リトライ機構
- [ ] ブロック検知
- [ ] メディア対応（画像/動画）
- [ ] グループ/ルーム対応
- [ ] 自動バックアップ
- [ ] モニタリング/アラート
- [ ] データベース対応（現在JSONファイルのみ）
- [ ] エラーハンドリングの標準化
- [ ] ログローテーション

---

## 12. 参照リンク
- LINE Developers: https://developers.line.biz/
- Messaging API Docs: https://developers.line.biz/ja/docs/messaging-api/
- 内部ドキュメント: README.md / CLAUDE.md / リッチメニュー設定手順.md

---
**最終更新日**: 2025-08-15  
**次回レビュー予定**: 2025-11-15