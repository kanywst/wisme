# WIMSE トークン構造図

## Workload Identity Token (WIT) 構造

以下のmermaid図は、Workload Identity Token (WIT) の構造を示しています。

```mermaid
graph TD
    subgraph "Workload Identity Token (WIT)"
        subgraph "JOSEヘッダー"
            alg["alg: 署名アルゴリズム\n(例: ES256, EdDSA)"]
            typ["typ: wimse-id+jwt"]
            kid["kid: 鍵識別子\n(オプション)"]
        end
        
        subgraph "JWTクレーム"
            iss["iss: 発行者\n(アイデンティティサーバーのURI)\n(推奨だがオプション)"]
            sub["sub: サブジェクト\n(ワークロードのアイデンティティURI)"]
            exp["exp: 有効期限\n(数時間単位)"]
            jti["jti: 一意の識別子\n(オプション)"]
            cnf["cnf: 確認クレーム\n(ワークロードの公開鍵を含む)"]
        end
        
        signature["署名\n(アイデンティティサーバーの秘密鍵で署名)"]
    end
    
    classDef header fill:#ffecb3,stroke:#e6ac00,stroke-width:2px;
    classDef claims fill:#d1e7dd,stroke:#198754,stroke-width:2px;
    classDef sig fill:#f8d7da,stroke:#842029,stroke-width:2px;
    
    class alg,typ,kid header;
    class iss,sub,exp,jti,cnf claims;
    class signature sig;
```

## DPoP方式: Workload Proof Token (WPT) 構造

以下のmermaid図は、DPoP方式で使用されるWorkload Proof Token (WPT) の構造を示しています。

```mermaid
graph TD
    subgraph "Workload Proof Token (WPT)"
        subgraph "JOSEヘッダー"
            alg2["alg: 署名アルゴリズム\n(WITの公開鍵に対応)"]
            typ2["typ: wimse-proof+jwt"]
        end
        
        subgraph "JWTクレーム"
            aud["aud: 対象者\n(HTTPターゲットURI)"]
            exp2["exp: 有効期限\n(数分または秒単位)"]
            jti2["jti: 一意の識別子"]
            wth["wth: WITのハッシュ"]
            ath["ath: アクセストークンのハッシュ\n(存在する場合)"]
            tth["tth: Txn-Tokenのハッシュ\n(存在する場合)"]
            oth["oth: その他のトークンのハッシュ\n(存在する場合)"]
        end
        
        signature2["署名\n(ワークロードの秘密鍵で署名)"]
    end
    
    classDef header fill:#ffecb3,stroke:#e6ac00,stroke-width:2px;
    classDef claims fill:#d1e7dd,stroke:#198754,stroke-width:2px;
    classDef sig fill:#f8d7da,stroke:#842029,stroke-width:2px;
    
    class alg2,typ2 header;
    class aud,exp2,jti2,wth,ath,tth,oth claims;
    class signature2 sig;
```

## HTTP Message Signatures方式

以下のmermaid図は、HTTP Message Signatures方式で署名されるコンポーネントを示しています。

```mermaid
graph TD
    subgraph "HTTP Message Signatures"
        subgraph "リクエスト署名コンポーネント"
            method["@method"]
            target["@request-target"]
            ctype["Content-Type\n(存在する場合)"]
            digest["Content-Digest\n(存在する場合)"]
            auth["Authorization\n(存在する場合)"]
            txn["Txn-Token\n(存在する場合)"]
            wit["Workload-Identity-Token"]
        end
        
        subgraph "署名パラメータ"
            created["created: 作成時刻"]
            expires["expires: 有効期限\n(数分単位)"]
            nonce["nonce: 一意の値"]
            tag["tag: wimse-workload-to-workload"]
        end
        
        signature3["署名\n(ワークロードの秘密鍵で署名)"]
    end
    
    classDef components fill:#cfe2ff,stroke:#0d6efd,stroke-width:2px;
    classDef params fill:#d1e7dd,stroke:#198754,stroke-width:2px;
    classDef sig fill:#f8d7da,stroke:#842029,stroke-width:2px;
    
    class method,target,ctype,digest,auth,txn,wit components;
    class created,expires,nonce,tag params;
    class signature3 sig;
```

## 認証フロー比較

以下のmermaid図は、2つの認証オプションのフローを比較しています。

```mermaid
sequenceDiagram
    participant A as ワークロードA
    participant IS as アイデンティティサーバー
    participant B as ワークロードB
    
    rect rgb(240, 240, 255)
    note right of A: 共通ステップ
    A->>IS: 認証情報のリクエスト
    IS->>A: WIT発行
    end
    
    rect rgb(255, 240, 240)
    note right of A: DPoP方式
    A->>A: WPT生成 (WITの公開鍵に対応する秘密鍵で署名)
    A->>B: HTTP リクエスト + WIT + WPT
    B->>B: WITの検証
    B->>B: WPTの検証
    B->>A: レスポンス
    end
    
    rect rgb(240, 255, 240)
    note right of A: HTTP Message Signatures方式
    A->>A: リクエストに署名 (WITの公開鍵に対応する秘密鍵で)
    A->>B: 署名付きHTTPリクエスト + WIT
    B->>B: WITの検証
    B->>B: 署名の検証
    B->>B: レスポンスに署名 (オプション)
    B->>A: 署名付きレスポンス
    end
