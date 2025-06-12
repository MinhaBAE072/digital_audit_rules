# Rule Converter

## 개요

이 시스템은 **엑셀 파일에 작성된 규칙을 자동으로 JSON 형식으로 변환**하는 도구입니다. 사용자는 엑셀에 지침을 추가하는 것만으로 `rules.json` 파일이 자동으로 업데이트됩니다.

## 주요 기능

-  **엑셀 → JSON 자동 변환**: 엑셀 파일의 규칙을 JSON 형식으로 자동 변환
-  **지능형 규칙 분류**: 규칙 유형을 자동으로 판단하여 적절한 타입 할당
-  **데이터 정제**: 사용자 친화적인 엑셀 데이터를 기계가 이해할 수 있는 형태로 변환
-  **실시간 업데이트**: 엑셀 파일 변경 시 즉시 JSON 파일 업데이트

## 시스템 구조

```
rules/
├── modules/
│   └── rule_converter.py     # 핵심 변환 스크립트
├── rules_source/
│   └── 탐지 가이드라인.xlsx   # 엑셀 규칙 파일
├── system_rules/
│   └── rules.json            # 변환된 JSON 파일
```

## 작동 원리

### 1. 규칙 유형 자동 분류 (Conditional Logic)

시스템은 다음 우선순위에 따라 규칙의 `type`을 자동으로 결정합니다:

#### 1.1 ID 접두사 우선 분류
| 접두사 | 규칙 타입 | 설명 |
|--------|-----------|------|
| `PRSN` | `check_personal_items` | 개인 물품 관련 규칙 |
| `MEAL` | `check_meal_related` | 식사 관련 규칙 |
| `VEH` | `check_vehicle` | 차량 관련 규칙 |
| `FRAU` | `check_fraud` | 부정 행위 관련 규칙 |

#### 1.2 키워드 기반 분류
- **증빙 누락 규칙**: '점검 조건'에 "증빙"과 "누락" 키워드가 모두 포함된 경우
  - `type`: `check_attachment`
- **금액 한도 규칙**: '위반 태그'에 "한도" 키워드가 있거나, '점검 조건'에 4자리 이상 숫자가 포함된 경우
  - `type`: `check_amount_limit`

#### 1.3 기본값 할당
- 위 조건에 해당하지 않는 경우: `type`: `check_by_keyword`

### 2. 데이터 파싱 및 정제 (Text Parsing & Cleansing)

#### 2.1 리스트 변환
```
# 입력: "A, B, C"
# 출력: ["A", "B", "C"]
parse_list_string(cell_value)
```

#### 2.2 숫자 추출
```
# 입력: "100,000원 초과"
# 출력: 100000
filter(str.isdigit, text)
```

### 3. JSON 구조 매핑 (Data Structuring)

#### 3.1 엑셀 컬럼 → JSON 키 매핑
| 엑셀 컬럼 | JSON 키 | 설명 |
|-----------|---------|------|
| 점검 계정 | `target_account` | 대상 계정 |
| 위반 태그 | `rule_name` | 규칙 이름 |
| 점검 조건 | `conditions` | 검사 조건 |
| 비고#1~#5 | `remark1`~`relevance_check_threshold` | 추가 파라미터 |

#### 3.2 최종 JSON 구조
```
{
  "rule_id": "규칙 ID",
  "rule_name": "규칙 이름",
  "type": "규칙 타입",
  "parameters": {
    "target_account": "대상 계정",
    "conditions": ["조건1", "조건2"],
    "exclude_accounts": [],
    "exclude_merchants": [],
    "remark1": [],
    "remark2": [],
    "remark3": [],
    "remark4": [],
    "relevance_check_threshold": null
  },
  "risk_score": 50,
  "violation_tag": "위반 태그",
  "violation_message": "위반 메시지"
}
```

## 사용 예시

### 엑셀에 새 규칙 추가
```
| ID     | 점검 계정  | 점검 조건         | 위반 태그            | 대분류    |
|--------|-----------|-------------------|---------------------|-----------|
| VEH-05 | 차량유지비 | 주말, 공휴일 사용 | 주말/공휴일 차량 사용 | 비용-차량 |
```

### 자동 변환 결과
```
{
  "rule_id": "VEH-05",
  "rule_name": "주말/공휴일 차량 사용",
  "type": "check_vehicle",
  "parameters": {
    "target_account": "차량유지비",
    "dont_conditions": ["주말", "공휴일 사용"],
    "do_conditions": [],
    "exclude_accounts": [],
    "exclude_merchants": [],
    "remark1": [],
    "remark2": [],
    "remark3": [],
    "remark4": [],
    "relevance_check_threshold": null
  },
  "risk_score": 50,
  "violation_tag": "비용-차량",
  "violation_message": "'주말/공휴일 차량 사용' 위반"
}
```

## 설치 및 실행

### 필요 라이브러리
```
pip install pandas openpyxl
```

## 핵심 기술 스택

- **Python 3.x**: 메인 프로그래밍 언어
- **pandas**: 엑셀 파일 읽기 및 데이터 처리
- **openpyxl**: 엑셀 파일 처리
- **json**: JSON 파일 생성 및 처리
- **re (정규표현식)**: 텍스트 패턴 매칭

## 특징

### 장점
- **자동화**: 수동 작업 없이 엑셀 변경 시 자동 변환
- **확장성**: 새로운 규칙 타입 쉽게 추가 가능
- **안정성**: 데이터 검증 및 오류 처리 포함
- **유지보수성**: 엑셀로 규칙 관리가 가능하여 비개발자도 사용 가능

### 기술적 특징
- 조건부 로직을 통한 지능형 분류
- 정규표현식을 활용한 텍스트 패턴 매칭
- 데이터 정제 및 구조화 자동화
- UTF-8 한글 지원

---

**Note**: 이 시스템은 사람이 관리하는 엑셀과 기계가 사용하는 JSON 사이의 지능적인 번역기 역할을 하며, 정해진 로직에 따라 자동으로 데이터를 분류하고 구조화합니다.
