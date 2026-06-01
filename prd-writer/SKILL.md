---
name: prd-writer
description: "Generate a developer-ready PRD (Product Requirements Document) from a screen spec or 기획서. Use this whenever the user wants to turn a Figma design link, a planning document (기획서), or a screen-definition image/text into a PRD — including phrases like 'PRD 만들어줘', 'PRD 작성해줘', '기획서로 PRD', 'Figma로 PRD', '이 화면 요구사항 정리해줘', 'turn this spec into a PRD', or when the user shares a Figma URL or spec image and asks for requirements. Reads the source (Figma via MCP, or pasted/attached text or image), then writes a structured PRD with strict no-guessing rules, measurable requirements, and Given-When-Then acceptance criteria. Use this even if the user only says 'PRD' alongside any design or spec."
---

# PRD Writer

기획서(Figma 화면 정의서 · 텍스트 · 이미지)를 입력받아, 엔지니어와 디자이너가 바로
착수할 수 있는 수준의 PRD를 일관된 구조로 작성하는 스킬.

이 스킬의 핵심 가치는 **"추측하지 않는 PRD"**다. 일반적인 LLM은 기획서의 빈칸을
그럴듯하게 메우는데, 그렇게 만든 요구사항은 개발 착수 후 재작업 비용을 만든다.
이 스킬은 빈칸과 모순을 메우는 대신 **드러내서** 기획자에게 되묻게 한다.

출력 언어는 **기획서의 언어를 따른다** (한국어 기획서 → 한국어 PRD).

---

## 작성 절차

### 1단계 — 입력 소스 판별 및 읽기

입력 형태에 따라 분기한다. **어떤 경우든 원문을 실제로 읽고 나서 작성한다. 읽지 않고
내용을 지어내지 않는다.**

#### (A) Figma 링크인 경우

URL에서 `fileKey`와 `nodeId`를 추출한다.
- 예: `https://figma.com/design/{fileKey}/{name}?node-id=12403-5679`
  → `fileKey = {fileKey}`, `nodeId = 12403:5679` (URL의 `-`는 API에서 `:`로 변환)
- URL에 `node-id`가 없으면, 특정 화면 노드 URL을 다시 요청한다.

읽는 순서:
1. **`get_metadata`** 를 먼저 호출해 노드의 구조·레이어 이름·텍스트 정의를 파악한다.
   기획서의 "화면 설명/정의" 텍스트 박스가 여기에 그대로 들어온다 — 이것이 명세 원문이다.
2. 시각 디테일(빈 상태 모습, 읽음/안읽음 강조 등) 확인이 필요하면 **`get_design_context`**
   로 스크린샷을 가져온다. 컨텍스트 절약을 위해 텍스트 명세로 충분하면 생략 가능.
3. `hidden="true"` 노드나 의미 없는 더미 텍스트(예: "ㄴㅇㄴㄴ", "asdf")는 무시한다.

Figma MCP 도구가 로드되어 있지 않으면 먼저 `tool_search`로 찾는다(예:
`figma get design context metadata`). 연결 자체가 안 되어 있으면, 추측으로 진행하지
말고 사용자에게 Figma 커넥터 연결 또는 노드 URL을 요청한다.

#### (B) 텍스트 기획서인 경우

대화에 붙여넣은 텍스트, 또는 업로드된 문서(.md/.txt/.docx 등)를 직접 읽는다.
업로드 파일은 `/mnt/user-data/uploads/`에 있으며, 파일 형식별 읽기 방법은
file-reading 스킬을 참고한다.

#### (C) 이미지 기획서인 경우

업로드된 화면 캡처/기획 이미지를 직접 보고(비전) 텍스트·UI 요소·상태를 읽어낸다.
이미지에서 글자가 흐리거나 잘려 확실치 않은 부분은 추측하지 말고 `[확인 필요]`로 둔다.

여러 소스가 섞여 들어오면(예: Figma + 보충 텍스트) 모두 읽고 통합하되, 출처 간
충돌이 있으면 4대 규칙의 모순 처리(규칙 4)를 적용한다.

