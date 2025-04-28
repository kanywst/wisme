# WIMSE アプリケーションレベルのワークロード間認証

## 概要

多くのデプロイメントでは、ワークロード間の通信にエンドツーエンドのTLSを使用できません。これらのデプロイメントスタイルに対して、WIMSEではアプリケーションレベルの保護を提案しています。

現在のドキュメントでは、新しく導入されたWorkload Identity Token（WIT）を使用する2つの代替案が含まれています：

1. OAuth DPoP仕様に触発されたオプション
2. HTTP Message Signatures RFCに基づくオプション

## Workload Identity Token (WIT)

Workload Identity Token（WIT）は、ワークロードのアイデンティティを表すJWS（JSON Web Signature）で署名されたJWT（JSON Web Token）です。これはアイデンティティサーバーによって発行され、公開鍵をワークロードアイデンティティにバインドします。

### WITの必須クレーム

WITには以下のクレームが含まれている必要があります（特記がない限り）：

#### JOSEヘッダー内

- **alg**: JWS非対称デジタル署名アルゴリズムの識別子（IANA JOSE Algorithmsレジストリに登録されているアルゴリズム識別子）。値「none」は使用してはなりません。
- **typ**: WITは明示的に型付けされ、wimse-id+jwtメディアタイプを使用します。

#### JWTクレーム内

- **iss**: トークンの発行者（アイデンティティサーバー）をURIで表します。issクレームは推奨されますが、オプションです。
- **sub**: トークンのサブジェクト（ワークロードのアイデンティティ）をURIで表します。
- **exp**: トークンの有効期限（RFC7519のセクション4.1.4で定義）。WITは定期的に更新されるべきです（例：数時間ごと）。
- **jti**: トークンの一意の識別子。このクレームはオプションです。jtiクレームは個々のWITの発行を監査したり、それらを取り消したりするのに頻繁に役立ちますが、一部のトークン生成環境ではサポートされていません。
- **cnf**: jwkメンバーを使用して、ワークロードの公開鍵を含む確認クレーム（RFC7800のセクション3.2で定義）。ワークロードはWITを別のパーティに提示する際に、対応する秘密鍵の所有を証明する必要があります。

### WITのHTTPヘッダー

WITはWorkload-Identity-TokenというHTTPヘッダーフィールドで伝達されます。

```
Workload-Identity-Token: eyJhbGciOiJFUzI1NiIsImtpZCI6Ikp1bmUgNSIsInR5cCI6IndpbXNlLWlkK2p3dCJ9.eyJjbmYiOnsiandrIjp7ImNydiI6IkVkMjU1MTkiLCJrdHkiOiJPS1AiLCJ4Ijoiclp3VUEwVHJIazRBWEs5MkY2Vll2bUhIWDN4VU0tSUdsck11VkNRaG04VSJ9fSwiZXhwIjoxNzQwNzU4MzQ4LCJpYXQiOjE3NDA3NTQ3NDgsImp0aSI6IjRmYzc3ZmNlZjU3MWIzYmIzM2I2NzJlYWYyMDRmYWY0Iiwic3ViIjoid2ltc2U6Ly9leGFtcGxlLmNvbS9zcGVjaWZpYy13b3JrbG9hZCJ9.j-WlF3bufTwWeVZQntPhlzvSTPwf37-4wfazJZARdHYmW9S_olB5nKEqwqTZpIX_LoVVIcyK0VBE7Fa0CMvw2g
```

## オプション1: DPoPに触発された認証

このオプションは、OAuth DPoP仕様に触発されたもので、リクエストのコンテキストで呼び出し元のワークロードを認証するためにDPoPに似たメカニズムを使用します。

### Workload Proof Token (WPT)

WITに加えて、Workload Proof Token（WPT）と呼ばれる追加のJWTがWITの公開鍵に対応する秘密鍵で署名されます。WPTはリクエストのWorkload-Proof-Tokenヘッダーフィールドで送信されます。

#### WPTの必須クレーム

