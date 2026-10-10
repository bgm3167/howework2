# 콘텐츠 생성 멀티에이전트

## 무엇을 해결하나요?

분석 보고서만으로 다음 달 콘텐츠를 쓰면 최신 사례와 출처가 빠지기 쉽습니다. 이 에이전트는 [9월 유입 하락 분석 보고서](starter/data/analysis_report.md)와 사용자 요청을 받아 자료를 조사하고, **출처가 달린 블로그 글 1편과 쇼츠 대본 1편**을 만듭니다. 초안을 자동 검수해 근거가 부족하면 조사나 작성을 다시 요청합니다.

## 구조

`planner → researcher × N → synthesizer → supervisor → writer → reviewer → publisher` 순서로 실행합니다. `planner`가 만든 작업은 `Send`로 병렬 조사합니다. `supervisor`는 핵심 자료가 부족하면 `planner`로, `reviewer`가 반려하면 `supervisor`를 거쳐 조사나 작성으로 되돌립니다. 작성자는 핸드오프 도구로 검수 또는 추가 조사를 요청합니다.

| 노드·에이전트 | 하는 일 | 쓰는 도구·판정 |
|---|---|---|
| `planner` | 보고서와 요청을 읽고 자료원별 조사 작업을 계획 | GPT 구조화 출력 |
| `researcher` | 작업마다 별도 조사원을 실행하고 사실과 출처 URL을 수집 | 작업의 자료원에 해당하는 MCP 도구만 사용: 네이버 검색, YouTube 검색·자막, Tavily 웹 검색 |
| `synthesizer` | 조사 결과를 공통 흐름, 콘텐츠 기회, 부족한 사실이 담긴 브리프로 종합 | GPT 구조화 출력 |
| `supervisor` | 보충 조사 여부와 반려 후 다음 담당을 결정하고 반복 횟수를 제한 | Jev 확률 판정, GPT 구조화 출력 |
| `writer` | 브리프와 출처를 바탕으로 블로그 글과 쇼츠 대본을 저장하고 다음 담당에게 넘김 | 파일 쓰기 MCP, `request_review`, `request_research` |
| `reviewer` | 출처 충실도, 독자 가치, 자료와의 모순을 판정하고 반려 시 수정 의견을 작성 | Jev 점수·확률 판정, GPT 피드백 |
| `publisher` | 통과한 글의 커버 이미지를 만들고 발행 상태와 출처를 기록 | OpenAI 이미지 API, `sources.json` 저장 |

코드는 [`starter/app/`](starter/app/)에 있습니다. 노드와 분배 경로는 `graph/`, 프롬프트·출력 형식·판정 기준은 `core/`, MCP 연결과 핸드오프는 `tools/`에 있습니다. 자세한 도구 이름과 분기 규칙은 [실습 설명](starter/README.md)을 참고하세요.

## 실행 방법

Python 3.12, `uv`, Node.js 22 이상(`npx` 포함)이 필요합니다. 저장소 최상위 폴더에서 의존성을 설치하고, `starter/.env.example`을 `starter/.env`로 복사해 필요한 API 키를 **로컬에서만** 입력합니다. `.env`는 Git 추적 대상에서 제외했습니다.

```powershell
git clone https://github.com/bgm3167/howework2.git
cd howework2
uv sync
Copy-Item starter/.env.example starter/.env
# starter/.env에 OPENAI_API_KEY, TYPESAFE_API_KEY, TAVILY_API_KEY,
# NAVER_CLIENT_ID, NAVER_CLIENT_SECRET, YOUTUBE_API_KEY를 입력
cd starter
uv run -m app.main "보고서 기준으로 다음 달 블로그 글 1편과 쇼츠 대본 1편 만들어 줘"
```

macOS·Linux에서는 복사 명령을 `cp starter/.env.example starter/.env`로 바꾸면 됩니다. 실행 결과는 `starter/output/<실행 시각>/`에 저장됩니다.

## 실행 결과

위 요청으로 실행한 [예시 결과 폴더](starter/output/20261010_150129/)에는 다음 파일이 생성됐습니다.

| 파일 | 결과 |
|---|---|
| [`blog.md`](starter/output/20261010_150129/blog.md) | 출처 링크가 달린 블로그 글 |
| [`shorts_script.md`](starter/output/20261010_150129/shorts_script.md) | 쇼츠 대본 |
| [`sources.json`](starter/output/20261010_150129/sources.json) | 발행 상태 `발행`, 검수 통과 `true`, 출처 34개, 재작업 0회 |
| [`cover.png`](starter/output/20261010_150129/cover.png) | 검수 통과 후 생성된 커버 이미지 |

실제 `.env`와 API 키는 저장소에 포함하지 않았습니다.