### 2단계 — 4대 작성 규칙 적용

PRD 전체에 다음 규칙을 **항상** 적용한다. 이 규칙들이 이 스킬의 정체성이다.

1. **추측 금지.** 기획서에 명시되지 않은 내용은 임의로 채우지 않는다. 추론·보강이
   필요한 부분은 본문에 `[확인 필요: …]`로 인라인 표시하고, 문서 끝
   "확인이 필요한 항목" 섹션(Q1, Q2, …)에 질문·분류·근거와 함께 모은다.
2. **측정 가능화.** 정성적 표현을 검증 가능한 형태로 구체화한다.
   ("빠르게" → "응답 2초 이내", "잘 보이게" → 구체적 대비/크기). 단, 기획서에 수치가
   없어 제안하는 값이라면 그 값 자체를 `[확인 필요]`로 표시한다.
3. **인수 조건(Given-When-Then).** 각 기능 요구사항에 사용자 스토리와 인수 조건을
   Given-When-Then 형식으로 작성한다. 정상 흐름뿐 아니라 경계/예외 케이스도 포함한다.
   (이 형식은 그대로 E2E/위젯 테스트 시나리오로 전환되므로 테스트 추적성이 생긴다.)
4. **모순·누락 탐지.** 기획서 내부의 불일치(예: 화면 설명과 목업이 다른 기능을 가리킴,
   복붙 흔적)를 발견하면 임의로 합치거나 한쪽을 버리지 말고, 그대로 짚어 `[확인 필요]`로
   분리하고 근거를 적는다.

### 3단계 — 8섹션 구조로 작성

아래 8개 섹션을 **순서·제목 고정**으로 작성한다. 각 섹션의 항목별 작성 가이드와
빈 템플릿은 `references/prd-template.md`를 읽고 그대로 채운다.

1. 개요 / 배경 및 문제 정의
2. 목표 및 성공 지표 (측정 가능한 KPI)
3. 사용자 스토리
4. 기능 요구사항 (우선순위 P0/P1/P2, 각 항목에 Given-When-Then 인수 조건)
5. 비기능 요구사항 (성능 · 보안/권한 · 접근성 · 데이터/표시 규칙)
6. 범위 외 (Out of Scope)
7. 의존성 및 리스크
8. 확인이 필요한 항목 (Q표: 번호·분류·질문·근거)

우선순위 정의: **P0**=MVP 필수, **P1**=중요하나 후속 가능, **P2**=개선.

### 4단계 — 출력

- **Markdown 파일**로 저장한다. Notion 붙여넣기 친화적으로 헤딩과 표를 활용한다.
- 파일명 규칙: `PRD_<프로젝트/제품>_<화면경로>.md`
  (예: `PRD_솔라시도_마이페이지_공지사항.md`)
- `present_files` 도구가 있으면 파일을 제시한다.
- 작성 후, 발견한 **모순/주요 누락(특히 개발 임팩트가 큰 것)**을 1~3개 짚어
  사용자에게 간단히 브리핑한다. 그 PRD를 받아 바로 개발에 들어갈 사람이 가장 먼저
  알아야 할 것을 우선한다.

---

## 작성 시 자주 빠지는 함정 (체크리스트)

- [ ] 기획서에 없는 KPI·수치를 단정해서 적지 않았는가? (제안값은 `[확인 필요]`로)
- [ ] 데이터 출처(API 스펙)·상태 저장 위치(서버/로컬) 같은, 명세에 없지만 구현에
      필수인 항목을 `[확인 필요]`로 끌어올렸는가?
- [ ] 정상 상태만 있는 기획서에서 로딩·에러·빈 상태·페이지네이션을 점검했는가?
- [ ] 색상으로만 구분되는 UI(읽음/안읽음 등)의 접근성 보조 표식을 짚었는가?
- [ ] 인수 조건이 "정상 흐름"만 있고 경계/예외가 빠지지 않았는가?
- [ ] 화면 설명과 목업이 가리키는 기능이 일치하는가? (불일치 시 규칙 4)