WPTには以下のクレームが含まれています：

##### JOSEヘッダー内

- **alg**: 関連するWITの確認キーに対応する適切なJWS非対称デジタル署名アルゴリズムの識別子。
- **typ**: WPTは明示的に型付けされ、application/wimse-proof+jwtメディアタイプを使用します。

##### JWTクレーム内

- **aud**: トークンの対象者には、WPTが添付されているリクエストのHTTPターゲットURI（クエリやフラグメント部分なし）が含まれます。
- **exp**: WITの有効期限。WPTの有効期間は短くなければなりません（例：分または秒単位）。
- **jti**: トークンの一意の識別子。
- **wth**: Workload Identity Tokenのハッシュ。値は、トークン値のASCIIエンコーディングのSHA-256ハッシュのbase64urlエンコーディングです。
- **ath**: リクエストに存在する場合、OAuthアクセストークンのハッシュ。
- **tth**: リクエストに存在する場合、Txn-Tokenのハッシュ。
- **oth**: リクエストに存在する場合、エンドユーザーのアイデンティティと認可コンテキストを伝える可能性のある他のトークンのハッシュ。

### WPTの検証

リクエスト内のWPTを検証するために、受信者は以下を確認する必要があります：

- リクエスト内に正確に1つのWorkload-Proof-Tokenヘッダーフィールドがあること
- Workload-Proof-Tokenヘッダーフィールド値が単一で整形式のJWTであること
- WPT署名がWITの確認クレームの公開鍵を使用して有効であること
- WPTのtypヘッダーパラメータがwimse-proof+jwtメディアタイプを伝えていること
- WPTのaudクレームがHTTPリクエストのターゲットURIと一致すること
- expクレームが存在し、まだ経過していない時間を伝えていること
- wthクレームが存在し、Workload-Identity-Tokenヘッダーで伝達されるトークン値のハッシュと一致すること
- オプションで、jtiクレームの値がWPTが有効と見なされる時間枠内で以前に使用されていないことを確認すること

## オプション2: HTTP Message Signaturesに基づく認証

このオプションは、リクエストと、オプションでレスポンスに署名するためにWorkload Identity Token（WIT）を使用します。これはHTTP Message Signatures仕様のプロファイルを定義します。

### 署名要件

#### リクエスト署名

リクエストはRFC9421に従って署名されます。以下の派生コンポーネントに署名する必要があります：

- @method
- @request-target

さらに、存在する場合は以下のリクエストヘッダーに署名する必要があります：

- Content-Type
- Content-Digest
- Authorization
- Txn-Token
- Workload-Identity-Token

#### レスポンス署名

レスポンスが署名される場合、以下のコンポーネントに署名する必要があります：

- @status
- @method;req
- @request-target;req
- 存在する場合はContent-Type
- 存在する場合はContent-Digest
- Workload-Identity-Token

#### 署名パラメータ

リクエストとレスポンスの両方について、以下の署名パラメータを含める必要があります：

- created
- expires - 有効期限は短くなければなりません（例：分単位）
- nonce
- tag - この仕様の実装の値はwimse-workload-to-workloadです

## 2つのオプションの比較

DPoPに触発されたオプションとMessage Signaturesオプションには、実装の複雑さ、拡張性、セキュリティに関して異なる長所と短所があります：

### DPoPに触発されたソリューションの利点

- HTTP以外のプロトコルにも適応しやすく、非同期通信シナリオに価値がある
- WITがJWTの一種であるため、同じくJWTを使用するDPoPに触発されたアプローチはMessage Signaturesよりも複雑さが低い

### Message Signaturesの利点

- 既存のHTTP固有のRFCと確立された実装の恩恵を受ける
- ミドルボックスによるメッセージ改変を軽減するなど、優れた整合性保護を提供
- レスポンス署名をサポート
- より大きな柔軟性を提供し、特定のフィールドのカバレッジなど、メッセージ署名の特定の側面を必須またはオプションにするかどうかを将来のバージョンで決定可能
