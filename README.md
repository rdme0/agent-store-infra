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

demo-agent는 API 컨테이너의 네트워크 네임스페이스를 공유합니다. 따라서 API와 demo-agent 사이의 기존
`127.0.0.1:8080` callback 및 `127.0.0.1:8090` Agent endpoint가 그대로 동작합니다.

## 실행

```powershell
Copy-Item ../agent-store-be/.env.example ../agent-store-be/.env
Copy-Item ../demo-agent/.env.example ../demo-agent/.env
docker compose --env-file ../agent-store-be/.env up --build -d
```

상태와 로그는 다음처럼 확인합니다.

```powershell
docker compose --env-file ../agent-store-be/.env ps
docker compose --env-file ../agent-store-be/.env logs -f api demo-agent
```

API와 demo-agent는 같은 네트워크 네임스페이스를 공유하므로, 두 서비스 중 하나를 변경한 뒤에는
항상 함께 다시 생성합니다.

```powershell
docker compose --env-file ../agent-store-be/.env up --build -d api demo-agent
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
Invoke-RestMethod http://localhost:8080/health
Invoke-RestMethod http://localhost:8090/health
```

`docker compose --env-file ../agent-store-be/.env down`은 PostgreSQL 데이터를 보존합니다. 데이터를 포함해 새로 시작할 때만
`docker compose --env-file ../agent-store-be/.env down -v`를 사용합니다.
