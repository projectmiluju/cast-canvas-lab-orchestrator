# CastCanvas Lab Orchestrator

CastCanvas Lab은 PDF 문서, 이미지, 노트를 하나의 무한 캔버스 위에서 연결해 리서치 흐름을 정리하는 공간형 워크스페이스입니다.

> Notion보다 더 공간적으로, Figma보다 더 문서 친화적으로.

## 프로젝트 개요

CastCanvas Lab은 아래 5개 저장소로 구성됩니다.

| 레포 | 역할 |
| --- | --- |
| `cast-canvas-lab-orchestrator` | 크로스 레포 계약, 태스크, 컨텍스트 관리 |
| `cast-canvas-lab-fe` | 워크스페이스 프론트엔드 |
| `cast-canvas-lab-be` | 메인 백엔드 API 서버 |
| `cast-canvas-lab-collab` | 실시간 협업 서버 |
| `cast-canvas-lab-site` | 퍼블릭 랜딩 사이트 |

레포 간 책임 분리와 통합 흐름은 이 저장소의 [CLAUDE.md](./CLAUDE.md)와 `contracts/`, `tasks/` 문서를 기준으로 유지합니다.

## 이 레포의 역할

`cast-canvas-lab-orchestrator`는 프로덕션 애플리케이션 코드를 직접 구현하는 저장소가 아니라, 여러 저장소 사이의 계약과 작업 흐름을 조율하는 운영 저장소입니다.

이 레포가 담당하는 범위:

- FE, BE, COLLAB 사이의 공유 계약 관리
- 크로스 레포 태스크 정의와 진행 상태 추적
- 레포별 작업 전 참고해야 하는 컨텍스트 정리
- 통합 작업 시 선후관계와 위임 기준 문서화

이 레포가 담당하지 않는 범위:

- 브라우저 UI 구현
- 메인 비즈니스 API 구현
- 실시간 협업 서버 구현
- 퍼블릭 마케팅 페이지 구현

즉, 이 저장소는 제품 기능을 직접 배포하는 레포가 아니라, 여러 레포가 같은 기준으로 움직이도록 만드는 단일 문서 소스에 가깝습니다.

## 문서 스택

| 항목 | 내용 |
| --- | --- |
| Main Guide | `CLAUDE.md` |
| API Contract | `contracts/openapi.yaml` |
| Collab Contract | `contracts/collab-protocol.md` |
| Task Tracking | `tasks/MASTER_TASKS.md` |
| Repo Context | `context/*.md` |
| Version Control | Git |

## 시작하기

요구 사항:

- Git
- Markdown/YAML을 읽고 수정할 수 있는 편집기

1. 작업하려는 변경이 어느 레포에 영향을 주는지 `tasks/MASTER_TASKS.md`에서 확인합니다.

2. 계약 변경 여부를 `contracts/` 문서에서 먼저 판단합니다.

3. 레포별 세부 맥락이 필요하면 `context/` 문서를 읽습니다.

4. 실제 구현 레포에서 작업한 뒤, 결과에 맞춰 이 저장소의 계약/태스크 문서를 업데이트합니다.

## 주요 파일

| 경로 | 설명 |
| --- | --- |
| `CLAUDE.md` | 오케스트레이터 운영 원칙과 위임 흐름 |
| `contracts/openapi.yaml` | FE↔BE REST API 계약 |
| `contracts/collab-protocol.md` | FE↔COLLAB WebSocket 프로토콜 계약 |
| `tasks/MASTER_TASKS.md` | 전체 레포 단위 작업 추적 |
| `context/be.md` | 백엔드 작업 컨텍스트 |
| `context/fe.md` | 프론트엔드 작업 컨텍스트 |
| `context/collab.md` | 협업 서버 작업 컨텍스트 |
| `context/site.md` | 랜딩 사이트 작업 컨텍스트 |

## 프로젝트 구조

```text
cast-canvas-lab-orchestrator/
├── CLAUDE.md
├── README.md
├── contracts/
│   ├── openapi.yaml
│   └── collab-protocol.md
├── context/
│   ├── be.md
│   ├── fe.md
│   ├── collab.md
│   └── site.md
└── tasks/
    └── MASTER_TASKS.md
```

## 협업 규칙

- 계약 변경이 필요한 작업은 구현보다 먼저 `contracts/`를 수정합니다.
- 크로스 레포 작업 상태는 `tasks/MASTER_TASKS.md`를 단일 소스로 유지합니다.
- 각 구현 레포의 사용자 노출 설명과 API 설명은 여기 문서와 충돌하지 않게 맞춥니다.
- 개인용 메모나 에이전트 런타임 파일은 커밋하지 않습니다.
