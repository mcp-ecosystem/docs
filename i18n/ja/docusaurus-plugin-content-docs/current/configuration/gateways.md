# ゲートウェイサービス設定

## 設定例

以下は、ルーティング、CORS、レスポンス処理などを含む完全な設定例です：

```yaml
name: "mock-server"             # プロキシサービス名、グローバルに一意

# ルーター設定
routers:
  - server: "mock-server"       # サービス名
    prefix: "/mcp/user"         # ルートプレフィックス、グローバルに一意、重複不可、サービスまたはドメイン+モジュールで区別することを推奨

    # CORS設定
    cors:
      allowOrigins:             # 開発テスト環境ではすべて開放可能、本番環境では必要に応じて開放することをお勧めします。（ほとんどのMCPクライアントはCORSを必要としません）
        - "*"
      allowMethods:             # 許可するリクエストメソッド、必要に応じて開放。MCP（SSEとStreamable）の場合、通常これら3つのメソッドのみが必要
        - "GET"
        - "POST"
        - "OPTIONS"
      allowHeaders:
        - "Content-Type"        # 必須の許可
        - "Authorization"       # 認証ニーズのためにリクエストでこのキーを携帯する必要がある
        - "Mcp-Session-Id"      # MCPの場合、Streamable HTTPが正常に機能するためにはリクエストでこのキーを携帯することが必要
      exposeHeaders:
        - "Mcp-Session-Id"      # MCPの場合、CORSが有効になっているときにこのキーを公開する必要がある、そうでなければStreamable HTTPが正常に機能しない
      allowCredentials: true    # Access-Control-Allow-Credentials: trueというヘッダーを追加するかどうか
```

### 1. 基本設定

- `name`: プロキシサービス名、グローバルに一意、異なるプロキシサービスを識別するために使用
- `routers`: ルーター設定リスト、リクエスト転送ルールを定義
- `servers`: サーバー設定リスト、サービスのメタデータと許可されたツールを定義
- `tools`: ツール設定リスト、特定のAPI呼び出しルールを定義

設定を名前空間として扱うことができ、サービスまたはドメインによって区別することを推奨します。サービスには多くのAPIインターフェイスが含まれ、各APIインターフェイスはツールに対応しています。

### 2. ルーター設定

ルーター設定はリクエスト転送ルールを定義するために使用されます：

```yaml
routers:
  - server: "mock-server"       # サービス名、serversの名前と一致する必要がある
    prefix: "/mcp/user"         # ルートプレフィックス、グローバルに一意、重複不可
```

- `body`: パラメータはJSONリクエストボディに配置されます
- `form-data`: パラメータはmultipart/form-dataリクエストボディに配置され、ファイルアップロードなどのシナリオで使用されます

各パラメータにはデフォルト値を設定できます。MCPリクエストでパラメータが提供されない場合、デフォルト値が自動的に使用されます。デフォルト値が空文字列（""）の場合でも使用されます。例えば：

```yaml
args:
  - name: "theme"
    position: "body"
    required: true
    type: "string"
    description: "User interface theme"
    default: "light"    # リクエストでthemeパラメータが提供されない場合、"light"がデフォルト値として使用されます
```

`form-data`をパラメータ位置として使用する場合、`requestBody`を指定する必要はありません。システムは自動的にパラメータをmultipart/form-data形式に組み立てます。例えば： 