---
name: squad_career_v2
description: 커리어 스쿼드 v2 - 51 컬럼 + 수료일 이후 활동 정책, 옵션B 신청자 한정 결과확인 (2026-04-28)
type: project
originSessionId: 94377727-a791-4869-a178-b12b609f41cf
---
# 커리어 스쿼드 v2 (2026-04-28 최신)

## 파일 위치
- v1 (제품실 KPI): `/Users/bokyeongkim/claudedocs/career/커리어_지표한판_weekly.sql`
- v2 (스쿼드 weekly): `/Users/bokyeongkim/claudedocs/career/커리어_v2_스쿼드_weekly.sql`
- 컬럼 가이드: `/Users/bokyeongkim/claudedocs/career/커리어_v2_컬럼_종합가이드.md`
- 분포 분석: `/Users/bokyeongkim/claudedocs/career/커리어_재방문분포_분석.sql`
- Cohort 분석: `/Users/bokyeongkim/claudedocs/career/커리어_cohort_분석.sql`

## 모수 정의 (v1, v2 공통)
- target_graduates (696명) = 2026 수료 + dropout_reason IN ('정상수료','80%이상수료') + hrd_category='실업자'
- eligible_users (~609명) = target_graduates ∩ career_visitors (crr_%_view 1회+)
- 모든 행동 메트릭 → eligible_users 모수에서만 카운트

## 핵심 정책 결정사항

### 1. 활동 시점 정책 (2026-04-28 v3)
- **위클리 (v2 SQL)**: graduation_date -14일 ~ +180일 윈도우만 카운트
  - 수료 직전 진입 효과 + 수료 후 6개월 추적 가능
  - PM 협의 결과 (취업 지원 세션 진행 일수 ≈ 14일 가정)
- **트랙별 SQL**: 윈도우 없음 (lifetime, cohort 누적 관점)
  - 트랙 전체의 cohort retention 측정용
- **v1 (제품실 KPI)**: graduation_date 이후 (현재 그대로)

### 2. 취업대상자 누적 (2026-04-27)
- weekly_active_pool에서 미취업 조건(is_employed=FALSE) 제거 → 누적 모수
- 컬럼명: `취업대상자(미취업)` → `취업대상자(누적)`
- 단조 증가 (취업해도 빠지지 않음)

### 3. 트랙 기반 모수 (2026-04-27)
- 자력/바로인턴/꿀알바 모수 5개 → 난항/순항/꿀알바 3개로 단순화
- is_hard_track 기반 (모든 수료자에 set 되어 있음, employment_type 의존 X)
- 검증: 난항_모수 + 순항_모수 = 꿀알바_모수 (= 전체 모수)

### 4. 결과확인 옵션 B (2026-04-28)
- 신청자 한정으로 결과확인 카운트 (직접 접근자 제외)
- coaching_applicants CTE 추가 (3개 코칭 타입)
- 결과: 결과확인 ≤ 신청 보장 (정상 funnel)

## v2 SQL 컬럼 구조 (51개, A~AY)

### 섹터 1: 유입 (1-11)
1. 주차, 2. 주시작일, 3. 신규수료대상자, 4. 취업대상자(누적), 5. WAU, 6. MAU(4주롤링),
7. Weekly리텐션(%), 8. Monthly리텐션(%), 9. WAU(기능사용), 10-11. Weekly/Monthly리텐션(기능사용)

### 섹터 2: 행동 - 취업코칭 DB (12-18)
12. AI이력서_신청유저수 (CANCEL/TEMP 제외)
13. AI이력서_신청건수
14. AI이력서_PASS유저수 (PASS + FEEDBACK_PASS)
15. AI이력서_PASS건수
16. AI이력서_첫Pass유저수, 17. NonPass, 18. 재Pass

### 섹터 2: 행동 - 취업코칭 Amplitude (19-26)
19. AI이력서코칭_신청유저수 (crr_coachingPage_modal_click + button_text='신청하기')
20. AI이력서코칭_결과확인유저수 (신청자 한정, first view)
21-22. 면접코칭_신청/결과확인 (결과확인은 신청자 한정 first view)
23-24. 포트폴리오_첫/재시도
25-26. 채용공고맞춤형_신청/결과확인 (결과확인 신청자 한정)

