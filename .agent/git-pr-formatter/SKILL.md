---
name: git-pr-formatter
description: Creates a GitHub or GitLab Pull Request linked to an existing issue. All PR content must be written in Korean.
---

# Execution Logic

1. **Detect Platform**
   - Identify whether the repository is GitHub or GitLab using `git remote -v`.

2. **Base Branch Detection**
   - Detect default base branch (`main` or `develop`).
   - Use it as the PR target unless explicitly specified.

3. **Language Rule**
   - **MUST** write the PR Title and Body in **Korean**.
   - No exceptions.

4. **Issue Binding**
   - PR **MUST** reference a single issue number.
   - Branch name is used as the primary source of the issue number (`#123`).

5. **Template Application**
   - Use the following Korean template for the PR body.
   - All sections are mandatory unless explicitly marked optional.

---

# PR Body Template (Korean)

"""
## 📌 개요
- 이 PR이 해결하는 문제와 변경 목적을 간결히 설명
- 관련 이슈: close #<issue-number>

---

## 🔧 변경 사항 요약
- 주요 변경 내용 1
- 주요 변경 내용 2
- 주요 변경 내용 3

---

## 🧪 테스트 내역
- [ ] 로컬 테스트 완료
- [ ] 주요 시나리오 검증
- [ ] 회귀 영향 없음 확인
- [ ] 추가 테스트 필요 (TODO 시 명시)

---

## ✅ 완료 조건 체크
(이슈의 Acceptance Criteria 기준)

- [ ] 조건 1
- [ ] 조건 2
- [ ] 조건 3

---

## 🧠 코드 리뷰 포인트
- 리뷰어가 집중해서 봐야 할 부분
- 구조적 판단이나 트레이드오프
- 의도적으로 선택한 설계 결정

---

## 📎 참고 사항 (Optional)
- 스크린샷
- 로그
- 추가 컨텍스트
"""

---

# Commands

- **GitHub**