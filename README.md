# LINE Migration Documentation

LINE公式アカウント連携システムの移行用ドキュメントとテンプレート

## 🚀 クイックスタート

このリポジトリをcloneして、以下のコマンドで即座にLINE連携システムを構築できます：

```bash
# 1. リポジトリをclone
git clone https://github.com/Regit-Hino/line-migration-docs.git
cd line-migration-docs

# 2. 開発環境用の設定ファイルをコピー
cp line_migration_pack/.env.development .env

# 3. ドキュメントに従って実装
# docs/LINE_Connection_Handoff.md を参照
```

## 📁 ファイル構成

```
.
├── docs/
│   └── LINE_Connection_Handoff.md    # 完全な移行手順書
├── line_migration_pack/
│   ├── export.json                   # システム設定のエクスポート
│   ├── .env.template                 # 環境変数テンプレート
│   └── .env.development              # 開発環境用設定（認証情報含む）
└── README.md                         # このファイル
```

## 🔑 含まれる認証情報

`.env.development`には以下の認証情報が含まれています：
- LINE Channel Access Token（実際の値）
- LINE Channel Secret（実際の値）
- リッチメニューID（2つ）

**注意**: このリポジトリはprivateですが、セキュリティのため本番環境では新しい認証情報の使用を推奨します。

## 📋 実装チェックリスト

- [ ] Webhookサーバー（ポート5001）
- [ ] リッチメニュー管理
- [ ] メッセージ送受信（reply/push/multicast/broadcast）
- [ ] ユーザー管理（JSON）
- [ ] 署名検証

## 🚨 重要事項

1. **開発環境**: `.env.development`をそのまま使用可能
2. **本番環境**: `SKIP_WEBHOOK_SIGNATURE_VERIFICATION=false`に変更必須
3. **Webhook URL**: ngrokなどで外部公開が必要

## 📖 詳細ドキュメント

完全な実装手順は `docs/LINE_Connection_Handoff.md` を参照してください。