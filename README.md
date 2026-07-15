# 📊 Dataplatform_study

> **공공 데이터를 활용하여 엔드투엔드(End-to-End) 데이터 플랫폼 및 데이터 파이프라인 구축을 학습하는 저장소입니다.**  
> 데이터 수집, 정제, 적재, 분석 및 시각화까지의 전 과정을 구현하며 데이터 엔지니어링 역량을 기르는 것을 목표로 합니다.

---

## 📌 프로젝트 개요
공공 데이터 포털(data.go.kr) 등에서 제공하는 다양한 공공 데이터를 연동하여 실전 데이터 플랫폼의 핵심 요소를 모방하고 구현합니다. 대용량 데이터 처리, 배치/실시간 파이프라인 설계, 데이터 품질 관리 등에 초점을 맞추어 학습을 진행하고 있습니다.

---

## 🛠 Tech Stack (기술 스택)

* **Language:** Python 3.x
* **Data Ingestion:** 내용 작성하면서 이용도구 작성 예정 
* **Data Processing:** 내용 작성하면서 이용도구 작성 예정 
* **Storage / DB:** 내용 작성하면서 이용도구 작성 예정 
* **Workflow Management:** 내용 작성하면서 이용도구 작성 예정 
* **Visualization:** 내용 작성하면서 이용도구 작성 예정 
* **Environment:** 내용 작성하면서 이용도구 작성 예정 

---

## 🏗 Data Pipeline Architecture (데이터 흐름도)

```text
 [공공 데이터 API / CSV] 
         │
         ▼ (Ingestion: requests)
  [Raw Data Storage] (Local / S3)
         │
         ▼ (Processing & Refine: Pandas / Spark / tqdm)
 [Staging & Mart DB] (PostgreSQL / DW)
         │
         ▼ (Service / Visualization)
   [Dashboard / BI] (Streamlit / Tableau)
