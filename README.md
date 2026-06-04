# My Claude Skills

Claude Code 커스텀 스킬 모음.

## 설치

글로벌 스킬로 사용하려면 `~/.claude/skills/`에 복사:

```bash
cp -r ./* ~/.claude/skills/
```

## 스킬 목록

| 스킬 | 명령어 | 설명 |
|------|--------|------|
| [checkmd](./checkmd) | `/checkmd` | 기능 구현 계획 문서(plan 파일) 생성/업데이트 |
| [cleanup](./cleanup) | `/cleanup` | 디버그 로그, 주석 코드, 미사용 import 정리 |
| [code-review](./code-review) | `/code-review` | 종합 코드 리뷰 (QA + 보안 + 최적화 + 코드품질) |
| [gocode](./gocode) | `/gocode` | 계획 문서 기반 전체 구현 실행 |
| [notion](./notion) | `/notion` | 개발 일일보고서 생성 (노션용) |
| [tracking](./tracking) | `/tracking` | 추적 코드 자동 세팅 (Meta, gtag, Clarity 등) |
