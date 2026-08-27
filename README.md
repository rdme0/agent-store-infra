# AgentStore Infra

AgentStore 개발 환경의 실행 진입점입니다. `agent-store-be`, `demo-agent`, `agent-store-fe` 저장소가 이 저장소와 같은 상위 폴더에 있어야 합니다.
API·PostgreSQL secret은 `agent-store-be/.env`가 소유합니다. demo-agent의 공개 설정은 체크인된
`demo-agent/config/application.yaml`이고, `demo-agent/.env`에는 OpenAI opt-in용 `OPEN_AI_KEY`만 둡니다.

## 구성

| 구성 | 실행 방식 | 주소 |
|---|---|---|
| Frontend | 로컬 Vite | http://localhost:5173 |
| Spring API | Docker Compose | http://localhost:8080 |
| Go demo-agent | Docker Compose | http://localhost:8090 |
| PostgreSQL | Docker Compose | localhost:5432/agent_store |

Compose 안에서는 일반 service network DNS를 사용합니다. API는 `api:8080`, demo-agent는
`demo-agent:8090`에서 서로 직접 통신하며, 호스트에는 demo-agent만 `127.0.0.1:8090`으로 노출됩니다.
API와 demo-agent가 healthy가 된 뒤 `catalog-bootstrap` one-shot service가 catalog를 등록·검증하고 Version을 publish합니다.
같은 catalog를 다시 실행하는 것은 성공하지만 ACTIVE 데이터와 다른 내용이면 drift 오류로 중단합니다.

## 실행

```powershell
Copy-Item ../agent-store-be/.env.example ../agent-store-be/.env
Copy-Item ../demo-agent/.env.example ../demo-agent/.env
$env:DEMO_AGENT_MODE = 'fixture' # fixture 또는 openai
docker compose --env-file ../agent-store-be/.env up --build -d --remove-orphans
```

상태와 로그는 다음처럼 확인합니다.

```powershell
docker compose --env-file ../agent-store-be/.env ps
docker compose --env-file ../agent-store-be/.env logs -f api demo-agent catalog-bootstrap
```

```powershell
# Compose 설정만 바꾼 경우
docker compose --env-file ../agent-store-be/.env up -d

# Spring 코드나 application.yaml을 바꾼 경우
docker compose --env-file ../agent-store-be/.env build api
docker compose --env-file ../agent-store-be/.env up -d api demo-agent

# Go demo-agent 또는 catalog를 바꾼 경우
docker compose --env-file ../agent-store-be/.env build demo-agent
docker compose --env-file ../agent-store-be/.env up -d api demo-agent catalog-bootstrap
```

로컬 Java 없이 Spring 검증을 실행합니다. 첫 실행은 Gradle distribution과 의존성을 받고, 이후 실행은
`agent-store-gradle-cache` named volume을 재사용합니다.

```powershell
docker compose --profile tools --env-file ../agent-store-be/.env run --rm gradle test
```

프론트엔드는 별도 터미널에서 실행합니다.

```powershell
Set-Location ../agent-store-fe
npm ci
npm run dev
```

## 검증과 정리

```powershell
docker compose --env-file ../agent-store-be/.env config
docker compose --profile tools --env-file ../agent-store-be/.env config
Invoke-RestMethod http://localhost:8080/health
Invoke-RestMethod http://localhost:8090/health
docker compose --env-file ../agent-store-be/.env ps catalog-bootstrap
Invoke-RestMethod 'http://localhost:8080/api/agents?limit=50&usageType=user_facing'
```

`docker compose --env-file ../agent-store-be/.env down`은 PostgreSQL 데이터를 보존합니다. 데이터를 포함해 새로 시작할 때만
`docker compose --env-file ../agent-store-be/.env down -v`를 사용합니다.
