# CastCanvas Lab BE — Context

> 이 파일은 `cast-canvas-lab-be` 레포 작업 시 필요한 상세 컨텍스트다.
> 오케스트레이터가 관리하며, `cast-canvas-lab-be/CLAUDE.md`는 이 파일을 참조한다.

## About this repo

Backend API server for CastCanvas Lab. Handles core application data — not CRDT sync.

## Stack

- Java, Spring Boot, Gradle
- Spring Security + JWT
- Spring Data JPA + PostgreSQL
- Redis
- S3-compatible object storage
- OpenAPI / Swagger

## Primary responsibilities

- user auth
- workspace access control
- canvas metadata management
- nodes/edges metadata
- asset/document metadata
- signed upload URL issuance
- search APIs
- document-processing job orchestration
- integration contracts with the collab service

## Non-responsibilities

Do **not** put these here — they belong to collab:
- Yjs document synchronization
- awareness/presence broadcasting
- collaborative WebSocket transport
- CRDT conflict resolution

## Repo priorities

1. clear module boundaries
2. maintainable business logic
3. stable API contracts
4. strong security boundaries
5. clean integration points
6. future collaboration compatibility

## Architecture expectations

- favor a modular monolith
- keep controllers thin
- keep services focused
- use DTOs for API boundaries
- centralize exception handling
- keep persistence concerns separate from transport concerns
- preserve explicit transaction boundaries

## Module layout

- auth
- user
- workspace
- canvas
- node
- edge
- asset
- document
- search
- job
- common/global

## Layer pattern (follow the `user` module as the canonical reference)

Every module uses this four-layer DDD structure under `com.castcanvaslab.api.{module}`:

```
{module}/
  domain/
    {Entity}.java               # JPA entity, no framework imports beyond JPA/Lombok
    {Entity}Repository.java     # domain interface (not Spring Data — pure Java)
  application/
    {Entity}Service.java        # business logic, depends on domain interface
    dto/
      {Entity}Response.java     # outbound DTO (record)
      {Action}Request.java      # inbound DTO (record, with validation annotations)
  presentation/
    {Entity}Controller.java     # thin controller, delegates to service
  infrastructure/
    {Entity}JpaRepository.java  # Spring Data JPA, implements domain interface
```

**Concrete example — user module:**
- `user/domain/User.java` — entity with `create()` factory and domain methods
- `user/domain/UserRepository.java` — plain Java interface
- `user/application/UserService.java` — `@Transactional(readOnly = true)`, uses `UserRepository`
- `user/application/dto/UserResponse.java` — record with `from(User)` static factory
- `user/application/dto/UpdateNicknameRequest.java` — record with Bean Validation
- `user/presentation/UserController.java` — `@RestController`, accepts `Authentication`
- `user/infrastructure/UserJpaRepository.java` — `extends JpaRepository`, `implements UserRepository`

**Rules:**
- Controllers must not import entity classes directly
- Services depend on the domain interface, not the Spring Data repository
- DTOs live in `application/dto/`, not in `presentation/`
- Domain methods (e.g. `user.changeNickname(...)`) encapsulate mutation; avoid direct field access from services

## Code style

- prefer readable code over abstraction-heavy code
- avoid giant service classes
- avoid leaking entities into API responses
- prefer explicit naming
- avoid unnecessary annotations and indirection
- keep validation close to request boundaries
- keep authorization logic explicit

## Code Style & Linting

- **Spotless & Checkstyle:** Google Java Format (AOSP 스타일, 들여쓰기 4칸)
- 커밋 전 반드시 `./gradlew spotlessApply` 실행 (Git Hook 자동 실행)
- 코드 수정 후 `./gradlew checkstyleMain`으로 컨벤션 위반 확인
- IDE 설정은 `.editorconfig` 따름

## Persistence guidance

- PostgreSQL is the source of truth for application metadata
- collaboration state should not be conflated with ordinary relational metadata
- keep schema evolution migration-driven
- be careful with cascading deletes and orphan removal
- make indexes intentional

## Security guidance

- assume workspace-level authorization is important
- avoid controller-only authorization assumptions
- keep token validation and user context retrieval consistent
- treat upload URL issuance as privileged behavior
- make permission checks obvious in service code

## Integration guidance

Design for collab service integration:
- clear workspace/document ownership
- explicit authorization contract
- explicit API for collab-side validation if needed
- no hidden coupling through shared side effects

## Async/job guidance

- heavy document work should run asynchronously
- keep job payloads stable and small
- aim for idempotent handlers
- track job status explicitly
- avoid mixing orchestration and domain rules without reason

## Preferred workflow

Before editing:
1. identify the module boundary involved
2. inspect nearby service/controller/entity/DTO patterns
3. preserve current conventions

When changing code:
1. keep edits small and focused
2. avoid broad refactors unless requested

After changing code:
1. check imports and package layout
2. keep response shapes consistent
3. keep security assumptions explicit

## Avoid

- 설정된 린트 규칙을 우회하는 코드 작성
- 들여쓰기, 임포트 순서 등 포맷팅을 무시한 커밋
- embedding CRDT sync behavior here
- mixing transport and domain logic
- returning entities directly from controllers
- adding new frameworks casually
- hiding business logic in annotations or implicit behavior
