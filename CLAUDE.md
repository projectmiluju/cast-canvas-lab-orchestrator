# CastCanvas Lab Orchestrator — CLAUDE.md

## 역할

이 레포는 CastCanvas Lab 전체 4개 레포를 조율하는 **오케스트레이터**다.

직접 프로덕션 코드를 소유하지 않는다. 대신:

- 레포 간 **API 계약**을 단일 소스로 관리한다
- **크로스 레포 태스크**를 정의하고 추적한다
- 각 레포 에이전트에게 **명확한 위임**을 수행한다
- 시스템 전체 **일관성**을 검증한다

---

## 필수 참조 문서

작업 전 반드시 읽어야 할 문서:

| 문서 | 위치 | 목적 |
|------|------|------|
| 시스템 아키텍처 | `../cast-canvas-lab-be/ARCHITECTURE.md` | 레포 경계, 통합 흐름 |
| MVP 범위 | `../cast-canvas-lab-be/MVP.md` | 현재 구현 목표 |
| REST API 계약 | `contracts/openapi.yaml` | FE↔BE 인터페이스 |
| Collab 프로토콜 | `contracts/collab-protocol.md` | FE↔COLLAB 인터페이스 |
| 전체 태스크 | `tasks/MASTER_TASKS.md` | 크로스 레포 진행 상황 |
| BE 구현 상태 | `../cast-canvas-lab-be/STATUS.md` | BE 상세 진행 상황 |

---

## 오케스트레이터 운영 방식

### 크로스 레포 태스크 처리 흐름

```
사용자 요청
  ↓
1. MASTER_TASKS.md 읽기 → 현재 상태 파악
2. ARCHITECTURE.md 읽기 → 영향 받는 레포 식별
3. contracts/ 읽기 → 계약 확인 및 변경 필요 여부 판단
   ↓
계약 변경이 필요한 경우:
  → contracts/ 먼저 수정
  → 각 레포 에이전트에게 계약 기반으로 구현 위임
   ↓
단일 레포 변경인 경우:
  → 해당 레포 CLAUDE.md 읽기
  → 해당 레포 에이전트에게 위임
   ↓
완료 후:
  → MASTER_TASKS.md 상태 업데이트
```

### 멀티 에이전트 위임 원칙

- 독립적인 레포 작업은 **병렬**로 서브에이전트에 위임한다
- 계약 변경이 선행되어야 하는 작업은 **순차**로 처리한다
- 각 서브에이전트는 자신의 레포 경계 안에서만 작동해야 한다
- 서브에이전트 결과는 계약과의 일관성을 검증한다

---

## 계약 관리 규칙

### API 계약 변경 시

1. `contracts/openapi.yaml` 수정
2. BE 에이전트에게 구현 위임 (`cast-canvas-lab-be`)
3. FE 에이전트에게 클라이언트 업데이트 위임 (`cast-canvas-lab-fe`)
4. 순서: 계약 → BE → FE (의존 방향)

### Collab 프로토콜 변경 시

1. `contracts/collab-protocol.md` 수정
2. COLLAB 에이전트에게 구현 위임 (`cast-canvas-lab-collab`)
3. FE 에이전트에게 클라이언트 업데이트 위임 (`cast-canvas-lab-fe`)
4. 순서: 계약 → COLLAB + FE (병렬 가능)

---

## 각 레포 에이전트 특성

### cast-canvas-lab-be (BE)
- 작업 시작 전: `MASTER_TASKS.md` → `context/be.md` → `ARCHITECTURE.md`, `MVP.md` → `STATUS.md` → 관련 `docs/modules/*.md` 순서로 읽기
- 작업 완료 후: `MASTER_TASKS.md` 항목 체크 + `STATUS.md` 업데이트 필수
- API 계약 참조: `docs/api-contracts.md` (BE 관점의 API 계약 문서)
- 새 모듈 추가 시 `docs/modules/{name}.md`, 설계 결정 시 `docs/decisions/NNN-*.md` 작성
- 실행: `./gradlew bootRun`, 빌드: `./gradlew build`, 테스트: `./gradlew test`, 포맷: `./gradlew spotlessApply`, 체크스타일: `./gradlew checkstyleMain`

### cast-canvas-lab-fe (FE)
- 작업 시작 전: `MASTER_TASKS.md` → `context/fe.md` → `ARCHITECTURE.md`, `STATUS.md` → 관련 `docs/features/*.md` 순서로 읽기
- 작업 완료 후: `MASTER_TASKS.md` 항목 체크 + `STATUS.md` 업데이트 + `docs/features/` 문서화
- 설계 결정 시 `docs/decisions/NNN-*.md`
- 빌드: `pnpm build`, 린트: `pnpm lint`, 타입체크: `pnpm typecheck`, 포맷: `pnpm format`, 포맷체크: `pnpm format:check`

