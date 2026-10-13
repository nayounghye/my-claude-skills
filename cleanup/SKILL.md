---
name: cleanup
description: "디버그 로그, 주석 코드, 미사용 import 정리. 파일 경로를 인자로 받아 불필요한 코드를 제거합니다."
disable-model-invocation: true
---

Look through $ARGUMENTS files and clean up any leftover debug logs, commented code, or unused imports.

## 정리 범위 (수술적 변경 원칙과 충돌하지 않게)

이 스킬은 **명시적으로 정리를 요청받은 경우**에만 삭제한다. gocode의 수술적 변경 원칙(요청 밖 코드는 건드리지 않는다)의 예외이므로, 그 예외 범위를 좁게 지킨다.

**지운다:**

- `$ARGUMENTS`로 지정된 파일 안에서만. 지정 안 된 파일로 번지지 않는다.
- 디버그 로그(`console.log`·`print`·임시 `debugger`), 주석 처리된 죽은 코드, 미사용 import.

**지우지 않고 보고만 한다:**

- 의도가 있어 보이는 로그 — 에러 로깅·관측가능성용 로그는 남긴다. (silent fail을 만들지 않기 위해)
- 미사용으로 보이지만 **외부에서 참조될 수 있는 것** — export된 함수·타입, 동적 참조(문자열 키·리플렉션), 테스트에서만 쓰는 헬퍼.
- 왜 있는지 모르겠는 코드 — 지우기 전에 물어본다.

**마무리:** 지운 것과 남긴 것을 각각 목록으로 보고한다. 남긴 이유도 한 줄씩. 지운 뒤 타입 체크/빌드를 돌려 깨지지 않았는지 확인하고, 못 돌렸으면 못 돌렸다고 쓴다.
