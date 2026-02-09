---
name: git-issue-registrar
description: Creates a GitHub or GitLab issue based on the implementation plan. All issue content must be written in Korean.
---

# Execution Logic

1. **Detect Platform**: Identify if the repository is GitHub or GitLab using `git remote -v`.
2. **Language Rule**: **MUST** write the issue Title and Body in **Korean**, regardless of the language used in the plan or instructions.
3. **Template Application**: Use the following Korean template for the issue body.
4. **Set Assignee**: Assign the issue to the logged-in user.

# Issue Body Template (Korean)

"""

## 📝 개요

<Planner가 수립한 계획의 목적과 필요성을 한국어로 요약>

## 🛠 작업 리스트

- [ ] <세부 작업 1>
- [ ] <세부 작업 2>
- [ ] <세부 작업 3>

## 🔗 컨텍스트

- **타입**: <feat|fix|refactor|etc>
- **작성자**: Antigravity AI Agent
- **참조**: <관련 링크나 이슈 번호>

## 🏁 완료 조건 (Acceptance Criteria)

- [ ] <작업이 완료되었다고 판단할 수 있는 핵심 기준 1>
- [ ] <작업이 완료되었다고 판단할 수 있는 핵심 기준 2>
      """

# Commands

- [GitHub]: `gh issue create --title "<type>: <subject_in_korean>" --body "<body>" --label "<type>"`
- [GitLab]: `glab issue create -t "<type>: <subject_in_korean>" -d "<body>" --label "<type>"`