### 섹터 2: 행동 - 이력서/지원내역/커리어톤/프로필 (27-39)
27. 이력서_생성, 28. 페이지뷰, 29. 저장
30. 지원내역_페이지뷰, 31. 공고추가하기버튼 (modal_view), 32. 공고등록 (modal_click), 33. 공고상태변경
34. 커리어톤_미션, 35. 리워드
36. 첫방문, 37. 같은주재방문 (graduation 후 first/second visit)
38. 프로필페이지진입, 39. 프로필완성

### 섹터 2: Flow (40-41)
40. AI이력서코칭PASS수(임시), 41. 최초공고등록수(임시)

### 섹터 3: 취업 (42-51)
42. 신규취업수, 43. 취업성취도(NULL placeholder)
44-48. 자력_난항/순항, 바로인턴_난항/순항, 꿀알바
49-51. 난항_모수, 순항_모수, 꿀알바_모수

## 주요 이벤트 정의 (Amplitude)
- 신청: crr_coachingPage_modal_click + coaching_type + button_text='신청하기' (AI이력서)
- 신청: crr_evaluation_reserve_completed + coaching_name (면접/채용공고/포폴)
- 결과확인: crr_evaluation_page_view + coaching_type + page_title='결과 페이지'
- 공고 추가버튼: crr_applyHistory_modal_view + modal_title='공고 추가'
- 공고 등록완료: crr_applyHistory_modal_click + modal_title='공고 추가' + button_text='추가하기'

## 시트 구조 (커리어 스쿼드 상세지표)
- Raw 시트명: **커리어_위클리스냅샷_raw**
- 주차 헤더: G4 셀에 시작 (G4=2026-01W, H4=2026-02W, ...)
- 시계열 수식: `=IFERROR(VLOOKUP(G$4, 커리어_위클리스냅샷_raw!$A:$AY, [컬럼번호], FALSE), "")`
- 현재 단일 값: `=IFERROR(VLOOKUP($F$3, 커리어_위클리스냅샷_raw!$A:$AY, [컬럼번호], FALSE), "")`

## 시트 카테고리 구조 (B/C/D 계층)
- 주요 지표: 취업률, 취업대상자, 신규 취업자 수
- 커리어 유입: 신규수료, WAU, **WAU 활성화율**, **평균 WAU(4주롤링)**, MAU, 리텐션, 재방문율, 프로필완성률, 취업성취도
- 행동 > **취업지원 기능**: AI 이력서 코칭율 (Amp), 면접 코칭율, 공고기록 (공고추가/등록완료)
- 행동 > **취업코칭** (DB): AI 이력서 코칭, 포트폴리오, 채용공고맞춤형, **리드타임** (미구현)
- 이력서 관리, 지원내역 (+ **재등록 유저수** = 공고등록 - 최초공고등록), 커리어톤
- 취업률 세부: **난항/순항 수료생 수** (모수), 자력/바로인턴 (난항/순항 + 취업률), 꿀알바

## 미구현 / 보류
- 20건+지원달성수 — 의도적 제외
- 취업 성취도 — NULL placeholder (정의 미정)
- AI 이력서 리드타임 — 신청→첫PASS 평균 일수 (보류, 사용자 요청 시 추가)
- hrd_category mutability — 사용자 검증 필요

## KPI 권장 (논의 결과)
- **North Star**: 챕터3 8주 평균 WAU ≥ 180 (sustained)
- Driver: Regular+ 비율 ≥ 36%, "1주만" 비율 ≤ 18%
- 핵심 인사이트: 51.6%가 2주 이내 이탈 → 챕터3 미션 = "정착자 풀 두껍게 만들기"
- 분포 데이터 기반 시뮬레이션: 평균 active_weeks 3.27 → 4.0이면 sustained 180 가능

---

## 최근 4주 실험·릴리스
- (위클리 미팅 후 한 줄씩 누적)

## 알려진 외부 이벤트
- (연휴·캠페인·정책 변경 등을 일자와 함께 누적)

## 주의할 패턴·해석 가이드
- 첫방문→재방문은 주간 코호트 + 14일 기한. 최근 2주 코호트는 미확정
- 51.6%가 2주 이내 이탈 — 단기 지표보다 8주 sustained WAU가 신호로 깔끔
- 활동 시점: 위클리 SQL은 graduation_date -14일 ~ +180일 윈도우. 트랙별 SQL은 lifetime (윈도우 없음)

## 열린 가설·검증 중
- (검증 진행 중 가설을 누적)

