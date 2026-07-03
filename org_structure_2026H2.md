---
name: org-structure-2026h2
description: "2026 하반기 조직 개편 - 스쿼드 → 파트 전환 매핑 (AI 어시스턴트 참조용, 착오 방지)"
metadata: 
  node_type: memory
  type: reference
  last_updated: 2026-07-02
  scope: 제품실 전체 파트/스쿼드 명칭 매핑
  originSessionId: 82a18ae3-066b-4c7d-a17f-ee9a9ce1d3dd
---

# 2026 하반기 조직 개편 - 파트 체계

## 배경

- 기존 스쿼드 체제 → 파트 체제로 전환
- 각 스쿼드가 파트의 하위 단위 또는 파트 자체로 편입
- AI 어시스턴트가 모니터링·분석·리포트 작성 시 신규 파트명 우선 사용

## 파트 ⇄ 스쿼드/사업 매핑

| 파트 (신규) | 하위 구성 | 구 명칭 | 담당 사업 |
|---|---|---|---|
| **구매전환 파트** | KDC / KDT | 그로스 스쿼드 | KDC(국비지원 시니어 AI 교육), KDT(내일배움캠프 부트캠프 모집) |
| **교육솔루션 파트** | 온라인 / 부트캠프 | 수강경험 스쿼드(온라인), 학습관리 스쿼드(부트캠프) | 온라인 강의 학습·수료, 부트캠프 학습·A.tani |
| **취업솔루션 파트** | (단일) | 커리어 스쿼드 | 부트캠프 수료생 취업 지원 (바로인턴·이력서·프로필 등) |

## 파트별 관련 메모리 파일

### 구매전환 파트

- `squad_growth_kdc.md` - KDC 도메인 + 데일리 모니터링 운영 (매일 9:30, n8n)
- `squad_growth_kdt.md` - KDT 도메인 + 위클리 모니터링 운영 (월 10:15/금 16:15, n8n)
- `kdc_werl_kpi_framework.md` - KDC 매출 연동 KPI 체계
- `kdc_learning_segment_trends.md` - KDC 수강경험 트렌드 (참조 시 파트 크로스)
- `kdc_lead_definition.md` - KDC 리드 정의
- `kdc_revenue_calculation.md` - KDC 매출 계산
- `kdc_completion_vs_practice.md` - KDC 수료·실습 정의

### 교육솔루션 파트

- `squad_learning_experience.md` - 온라인 (구 수강경험) 지표 체계
- `squad_learning_management.md` - 부트캠프 (구 학습관리) A.tani 사용패턴 + 챕터 KPI

### 취업솔루션 파트

- `squad_career.md` - 커리어 지표한판 v1
- `squad_career_v2.md` - 커리어 v2 (유입/행동/취업 3섹터)
- `squad_career_employment.md` - 취업솔루션 파트 지표 체계 전반 (WAU/WoW/베이스라인)
- `career_two_sheets_difference.md` - 커리어 시트 소급 변동 이슈
- `career_behavior_employment_window_gap.md` - 행동로그 vs 취업성숙 윈도우 갭

## AI 어시스턴트 사용 가이드 (착오 방지)

### 모니터링·리포트 작성 시

- 신규 파트명 우선 사용 - "구매전환 파트 KDC", "교육솔루션 파트 - 부트캠프" 등
- 슬랙 채널·메시지 헤더는 사용자 확인 후 반영 (기존 채널명은 즉시 변경 X)
- 문서·기획서에는 "(구 그로스 스쿼드)" 같은 병기 초반 1회 정도만

### 사용자 발화 매핑

- "그로스 KDT" 또는 "그로스 KDC" 언급 → 구매전환 파트로 인식
- "수경스", "수강경험" → 교육솔루션 파트 - 온라인
- "학관스", "학습관리" → 교육솔루션 파트 - 부트캠프
- "커리어" → 취업솔루션 파트

### 이전 명칭 사용 금지 상황

- 신규 노션 문서 제목·헤더
- 슬랙 공지·메시지 새로 만들 때
- 파트별 KPI·기획서 작성 시

### 이전 명칭 유지 가능 상황

- 코드·SQL 안 컬럼명·변수명 (기존 명명 관성 유지)
- 시트 탭 이름 (일괄 변경 부담)
- 슬랙 기존 채널 이름 (별도 리네이밍 안건 확정 전)

## 진행 이력

- 2026-07-02: 조직 개편 확정, 메모리 반영 시작

## 관련 메모리

- `MEMORY.md` - 파트별 그룹핑으로 인덱스 재구조화됨
- `da_chapter_kpi.md` - DA 챕터 관점의 파트별 지원 범위
- `da_chapter_collab_axes.md` - DA-PM 협업 3축 모델 (파트별 검토 범위 규칙)
