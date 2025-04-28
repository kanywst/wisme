# WIMSE (Workload Identity in Multi System Environments)

このリポジトリは、IETF WIMSE ワーキンググループによるドラフト「draft-ietf-wimse-s2s-protocol-03.txt」の内容を日本語で整理し、サンプル実装を提供するものです。

## 概要

WIMSE（Workload Identity in Multi System Environments）は、様々なランタイム環境におけるソフトウェアワークロードの認証と認可のためのアーキテクチャを定義しています。基本的な環境から複雑なマルチサービス、マルチクラウド、マルチテナントのデプロイメントまで対応しています。

このリポジトリでは、WIMSEアーキテクチャの最も基本的な単位である、安全に通信するために互いのアイデンティティを検証する必要がある2つのワークロード間のプロトコルについて説明します。

## ディレクトリ構造

```
/wimse/
  ├── docs/                      # 日本語ドキュメント
  │   ├── overview.md            # WIMSEの概要
  │   ├── architecture.md        # アーキテクチャと通信フロー
  │   ├── identity.md            # ワークロードアイデンティティの概念
  │   ├── app_level_auth.md      # アプリケーションレベル認証
  │   ├── mtls_auth.md           # mTLS認証
  │   └── security.md            # セキュリティに関する考慮事項
  ├── diagrams/                  # 図表
  │   ├── architecture.md        # アーキテクチャ図（mermaid）
  │   ├── message_flow.md        # メッセージフロー図（mermaid）
  │   └── token_structure.md     # トークン構造図（mermaid）
  └── sample/                    # サンプルGolangアプリケーション
      ├── common/                # 共通コード
      │   └── wit/              # WIT実装
      ├── workload-a/            # ワークロードA実装
      ├── workload-b/            # ワークロードB実装
      ├── identity-server/       # アイデンティティサーバー実装
      └── kubernetes/            # Kubernetesマニフェスト
```

## ドキュメント

### 基本概念

- [概要](docs/overview.md) - WIMSEの基本概念と目的
- [アーキテクチャ](docs/architecture.md) - デプロイメントアーキテクチャと通信フロー
- [ワークロードアイデンティティ](docs/identity.md) - 信頼ドメインとワークロード識別子

### 認証プロトコル

- [アプリケーションレベル認証](docs/app_level_auth.md) - DPoP方式とHTTP Message Signatures方式
- [mTLS認証](docs/mtls_auth.md) - 相互TLSを使用した認証

### その他

- [セキュリティに関する考慮事項](docs/security.md) - セキュリティとプライバシーの考慮事項

## 図表

- [アーキテクチャ図](diagrams/architecture.md) - WIMSEのデプロイメントアーキテクチャ
- [メッセージフロー図](diagrams/message_flow.md) - 認証プロトコルのメッセージフロー
- [トークン構造図](diagrams/token_structure.md) - WITとWPTの構造

## サンプルアプリケーション

[サンプルアプリケーション](sample/README.md)は、WIMSEプロトコルのGolang実装を提供します。このサンプルは、DPoP方式のアプリケーションレベル認証を実装しており、Kindを使用してKubernetes上で実行することができます。

### 主要コンポーネント

- **アイデンティティサーバー**: ワークロードにWorkload Identity Token（WIT）を発行
- **ワークロードA**: クライアントとして機能し、ワークロードBを呼び出す
- **ワークロードB**: サーバーとして機能し、ワークロードAからの認証済み呼び出しを受け付ける

### 実行方法

サンプルアプリケーションの実行方法については、[サンプルREADME](sample/README.md)を参照してください。

## 参考資料

- [draft-ietf-wimse-s2s-protocol-03.txt](https://datatracker.ietf.org/doc/draft-ietf-wimse-s2s-protocol/) - 原文ドラフト
- [IETF WIMSE ワーキンググループ](https://datatracker.ietf.org/wg/wimse/about/) - ワーキンググループ情報
