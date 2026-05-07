# 데이터 설계서 (DDD)
**산출물 ID:** MD05  
**모듈:** RETS-MODULE-02 — 이해관계자 및 인터뷰 매니징  
**파일명:** `rets_module02_05_ddd.md`  
**버전:** 1.0  
**작성일:** 2026-05-07  
**참고 산출물:** `rets_module02_03_srs.xlsx` (SRS 추적 기준), `rets_02c_req_schema.json`

---

## 1. 개요

### 1.1 목적

본 문서는 RETS-MODULE-02 웹앱 내에서 처리되는 주요 데이터 객체의 구조, 속성, 관계, 제약조건을 정의하고, 각 데이터 객체와 SRS 요구사항 간의 추적 관계를 정리한다.

### 1.2 구현 특성

- 백엔드 DB 없이 **브라우저 메모리(런타임 상태)** 및 **localStorage** 기반으로 데이터를 관리한다.
- 공통 벤치마크 스키마 `rets_02c_req_schema.json`에 정의된 `Requirement` 객체는 외부 JSON 파일에서 불러온다(Read-Only).
- 나머지 데이터 객체(이해관계자, 인터뷰 계획, 질문지 등)는 모듈 내부 스키마로 정의한다.

---

## 2. 데이터 모델 개요 (ERD 설명)

```
Requirement (외부 입력, Read-Only)
    │
    ├─── InterviewPlan (N개) ──── Stakeholder (1개)
    │         │
    │         ├─── Questionnaire (1개) ──── QuestionItem (N개)
    │         │
    │         └─── InterviewResult (1개)
    │                   │
    │                   ├─── InterviewResultItem (N개)
    │                   │         │
    │                   │         └─── RequirementMapping (N개) ──── Requirement
    │                   │
    │                   └─── ActionItem (N개)
    │
Stakeholder (독립 관리)
```

---

## 3. 데이터 객체 정의

---

### 3.1 Requirement (요구사항)

> **출처:** 외부 JSON 파일 (`rets_02c_req_schema.json` 준수). 모듈 내 Read-Only.

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `req_type` | enum | ✅ | "RFP" 또는 "DEV" | RFP / DEV |
| `req_id` | string | ✅ | 요구사항 고유 ID | 유일값 |
| `req_version` | string | ✅ | 버전 (예: "1.0") | |
| `req_level` | integer | ✅ | 계층 수준 (1~5) | 1 ≤ x ≤ 5 |
| `req_name` | string | ✅ | 요구사항명 | |
| `functional_class` | enum | ✅ | "기능" / "비기능" | |
| `req_category` | enum | ✅ | 요구사항 유형 | |
| `description` | string | ✅ | 상세 설명 | |
| `priority` | enum | — | MoSCoW 우선순위 | Must/Should/Could/Won't |
| `stakeholders` | string[] | — | 연관 이해관계자 ID 목록 | |
| `req_status` | enum | ✅ | 요구사항 상태 | |
| `created_by` | string | ✅ | 발의자 | |
| `last_modified_date` | string | ✅ | 최종 수정일 (ISO 8601) | |

**SRS 추적:** MOD02-F-01-01, MOD02-F-01-02, MOD02-F-01-03, MOD02-F-05-03

---

### 3.2 Stakeholder (이해관계자)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `stakeholder_id` | string | ✅ | 고유 ID (예: "STK-001") | 유일값, 자동 채번 |
| `name` | string | ✅ | 이름 | 공백 불가 |
| `role` | string | ✅ | 역할 (예: "발주사 PM") | 공백 불가 |
| `organization` | string | — | 소속 조직 | |
| `contact` | string | — | 연락처 (이메일 또는 전화) | |
| `power` | integer | ✅ | 영향력 점수 | 1 ≤ x ≤ 5 |
| `interest` | integer | ✅ | 관심도 점수 | 1 ≤ x ≤ 5 |
| `notes` | string | — | 메모 | |
| `created_at` | string | ✅ | 생성 일시 (ISO 8601) | |
| `updated_at` | string | ✅ | 수정 일시 (ISO 8601) | |

**파생 필드 (UI 계산):**
- `quadrant`: power/interest 조합으로 "Manage Closely" / "Keep Satisfied" / "Keep Informed" / "Monitor" 자동 결정

**SRS 추적:** MOD02-F-02-01, MOD02-F-02-02, MOD02-F-02-03

---

