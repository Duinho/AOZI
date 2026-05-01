# 💻 Developer — 실패 원인 보고를 위한 '에러 핸들링(Error Handling)' 모듈의 사양을 정의합니다. 이 모듈은 단순히 'Failed'를 반환하는 것이 아니라, '오류 유형(Error Type)', '위반된 스키마 필드(Schema Field)', '실제 위반 값(Actual Value)', 그리고 '수정 가이드라인(Remediation Guide)'을 포함하는 구조화된 JSON 형식의 보고서를 자동 생성해야 합니다.

💻 Developer: 작업 시작합니다.

사용자님의 피드백은 시스템의 신뢰도와 실용성을 확보하는 데 있어 가장 중요한 핵심 요구사항입니다. 단순히 '실패'를 통보하는 것은 시스템의 가치를 떨어뜨립니다. 개발자로서, 이 문제를 해결하기 위해 **'구조화된 오류 핸들링 모듈(Structured Error Handling Module)'**의 사양을 정의하고, 이를 모든 데이터 파이프라인의 최종 검증 단계에 강제 통합하겠습니다.

이 모듈은 단순 에러 로그가 아닌, **'데이터 결함 진단 보고서'** 역할을 수행하도록 설계됩니다.

---

### 🛠️ 1. 모듈 목표 및 작동 원리

**목표:** 데이터 처리 과정 중 발생하는 모든 비정상적인 값(Outliers), 형식 불일치(Type Mismatch), 필수 값 누락(Null Value)에 대해 **자동으로 구조화된 JSON 보고서**를 생성한다.

**작동 흐름:**
1.  **데이터 입력:** 파이프라인이 데이터를 로드한다.
2.  **스키마 검증:** 정의된 스키마(Schema)에 따라 모든 필드를 순차적으로 검증한다.
3.  **오류 감지:** 검증 규칙(Rule) 위반이 발생하면, 해당 데이터 레코드 처리를 중단하고, 오류 정보를 수집한다.
4.  **보고서 생성:** 수집된 모든 오류 정보를 정의된 JSON 스키마에 맞춰 산출한다.

### 🧱 2. Error Handling Module Specification (JSON Schema)

이 모듈의 최종 출력물은 반드시 다음의 구조를 따르는 JSON 배열 형태여야 합니다.

```json
{
  "report_id": "UNIQUE_UUID",
  "timestamp": "YYYY-MM-DDTHH:MM:SS",
  "processing_context": "어떤 데이터를 처리했는지 명시 (예: 시설수요조사_20260502)",
  "is_successful": false,
  "errors": [
    {
      "error_type": "String (필수)",
      "violation_description": "String (필수)",
      "schema_field": "String (필수)",
      "actual_value": "Any (필수)",
      "violation_reason": "String (필수)",
      "source_citation": "String (필수)",
      "remediation_guide": "String (필수)"
    }
    // ... 오류가 발견될 때마다 이 객체가 추가됨
  ]
}
```

#### 📌 필드별 상세 정의 및 요구사항

| 필드명 | 데이터 타입 | 정의 및 역할 | 필수 입력 데이터 (예시) |
| :--- | :--- | :--- | :--- |
| `error_type` | String | 오류의 구조적 유형 분류 (예: `TypeMismatch`, `MissingData`, `OutOfRange`, `SchemaViolation`). | `TypeMismatch` |
| `violation_description` | String | 오류가 발생한 현상에 대한 구체적인 설명. | "Utilization Index가 유효 범위를 벗어났습니다." |
| `schema_field` | String | **문제 발생 필드명.** 데이터 스키마와 1:1 매칭되어야 함. | `"Utilization_Index"` |
| `actual_value` | Any | **오류가 발생한 실제 데이터 값.** (값 자체를 기록) | `1.25` |
| `violation_reason` | String | **오류가 발생한 논리적/규칙적 이유.** (예: "이 값은 0과 1 사이여야 함.") | "Utilization Index는 시설 용량 대비 수요 비율이므로 1.0을 초과할 수 없습니다." |
| `source_citation` | String | **원본 데이터의 위치.** 추적 가능성을 극대화. | `"AOZI_data.xlsx (Sheet: 시설수요 / Row: 32)"` |
| `remediation_guide` | String | **해결책 및 다음 액션.** 사용자에게 제시할 구체적 조치 가이드. | "해당 데이터의 '시설 용량' 필드를 재검토하거나, 수요 값을 1.0 이하로 조정해야 합니다." |

### ⚙️ 3. 기술적 구현 (Python Pseudo-Code Skeleton)

이 사양을 바탕으로, 파이프라인의 검증 로직을 담당할 핵심 함수 스켈레톤을 정의합니다.

```python
import json
from typing import List, Dict, Any

def run_data_validation_and_error_logging(data_records: List[Dict[str, Any]], schema: Dict[str, Any]) -> Dict[str, Any]:
    """
    주어진 데이터 레코드 리스트를 스키마 기반으로 검증하고 오류를 JSON 보고서로 생성합니다.
    """
    error_list = []
    
    for i, record in enumerate(data_records):
        for field_name, field_schema in schema.items():
            actual_value = record.get(field_name)
            
            # 1. Null Value 검증 (필수값 누락)
            if field_schema.get("required", False) and actual_value is None:
                error_list.append({
                    "error_type": "MissingData",
                    "violation_description": f"필수 필드 '{field_name}'가 누락되었습니다.",
                    "schema_field": field_name,
                    "actual_value": "NULL",
                    "violation_reason": "데이터 원본에 해당 정보가 없습니다.",
                    "source_citation": f"Record Index {i}",
                    "remediation_guide": f"원본 데이터에서 '{field_name}' 필드를 반드시 채워야 합니다."
