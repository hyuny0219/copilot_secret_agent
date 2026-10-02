# Declassify Station 전시 프로그램 요구사항 명세 (Draft v0.1)

> 대상: `AX_Day_2026_Copilot_Secret_Agent_HQ_Proposal.md` 5-2절의 **Declassify Station**(START 한 번 → 90초 자동 작전 재현)을 구현할 부스용 프로그램
> 목적: 구현 전에 범위, 구조, 운영 요건을 확정한다.

---

## 1. 범위

| 구분 | 포함 | 제외 |
|---|---|---|
| **P1 Station App** | 사건 선택 → START → 90초 자동 시퀀스 → 종료 → 대기 화면 | 실시간 Copilot 호출, 로그인, 개인정보 입력 |
| **P2 Mirror / Attract** | 상단 55" 사이니지: 지금 재생 중인 Station 확대 미러, 유휴 시 Attract 루프 | 원격 제어 |
| **P3 Live Counter** | "오늘 기밀 해제된 사건 수"와 "의뢰 접수 수"를 Intelligence Map LED에 표시 | 외부 서버 전송 |
| **P4 Content Pack** | 사례 4건(CASE #001~#004)의 데이터 파일과 에셋 | 사례 내용 자체(현업 수집) |
| **P5 Operator Panel** | 스태프용 숨김 메뉴: 재시작, 카운터 리셋, 상태 확인 | – |

## 2. 핵심 원칙
1. **완전 오프라인**: 네트워크가 끊겨도 100% 동작한다. 모든 에셋은 로컬에 둔다.
2. **입력 최소화**: 방문객 입력은 **사건 버튼 4개 + START 1개** 가 전부. 터치 없음.
3. **콘텐츠와 엔진 분리**: 사례가 바뀌어도 코드를 고치지 않는다. **사례 = JSON 1개 + 에셋 폴더**.
4. **무음 이해**: 소리가 없어도 자막과 화면만으로 100% 이해된다.
5. **실패해도 보기 좋게**: 어떤 오류가 나도 검은 화면이나 브라우저 오류창을 띄우지 않고 Attract 루프로 복귀한다.

## 3. 하드웨어 · 실행 환경 (가정)

| 항목 | 사양 |
|---|---|
| 디스플레이 | 43" 세로(2160×3840 또는 1080×1920) × 4, 상단 미러 55" 가로 × 1~2 |
| PC | Station당 미니 PC 1대 (또는 1대가 2화면 구동) + 예비 1대 |
| 입력 | USB 아케이드 인코더 → 키보드 입력으로 매핑 (`1`~`4` = 사건 선택, `Space`/`Enter` = START) |
| 런타임 | **웹앱(HTML/CSS/JS) + Chromium 키오스크 모드** (`--kiosk --autoplay-policy=no-user-gesture-required`) |
| 부팅 | 전원 ON → OS 자동 로그인 → 앱 자동 실행. 사람 개입 0 |

## 4. 화면 상태 머신 (Station)

```
          (1~4 key)            (START)             (90s end)          (10s)
 [IDLE] ───────────▶ [ARMED] ──────────▶ [PLAYING] ──────────▶ [OUTRO] ───────▶ [IDLE]
   ▲                   │ 15s no input                │ error
   └───────────────────┘                             └──────────▶ [IDLE] (+log)
```

| 상태 | 화면 | 입력 처리 |
|---|---|---|
| IDLE | 잠긴 사건 폴더 4개 회전, "SELECT CASE → PRESS START". 30초마다 Redaction 맛보기 | `1`~`4` → ARMED. START만 누르면 **마지막 선택 사건 또는 랜덤** 재생 |
| ARMED | 선택한 사건 표지, START 버튼 깜빡임 안내 | 다른 숫자 → 사건 변경. START → PLAYING. 15초 무입력 → IDLE |
| PLAYING | 90초 시퀀스(5절) | **모든 입력 무시** (연타 방지). 운영자 키만 허용 |
| OUTRO | "MISSION COMPLETE · 포켓 카드를 Debriefing Desk에서 받으세요" | 10초 후 IDLE. 숫자 입력 시 바로 ARMED |

## 5. 90초 시퀀스 엔진

### 5-1. 고정 레이아웃 (세로 화면)
`상태바 → 사건 헤더 → ① 무대(25%) → ② 증거 트레이(15%) → ③ 작전 콘솔(25%) → ④ 결과물(25%) → ⑤ 진행 바`

### 5-2. 타임라인 (CASE #001 기준, 사례마다 JSON으로 조정)

| 구간 | 단계 | 엔진이 하는 일 |
|---|---|---|
| 00:00–00:05 | AUTH | 스캔 라인, "CLEARANCE UPGRADED" |
| 00:05–00:18 | ① INCIDENT | 장면 이미지와 문제 수치 카운트업 |
| 00:18–00:30 | ② EVIDENCE | 증거 카드 E-01~E-03 순차 드롭 + 분량 태그 |
| 00:30–00:50 | ③ TRIGGER + PROMPT | 트리거 표시 → 검게 가려진 프롬프트를 **타자 효과로 해제**, 키워드 하이라이트 |
| 00:50–01:05 | ③ PROCESSING | 증거 하이라이트 → 결과물 칸으로 **연결선 애니메이션** (출처 매핑) |
| 01:05–01:15 | ③ HUMAN CHECK | 빨간 펜 수정 표시 → "APPROVED BY HANDLER" 도장 |
| 01:15–01:25 | ④ RECOVERED | 결과물 확대 표시 |
| 01:25–01:30 | ⑤ RESULT | Before/After 막대 축소 애니메이션 → OUTRO |

### 5-3. 사례 데이터 스키마 (초안)

```json
{
  "id": "CASE-001",
  "agent": "BLACK",
  "color": "#111111",
  "title": "THE LOST ACTION ITEMS",
  "titleKo": "사라진 액션아이템 사건",
  "durationSec": 90,
  "incident": {
    "image": "assets/case001/incident.png",
    "lines": ["주간 영업회의 90분", "회의록 작성 40분", "액션 누락 30%"]
  },
  "evidence": [
    { "id": "E-01", "label": "Teams 트랜스크립트", "volume": "12p", "image": "assets/case001/e01.png" },
    { "id": "E-02", "label": "공유 PPT", "volume": "8 slides" },
    { "id": "E-03", "label": "이전 회의 메일", "volume": "6 mails" }
  ],
  "operation": {
    "trigger": "Teams 회의 종료 → Copilot 패널 열기",
    "prompt": "이번 회의의 결정사항, 담당자별 액션아이템, 기한, 미결 이슈를 표로 정리해줘. ...",
    "highlight": ["결정사항", "액션아이템", "기한", "미결 이슈"],
    "mappings": [
      { "from": "E-01", "ref": "14:32", "to": "decision-1" },
      { "from": "E-02", "ref": "slide 5", "to": "action-3" }
    ],
    "humanCheck": ["기한 1건 수정", "담당자 1명 추가"]
  },
  "recovered": { "image": "assets/case001/minutes.png", "anchors": ["decision-1", "action-3"] },
  "result": {
    "before": { "label": "회의록 작성", "value": 40, "unit": "분" },
    "after":  { "label": "회의록 작성", "value": 5,  "unit": "분" },
    "extra": "액션 누락 30% → 5%",
    "quote": "회의 끝나고 바로 공유됩니다 — 영업기획팀"
  },
  "timeline": [
    { "phase": "auth", "start": 0 },
    { "phase": "incident", "start": 5 },
    { "phase": "evidence", "start": 18 },
    { "phase": "prompt", "start": 30 },
    { "phase": "processing", "start": 50 },
    { "phase": "humanCheck", "start": 65 },
    { "phase": "recovered", "start": 75 },
    { "phase": "result", "start": 85 }
  ]
}
```

## 6. 미러 · 카운터 연동 (P2/P3)
- Station 4대와 미러/카운터 PC를 **부스 내부 로컬 네트워크(공유기 1대, 인터넷 불필요)** 로 연결한다.
- Station은 상태가 바뀔 때마다 `{stationId, state, caseId, t}` 이벤트를 로컬 브로드캐스트한다 (WebSocket 서버 1개를 미러 PC에 둠).
- 미러 화면은 재생 중인 Station 중 **가장 최근에 시작한 것** 을 같은 엔진으로 동시 렌더링한다. 영상 스트리밍이 아니라 이벤트 동기화 방식이다.
- 로컬 네트워크가 끊기면 미러는 Attract 루프로, Station은 단독 모드로 계속 동작한다.
- 카운터: PLAYING에 진입할 때마다 +1. 날짜별로 로컬 파일에 저장해서 재부팅해도 유지한다.

## 7. 운영자 기능 (P5)
| 키 조합 | 기능 |
|---|---|
| `Ctrl+Shift+O` | 운영 패널 (상태, 재생 수, 오류 로그, 버전) |
| `Ctrl+Shift+R` | 앱 재시작 |
| `Ctrl+Shift+C` | 오늘 카운터 리셋 (확인 2회) |
| 숨김 핫코너 | 키보드가 없을 때를 대비해 START를 10초 길게 누르면 운영 패널 |

## 8. 비기능 요건
| 항목 | 기준 |
|---|---|
| 연속 운영 | 10시간 무중단, 메모리 누수 없음 (1,000회 연속 재생 테스트) |
| 성능 | 4K 세로에서 60fps 목표, 최저 30fps |
| 기동 시간 | 전원 ON → IDLE 화면까지 60초 이내 |
| 복구 | 앱 크래시 시 5초 안에 자동 재실행 (watchdog) |
| 접근성 | 최소 글자 크기 1.5m 거리에서 읽힘(세로 4K 기준 48px 이상), 색만으로 정보 구분 금지 |
| 보안 | 실데이터 미포함. 모든 에셋은 가상 데이터. 로그에 개인정보 없음 |
| 다국어 | 국문 기본, 영문 자막 토글 (JSON 필드) |

## 9. 산출물 · 일정
| 순서 | 산출물 |
|---|---|
| 1 | 엔진 프로토타입 + CASE #001 1건 (실제 하드웨어 리허설) |
| 2 | 사례 JSON 스키마 확정 → 나머지 3건 콘텐츠 투입 |
| 3 | 미러 · 카운터 연동 |
| 4 | 연속 운영 테스트, 키오스크 설치 스크립트 |

## 10. 미결 사항 (검토 요청)
1. 웹앱 + Chromium 키오스크 vs 사전 렌더링 영상(MP4) 재생 방식: 어느 쪽이 현장 리스크와 제작비 면에서 나은가?
2. 미러를 이벤트 동기화로 할지, HDMI 분배기로 단순화할지
3. 사례 4건 × 90초가 적정한지 (60초 축약판 필요 여부)
4. 카운터와 의뢰서 센서(투입구)를 같은 시스템에 묶을지
