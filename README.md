# my-skills

자주 쓰는 [Claude 스킬](https://docs.claude.com/en/docs/claude-code/skills)을 모아두는 개인 저장소입니다.
새로 만든 스킬을 하나씩 추가해 나갑니다.

## 스킬 목록

| 스킬 | 설명 |
| --- | --- |
| [prd-writer](prd-writer/SKILL.md) | 기획서(Figma 화면 정의서·텍스트·이미지)를 입력받아, 추측하지 않고 `[확인 필요]`로 빈칸을 드러내는 개발 착수용 PRD를 작성한다. |

## Claude에 적용하기

스킬은 디렉터리(`SKILL.md` 포함)를 Claude가 읽는 `skills/` 위치에 두면 적용됩니다.
범위에 따라 두 곳 중 하나를 고릅니다.

| 범위 | 위치 | 적용 대상 |
| --- | --- | --- |
| 개인 (모든 프로젝트) | `~/.claude/skills/<skill-name>/` | 내 머신의 모든 세션 |
| 프로젝트 (팀 공유) | `<프로젝트>/.claude/skills/<skill-name>/` | 해당 저장소를 여는 모두 |

### 적용 방법 (심볼릭 링크 권장)

이 저장소를 한 곳에 두고 심볼릭 링크만 걸면, 여기서 수정한 내용이 바로 반영됩니다.

```bash
# 개인 스킬로 적용 (예: prd-writer)
ln -s ~/workspace/my-skills/prd-writer ~/.claude/skills/prd-writer

# 또는 프로젝트 스킬로 적용
ln -s ~/workspace/my-skills/prd-writer <프로젝트>/.claude/skills/prd-writer
```

복사해서 쓰고 싶다면 `cp -r prd-writer ~/.claude/skills/` 처럼 디렉터리째 복사해도 됩니다.

### 사용

적용 후 Claude Code 세션에서 두 가지로 호출됩니다.

- **자동 호출** — 대화 내용이 `description`의 트리거와 맞으면 Claude가 알아서 스킬을 사용합니다. (예: "이 Figma로 PRD 만들어줘")
- **수동 호출** — `/<skill-name>` 슬래시 커맨드로 직접 실행합니다. (예: `/prd-writer`)

> 새 스킬을 추가하거나 `SKILL.md`를 수정한 뒤에는 세션을 새로 시작해야 반영됩니다.

## 구조

각 스킬은 자기 이름의 디렉터리 하나를 가지며, 최소한 `SKILL.md`를 포함합니다.

```
my-skills/
├── README.md
└── <skill-name>/
    ├── SKILL.md          # 스킬 본문 (frontmatter + 작성 절차)
    └── references/       # 템플릿·예시 등 보조 자료 (선택)
```

`SKILL.md`는 다음과 같은 frontmatter로 시작합니다.

```markdown
---
name: <skill-name>
description: "이 스킬이 무엇을 하는지, 그리고 언제 발동되어야 하는지를 한 문단으로. 트리거 문구를 구체적으로 적을수록 자동 호출 정확도가 올라간다."
---

# 스킬 제목

...본문...
```

## 새 스킬 추가하기

1. 스킬 이름으로 디렉터리를 만든다. (`kebab-case` 권장)
2. 그 안에 `SKILL.md`를 작성한다. `description`에는 **무엇을 / 언제** 둘 다 적는다.
3. 필요하면 `references/`에 템플릿이나 예시를 둔다.
4. 위 "스킬 목록" 표에 한 줄 추가한다.