### 3.3 InterviewPlan (인터뷰 계획)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `interview_id` | string | ✅ | 고유 ID (예: "INT-001") | 유일값, 자동 채번 |
| `title` | string | ✅ | 인터뷰 제목 | |
| `purpose` | string | — | 인터뷰 목적 | |
| `stakeholder_id` | string | ✅ | 대상 이해관계자 ID | → Stakeholder 참조 |
| `related_req_ids` | string[] | — | 관련 요구사항 ID 목록 | → Requirement 참조 |
| `participants` | string[] | — | 참여자 이름 목록 | |
| `scheduled_at` | string | ✅ | 예정 일시 (ISO 8601) | |
| `location` | string | — | 장소 또는 URL | |
| `estimated_duration_min` | integer | — | 예상 소요시간 (분) | x > 0 |
| `status` | enum | ✅ | 상태 | "예정" / "완료" / "취소" |
| `questionnaire_id` | string | — | 연결된 질문지 ID | → Questionnaire 참조 |
| `result_id` | string | — | 연결된 결과 ID | → InterviewResult 참조 |
| `created_at` | string | ✅ | 생성 일시 | |
| `updated_at` | string | ✅ | 수정 일시 | |

**SRS 추적:** MOD02-F-03-01, MOD02-F-03-02, MOD02-F-03-03

---

### 3.4 Questionnaire (질문지)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `questionnaire_id` | string | ✅ | 고유 ID (예: "QST-001") | 유일값 |
| `interview_id` | string | ✅ | 소속 인터뷰 계획 ID | → InterviewPlan 참조 |
| `version` | integer | ✅ | 버전 번호 | x ≥ 1, 자동 증가 |
| `generated_by` | enum | ✅ | 생성 방식 | "LLM" / "Manual" |
| `llm_prompt_context` | string | — | LLM 호출 시 사용된 컨텍스트 요약 | |
| `items` | QuestionItem[] | ✅ | 질문 항목 목록 | |
| `created_at` | string | ✅ | 생성 일시 | |

**SRS 추적:** MOD02-F-04-01, MOD02-F-04-04

---

### 3.5 QuestionItem (질문 항목)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `item_id` | string | ✅ | 고유 ID (예: "QI-001") | 유일값 |
| `questionnaire_id` | string | ✅ | 소속 질문지 ID | → Questionnaire 참조 |
| `order` | integer | ✅ | 정렬 순서 | x ≥ 1 |
| `content` | string | ✅ | 질문 내용 | 공백 불가 |
| `source` | enum | ✅ | 출처 | "LLM" / "Manual" |
| `acceptance_status` | enum | ✅ | 수락 여부 | "Pending" / "Accepted" / "Rejected" |
| `related_req_ids` | string[] | — | 관련 요구사항 ID | |
| `created_at` | string | ✅ | 생성 일시 | |

**SRS 추적:** MOD02-F-04-01, MOD02-F-04-02, MOD02-F-04-03

---

### 3.6 InterviewResult (인터뷰 결과)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `result_id` | string | ✅ | 고유 ID (예: "RES-001") | 유일값 |
| `interview_id` | string | ✅ | 소속 인터뷰 계획 ID | → InterviewPlan 참조 |
| `raw_notes` | string | ✅ | 원본 인터뷰 메모(자유 텍스트) | |
| `structured_items` | InterviewResultItem[] | — | LLM 구조화 결과 항목 | |
| `action_items` | ActionItem[] | — | 도출된 액션아이템 | |
| `created_at` | string | ✅ | 생성 일시 | |
| `updated_at` | string | ✅ | 수정 일시 | |

**SRS 추적:** MOD02-F-05-01, MOD02-F-05-02

---

### 3.7 InterviewResultItem (구조화된 인터뷰 항목)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `item_id` | string | ✅ | 채번 ID (예: "INT-001-01") | 유일값 |
| `result_id` | string | ✅ | 소속 인터뷰 결과 ID | → InterviewResult 참조 |
| `content` | string | ✅ | 구조화된 항목 내용 | |
| `source` | enum | ✅ | 생성 방식 | "LLM" / "Manual" |
| `acceptance_status` | enum | ✅ | 수락 여부 | "Pending" / "Accepted" / "Rejected" |
| `mappings` | RequirementMapping[] | — | 요구사항 매핑 목록 | |
| `is_action_item_candidate` | boolean | — | 액션아이템 후보 여부 | |

**SRS 추적:** MOD02-F-05-02, MOD02-F-05-03

---

### 3.8 RequirementMapping (요구사항 매핑)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `mapping_id` | string | ✅ | 고유 ID | 유일값 |
| `interview_id` | string | ✅ | 인터뷰 ID | → InterviewPlan 참조 |
| `item_id` | string | ✅ | 인터뷰 결과 항목 ID | → InterviewResultItem 참조 |
| `req_id` | string | ✅ | 매핑된 요구사항 ID | → Requirement 참조 |
| `mapping_type` | enum | ✅ | 매핑 방식 | "Explicit" / "Inferred" |
| `confidence_level` | enum | — | 추론 신뢰도 | "High" / "Medium" / "Low" |
| `content_excerpt` | string | — | 관련 인터뷰 내용 발췌 | |
| `acceptance_status` | enum | ✅ | 수락 여부 | "Pending" / "Accepted" / "Rejected" |
| `created_at` | string | ✅ | 생성 일시 | |

