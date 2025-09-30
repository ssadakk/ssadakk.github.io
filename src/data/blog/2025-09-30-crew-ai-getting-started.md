---
author: HM Joo
pubDatetime: 2025-09-30T03:00:00.000Z
title: Crew AI 입문 - 간단한 에이전트 워크플로우 만들기
slug: crew-ai-getting-started
featured: false
draft: false
tags:
  - CrewAI
  - Agents
  - Python
  - LLM
description: Crew AI로 리서처와 라이터 에이전트를 구성해 순차 실행하고 결과를 Markdown 파일로 저장하는 최소 예제.
---

Crew AI로 "리서처 → 라이터" 두 개의 에이전트를 구성해 순차(Sequential)로 실행하고, 결과를 Markdown 파일로 저장하는 최소 예제. Python 가상환경 설정부터 의존성 설치, 환경변수 구성, 오프라인 모드, 결과 저장까지 정리.

## Table of contents

## 프로젝트 구조

```
.
├── crew_min.py        # Crew 구성/실행 스크립트
├── check_env.py       # 환경변수 점검 스크립트
├── .env               # 로컬 전용, Git에 커밋 금지
└── outputs/           # 보고서/포스팅 등 md 결과물 저장 위치
```

CrewAI의 `Agent`, `Task`, `Crew`를 사용하며, 기본 LLM 모델은 `gpt-4o-mini`로 설정. `--topic`, `--output`, `--offline` CLI 옵션으로 주제와 저장 경로, 오프라인 모드 사용을 제어.

## 코드 구조와 핵심 구성요소

### 환경 변수 로드

키 이름 혼용 대응(OPENAI_API_KEY와 OPEN_API_KEY):

```python
from dotenv import load_dotenv
load_dotenv(dotenv_path=ENV_PATH, override=True)
openai_key = os.getenv("OPENAI_API_KEY") or os.getenv("OPEN_API_KEY")
if not os.getenv("OPENAI_API_KEY") and openai_key:
    os.environ["OPENAI_API_KEY"] = openai_key
```

### LLM 및 선택적 도구

```python
from crewai import LLM
llm = LLM(model="gpt-4o-mini")
try:
    from crewai_tools import SerperDevTool
    search_tool = SerperDevTool()  # SERPER_API_KEY 필요
except Exception:
    search_tool = None  # 키/패키지 없으면 비활성화
```

### 에이전트 정의

역할/목표/툴/LLM 지정:

```python
from crewai import Agent
researcher = Agent(
    role="Researcher",
    goal="주어진 토픽을 최신 자료로 조사",
    backstory="정확한 정보 수집 담당",
    tools=[t for t in [search_tool] if t is not None],
    llm=llm,
)
writer = Agent(
    role="Writer",
    goal="리서치 결과를 한국어 요약 보고서로 작성",
    backstory="기술 글쓰기에 능숙",
    llm=llm,
)
```

### 태스크 정의

설명/기대 출력/할당 에이전트:

```python
from crewai import Task
research_task = Task(
    description="'{topic}' 동향 3~5가지 + 출처 URL",
    expected_output="불릿 포인트 + References",
    agent=researcher,
)
write_task = Task(
    description="TL;DR + 본문 요약(5~8문장)",
    expected_output="간결한 한국어 보고서",
    agent=writer,
    context=[research_task],
)
```

### 크루 정의 및 실행

순차 처리:

```python
from crewai import Crew, Process
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential,
    verbose=True,
)
result = crew.kickoff(inputs={"topic": args.topic})
```

### CLI 옵션 및 결과 저장

```python
import argparse, os
parser = argparse.ArgumentParser()
parser.add_argument("--topic", default="Android 15 SDK 주요 변화")
parser.add_argument("--output", default="")
parser.add_argument("--offline", action="store_true")
args = parser.parse_args()
if args.offline:
    os.environ["CREW_OFFLINE"] = "1"
content = "- TL;DR: 샘플 요약\n- 본문: 학습용 더미 결과" if os.getenv("CREW_OFFLINE") else str(result)
out_path = args.output or f"outputs/report-YYYYMMDD-HHMMSS.md"
with open(out_path, "w", encoding="utf-8") as f:
    f.write(content)
```

## 환경 준비 및 설치

```bash
# Python 3.10+ 권장
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

pip install -U pip
pip install crewai crewai-tools python-dotenv
```

## 환경변수 설정

루트에 `.env` 파일 생성. 실제 키 대신 자리표시자 사용:

```dotenv
OPENAI_API_KEY=your_openai_api_key_here
# 선택: 검색 툴 사용 시 필요
SERPER_API_KEY=your_serper_api_key_here
```

확인:

```bash
python check_env.py
```

## 실행 및 결과 저장

### 온라인 실행

결과는 콘솔 출력되고 Markdown 파일로 저장. 저장 경로 미지정 시 `outputs/report-YYYYMMDD-HHMMSS.md`로 생성:

```bash
# 온라인 실행 (실제 LLM 호출)
python crew_min.py --topic "Android 15 SDK 주요 변화"

# 저장 경로 지정
python crew_min.py --topic "테스트 토픽" --output outputs/my-report.md
```

### 오프라인 모드

API 사용량 제한/결제 문제로 호출이 어려울 때 사용. LLM 호출 없이 학습/데모용 더미 결과를 Markdown으로 저장:

```bash
# CLI 플래그로 오프라인 모드
python crew_min.py --offline --topic "오프라인 테스트" --output outputs/offline-report.md

# 또는 환경변수로 지정
CREW_OFFLINE=1 python crew_min.py --topic "오프라인 테스트"
```

### 생성되는 Markdown 기본 골격

```md
# Report
- Topic: <입력 토픽>
- Model: gpt-4o-mini
- Timestamp: 2025-09-30 12:34:56

## Output
<에이전트가 작성한 내용>
```

## 문제 해결

- **ModuleNotFoundError**(예: dotenv): 가상환경 활성화 후 `pip install crewai crewai-tools python-dotenv` 재설치
- **RateLimit/Quota/Billing 오류**: OpenAI 사용량/결제 상태 확인, 임시로 `--offline` 사용
- **키 불일치**: 이 예제는 `OPENAI_API_KEY`와 `OPEN_API_KEY`를 모두 인식하지만, 표준은 `OPENAI_API_KEY` 사용을 권장

## 마무리

이 예제는 최소 구성으로 구성된 Crew AI 파이프라인. 주제/프롬프트를 확장하고, 도구(tools)와 테스트를 추가해 더 풍부한 에이전트 협업 흐름을 만들 수 있다. 작성된 `.md` 결과 파일은 GitHub Pages 블로그 리포지토리로 옮겨 게시 가능.

**주의**: 민감한 키 값은 절대 공개 저장소에 올리지 말 것.