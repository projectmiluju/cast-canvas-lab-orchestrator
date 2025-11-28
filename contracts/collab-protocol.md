# CastCanvas Lab — Collab WebSocket Protocol

## 이 문서의 역할

FE↔COLLAB 간 WebSocket 프로토콜의 **단일 소스**다.

- COLLAB 서버는 이 명세를 기준으로 구현한다
- FE 클라이언트는 이 명세를 기준으로 연결한다
- 프로토콜 변경 시 이 파일을 먼저 수정한다

---

## 1. 연결

### 엔드포인트

```
ws://localhost:3001/collab
wss://collab.castcanvaslab.com/collab
```

### 연결 파라미터

WebSocket 연결 시 쿼리스트링으로 전달:

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| `roomId` | string (UUID) | 필수 | 캔버스 ID (BE의 Canvas.id와 동일) |
| `token` | string | 필수 (로그인 시) | BE에서 발급한 JWT 액세스 토큰 |

```
ws://localhost:3001/collab?roomId={canvasId}&token={accessToken}
```

### 연결 거부 코드

| 코드 | 의미 |
|------|------|
| 4001 | 토큰 없음 또는 유효하지 않은 토큰 |
| 4003 | 워크스페이스 접근 권한 없음 |
| 4004 | 존재하지 않는 room |

---

## 2. 메시지 포맷

모든 메시지는 **바이너리(Uint8Array)** 로 전송한다.
Yjs `y-protocols` 인코딩을 따른다.

### 메시지 타입

| 타입 값 | 이름 | 방향 | 설명 |
|--------|------|------|------|
| 0 | `sync` | 양방향 | 초기 동기화 (state vector / update 교환) |
| 1 | `update` | 양방향 | 증분 Yjs 업데이트 |
| 2 | `awareness` | 양방향 | awareness 상태 업데이트 |
| 3 | `auth` | 서버→클라이언트 | 인증 결과 |

---

## 3. 초기 동기화 흐름

```
클라이언트                           서버
   |                                  |
   |── WebSocket 연결 ────────────────>|
   |                                  | (토큰 검증, 워크스페이스 권한 확인)
   |                                  | (room 생성 또는 로드)
   |                                  |
   |<── sync(step1: server stateVector)|  서버 state vector 전송
   |                                  |
   |── sync(step2: missing updates) ─>|  클라이언트가 누락된 업데이트 전송
   |                                  |
   |── sync(step1: client stateVector)>|  클라이언트 state vector 전송
   |                                  |
   |<── sync(step2: missing updates) ─|  서버가 누락된 업데이트 전송
   |                                  |
   | [동기화 완료 — 정상 협업 상태]    |
```

### sync 메시지 구조

`y-protocols/sync` 표준 인코딩 사용:

```
[messageType=0][syncStep][...yjs encoded data]
```

- `syncStep=1`: state vector 전송
- `syncStep=2`: update 전송 (상대방의 state vector 기반)

---

## 4. 증분 업데이트

```
클라이언트                           서버
   |                                  |
   |── update(yjsBinaryUpdate) ──────>|
   |                                  | (Y.Doc에 적용)
   |                                  |
   |                         <────────|── update(yjsBinaryUpdate) ── (같은 room의 다른 클라이언트)
```

### update 메시지 구조

```
[messageType=1][...yjs binary update]
```

- 서버는 업데이트를 Y.Doc에 적용한 뒤 같은 room의 **다른** 연결에 브로드캐스트한다
- 원본 발신자에게는 재전송하지 않는다

---

## 5. Awareness

```
클라이언트                           서버
   |                                  |
   |── awareness(state) ─────────────>|
   |                                  | (awareness 상태 갱신)
   |                         <────────|── awareness(allStates) ── (같은 room의 다른 클라이언트)
   |                                  |
   |── disconnect ────────────────────>|
   |                                  | (해당 클라이언트 awareness 제거)
   |                         <────────|── awareness(removedState) ── (다른 클라이언트)
```

### awareness 메시지 구조

`y-protocols/awareness` 표준 인코딩 사용:

```
[messageType=2][...awareness encoded data]
```

awareness 상태 스키마 (클라이언트가 설정하는 값):

```typescript
interface AwarenessState {
  user: {
    id: string;       // BE User.id (UUID)
    nickname: string; // BE User.nickname
    color: string;    // 커서 색상 (클라이언트가 결정)
  };
  cursor: {
    x: number;        // 캔버스 좌표
    y: number;
  } | null;
}
```

---

## 6. Room 식별자 규칙

- Room ID는 **BE의 Canvas ID (UUID)** 와 동일하다
- 클라이언트가 임의로 생성하지 않는다
- COLLAB 서버는 room ID를 기반으로 BE에 접근 권한을 확인한다

---

## 7. 인증 흐름

```
1. FE: WebSocket 연결 요청 (token 쿼리파라미터 포함)
2. COLLAB: token 파싱 및 기본 유효성 확인
3. COLLAB: BE /api/v1/collab/validate 호출 (workspaceId + userId)
4. BE: 접근 권한 확인 후 role 반환 (owner/editor/viewer)
5. COLLAB: viewer는 읽기만 허용, owner/editor는 쓰기 허용
6. COLLAB: room join 완료 → 초기 동기화 시작
```

---

## 8. 비로그인 사용자 정책

MVP 기준:
- 비로그인 사용자는 COLLAB 서버에 연결할 수 없다 (협업 불가)
- 로컬 전용 캔버스는 FE에서 Yjs를 로컬로만 사용한다 (COLLAB 연결 없음)
- 공유 링크로 진입한 비로그인 사용자는 조회 전용 UI만 제공 (COLLAB 연결 없음)

---

## 9. 구현 상태

| 기능 | COLLAB | FE |
|------|--------|-----|
| WebSocket 연결 | [planned] | [planned] |
| 인증 검증 | [planned] | [planned] |
| 초기 동기화 | [planned] | [planned] |
| 증분 업데이트 | [planned] | [planned] |
| Awareness | [planned] | [planned] |
| 연결 해제 정리 | [planned] | [planned] |
| 상태 영속성 | [planned] | N/A |