### cast-canvas-lab-collab (COLLAB)
- 작업 시작 전: `MASTER_TASKS.md` → `context/collab.md` → `ARCHITECTURE.md`, `MVP.md`, `COLLAB_PROTOCOL.md` → `contracts/collab-protocol.md` → 관련 `docs/modules/*.md` 순서로 읽기
- 프로토콜 변경은 `contracts/collab-protocol.md` 기준
- 작업 완료 후: `MASTER_TASKS.md` 항목 체크 + `STATUS.md` 업데이트 필수
- 새 모듈 추가 시 `docs/modules/{name}.md`, 설계 결정 시 `docs/decisions/NNN-*.md` 작성
- 빌드: `pnpm build`, 린트: `pnpm lint`, 타입체크: `pnpm typecheck`, 포맷: `pnpm format`, 포맷체크: `pnpm format:check`, 테스트: `pnpm test`

### cast-canvas-lab-site (SITE)
- 작업 시작 전: `MASTER_TASKS.md` → `context/site.md` → `ARCHITECTURE.md`, `MVP.md` → `STATUS.md` → 관련 `docs/decisions/*.md` 순서로 읽기
- 작업 완료 후: `MASTER_TASKS.md` 항목 체크 + `STATUS.md` 업데이트 필수
- 설계 결정 시 `docs/decisions/NNN-*.md` 추가
- 빌드: `pnpm build`, 린트: `pnpm lint`, 타입체크: `pnpm typecheck`, 포맷: `pnpm format`, 포맷체크: `pnpm format:check`

---

## 공통 문서화 패턴

모든 레포는 비슷한 문서화 구조를 따른다:

| 문서 유형 | 경로 패턴 | 적용 레포 | 작성 시점 |
|-----------|-----------|-----------|----------|
| 설계 결정 | `docs/decisions/NNN-*.md` | **전체** (BE, FE, COLLAB, SITE) | 비자명한 설계 결정이 있을 때 |
| 모듈 문서 | `docs/modules/{name}.md` | BE, COLLAB | 새 모듈 추가 시 |
| 기능 문서 | `docs/features/{name}.md` | FE | feature 구조 변경 시 |
| API 계약 | `docs/api-contracts.md` | BE | API 변경 시 |
| 스키마 문서 | `docs/schema.md` | BE | DB 스키마 변경 시 |

> 각 레포에 `docs/decisions/_template.md` 등 템플릿이 존재한다. 새 문서 작성 시 반드시 템플릿을 따른다.

---

## 크로스 레포 통합 체크리스트

새 기능 구현 시 다음을 확인한다:

**인증 관련**
- [ ] BE: JWT 토큰 발급 엔드포인트 구현
- [ ] FE: 토큰 저장 및 API 요청 헤더 주입
- [ ] COLLAB: 연결 시 토큰 검증

**데이터 흐름 관련**
- [ ] `contracts/openapi.yaml`에 엔드포인트 명세 존재
- [ ] BE: 명세대로 구현
- [ ] FE: 명세 기반 API 클라이언트 호출

**협업 관련**
- [ ] `contracts/collab-protocol.md`에 메시지 타입 명세 존재
- [ ] COLLAB: 명세대로 구현
- [ ] FE: 명세 기반 WebSocket 클라이언트 구현

---

## 디렉터리 구조

```
cast-canvas-lab-orchestrator/
├── CLAUDE.md                    ← 이 파일
├── contracts/
│   ├── openapi.yaml             ← FE↔BE REST API 계약 (단일 소스)
│   └── collab-protocol.md       ← FE↔COLLAB WebSocket 프로토콜 계약
├── context/
│   ├── be.md                    ← BE 상세 컨텍스트 (스택, 아키텍처, 가이드라인)
│   ├── fe.md                    ← FE 상세 컨텍스트 (스택, 프로젝트 구조, 디자인 토큰)
│   ├── collab.md                ← COLLAB 상세 컨텍스트 (프로토콜, 설계 원칙)
│   └── site.md                  ← SITE 상세 컨텍스트 (아이덴티티, 디자인 시스템)
└── tasks/
    └── MASTER_TASKS.md          ← 크로스 레포 태스크 추적
```

> **레포별 CLAUDE.md 역할**: 각 레포의 `CLAUDE.md`는 작업 흐름(시작 전/후 필수)과 커맨드만 포함한다.
> 상세 컨텍스트는 위 `context/` 파일을 읽는다.

---

## 이 레포가 하지 않는 것

- 직접 프로덕션 코드 작성
- 특정 레포의 기술적 결정 대행
- 레포 내부 구조 변경
- 배포 설정 관리
