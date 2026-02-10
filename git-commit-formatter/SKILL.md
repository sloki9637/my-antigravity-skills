---
name: git-commit-formatter
description: Formats git commit messages according to Conventional Commits specification. Use this when the user asks to commit changes or write a commit message.
---

# Git Commit Formatter Skill

When writing a git commit message, you MUST follow the Conventional Commits specification and include the issue number.

## Format
`<type>(#<issue_number>): <description>` 또는 `<type>(#<issue_number>)[optional scope]: <description>`

## Allowed Types
- **feat**: 새로운 기능 추가
- **fix**: 버그 수정
- **docs**: 문서 수정만 해당
- **style**: 코드 의미에 영향을 주지 않는 변경 (포맷팅, 세미콜론 누락 등)
- **refactor**: 버그 수정이나 기능 추가가 없는 코드 변경
- **perf**: 성능 개선을 위한 코드 변경
- **test**: 테스트 코드 추가 및 수정
- **chore**: 빌드 프로세스, 보조 도구 및 라이브러리 변경

## Instructions
1. **Link Issue**: `git-issue-registrar`로 생성된 이슈 번호를 반드시 포함한다.
2. **Analyze Changes**: 변경 사항을 분석하여 적절한 `type`을 결정한다.
3. **Identify Scope**: 필요한 경우 괄호 뒤에 `scope`를 추가한다 (예: `feat(#1)(auth): ...`).
4. **Korean Description**: 상세 설명은 반드시 **한국어**로 작성한다. (영어 사용 엄격히 금지)
5. **Language Mix**: `type`과 `scope`는 영문으로 유지하되, 그 외 설명은 한국어로 작성한다.
6. **Breaking Changes**: 파괴적 변경이 있다면 `BREAKING CHANGE:` 뒤에 한국어로 설명을 추가한다.

## Examples
- `feat(#12)(auth): 구글 로그인 기능 구현`
- `fix(#45): 사용자 응답 데이터 누락 수정`
- `style(#7)(ui): 메인 버튼 색상 변경`
- `refactor(#23): API 호출 로직 구조 개선`