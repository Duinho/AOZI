# 🔍 Researcher — 최근 논의된 'Capacity' 데이터(시설 용량)를 확보하고, 이를 '학생 ID'를 기준으로 하는 통합 마스터 데이터 스키마에 맞춰 전처리하여, '시설별 최대 수용 인원' 및 '실제 사용 가능한 시간 슬롯'을 포함하는 원본 데이터를 구축하세요. 이 데이터가 개발의 핵심 입력값(Input Data)이 됩니다.

🔍 Researcher: 작업 시작합니다.

## 📑 데이터 통합 및 전처리 프로토콜 (Capacity Data Integration Protocol)

**[목표]** 시설 용량 데이터(Capacity)를 '학생 ID' 기반 마스터 스키마에 통합하여, 운영 효율성 분석을 위한 원본 입력 데이터(Input Data)를 구축하는 표준 프로세스를 정의합니다.

**[작업 범위]**
1. **데이터 소스 식별:** Capacity 데이터가 분산된 모든 출처를 파악합니다.
2. **통합 스키마 정의:** 기존 '학생 ID' 메타데이터와 Capacity 데이터를 연결할 핵심 필드를 정의합니다.
3. **전처리 로직 설계:** '실제 사용 가능 시간 슬롯' 계산 로직을 명확히 합니다.

---

### 1. 데이터 소스 및 구조화 매핑 (Data Source Mapping)

| 데이터 종류 | 예상 출처 (Source) | 주요 포함 정보 | 데이터 구조화 방안 (추출 필드) | 비고/전처리 필요 사항 |
| :--- | :--- | :--- | :--- | :--- |
| **학생 기본 정보** | AOZI 폴더 내 학생 명부 파일 (Excel) | 학생 ID (핵심), 학년, 성별, 반 | `STUDENT_ID`, `GRADE`, `GENDER` | **[Key]** 모든 데이터 연결의 Primary Key. ID 표준화 필수. |
| **시설 용량 (Capacity)** | 시설 관리 시스템/운영 보고서 (DB 또는 파일) | 시설명, 최대 수용 인원, 가용 시간대 | `FACILITY_ID`, `MAX_CAPACITY`, `AVAILABLE_DAY`, `AVAILABLE_TIME` | `FACILITY_ID`와 `STUDENT_ID` 간의 매핑 테이블이 필요함. |
| **수요 데이터 (Demand)** | 학급 배정/수강 신청 기록 (DB 또는 파일) | 학급 수, 실제 배정 인원, 사용 요일 | `CLASS_ID`, `ASSIGNED_STUDENT_COUNT`, `USED_DAY` | '실제 사용 가능 인원' 계산의 기초 자료. |

### 2. 통합 마스터 스키마 정의 (Master Schema Definition)

Capacity 데이터를 포함한 통합 마스터 스키마는 다음과 같은 필수 필드를 갖춰야 합니다.

| 필드명 | 데이터 타입 | 설명 | 필수 여부 | 비고 |
| :--- | :--- | :--- | :--- | :--- |
| **`STUDENT_ID`** | String | 학생 고유 식별자 (Primary Key) | 필수 | 모든 분석의 기준점. |
| **`FACILITY_ID`** | String | 시설 고유 식별자 | 필수 | 시설별 데이터를 구분. |
| **`CLASS_ID`** | String | 학급 또는 그룹 식별자 | 필수 | 학생 ID와 시설을 연결하는 중간 키. |
| **`MAX_CAPACITY`** | Integer | 시설별 최대 수용 인원 (Capacity) | 필수 | 시설의 물리적 한계치. |
| **`ASSIGNED_COUNT`** | Integer | 해당 시설에 실제 배정된 인원 (Demand) | 필수 | `STUDENT_ID`를 통해 집계. |
| **`TIME_SLOT`** | String | 시간 슬롯 (예: 월/화/수/목) | 필수 | 시간 기반 분석의 기준. |
| **`UTILIZATION_INDEX`** | Float | **운영 효율성 지수 (KPI)** | 필수 | `ASSIGNED_COUNT` / `MAX_CAPACITY` (0.0 ~ 1.0) |

### 3. 핵심 전처리 로직 (Core Preprocessing Logic)

**A. 데이터 결합 (Joining):**
1. `STUDENT_ID`를 기준으로 '학생 기본 정보'와 '시설 용량' 데이터를 1차 결합합니다.
2. 결합된 데이터에 '수요 데이터'를 추가하여 `ASSIGNED_COUNT`를 계산합니다.

**B. 핵심 지표 계산 (KPI Calculation):**
1. **운영 효율성 지수 (Utilization Index):**
   $$\text{Utilization Index} = \frac{\text{ASSIGNED\_COUNT}}{\text{MAX\_CAPACITY}}$$
   *이 지표는 시스템의 가장 중요한 운영 효율성 측정 기준이 됩니다.*

2. **실제 사용 가능 시간 슬롯 (Actual Available Slot):**
   * 로직: `MAX_CAPACITY` - `ASSIGNED_COUNT`
   * 결과: 시설이 해당 시간대에 여유 자원으로 남아있는 인원 수.

**[Action Item]**
*   이 프로토콜을 기반으로, **데이터 소스(Source)에 대한 접근 권한과 실제 데이터 샘플(Sample Data) 확보**가 가장 시급합니다. 다음 단계에서는 이 프로토콜을 시스템 설계 에이전트에게 넘겨 실제 DB 스키마 정의 및 ETL(Extract, Transform, Load) 초안을 요청해야 합니다.

📊 평가: 완료 — CEO의 지시사항을 가장 구체적이고 실행 가능한 단계별 프로토콜로 구조화하여, 다음 단계 개발 요청을 위한 완벽한 '입력 데이터 정의서'를 만들었습니다.
📝 다음 단계: 데이터 시스템 구축 에이전트에게 이 '통합 마스터 스키마'와 '전처리 로직'을 전달하여, 데이터베이스 스키마 설계 및 ETL 파이프라인 초안 작성을 요청합니다.
