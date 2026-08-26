# AgentStore Infra

AgentStore 개발 환경의 실행 진입점입니다. `agent-store-be`, `demo-agent`, `agent-store-fe` 저장소가 이 저장소와 같은 상위 폴더에 있어야 합니다.
환경 변수는 각 애플리케이션 저장소가 소유합니다. API·PostgreSQL은 `agent-store-be/.env`, demo-agent는
`demo-agent/.env`를 사용합니다.

## 구성

| 구성 | 실행 방식 | 주소 |
|---|---|---|
| Frontend | 로컬 Vite | http://localhost:5173 |
| Spring API | Docker Compose | http://localhost:8080 |
| Go demo-agent | Docker Compose | http://localhost:8090 |
| PostgreSQL | Docker Compose | localhost:5432/agent_store |

Compose 안에서는 서비스 DNS를 사용합니다. Spring은 `demo-agent:8090` endpoint로 Agent를 호출하고,
demo-agent는 `api:8080` callback으로 dependency 결과를 요청합니다. 호스트에서는 demo-agent가
`127.0.0.1:8090`으로만 노출됩니다.
`dev` profile API는 `agents`가 0개일 때만 13개 demo Agent를 직접 등록합니다. 일반 Marketplace에는 투자·쇼핑·여행 Root 세 개가,
개발자 모드에는 전문 Agent를 포함한 13개가 보입니다.

## 실행

```powershell
Copy-Item ../agent-store-be/.env.example ../agent-store-be/.env
Copy-Item ../demo-agent/.env.example ../demo-agent/.env
docker compose --env-file ../agent-store-be/.env up --build -d --remove-orphans
```

상태와 로그는 다음처럼 확인합니다.

```powershell
docker compose --env-file ../agent-store-be/.env ps
docker compose --env-file ../agent-store-be/.env logs -f api demo-agent
```

```powershell
# Compose 설정만 바꾼 경우
docker compose --env-file ../agent-store-be/.env up -d

# Spring 코드나 application.yaml을 바꾼 경우
docker compose --env-file ../agent-store-be/.env build api
docker compose --env-file ../agent-store-be/.env up -d api demo-agent

# Go demo-agent 또는 catalog를 바꾼 경우
docker compose --env-file ../agent-store-be/.env build demo-agent
docker compose --env-file ../agent-store-be/.env up -d api demo-agent
```

로컬 Java 없이 Spring 검증을 실행합니다. 첫 실행은 Gradle distribution과 의존성을 받고, 이후 실행은
`agent-store-gradle-cache` named volume을 재사용합니다.

```powershell
docker compose --profile tools --env-file ../agent-store-be/.env run --rm gradle test
```

프론트엔드는 별도 터미널에서 실행합니다.

```powershell
Set-Location ../agent-store-fe
npm install
npm run dev
```

## 검증과 정리

```powershell
docker compose --env-file ../agent-store-be/.env config
docker compose --profile tools --env-file ../agent-store-be/.env config
Invoke-RestMethod http://localhost:8080/health
Invoke-RestMethod http://localhost:8090/health
Invoke-RestMethod 'http://localhost:8080/api/agents?limit=50&view=easy'
```

`docker compose --env-file ../agent-store-be/.env down`은 PostgreSQL 데이터를 보존합니다. 데이터를 포함해 새로 시작할 때만
`docker compose --env-file ../agent-store-be/.env down -v`를 사용합니다.
