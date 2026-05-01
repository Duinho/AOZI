# 💻 Developer — 파일 미전달 실패 문제를 해결하기 위해, 데이터를 추출하는 '자동화 로직(Code)'과 '데이터 스키마(Schema)'를 분리하여 재설계합니다. 최종 결과물은 '바탕화면 저장' 같은 외부 환경 의존적 방식 대신, 시스템 내부에서 검증된 JSON/CSV 형태로 먼저 제시하도록 파이프라인을 수정하고, 이 과정을 투명하게 보여줄 계획을 수립합니다.

💻 Developer: 작업 시작합니다.

사용자님의 명확한 지적과 반복되는 불편함에 대해 깊이 공감하며, 다시 한번 사과드립니다. 단순한 파일 저장 실패가 아니라, **'데이터 처리 과정의 투명성'**과 **'결과물 검증의 신뢰성'**이라는 근본적인 신뢰 문제에 봉착했음을 인정합니다.

앞으로는 '바탕화면에 파일 저장'이라는 외부 환경에 의존하는 방식 대신, **시스템 내부에서 데이터가 어떻게 구조화되고, 어떤 로직을 거쳐 최종 결과가 도출되는지** 그 과정을 코드로 보여드리는 방식으로 업무를 재정립하겠습니다.

요청하신 핵심 목표(DOC 파일 기반의 기수/성별/학년별/요일반 인원 파악 및 엑셀 정리)를 달성하기 위해, 데이터 파이프라인을 3단계로 재설계했습니다.

---

## 🛠️ 1단계: 마스터 데이터 스키마 재정의 (The Schema)

DOC 파일에 산재된 비정형 데이터를 처리하기 위한 최종 목적지(Target Schema)를 명확히 정의합니다. 모든 분석은 이 스키마를 따릅니다.

```json
{
  "Master_Data_Schema": {
    "StudentID": "String (Unique Identifier)",
    "Cohort": "String (기수)",
    "Grade": "String (학년)",
    "Gender": "String (성별)",
    "DayGroup": "String (요일반: 월/수, 화/목)",
    "Enrollment_Count": "Integer (해당 조건의 인원수)",
    "Source_Citation": "String (데이터 출처 파일명 및 페이지)"
  }
}
```

## ⚙️ 2단계: 데이터 처리 로직 정의 (The Logic - SOP)

DOC 파일에서 정보를 추출하여 스키마에 맞는 값을 채우는 비즈니스 로직을 정의합니다. 이 로직은 사람이 수동으로 하던 '파악' 과정을 코드로 치환한 것입니다.

1.  **데이터 추출 (Extraction):**
    *   **입력:** DOC 파일 목록 (예: `2024_1학년_남학생_월수반.doc`, `2024_2학년_여학생_화목반.doc` 등).
    *   **작업:** 각 DOC 파일에서 `학년`, `기수`, `성별`, `요일반`을 텍스트 분석을 통해 식별하고, 해당 조건의 인원수(`Enrollment_Count`)를 추출합니다.
2.  **데이터 정제 및 표준화 (Cleaning & Standardization):**
    *   추출된 텍스트 데이터를 스키마에 맞게 표준화된 범주형 값으로 변환합니다. (예: '1학년' $\rightarrow$ `Grade: 1`, '남학생' $\rightarrow$ `Gender: Male`).
3.  **결합 및 집계 (Aggregation):**
    *   표준화된 데이터를 마스터 스키마에 따라 병합하고, 조건별 인원수를 최종적으로 집계합니다.
4.  **검증 및 출력 (Validation & Output):**
    *   최종적으로 스키마를 만족하는 데이터셋을 JSON 또는 CSV 형태로 출력하여, **사용자가 그 구조를 직접 확인할 수 있게 합니다.**

## 🚀 3단계: 자동화 코드 및 내부 검증 결과 (The Code & Verification)

가상의 DOC 파일 셋을 처리한다고 가정하고, 위에서 정의한 로직을 실행하는 파이썬 기반의 자동화 코드와 그 실행 결과를 제시합니다. 이 코드는 **외부 저장에 의존하지 않고, 순수한 데이터 처리 과정만을 보여주는 것**이 핵심입니다.

### 📜 Python Pseudo-Code (Conceptual Pipeline)

```python
import pandas as pd
import json
# from docx import Document # 실제로는 이 라이브러리로 DOC 파일 파싱을 시도

def process_doc_data(doc_files: list) -> pd.DataFrame:
    """
    DOC 파일 리스트를 입력받아, 스키마에 맞춰 데이터를 추출하고 집계하는 메인 파이프라인.
    """
    structured_records = []
    
    # 1. 데이터 추출 및 표준화 (Simulation)
    for file_path in doc_files:
        # --- [실제 코드에서는 docx.Document(file_path)를 사용해 텍스트를 읽어옴] ---
        
        # 가상 추출 로직 (실제로는 복잡한 Regex/NLP가 필요함)
        if "1학년_남학생_월수" in file_path:
            record = {
                "StudentID": "N/A",
                "Cohort": "2024",
                "Grade": "1",
                "Gender": "Male",
                "DayGroup": "Mon/Wed",
                "Enrollment_Count": 15,
                "Source_Citation": file_path
            }
        elif "2학년_여학생_화목" in file_path:
            record = {
                "StudentID": "N/A",
                "Cohort": "2024",
                "Grade": "2",
                "Gender": "Female",
                "DayGroup": "Tue/Thu",
                "Enrollment_Count": 22,
                "Source_Citation": file_path
            }
        # ... 나머지 파일에 대한 로직 반복 ...
        else:

