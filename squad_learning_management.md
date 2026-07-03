---
name: squad_learning_management
description: "교육솔루션 파트 - 부트캠프 (구 학습관리 스쿼드) 지표한판 + A.tani 사용패턴 분석 - chathistory.timestamp 기반, 평일만 집계, 챕터 KPI 13% 권장 (2026-04-21 최종). 2026 하반기 조직 개편으로 교육솔루션 파트로 재편"
type: project
originSessionId: bf0b0828-ed19-48fe-982a-28f40756bf13
---
# 교육솔루션 파트 - 부트캠프 (구 학습관리 스쿼드) A.tani

> 2026 하반기 조직 개편 - 학습관리 스쿼드 → 교육솔루션 파트 - 부트캠프. 온라인(구 수강경험)과 함께 파트 구성.
> 사용자 발화에서 "학관스", "학습관리" 언급 시 교육솔루션 파트 - 부트캠프로 매핑.
> A.tani(에이타니) - 팀스파르타 LMS에 담긴 AI 튜터.

> **비즈니스 지표**: 학습 성취도
> **제품 연말 KPI**: DAU 90% / 리텐션 70% / 만족도 4.0
> **챕터(2개월) KPI**: 자력 10% (현재) → 권장 13% (학관스 KPI 그대로 달성 시 자연 도달)

---

## 구현 파일

| 파일 | 용도 |
|------|------|
| `claudedocs/learning/학관스_atani_weekly_dashboard.sql` | 주차별 지표한판 |
| `claudedocs/learning/학관스_atani_daily_dau.sql` | 일별 DAU |
| `claudedocs/learning/학관스_atani_지표_명세서.md` | PM 전달용 |
| `claudedocs/learning/학관스_atani_사용패턴_분석.md` | 사용 패턴 분석 (chathistory 기반) |

---

## 주요 설계 결정

### 활동일 산정 (★ 중요)
- **chathistory[].timestamp 기반** (이어쓰기 케이스 정확 카운트)
- chathistory NULL/빈 세션(11.6%)은 createdat fallback
- SUPER unnest 우회: `CROSS JOIN (SELECT 0 UNION SELECT 1 ... UNION SELECT 59)` 인덱스 시리즈
- read-only 권한에서 `FROM tbl, tbl.array` 막힘 → 인덱스 접근으로 우회

### 평일만 집계
- `EXTRACT(DOW FROM date) BETWEEN 1 AND 5` (월~금)
- Daily 리텐션 금→월 +3일 처리
- 주말 활동 제외

### 분모 (수강중수강생)
- `dbnbcamp_enrolleds`: `isnotdroppedout=true AND isregistered=true`
- `dbnbcamp_rounds`: 수강기간 겹침 (`week_end >= round_start AND week_start <= round_end`)

### A.tani 데이터 소스 (★ 채팅 vs 퀴즈 구분, 2026-05-12)
- **채팅 세션**: `dbnbcamp_qna_sessions WHERE quizrecordid IS NULL`
- **퀴즈 응시**: `dbnbcamp_quiz_records` ← 별도 테이블 (컬럼: _id, enrolledid, quizid, question SUPER, submission SUPER, achievementid, retry, createdat)
- **퀴즈 관련 채팅**: `qna_sessions WHERE quizrecordid IS NOT NULL` (퀴즈 풀다 도움 요청)
- 잘못된 패턴: `qna_sessions.quizrecordid`만으로 퀴즈 사용자 카운트 → 56명 중 18명만 잡힘 (38명 누락)
- 첫 액션 분류: 채팅 371(95%) / 퀴즈 19(5%) — 둘 다 사용 47명 / 채팅만 334 / 퀴즈만 9

### 세그먼트
- **미사용/1회사용/재사용**: 이번 주 활동 기준 (누적 아님, 기수 유입/수료에 흔들리지 않게)
- **고관여(≤3일)/중관여(3~14일)/저관여(>14일)**: 재사용자 × 최근 4주 rolling
- **빈도×깊이 매트릭스**: 헤비/자주짧게/가끔길게/라이트 4분면

### 첫방문→재방문
- 주간 코호트 + 14일 기한
- 과거 코호트 불공정 비교 방지
- 최근 2주 코호트는 미확정

### 출력 칼럼
- DAU율(%) 제거 (방문 로그 없어서 Adoption rate로 변질)
- Stickiness = 일평균DAU / WAU
- 퀴즈/피드백 사용률 (분모=WAU)
- 비율은 소수점 (0.314 형식, 엑셀 % 적용용)

### Daily 리텐션 가중 합산
- `SUM(D+1잔존) / SUM(D유저)` (평균의 평균 왜곡 회피)

### 타임존
- 모든 테이블 KST 저장 → 변환 불필요

---

## 챕터 KPI 시뮬레이션 (수강중 1,400명, 8주 균등 진입 가정)

매주 신규 175명, 학관스 KPI 그대로:
- 정상 상태 WAU 430명 (30.7%)
- DAU = 30.7% × 45% Stickiness = **13.8%**
- → 챕터 KPI **13% 권장** (현재 10%는 KPI 미달 시에도 달성 가능)

리텐션 70% 달성 시 DAU 18.7%, 80% 달성 시 28%

---

## 사용 패턴 분석 핵심 (chathistory.timestamp 기반)

| 지표 | 값 |
|------|------|
| 채팅 첫 액션 | 374명 (98%) |
| 퀴즈 첫 액션 | 6명 (2%) |
| **첫 사용 후 이탈** | **56%** (chathistory 정밀 측정) |
| 재사용 | 44% |
| 반복:탐색 | 32:1 (거의 같은 기능 반복) |
| 평균 리드타임 | 9일 (목표 3일) |

### 빈도별 패턴 (재사용자 166명)
| 관여도 | 유저 | 세션당 메시지 | "더 많은 힌트"% |
|--------|:-:|:-:|:-:|
| 고관여 | 40 | 8.9 | 33% |
| 중관여 | 73 | 5.9 | 23% |
| 저관여 | 53 | 4.8 | 19% |

### 빈도×깊이 4분면
- 헤비유저 25% / 라이트 34% / 자주짧게 23% / 가끔길게 17%

### 트랙별 (≥30명만 유효)
- unreal(140), spring(55), digital_marketer(50), advanced-java(37)
- qa(28)/ai_designer(16): 고관여 비율 높지만 표본 적음

---

## 데이터 한계

### chathistory NULL 11.6% → 원인 규명 + 보정 완료 (2026-05-12)
- **임계값 가설 확정**: chatHistory 직렬화 사이즈 ~64KB 초과 시 Hevo 적재 실패
  - 정상 적재 max: 58 msg / 35KB JSON (749건/4-5월)
  - 누락 min: 4 msg / 41KB BSON (125건/4-5월)
  - 완전 분리됨 → Redshift VARCHAR(MAX) 65,535B 한계 또는 Hevo payload 제한 추정
- **MongoDB 직접 검증**: 누락 50건 샘플 100% chatHistory 정상 존재
- **보정 CTE 작성**: `claudedocs/learning/학관스_atani_hevo_보정_activity.sql`
  - 4-5월 125 세션 × 224 (session, kst_date) 행 (UNION ALL SELECT 형식)
  - 매월 1회 재추출 절차 파일 하단 주석
- **보정 효과**: createdat fallback 103 user-days → 정확 보정 167 user-days (+62%)
- **DE팀 액션**: Hevo chathistory 컬럼 매핑/payload limit 점검 요청 필요

### personal_reports = 시스템 자동
- 매일 KST 13시 일괄 배치 생성
- 유저 조회 추적 불가
- 학습 진척 proxy로만 활용 가능

### 미구현 지표
- 만족도/세부항목/만족도별 사용패턴/기능별 만족도 (설문 미개발)
- CX 부정VoC (VoC 데이터 미확보)

---

## Claude Code MCP (배포 전)
- `dbnbcamp_atani_claude_code_session_logs` 테이블 적재 대기
- 쿼리에 UNION ALL 주석 처리로 준비됨 (배포 후 해제)

## MongoDB MCP (진행 중)
- 빈 chathistory 11.6% 원본 확인용
- 등록은 했으나 세션 재시작 필요

---

## 최근 4주 실험·릴리스
- (위클리 미팅 후 한 줄씩 누적)

## 알려진 외부 이벤트
- (연휴·캠페인·정책 변경 등을 일자와 함께 누적)

## 주의할 패턴·해석 가이드
- 활동일 산정은 chathistory[].timestamp 기반 (이어쓰기 정확 카운트). NULL/빈 11.6%는 createdat fallback
- 평일만 집계 (월~금). 주말 활동 제외, Daily 리텐션 금→월 +3일 처리
- 미사용/1회사용/재사용은 이번 주 활동 기준 (누적 아님 — 기수 유입·수료에 흔들리지 않게)
- 첫 액션 분류: 채팅 95% / 퀴즈 5%. 채팅만 vs 퀴즈만 vs 둘 다 구분 필요
- 첫 사용 후 56% 이탈, 재사용 44%. 반복:탐색 비율 32:1 (거의 같은 기능 반복)
- chathistory NULL 11.6%는 직렬화 사이즈 약 64KB 초과 시 Hevo 적재 실패 가능성
- **학관스 활동일·메시지 카운트 정의 두 가지 공존 (2026-06-08 발견, 2026-06-09 메시지 카운트 추가)**:
  - **운영 대시보드 쿼리**: `qna_sessions.createdat::date` 기반 (세션 생성일만 카운트, 빈 세션도 활성)
  - **weekly_dashboard.sql 문서화 표준**: `chathistory[].timestamp::date` + `msg.role='user'` (실제 메시지 오간 날 + 사용자 의도)
  - 두 정의 같은 retention 비율 산출 (W21 운영=57.6% vs 문서화=63.8%, 모집단 8명 차이) - 비율 본질 유효
  - **유저 행동 정확성 측정 시 timestamp 기반 권장** (다일 세션·이어쓰기·assistant만 응답 빈 세션 구분 필요할 때)
  - 학관스 PM 보고 시 운영 대시보드 수치(createdat 기반)와 일치 여부 확인
  - **메시지 카운트 정의 차이 (2026-06-09 추가)**:
    - 운영 대시보드: `GET_ARRAY_LENGTH(chathistory)` - user + assistant 메시지 합산
    - 본 EDA·문서화 표준: `msg.role = 'user'` 필터링 - user 메시지만
    - 예: user 3개 + assistant 3개 응답 세션 → 운영 대시보드 6개 / 본 EDA 3개로 카운트
    - 학관스 PM 보고 시 절대 수치 약 2배 차이 가능. 비율 결과(상위 사용자 점유율 등)는 거의 동일

## 열린 가설·검증 중
- 챕터 KPI 13% 권장 (현재 10%는 학관스 KPI 미달 시에도 달성 가능)
- 평균 active_weeks 3.27 → 4.0 도달 시 sustained WAU 180 가능

