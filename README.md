# :triangular_flag_on_post: Dataplatform_study

> **공공·공개 데이터를 활용해 엔드투엔드(End-to-End) 데이터 플랫폼과 데이터 파이프라인 구축을 학습하는 저장소입니다.**
> 데이터 수집, 정제, 적재, 분석, 시각화까지 전 과정을 직접 구현하며 데이터 엔지니어링 역량을 기르는 것을 목표로 합니다.

---

## :fire: 프로젝트 개요
공공데이터포털(data.go.kr), 거래소 공개 시세 API 등 누구나 쓸 수 있는 데이터를 연동해 실전 데이터 플랫폼의 핵심 요소를 구현합니다.
**배치 파이프라인과 실시간 스트리밍 파이프라인**을 모두 다루며, 장애 복구, 중복 없는 적재, 데이터 품질 관리에 초점을 맞춥니다.

---

## :file_folder: Projects

| # | 프로젝트 | 유형 | 데이터 | 상태 |
|---|---|---|---|---|
| 1 | 실시간 코인 시세 파이프라인 & 대시보드 | 스트리밍 | 업비트 공개 시세 API | :construction: 진행 중 |
| 2 | 공공 데이터 배치 파이프라인 | 배치 | data.go.kr | :memo: 계획 |

---

## 1. 실시간 코인 시세 파이프라인 & 대시보드

### 목표
업비트 실시간 체결 데이터를 Kafka로 수집하고, 스트림에서 직접 캔들을 집계해 DB에 적재합니다.
적재된 데이터로 시장 변화(거래량 급증, 변동폭 확대 등)를 탐지하고, **근거·주의 요인·해제 조건**을 함께 보여주는 대시보드를 만듭니다.

> 자동 주문은 하지 않습니다. 신호는 수익을 보장하지 않으며, 검증 전 승률·확률은 표시하지 않습니다.

### Architecture
```text
 [Upbit WebSocket] ──> Producer ──> [Kafka] ──┬──> 캔들 집계 Consumer ──> [DB]
                                              └──> 신호 분석 Consumer ──> [DB]
 [Upbit REST] ──> 빈 구간 복구 작업 ───────────────────────────────────> [DB]
                                                                  [DB] ──> [Dashboard]
```

### 핵심 엔지니어링 포인트
- **스트림 집계**: 체결 데이터를 5분·1시간 구간으로 직접 묶어 캔들을 만듭니다. 늦게 도착한 데이터도 처리합니다.
- **멱등 적재**: `(종목, 봉 단위, 시각)`을 고유 키로 두고 덮어쓰기 저장합니다. 같은 데이터를 다시 처리해도 결과가 같습니다.
- **장애 복구**: Consumer는 재시작하면 Kafka에서 마지막으로 읽은 위치부터 이어서 처리합니다. 수집이 끊긴 구간은 REST API로 다시 채웁니다.
- **요청 제한 관리**: 업비트 API 제한(REST 초당 10회, 요청당 최대 200개)을 고려해, 받은 데이터를 쌓아 두고 새 데이터만 요청합니다.
- **모니터링**: Consumer 지연(lag)과 마지막 수신 시각을 대시보드에 표시합니다.

### 진행 상황
- [x] 요구사항·분석 규칙 정리 (대상: BTC·ETH·XRP, 5분봉 신호 + 1시간봉 추세 필터)
- [x] Colab 프로토타입: REST 수집, 완료 봉 분리, 거래량 비율, 요약표, Gradio 대시보드, SQLite 멱등 저장
- [ ] Docker로 Kafka 환경 구성
- [ ] WebSocket Producer
- [ ] 캔들 집계 Consumer
- [ ] 빈 구간 복구 작업
- [ ] 신호 분석 Consumer
- [ ] DB 기반 대시보드
- [ ] 모니터링

### 배운 점
- 업비트 캔들은 최신순으로 오고, 맨 위 봉은 아직 진행 중입니다. 분석에는 **완료된 봉만** 써야 합니다.
- 거래량 급증 기준 "직전 20봉 중앙값의 3배"는 BTC 5분봉에서 16시간 동안 23번 발생해 너무 민감했습니다. 충분한 데이터를 쌓은 뒤 기준을 다시 정하기로 했고, 이것이 데이터를 계속 저장하는 구조가 필요한 이유이기도 합니다.

---

## :wrench: Tech Stack

| 영역 | 사용 기술 | 비고 |
|---|---|---|
| Language | Python 3.x | |
| Data Ingestion | requests (REST), websockets (WebSocket) | |
| Message Queue | Apache Kafka | 실시간 프로젝트 |
| Data Processing | Pandas | |
| Storage / DB | SQLite → PostgreSQL | 프로토타입은 SQLite |
| Visualization | Gradio, Plotly | |
| Environment | Docker, Google Colab | |
| Workflow Management | 미정 | 배치 프로젝트에서 도입 예정 |

---

## :pushpin: 공통 데이터 흐름

```text
 [공공 데이터 API / 공개 시세 API / CSV]
         │
         ▼ (Ingestion: requests / WebSocket → Kafka)
  [Raw Data Storage] (Kafka topic / Local)
         │
         ▼ (Processing: Pandas / 스트림 집계)
  [Staging & Mart DB] (SQLite → PostgreSQL)
         │
         ▼ (Service / Visualization)
   [Dashboard] (Gradio / Plotly)
```
