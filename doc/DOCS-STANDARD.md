# 문서화 표준

> 기준일: 2026-07-12 — peter-cho-70의 모든 프로젝트에 공통으로 적용하는 문서 정리·기록·발행 규칙. `my-education-docs`를 만들면서 정리했고, 앞으로 새 프로젝트는 매번 다시 정의하지 않고 이 표준을 그대로 따른다. 실행용 원본 템플릿은 로컬 `~/.claude/templates/project-docs-site/`에 있다.

## 왜 이렇게 하는가

- 문서가 소스 코드 저장소 루트에 흩어져 쌓이면 뭐가 최신이고 뭐가 죽은 문서인지 구분이 안 된다 → **성격별 폴더 + 상태 배지**로 해결.
- "지금 뭐가 되어있고 뭐가 안 되어있는지"를 다시 설명하는 데 매번 시간이 든다 → **STATUS.md(현재 스냅샷) + DEVLOG.md(히스토리)**를 분리해서, 현재만 궁금하면 STATUS만 읽으면 되게 한다.
- 마크다운을 GitHub에서 그대로 읽으면 표·코드블록이 예쁘게 안 보이고, 목차도 없다 → **GitHub Pages + 커스텀 뷰어**로 사람이 읽기 좋은 형태로 공개한다.
- 문서 사이트가 프로젝트마다 따로 흩어지면 뭐가 있는지 잊어버린다 → **이 허브 페이지**에서 전부 한 곳에 모은다.

## 1. 소스 저장소 안 로컬 문서 구조

```
docs/  (프로젝트에 이미 doc/ 관행이 있으면 그 이름 유지)
├── STATUS.md          현재 상태 스냅샷(완료/진행중/이슈/다음 우선순위) — 매번 "지금 기준"으로 덮어쓴다
├── DEVLOG.md           날짜별 히스토리 — 절대 덮어쓰지 않고 새 날짜 섹션을 계속 추가한다
├── IDEAS.md             (선택) 아이디어 백로그
├── spec/                PRD·요구사항 등 기획·설계 문서
├── data/ 또는 guide/    콘텐츠 데이터 또는 학습·참고 자료
└── report/              날짜별 점검 리포트 (`YYYY-MM-DD-주제.md`)
```

같은 성격 문서는 같은 폴더로 모으고, 캐시/디버그 산출물(`.DS_Store`, 브라우저 자동화 로그 등)은 삭제 + `.gitignore`. 서로 참조만 하고 내용이 겹치지 않는 문서는(중복이 아니라 분업이면) 함부로 합치거나 지우지 않는다.

## 2. 공개 문서 사이트 (`<project>-docs`)

소스코드 저장소와 **별도의 public 저장소**로 분리한다 (`<project-slug>-docs`, 소스코드가 private이어도 문서 사이트는 public).

```
<project>-docs/
├── README.md            프로젝트 개요, 핵심 기능, 문서 링크 (자유 형식)
├── index.html            doc/index.html로 즉시 리다이렉트
└── doc/
    ├── index.html         필터형 문서 허브 (검색창 + 섹션별 카드)
    ├── viewer.html         마크다운 뷰어 — 모든 프로젝트에서 100% 동일한 파일을 그대로 복사(외부 라이브러리 없는 자체 파서, 다크/라이트 토글, 자동 목차)
    ├── STATUS.md / DEVLOG.md / IDEAS.md
    ├── spec/, data|guide/, report/
```

**배포 절차**
1. `gh repo create peter-cho-70/<slug>-docs --public --source=. --remote=origin --push` (기본 브랜치 `main`)
2. `gh api -X POST repos/peter-cho-70/<slug>-docs/pages -f "source[branch]=main" -f "source[path]=/"`
3. `gh api repos/peter-cho-70/<slug>-docs/pages/builds/latest`로 `"status":"built"` 확인 + 실제 URL curl 검증
4. 이 허브([index.html](../index.html))의 `.grid`에 카드 추가 — `data-check`는 그 프로젝트의 `doc/STATUS.md`를 가리키게 해서 "최종 갱신" 날짜가 자동으로(Last-Modified 실측값) 뜨게 한다. 날짜를 손으로 적지 않는다.

## 3. 갱신 규칙

| 문서 | 갱신 방식 |
|---|---|
| STATUS.md | append 아님 — 매번 "지금 시점" 기준으로 전체를 덮어쓴다 |
| DEVLOG.md | 의미 있는 작업 단위(기능 추가/버그 수정/막힌 이슈)가 끝날 때마다 날짜 섹션 append |
| report/ | 점검·리뷰를 할 때마다 `YYYY-MM-DD-주제.md`로 새 파일 추가, 기존 파일은 수정하지 않음 |

버그를 고쳤으면 DEVLOG에 "무엇이 문제였고 → 어떻게 고쳤는지"를 남긴다. 원인이 아직 안 밝혀진 이슈는 STATUS.md의 "알려진 이슈"에 넣는다.

## 4. 자동 적용 범위

"문서 정리해줘 / 진행상황 기록해줘 / 문서화해줘" 라고 요청하면, 위 전체(로컬 정리 → STATUS/DEVLOG 작성 → 공개 사이트 발행 → 이 허브 등록)를 **매번 다시 확인받지 않고** 수행한다. 단, 이 사전 승인은 **문서(마크다운·정적 HTML) 전용 public 저장소**에만 해당하며, 소스코드 저장소의 공개 전환·소스코드 자체 푸시·배포/프로덕션 설정 변경에는 적용되지 않는다. 개인정보(식별번호 등)·API 키·계약금액 같은 민감 정보는 공개 문서 사이트에 올리지 않는다.

## 5. 참고 구현

- [my-education-docs](https://peter-cho-70.github.io/my-education-docs/) — 이 표준을 처음 적용한 사례
- [stockdashboard-docs](https://peter-cho-70.github.io/stockdashboard-docs/) — STATUS/DEVLOG/IDEAS/report 폴더 구조가 가장 성숙하게 갖춰진 사례
