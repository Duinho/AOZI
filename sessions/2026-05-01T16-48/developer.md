# 💻 Developer — Researcher가 제공한 전처리된 Capacity 데이터를 기반으로, '요일반 포화 지수(Utilization Index)'를 계산하는 핵심 로직을 포함하는 데이터 파이프라인(자동화 코드) 초안을 작성하세요. 이 코드는 (실제 수요 / 시설 용량)을 계산하고, 이를 통해 과부하/유휴 자원 발생 여부를 판단하는 기능이 포함되어야 합니다. 출력은 동작 가능한 코드 스니펫과 테스트 결과여야 합니다.

💻 Developer: 작업 시작합니다.

## 🛠️ 자동화 코드: 요일반 포화 지수(Utilization Index) 계산 파이프라인

요청하신 '요일반 포화 지수' 계산 로직은 데이터 파이프라인의 핵심 비즈니스 규칙(Business Rule)을 정의하는 단계입니다. 이 코드는 Pandas와 Numpy를 사용하여, 통합 마스터 스키마에 정의된 `실제 수요`와 `시설 용량`을 기반으로 포화 지수를 계산하고, 이 지수를 바탕으로 자원 운영 상태를 판단합니다.

### 1. Python 코드 스니펫 (data_pipeline.py)

이 코드는 환경 변수나 파일 입력을 가정하여 설계되었으며, 실행 가능한 형태로 제공됩니다.

```python
import pandas as pd
import numpy as np
from typing import Tuple

def calculate_utilization_index(df: pd.DataFrame) -> Tuple[pd.DataFrame, pd.DataFrame]:
    """
    Capacity Data를 기반으로 요일반 포화 지수(Utilization Index)를 계산하고
    운영 상태를 판단하는 파이프라인 함수.

    Args:
        df: 통합 마스터 스키마를 따르는 DataFrame (필수 컬럼: ASSIGNED_STUDENT_COUNT, MAX_CAPACITY).

    Returns:
        (df_result, df_status): 결과 지수와 운영 상태가 추가된 DataFrame 튜플.
    """
    print("--- [INFO] 1. 포화 지수 계산 시작 ---")

    # 1. 포화 지수 계산 (실제 수요 / 시설 용량)
    # MAX_CAPACITY가 0인 경우 ZeroDivisionError 방지 및 NaN 처리
    df['Utilization_Index'] = np.where(
        df['MAX_CAPACITY'] > 0,
        df['ASSIGNED_STUDENT_COUNT'] / df['MAX_CAPACITY'],
        np.nan  # 용량이 0이면 지수도 NaN 처리
    )

    print("--- [INFO] 2. 운영 상태 분류 및 리포팅 ---")

    # 2. 운영 상태(Operational Status) 판단 로직 적용
    def determine_status(index):
        if pd.isna(index):
            return "데이터 부족/오류"
        elif index > 1.05:  # 5% 초과 과부하 (Critical Overload)
            return "🔴 과부하 (Overloaded) - 긴급 증설 필요"
        elif index > 1.0:  # 100% 초과 과부하 (Overload)
            return "🟠 과부하 (Overloaded) - 즉각 조정 필요"
        elif 0.8 <= index <= 1.0: # 80% ~ 100% (Optimal)
            return "🟢 적정 운영 (Optimal) - 정상 범위"
        elif 0.6 <= index < 0.8: # 60% ~ 80% (Warning)
            return "🟡 주의 (Warning) - 여유 공간 확보 필요"
        else: # 60% 미만 (Underutilized)
            return "🔵 유휴 (Underutilized) - 자원 재배치 검토"

    df['Operational_Status'] = df['Utilization_Index'].apply(determine_status)

    df_result = df.copy()
    df_result['Utilization_Index'] = df_result['Utilization_Index'].round(3)
    
    print("\n✅ 파이프라인 실행 완료. 결과 DataFrame을 반환합니다.")
    return df_result, pd.DataFrame({'Message': ['Success']})

# ====================================================================
# 테스트 환경 설정 및 실행 (Mock Data)
# ====================================================================

# 가상의 데이터셋 (Researcher가 정의한 마스터 스키마 기반)
data = {
    'STUDENT_ID': ['S001', 'S002', 'S003', 'S004', 'S005', 'S006'],
    'CLASS_ID': ['C101', 'C102', 'C101', 'C103', 'C102', 'C101'],
    'FACILITY_ID': ['F_A', 'F_B', 'F_A', 'F_C', 'F_B', 'F_A'],
    'USED_DAY': ['월', '화', '월', '목', '화', '월'],
    'MAX_CAPACITY': [30, 20, 30, 50, 20, 30], # 시설 최대 용량
    'ASSIGNED_STUDENT_COUNT': [35, 18, 30, 55, 22, 15] # 실제 수요
}

df_test = pd.DataFrame(data)

# 함수 실행
try:
    df_result, df_status = calculate_utilization_index(df_test)
    print("\n======================================================")
    print("               ✅ 최종 계산 결과 (Utilization Index)")
    print("======================================================")
    print(df_result[['STUDENT_ID', 'USED_DAY', 'MAX_CAPACITY', 'ASSIGNED_STUDENT_
