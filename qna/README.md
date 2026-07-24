# qna — QnA 원본 축적소

[백엔드 학습 트랙](https://linear.app/klilly/project/백엔드-실력-다지기-이직-준비-2027-950ee5d4b255)에서 Claude(강사)와 나눈 QnA 원본을 주제별로 쌓는 곳. 강의 영상 대신 QnA로 학습하는 방식에서 나온 **질문·답변·트레이드오프를 날것 그대로** 남긴다.

## 왜 남기나

- velog TIL 글의 재료 (질문 흐름 → 스토리로 재구성. 예: `phase5/clustered-index-mvcc.md` → velog "붙여둘까 떼어둘까")
- 나중에 **전자책**으로 엮을 소스
- 내가 뭘 궁금해했고 어떻게 이해했는지의 기록 (성장 로그)

## 구조

```
qna/
├── phase1/ ~ phase10/     # Linear Phase(마일스톤)와 대응
│   └── <주제>.md  또는  <이슈번호-섹션-주제>.md
```

- 강의 TIL: `<이슈번호>-<섹션>-<주제>.md` (예: `165-02-project-setup.md`)
- 독립 개념: `<주제>.md` (예: `clustered-index-mvcc.md`)

## 파일 포맷

```markdown
# [주제]
> Linear: LIL-xxx | Phase N | YYYY-MM-DD | 관련 강의/섹션

## Q1. (실제로 물어본 질문 원문)
**A.** 강사 답변 핵심.

**💡 왜/트레이드오프:** ...

## Q2. ...
```

## 운영 규칙

1. 학습 세션이 끝날 때 그날의 QnA를 해당 파일에 append.
2. Linear 이슈 본문엔 요약 + 이 파일 경로만 링크(원본을 중복 저장하지 않는다).
3. velog 글로 발전시키면 파일 상단에 `→ velog: <slug/url>` 한 줄 추가.

## 인덱스

| 파일 | 주제 | Linear | velog |
|---|---|---|---|
| phase1/165-02-project-setup.md | 프로젝트 환경설정(Gradle·전이의존성·톰캣·jar) | LIL-193 | back-to-basics-02 |
| phase1/165-03-web-basics.md | 웹 개발 3가지 방식(정적·MVC·API) | LIL-165 | (예정) |
| phase5/clustered-index-mvcc.md | 클러스터형 인덱스·MVCC | LIL-198 | back-to-basics-03 |
