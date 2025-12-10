# 🛡️ LogSentinel-AI: AI 기반 이상 탐지 로그 모니터링 시스템

> **Nginx 서버 로그를 분석하여 머신러닝으로 비정상 패턴을 감지하는 지능형 모니터링 시스템**  
> "단순한 텍스트 로그를 데이터로 변환하고, AI를 통해 숨겨진 위협을 찾아냅니다."

![Badge](https://img.shields.io/badge/Python-FastAPI-blue?logo=python)
![Badge](https://img.shields.io/badge/Target-Nginx-009639?logo=nginx)
![Badge](https://img.shields.io/badge/ML-Scikit--learn-orange?logo=scikit-learn)
![Badge](https://img.shields.io/badge/Data-CSV%2FJSON-lightgrey)

---

## 📖 프로젝트 개요 (Overview)

**LogSentinel-AI**는 웹 서버(Nginx)에서 발생하는 방대한 로그 데이터를 수집하여, **머신러닝 모델이 이해할 수 있는 형태(CSV/JSON)로 변환**하고, 학습된 AI 모델을 통해 실시간으로 이상 징후를 탐지하는 시스템입니다.

기존의 룰 기반(Rule-based) 탐지가 잡아내지 못하는 미세한 패턴 변화나 새로운 형태의 공격 시도를 **비지도 학습(Unsupervised Learning)** 알고리즘을 통해 식별하는 것을 목표로 합니다.

---

## 📊 데이터 처리 및 분석 흐름 (Data Processing Flow)

이 프로젝트의 핵심인 로그 데이터가 시스템에서 수집되어 AI 예측 결과로 나오기까지의 과정을 도식화하였습니다.

```mermaid
graph TD
    subgraph Local Environment
        A[Nginx Server] -->|Write Logs| B[access.log / error.log]
    end

    subgraph 1. Preprocessing (Python)
        B -->|Read File| C(Log Parser Script)
        C -->|Regex Parsing| D{Data Conversion}
        D -->|Training Data| E[History.csv]
        D -->|Real-time Data| F[Stream.json]
    end

    subgraph 2. AI Analysis
        E -->|Fit / Train| G[ML Model Engine]
        F -->|Input| G
        G -->|Predict| H[Anomaly Score Calculation]
    end

    subgraph 3. Action
        H -->|Score > Threshold| I[🚨 Alert (Anomaly Detected)]
        H -->|Score < Threshold| J[✅ Normal Pattern]
    end
```

### 단계별 상세 프로세스

1.  **로그 수집 (Collection)**: 로컬 시스템의 `/var/log/nginx/` 경로에 쌓이는 `access.log` 및 `error.log` 파일을 타겟으로 합니다.
2.  **데이터 변환 (Transformation)**:
    *   Python 스크립트가 Raw Text 형태의 로그를 읽어 정규표현식(Regex)을 통해 파싱합니다.
    *   **학습용**: 과거 로그를 대량으로 읽어 `CSV` 파일로 저장합니다. (Feature: IP, Timestamp, Method, URL, Status Code, Body Bytes 등)
    *   **탐지용**: 실시간으로 생성되는 로그 라인을 `JSON` 객체로 변환하여 모델에 주입합니다.
3.  **AI 예측 (Prediction)**:
    *   변환된 데이터를 `Isolation Forest` 또는 `LSTM` 모델에 입력합니다.
    *   모델은 해당 로그가 정상 패턴 분포에서 얼마나 벗어났는지(Anomaly Score)를 예측합니다.

---

## 🏗️ 전체 시스템 아키텍처 (System Architecture)

```mermaid
graph LR
    A[Log Parser (Python)] -->|JSON Stream| B(Kafka / Queue)
    B --> C{AI Engine Container}
    C -->|Store Logs| D[(Elasticsearch)]
    C -->|Detect and Metadata| E[(PostgreSQL)]
    C -->|Webhook| F[Slack Alarm]
    G[FastAPI Server] -->|Read| D
    G -->|Read| E
    G -->|WebSocket| H[Dashboard (React)]
```

---

## 🛠️ 기술 스택 (Tech Stack)

### Core
*   **Python 3.10+**: 로그 파싱, 데이터 전처리, ML 모델링
*   **FastAPI**: 백엔드 API 및 실시간 데이터 전송

### Data Source & Processing
*   **Source**: Nginx Access/Error Logs
*   **Preprocessing**: Python `re` (Regex), `pandas` (CSV 변환)
*   **Streaming**: Kafka (or Redis Streams)
*   **Database**:
    *   **Elasticsearch**: 대용량 로그 저장 및 전문 검색
    *   **PostgreSQL**: 알림 설정, 유저 관리, 탐지 통계 데이터

### AI / ML
*   **Scikit-learn**: Isolation Forest (이상 탐지 모델)
*   **Pandas/NumPy**: 데이터 벡터화 및 전처리

### Frontend
*   **Framework**: React (Vite)
*   **State Management**: React Query / Zustand
*   **Visualization**: D3.js (커스텀 차트) or Recharts

---

## ✨ 주요 기능 (Key Features)

1. **실시간 로그 스트리밍 처리**
   - 초당 수백 건의 로그를 지연 없이 처리하는 파이프라인 구축
2. **AI 이상 탐지 (Anomaly Detection)**
   - 룰 기반(Rule-based)이 아닌 학습 기반의 이상 징후 포착
   - 위험도 등급 자동 산정 (Critical / High / Medium / Low)
3. **실시간 대시보드**
   - D3.js를 활용한 시계열 이상치 시각화
   - WebSocket을 통한 데이터 자동 갱신
4. **스마트 알림 시스템**
   - 위험도 임계치(Threshold) 초과 시 Slack/Discord 알림 발송
   - 이슈 티켓 자동 생성

---

## 🚀 실행 가이드 (How to Run)

이 프로젝트는 여러 인프라(Kafka, ES, Postgres)가 필요하므로 `Docker Compose`를 사용하여 한 번에 실행하는 것을 권장합니다.

### 1. Nginx 로그 파서 실행 및 데이터 변환
로컬의 Nginx 로그를 읽어 학습 가능한 CSV 데이터로 변환하는 단계입니다.

```bash
# 로그 파서 스크립트 실행 (CSV 변환 모드)
python scripts/log_parser.py \
    --input /var/log/nginx/access.log \
    --output ./data/training_data.csv \
    --format csv
```

### 2. AI 모델 학습
변환된 CSV 데이터를 사용하여 정상 패턴을 학습시킵니다.

```bash
# 모델 학습 스크립트 실행
python src/ml/train_model.py --data ./data/training_data.csv
# 학습 완료 후 model.pkl 파일 생성됨
```

### 3. 실시간 탐지 시스템 시작
실시간으로 로그를 테일링(Tailing)하며 JSON으로 변환 후, 이상 여부를 판별합니다.

```bash
# 메인 시스템 실행
docker-compose up -d  # DB 등 인프라 실행
uvicorn app.main:app --reload # API 서버 실행

# 별도 터미널에서 실시간 감지 에이전트 실행
python src/agent/realtime_detector.py --source /var/log/nginx/access.log
```

---

## 📂 디렉토리 구조 (Directory Structure)

```
log-sentinel-ai/
├── backend/
│   ├── app/
│   │   ├── api/          # FastAPI 라우터
│   │   ├── core/         # 설정 및 보안
│   │   ├── db/           # ES, PG 연결 로직
│   │   ├── ml/           # 모델 학습 및 추론 엔진
│   │   └── services/     # 비즈니스 로직 (Kafka Consumer 등)
│   └── main.py
├── frontend/
│   ├── src/
│   │   ├── components/   # 차트 및 UI 컴포넌트
│   │   └── hooks/        # WebSocket 훅
├── infra/                # Docker Compose 설정 파일
├── scripts/              # 로그 파서 및 생성기
└── README.md
```

---

## 🤝 기여하기 (Contributing)
이 프로젝트는 교육용 오픈소스 프로젝트입니다. PR과 Issue 등록은 언제나 환영합니다!

## 📝 License
MIT License
