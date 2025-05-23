# 게이트웨이 서비스 구성

## 구성 예제

다음은 라우팅, CORS, 응답 처리 등을 포함한 완전한 구성 예제입니다:

```yaml
name: "mock-server"             # 프록시 서비스 이름, 전역적으로 고유함

# 라우터 구성
routers:
  - server: "mock-server"       # 서비스 이름
    prefix: "/mcp/user"         # 라우트 접두사, 전역적으로 고유함, 중복 불가, 서비스 또는 도메인+모듈로 구분하는 것이 좋음

    # CORS 구성
    cors:
      allowOrigins:             # 개발 테스트 환경에서는 모두 개방할 수 있으며, 프로덕션에서는 필요에 따라 개방하는 것이 좋습니다. (대부분의 MCP 클라이언트는 CORS가 필요하지 않음)
        - "*"
      allowMethods:             # 허용된 요청 메서드, 필요에 따라 개방. MCP(SSE 및 Streamable)의 경우 일반적으로 이 3가지 메서드만 필요함
        - "GET"
        - "POST"
        - "OPTIONS"
      allowHeaders:
        - "Content-Type"        # 반드시 허용해야 함
        - "Authorization"       # 인증 요구 사항에 대해 요청에 이 키를 포함하도록 지원해야 함
        - "Mcp-Session-Id"      # MCP의 경우, Streamable HTTP가 정상적으로 사용될 수 있도록 요청에 이 키를 포함하도록 지원해야 함
      exposeHeaders:
        - "Mcp-Session-Id"      # MCP의 경우, CORS가 활성화된 경우 이 키를 노출해야 함, 그렇지 않으면 Streamable HTTP를 정상적으로 사용할 수 없음
      allowCredentials: true    # Access-Control-Allow-Credentials: true 헤더를 추가할지 여부
```

### 1. 기본 구성

- `name`: 프록시 서비스 이름, 전역적으로 고유함, 다른 프록시 서비스를 식별하는 데 사용됨
- `routers`: 라우터 구성 목록, 요청 전달 규칙을 정의함
- `servers`: 서버 구성 목록, 서비스 메타데이터 및 허용된 도구를 정의함
- `tools`: 도구 구성 목록, 특정 API 호출 규칙을 정의함

구성을 네임스페이스로 취급할 수 있으며, 서비스 또는 도메인별로 구분하는 것이 좋습니다. 서비스에는 많은 API 인터페이스가 포함되어 있으며, 각 API 인터페이스는 도구에 해당합니다.

### 2. 라우터 구성

라우터 구성은 요청 전달 규칙을 정의하는 데 사용됩니다:

```yaml
routers:
  - server: "mock-server"       # 서비스 이름, servers의 이름과 일치해야 함
    prefix: "/mcp/user"         # 라우트 접두사, 전역적으로 고유함, 중복 불가
```

- `body`: 매개변수는 JSON 요청 본문에 배치됩니다
- `form-data`: 매개변수는 multipart/form-data 요청 본문에 배치되며, 파일 업로드 및 기타 시나리오에 사용됩니다

각 매개변수는 기본값을 가질 수 있습니다. MCP 요청에서 매개변수가 제공되지 않으면 기본값이 자동으로 사용됩니다. 기본값이 빈 문자열("")인 경우에도 사용됩니다. 예를 들어:

```yaml
args:
  - name: "theme"
    position: "body"
    required: true
    type: "string"
    description: "User interface theme"
    default: "light"    # 요청에서 theme 매개변수가 제공되지 않으면 "light"가 기본값으로 사용됩니다
```

`form-data`를 매개변수 위치로 사용할 때는 `requestBody`를 지정할 필요가 없으며, 시스템은 자동으로 매개변수를 multipart/form-data 형식으로 조립합니다. 예를 들어: 