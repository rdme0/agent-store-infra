# AgentStore Infra 인수인계서

최종 갱신: 2026-08-27

## 소유 범위

- 경로: 이 저장소 루트
- `compose.yaml`은 PostgreSQL, Spring API, 독립 Go demo-agent, catalog-bootstrap의 로컬 실행 구성을
  소유한다.
- Compose는 일반 service network를 사용한다. API는 `api:8080`, Go는 `demo-agent:8090`으로 통신하며
  Spring/Go의 exact origin allowlist와 일치한다.
- PostgreSQL은 `postgres:17`과 named volume을 사용한다. 백엔드가 유일한 DB writer다.
- demo-agent image는 같은 binary로 실행하고 `/app/config/application.yaml`을 사용한다. bootstrap은 API와
  Go health 이후 한 번 실행된다.

## 비밀값과 실행

- API secret은 `agent-store-be/.env`, Go OpenAI secret은 `demo-agent/.env`에서 주입한다. secret 값을
  compose 파일이나 문서에 복사하지 않는다.
- `DEMO_AGENT_MODE`는 Compose가 command flag로 명시해야 하는 유일한 실행 mode 선택값이다. fixture와
  openai 사이에 암묵적 fallback을 두지 않는다.
- host port는 API `8080`, Go `127.0.0.1:8090`이며 Go callback 보안 경계를 약화시키지 않는다.

## 검증

```powershell
$env:DEMO_AGENT_MODE = 'fixture'
docker compose config
```

Docker engine이 실행된 상태에서 `DEMO_AGENT_MODE=fixture`로 `docker compose up -d postgres api demo-agent` 후
각 health endpoint를 확인하고, `docker compose run --rm catalog-bootstrap`을 최초·재실행한다. 2026-08-27
fresh named volume에서 V1~V25 migration, health 200, 13개 Agent·9개 dependency와 두 번의 bootstrap을
확인했다. 일반 Compose와 tools profile의 `config --quiet`도 통과했다. 기존 volume은 V23의 historical
simulated payment 보호 동작을 확인하기 위해 삭제하지 않았다. 변경 후 `git diff --check`를 실행한다.
