# WIMSE メッセージフロー図

## アプリケーションレベル認証のメッセージフロー

以下のmermaid図は、アプリケーションレベル認証のメッセージフローを示しています。

### DPoP方式のメッセージフロー

```mermaid
sequenceDiagram
    participant A as ワークロードA
    participant IS as アイデンティティサーバー
    participant B as ワークロードB
    
    A->>IS: 認証情報のリクエスト
    IS->>A: WIT発行 (アイデンティティサーバーの秘密鍵で署名)
    Note over A: WITには公開鍵が含まれる
    
    Note over A: HTTPリクエスト準備
    A->>A: WPT生成 (WITの公開鍵に対応する秘密鍵で署名)
    Note over A: WPTには以下が含まれる:<br/>- ターゲットURI<br/>- WITのハッシュ<br/>- 有効期限<br/>- 一意の識別子
    
    A->>B: HTTP リクエスト<br/>Workload-Identity-Token: [WIT]<br/>Workload-Proof-Token: [WPT]
    
    Note over B: 認証処理
    B->>B: WITの検証 (アイデンティティサーバーの公開鍵で)
    B->>B: WPTの検証:<br/>1. WITの公開鍵で署名を検証<br/>2. ターゲットURIの確認<br/>3. WITハッシュの確認<br/>4. 有効期限の確認<br/>5. 一意性の確認
    
    alt 認証成功
        B->>B: 認可処理
        B->>A: 200 OK レスポンス
    else 認証失敗
        B->>A: 401 Unauthorized レスポンス
    end
```

### HTTP Message Signatures方式のメッセージフロー

```mermaid
sequenceDiagram
    participant A as ワークロードA
    participant IS as アイデンティティサーバー
    participant B as ワークロードB
    
    A->>IS: 認証情報のリクエスト
    IS->>A: WIT発行 (アイデンティティサーバーの秘密鍵で署名)
    Note over A: WITには公開鍵が含まれる
    
    Note over A: HTTPリクエスト準備
    A->>A: リクエストに署名 (WITの公開鍵に対応する秘密鍵で)
    Note over A: 署名対象:<br/>- @method<br/>- @request-target<br/>- Content-Type (存在する場合)<br/>- Content-Digest (存在する場合)<br/>- Authorization (存在する場合)<br/>- Txn-Token (存在する場合)<br/>- Workload-Identity-Token
    
    A->>B: 署名付きHTTPリクエスト<br/>Workload-Identity-Token: [WIT]<br/>Signature-Input: [署名入力]<br/>Signature: [署名値]
    
    Note over B: 認証処理
    B->>B: WITの検証 (アイデンティティサーバーの公開鍵で)
    B->>B: 署名の検証:<br/>1. WITの公開鍵で署名を検証<br/>2. 署名されたコンポーネントの確認<br/>3. 有効期限の確認<br/>4. ノンスの一意性確認
    
    alt 認証成功
        B->>B: 認可処理
        B->>B: レスポンスに署名 (オプション)
        B->>A: 署名付き200 OKレスポンス
    else 認証失敗
        B->>A: 401 Unauthorized レスポンス
    end
```

## 相互TLS（mTLS）認証のメッセージフロー

以下のmermaid図は、相互TLS認証のメッセージフローを示しています。

```mermaid
sequenceDiagram
    participant A as ワークロードA
    participant IS as アイデンティティサーバー
    participant B as ワークロードB
    
    A->>IS: 証明書リクエスト
    IS->>A: X.509証明書発行<br/>(SubjectAltName URI型にWIMSE識別子を含む)
    
    B->>IS: 証明書リクエスト
    IS->>B: X.509証明書発行<br/>(SubjectAltName URI型にWIMSE識別子を含む)
    
    Note over A,B: TLSハンドシェイク
    A->>B: ClientHello
    B->>A: ServerHello, 証明書, CertificateRequest
    A->>B: 証明書, CertificateVerify
    Note over A,B: 相互認証完了
    
    A->>B: HTTP リクエスト
    
    Note over B: 認可処理
    B->>B: TLS層からクライアント証明書のWIMSE識別子を取得
    B->>B: WIMSE識別子に基づいて認可決定
    
    B->>A: HTTP レスポンス
```

## 認証オプションの比較

以下の表は、WIMSEで提供される3つの認証オプションの主な特徴を比較しています：

| 特徴 | DPoP方式 | HTTP Message Signatures方式 | 相互TLS (mTLS) |
|------|---------|------------------------|--------------|
| **認証レベル** | アプリケーション | アプリケーション | トランスポート |
| **トークン形式** | JWT | HTTP署名 | X.509証明書 |
| **ミドルボックス対応** | ✅ | ✅ | ❌ |
| **レスポンス署名** | ❌ | ✅ | ✅ (TLSレベル) |
| **実装の複雑さ** | 中 | 高 | 低 |
| **HTTP以外のプロトコル適応性** | 高 | 低 | 高 |
| **既存インフラ活用** | JWT | HTTP署名 | PKI |
| **メッセージ整合性保護** | 部分的 | 高 | 高 |