**SRS 추적:** MOD02-F-05-03, MOD02-F-05-06

---

### 3.9 ActionItem (액션아이템)

| 필드명 | 타입 | 필수 | 설명 | 제약 |
|--------|------|------|------|------|
| `action_id` | string | ✅ | 고유 ID (예: "ACT-001") | 유일값 |
| `interview_id` | string | ✅ | 출처 인터뷰 ID | → InterviewPlan 참조 |
| `result_item_id` | string | — | 출처 구조화 항목 ID | → InterviewResultItem 참조 |
| `content` | string | ✅ | 액션아이템 내용 | 공백 불가 |
| `assignee` | string | — | 담당자 | |
| `due_date` | string | — | 완료 목표일 (ISO 8601) | |
| `status` | enum | ✅ | 상태 | "미완료" / "완료" / "취소" |
| `created_at` | string | ✅ | 생성 일시 | |
| `updated_at` | string | ✅ | 수정 일시 | |

**SRS 추적:** MOD02-F-05-04, MOD02-F-06

---

## 4. 데이터 흐름 정의

### 4.1 입력 데이터 흐름

```
[외부 JSON 파일]
  └─ rets_02c_req_schema.json 형식
       └─ 파일 불러오기 (TC-01-01)
            └─ Requirement[] 객체로 파싱
                 └─ 브라우저 메모리 저장
```

### 4.2 LLM 입출력 데이터 흐름

```
[질문지 생성 요청]
  입력: { stakeholder: Stakeholder, requirements: Requirement[], purpose: string }
  출력: { questions: [{ content: string, related_req_ids: string[] }] }
  파싱: JSON 배열 → QuestionItem[] 생성

[결과 구조화 요청]
  입력: { raw_notes: string, requirements: Requirement[] }
  출력: { items: [{ id: string, content: string, is_action_candidate: boolean, suggested_req_ids: string[], confidence: string }] }
  파싱: → InterviewResultItem[] + RequirementMapping[] 생성
```

### 4.3 출력 데이터 흐름

```
[Markdown 출력]
  InterviewPlan + Stakeholder + InterviewResult + InterviewResultItem[] + ActionItem[]
       └─ Markdown 문자열 생성 → Blob 다운로드

[CSV 출력]
  RequirementMapping[] (acceptance_status == "Accepted")
       └─ CSV 문자열 생성 → Blob 다운로드
  컬럼: req_id, interview_id, item_id, content, confidence_level, accepted
```

---

## 5. 데이터 제약 및 검증 규칙

| 규칙 ID | 객체 | 규칙 설명 |
|---------|------|-----------|
| DV-01 | Stakeholder | `name`, `role` 필드는 공백 불가 |
| DV-02 | Stakeholder | `power`, `interest`는 1~5 정수만 허용 |
| DV-03 | InterviewPlan | `stakeholder_id`는 등록된 Stakeholder ID여야 함 |
| DV-04 | InterviewPlan | `scheduled_at`는 ISO 8601 형식 준수 |
| DV-05 | QuestionItem | `content`는 공백 불가 |
| DV-06 | RequirementMapping | `req_id`는 로드된 Requirement ID여야 함 |
| DV-07 | ActionItem | `content`는 공백 불가 |
| DV-08 | Requirement | 외부 JSON 로드 시 `req_id` 중복 불가 |

---

## 6. SRS 추적 매트릭스 (데이터 관점)

| 데이터 객체 | 관련 SRS 요구사항 |
|------------|-----------------|
| Requirement | MOD02-F-01-01, MOD02-F-01-02, MOD02-F-01-03 |
| Stakeholder | MOD02-F-02-01, MOD02-F-02-02, MOD02-F-02-03 |
| InterviewPlan | MOD02-F-03-01, MOD02-F-03-02, MOD02-F-03-03 |
| Questionnaire | MOD02-F-04-01, MOD02-F-04-04 |
| QuestionItem | MOD02-F-04-01, MOD02-F-04-02, MOD02-F-04-03 |
| InterviewResult | MOD02-F-05-01, MOD02-F-05-02, MOD02-F-05-05 |
| InterviewResultItem | MOD02-F-05-02, MOD02-F-05-03 |
| RequirementMapping | MOD02-F-05-03, MOD02-F-05-06 |
| ActionItem | MOD02-F-05-04, MOD02-F-06 |

---

*스키마 정의 파일은 `rets_module02_05_ddd_schema.json`을 참고한다.*
