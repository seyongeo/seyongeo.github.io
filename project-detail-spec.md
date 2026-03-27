# project-detail.html — 페이지 명세서

> 이 문서는 `project-detail.html`의 전체 구조·스타일·동작을 기술한 재현 명세입니다.
> 이 문서만으로 동일한 HTML 페이지를 처음부터 재작성할 수 있도록 작성되었습니다.

---

## 1. 개요

| 항목 | 내용 |
|------|------|
| 파일명 | `project-detail.html` |
| 제목 | `TaskFlow - 프로젝트 진행 상세` |
| 역할 | 단일 프로젝트의 진행 상세 정보를 표시하는 뷰 페이지 |
| 데이터 연동 | `localStorage` 키 `pgDetailData` (JSON)에서 동적 데이터 로드 |
| 연결 페이지 | `project-detail-input.html` (데이터 입력), `project-management-mockup.html` (대시보드) |

---

## 2. 디자인 시스템

### 2-1. 색상 토큰 (CSS Custom Properties)

```css
:root {
  --color-bg:               #0a0e1a;   /* 페이지 배경 (최어두운) */
  --color-surface:          #141925;   /* 카드·사이드바 배경 */
  --color-surface-elevated: #1c2331;   /* 카드 내 강조 배경 */
  --color-border:           #252d3f;   /* 구분선·테두리 */
  --color-text-primary:     #e4e8f0;   /* 본문 텍스트 */
  --color-text-secondary:   #8b95a8;   /* 보조 텍스트 */
  --color-text-tertiary:    #5a6375;   /* 비활성·힌트 텍스트 */
  --color-accent:           #4f7cff;   /* 주요 강조색 (파랑) */
  --color-accent-light:     #6b91ff;   /* 강조색 밝은 변형 */
  --color-success:          #22c55e;   /* 완료·성공 (초록) */
  --color-warning:          #f59e0b;   /* 경고 (황색) */
  --color-danger:           #ef4444;   /* 위험·오류 (빨강) */
  --color-purple:           #a855f7;   /* 보조 강조색 (보라) */
  --color-cyan:             #06b6d4;   /* 보조 강조색 (청록) */
  --shadow-sm:  0 1px 2px rgba(0,0,0,0.3);
  --shadow-md:  0 4px 12px rgba(0,0,0,0.4);
  --shadow-lg:  0 8px 24px rgba(0,0,0,0.5);
}
```

### 2-2. 폰트

- **본문**: `'Instrument Sans'`, fallback: `-apple-system, sans-serif`
- **코드**: `'JetBrains Mono'` (프리뷰 패널 등에서 사용)
- Google Fonts CDN에서 로드:
  `https://fonts.googleapis.com/css2?family=Instrument+Sans:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap`

### 2-3. 배경 장식 (body pseudo-elements)

배경에 두 개의 반투명 radial-gradient 원형 글로우를 `position: fixed`로 배치.

| | 위치 | 크기 | 색상 |
|--|------|------|------|
| `::before` | top: -50%, right: -20% | 800×800px | `rgba(79,124,255,0.15)` (파랑) |
| `::after` | bottom: -30%, left: -10% | 600×600px | `rgba(168,85,247,0.12)` (보라) |

두 요소 모두 `pointer-events: none; z-index: 0`.

### 2-4. 애니메이션

```css
@keyframes slideInLeft  { from { opacity:0; transform: translateX(-20px); } to { opacity:1; transform: translateX(0); } }
@keyframes slideInDown  { from { opacity:0; transform: translateY(-20px); } to { opacity:1; transform: translateY(0); } }
@keyframes fadeInUp     { from { opacity:0; transform: translateY(20px);  } to { opacity:1; transform: translateY(0); } }
```

---

## 3. 페이지 레이아웃

```
┌─────────────────────────────────────────────────────┐
│  .container  (display: grid, grid-template-columns: 280px 1fr, height: 100vh)
│  ┌──────────┬────────────────────────────────────────┤
│  │ .sidebar │ .main-content                          │
│  │ 280px    │  ┌─────────────────────────────────┐   │
│  │          │  │ .header (sticky top bar)        │   │
│  │          │  ├─────────────────────────────────┤   │
│  │          │  │ .content-area (overflow-y:auto) │   │
│  │          │  │  [section cards...]             │   │
│  │          │  └─────────────────────────────────┘   │
│  └──────────┴────────────────────────────────────────┘
```

- `.container`: `position: relative; z-index: 1` (배경 글로우 위에 렌더)
- `.main-content`: `display: flex; flex-direction: column; overflow: hidden`

---

## 4. 사이드바 (`.sidebar`)

```
animation: slideInLeft 0.5s ease-out
background: var(--color-surface)
border-right: 1px solid var(--color-border)
padding: 24px 0
display: flex; flex-direction: column
```

### 4-1. 로고 영역 (`.logo`)

```
padding: 0 24px 24px
display: flex; align-items: center; gap: 12px
```

| 요소 | 스타일 | 내용 |
|------|--------|------|
| `.logo-icon` | 36×36px, border-radius 8px, gradient(accent→purple), 중앙 정렬, font-weight 700, font-size 18px, color white | `TF` |
| `.logo-text` | font-size 20px, font-weight 700, letter-spacing -0.5px | `TaskFlow` |

### 4-2. 내비게이션 (`.nav`)

`flex: 1; padding: 0 12px`
섹션 구분: `.nav-section { margin-bottom: 32px }`
섹션 레이블: `.nav-label` — font-size 11px, uppercase, letter-spacing 1px, color tertiary

**메뉴 아이템 (`.nav-item`):**
- padding 10px 12px, border-radius 8px, color secondary, font-weight 500, font-size 14px
- hover: background surface-elevated, color primary
- `.active`: background accent, color white, box-shadow `0 4px 12px rgba(79,124,255,0.3)`

**메뉴 목록 (섹션 1 — 메인 메뉴):**

| 아이콘 SVG path | 텍스트 | href | 상태 |
|----------------|--------|------|------|
| 바 차트 아이콘 | 대시보드 | `project-management-mockup.html` | 일반 |
| 문서 아이콘 | 프로젝트 | `project-detail.html` | **active** |
| 체크 원형 아이콘 | 계약 | `#` | 일반 |
| 시계 아이콘 | 타임라인 | `#` | 일반 |

**메뉴 목록 (섹션 2 — 워크스페이스):**

| 아이콘 SVG path | 텍스트 | href |
|----------------|--------|------|
| 팀 아이콘 | 팀원 | `#` |
| 캘린더 아이콘 | 캘린더 | `#` |
| 리포트 아이콘 | 리포트 | `#` |

### 4-3. 사용자 프로필 (`.user-profile`)

```
padding: 16px 24px
border-top: 1px solid var(--color-border)
display: flex; align-items: center; gap: 12px
cursor: pointer
hover: background surface-elevated
```

| 요소 | 스타일 | 내용 |
|------|--------|------|
| `.user-avatar` | 36×36px, border-radius 50%, gradient(cyan→accent), font-weight 600, font-size 14px | `김` |
| `.user-name` | font-size 14px, font-weight 600, margin-bottom 2px | `김태희` |
| `.user-role` | font-size 12px, color tertiary | `프로젝트 매니저` |

---

## 5. 상단 헤더 (`.header`)

```
animation: slideInDown 0.5s ease-out
padding: 24px 32px
border-bottom: 1px solid var(--color-border)
background: var(--color-surface)
display: flex; align-items: center; justify-content: space-between
```

### 5-1. 왼쪽 (`.header-left`)

- `h1`: font-size 24px, font-weight 700, letter-spacing -0.5px, margin-bottom 4px
  내용: `메리츠 증권 2단계 개발 — 진행 상세`
- 브레드크럼 (`.breadcrumb`): font-size 13px, color tertiary
  `대시보드` (link, color accent) `/` `프로젝트` (link) `/` `메리츠 증권 2단계 개발`

### 5-2. 오른쪽 (`.header-right`) — 버튼 4개

공통 버튼 스타일 `.btn`: padding 10px 20px, border-radius 8px, font-weight 600, font-size 14px

| 버튼 | 스타일 | 동작 |
|------|--------|------|
| ← 대시보드로 돌아가기 | `.btn-secondary` (bg: surface-elevated, border: border색) | `project-management-mockup.html`로 이동 |
| ✏ 진행상황 입력 | `.btn-secondary` + inline: bg rgba(79,124,255,0.15), border rgba(79,124,255,0.4), color #6b91ff | `project-detail-input.html`로 이동 |
| ↻ 갱신 | id=`refresh-btn`, `.btn-secondary` + inline: bg rgba(34,197,94,0.12), border rgba(34,197,94,0.35), color #22c55e | `refreshData()` 호출 |
| 보고서 출력 | `.btn-primary` (bg: accent, color white, box-shadow) | 미구현 |

---

## 6. 콘텐츠 영역 (`.content-area`)

```
flex: 1; overflow-y: auto; padding: 32px
```

공통 카드 스타일 `.section-card`:
```css
background: var(--color-surface);
border: 1px solid var(--color-border);
border-radius: 16px;
padding: 28px;
margin-bottom: 24px;
animation: fadeInUp 0.5s ease-out both;
```

공통 제목 스타일 `.section-title`:
```css
font-size: 16px; font-weight: 700;
margin-bottom: 20px;
display: flex; align-items: center; gap: 8px;
/* ::before: width 3px, height 16px, background accent, border-radius 2px */
```

콘텐츠 영역에는 아래 4개의 섹션 카드가 순서대로 배치됩니다:
1. 프로젝트 진행 상황 요약 (animation-delay: 0.05s)
2. 마일스톤 진행 현황 (animation-delay: 0.1s)
3. 월별 인력 투입 계획 / 현황 (animation-delay: 0.15s)
4. 월별 비용 계획 / 현황 (animation-delay: 0.2s)

---

## 7. 섹션 1 — 프로젝트 진행 상황 요약

> **동적 ID 매핑**: 아래 요소들은 `loadProjectData()`에 의해 localStorage 데이터로 채워짐

카드: `animation-delay: 0.05s`

### 7-1. 상단 행 (flex, justify-content: space-between)

**왼쪽:**

| 요소 | ID | 스타일 | 기본값 |
|------|----|--------|--------|
| 프로젝트명 `<h2>` | `pg-title` | font-size 22px, font-weight 700, margin-bottom 8px | 메리츠 증권 2단계 개발 |
| 고객사 `<span>` | `pg-client` | font-size 14px, color secondary, flex + 빌딩 SVG 아이콘(16px) | 메리츠화재 |
| 계약기간 `<span>` | `pg-period` | font-size 14px, color secondary, flex + 캘린더 SVG 아이콘(16px) | 2026.01.01 ~ 2026.10.31 |

**오른쪽:**

| 요소 | ID | 클래스 | 기본값 |
|------|----|--------|--------|
| 우선순위 뱃지 | `pg-priority` | `.priority-badge .priority-high` | High |

우선순위 뱃지 스타일:
```css
.priority-badge { padding: 4px 10px; border-radius: 6px; font-size: 11px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.5px; }
.priority-high   { background: rgba(239,68,68,0.15); color: var(--color-danger); }
.priority-medium { background: rgba(245,158,11,0.15); color: var(--color-warning); }
.priority-low    { background: rgba(34,197,94,0.15);  color: var(--color-success); }
```

### 7-2. 전체 진행률 바

```
margin-bottom: 12px
```

| 요소 | ID | 스타일 | 기본값 |
|------|----|--------|--------|
| 레이블 텍스트 | — | font-size 13px, font-weight 600, color secondary | `전체 진행률` |
| 퍼센트 텍스트 | `pg-progress-text` | font-size 13px, font-weight 700, color accent | `65%` |
| 바 트랙 | — | height 8px, background border색, border-radius 4px, overflow hidden | — |
| 바 필 | `pg-progress-bar` | height 100%, width=진행률%, gradient(accent→accent-light), border-radius 4px | width: 65% |

### 7-3. 하단 통계 그리드

`display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px; margin-top: 20px`

| 컬럼 | 레이블 | 값 요소 ID | 기본값 |
|------|--------|-----------|--------|
| 경과 일수 | font-size 12px, color tertiary | `pg-elapsed` | `42일` |
| 남은 일수 | font-size 12px, color tertiary | `pg-remain` | `261일` |
| 담당자 | font-size 12px, color tertiary | — | — |

값 스타일: font-size 22px, font-weight 700

**담당자 영역** (flex, align-items: center, gap: 8px):

| 요소 | ID | 스타일 | 기본값 |
|------|----|--------|--------|
| 아바타 원형 | `pg-manager-avatar` | `.assignee-avatar` (28×28px, border-radius 50%, gradient(accent→purple), font-size 12px, font-weight 600) | `박` |
| 이름 | `pg-manager-name` | font-size 14px, font-weight 600 | `박민수` |

---

## 8. 섹션 2 — 마일스톤 진행 현황

카드: `animation-delay: 0.1s`
제목: `마일스톤 진행 현황` (`.section-title` 클래스)

### 8-1. 타임라인 컨테이너

```css
#pg-milestones {
  position: relative;
  padding-left: 32px;
}
/* 수직 연결선 */
#pg-milestones::before (또는 내부 div) {
  position: absolute; left: 11px; top: 24px; bottom: 24px;
  width: 2px; background: var(--color-border);
}
```

### 8-2. 마일스톤 아이템 구조

각 아이템: `position: relative; margin-bottom: 28px` (마지막 항목은 margin-bottom: 0)

**상태별 도트(타임라인 마커) — position: absolute; left: -32px; top: 4px; width/height: 24px:**

| 상태 | 배경색 | 테두리 | 내부 아이콘 |
|------|--------|--------|-------------|
| 완료 | `var(--color-success)` | `3px solid var(--color-surface)` | 체크 SVG (stroke white, stroke-width 3) |
| 진행중 | `var(--color-accent)` | `3px solid var(--color-surface)` + box-shadow `0 0 0 4px rgba(79,124,255,0.2)` | 흰색 원형 점 (8×8px) |
| 대기 | `var(--color-surface-elevated)` | `2px solid var(--color-border)` | 없음 |

**상태별 카드 스타일 (`border-radius: 10px; padding: 16px`):**

| 상태 | background | border | 추가 |
|------|-----------|--------|------|
| 완료 | `var(--color-surface-elevated)` | `1px solid var(--color-border)` | — |
| 진행중 | `var(--color-surface-elevated)` | `1px solid var(--color-accent)` | `box-shadow: 0 0 0 1px rgba(79,124,255,0.1)` |
| 대기 | `var(--color-surface-elevated)` | `1px solid var(--color-border)` | `opacity: 0.7` |

**카드 내부 구조:**

```
[상단 행: flex, justify-content: space-between]
  ├─ [단계번호+단계명]  font-size 15px, font-weight 600, margin-bottom 4px
  ├─ [기간]            font-size 13px, color tertiary  (YYYY.MM.DD ~ YYYY.MM.DD 형식)
  └─ [상태 뱃지]       font-size 12px, padding 4px 10px, border-radius 6px

[작업 내용]  font-size 13px, "• 항목\n• 항목" 형식
  - 완료/진행중: color secondary
  - 대기: color tertiary

[단계 진행률 바] ← 진행중 상태일 때만 표시
  - 상단: 레이블 "단계 진행률" + 퍼센트 (color accent)
  - 바: height 6px, background border색, border-radius 3px
  - 필: background accent, width = 단계_진행률%
```

**상태별 뱃지 색상:**

| 상태 | background | color |
|------|-----------|-------|
| 완료 | `rgba(34,197,94,0.15)` | `var(--color-success)` |
| 진행중 | `rgba(79,124,255,0.15)` | `var(--color-accent)` |
| 대기 | `rgba(90,99,117,0.15)` | `var(--color-text-tertiary)` |

### 8-3. 기본 샘플 데이터 (4개 마일스톤)

| 단계 | 단계명 | 기간 | 상태 | 작업 내용 | 진행률 |
|------|--------|------|------|-----------|--------|
| 1 | 요구사항 분석 | 2026.01.01 ~ 2026.02.15 | 완료 | 사용자 요구사항 정의서 작성 완료 / 시스템 아키텍처 설계 완료 | — |
| 2 | 개발 및 구현 | 2026.02.16 ~ 2026.07.31 | 진행중 | 프론트엔드 개발 70% 완료 / 백엔드 API 개발 60% 완료 / 데이터베이스 스키마 구축 완료 | 65% |
| 3 | 테스트 및 QA | 2026.08.01 ~ 2026.09.15 | 대기 | 통합 테스트 예정 / 성능 테스트 예정 / 사용자 인수 테스트 예정 | — |
| 4 | 배포 및 안정화 | 2026.09.16 ~ 2026.10.31 | 대기 | 운영 환경 배포 예정 / 모니터링 및 안정화 예정 / 최종 인수 및 종료 예정 | — |

---

## 9. 섹션 3 — 월별 인력 투입 계획 / 현황

카드: `animation-delay: 0.15s`
제목: `월별 인력 투입 계획 / 현황` (`.section-title` 클래스)

### 9-1. 테이블 컨테이너

```html
<div style="overflow-x: auto; border-radius: 12px; border: 1px solid var(--color-border);">
  <table style="width:100%; border-collapse:collapse; font-size:12px; min-width:900px;">
```

**colgroup 너비 설정:**

| 컬럼 | 너비 |
|------|------|
| 조직 | 80px |
| 소속 | 60px |
| 등급 | 60px |
| 역할 | 100px |
| 01~10월 (10개) | 52px each (span="10") |
| 합계 | 64px |
| 비고 | 64px |

### 9-2. 테이블 헤더 구조 (2행)

**1행 (colspan/rowspan):**

| 컬럼 | rowspan | colspan | background | color | 내용 |
|------|---------|---------|-----------|-------|------|
| 조직 | 2 | — | `#1c2331` | `#8b95a8` | 조직 |
| 소속 | 2 | — | `#1c2331` | `#8b95a8` | 소속 |
| 등급 | 2 | — | `#1c2331` | `#8b95a8` | 등급 |
| 역할 | 2 | — | `#1c2331` | `#8b95a8` | 역할 |
| 연도 | — | 10 | `#1a2540` | `#6b91ff` (font-weight 700) | `2026` |
| 합계 | 2 | — | `#1c2331` | `#f59e0b` (font-weight 700) | 합계 |
| 비고 | 2 | — | `#1c2331` | `#8b95a8` | 비고 |

**2행 (월 컬럼 10개):**
- background: `#141925`, color: `#8b95a8`, font-weight 600
- border-bottom: `2px solid var(--color-accent)` (강조 구분선)
- 내용: `01` ~ `10`

모든 헤더 셀 공통: `padding: 10px 8px; text-align: center; border-right: 1px solid var(--color-border)`

### 9-3. 데이터 행 구조 (계획/실적 쌍)

각 역할은 **계획 행 + 실적 행** 2행 한 쌍으로 구성. 조직·소속·등급·역할 셀은 rowspan="2".

**계획 행:**
- 월별 값: color `#e4e8f0`, font-weight 500
- 합계: color `#f59e0b`, font-weight 700
- 비고: `계획` (color `#5a6375`, font-size 11px)
- row style: `border-bottom: 1px solid #1e2738`

**실적 행:**
- 실적이 계획과 일치: color `#22c55e`
- 실적이 계획과 차이 발생: color `#ef4444`
- 미집행(-): color `#5a6375`
- 합계: color `#22c55e`, font-weight 700
- 비고: `실적` (color `#5a6375`, font-size 11px)
- row style: `border-bottom: 1px solid var(--color-border); background: rgba(79,124,255,0.03)`

**등급 뱃지 스타일:**

| 등급 | background | color |
|------|-----------|-------|
| 특급 | `rgba(79,124,255,0.15)` | `#6b91ff` |
| 고급 | `rgba(168,85,247,0.15)` | `#a855f7` |
| 중급 | `rgba(34,197,94,0.15)` | `#22c55e` |
| 초급 | `rgba(245,158,11,0.15)` | `#f59e0b` |

뱃지 공통: `padding: 2px 7px; border-radius: 4px; font-weight 600; font-size 11px`

### 9-4. 샘플 데이터 (5개 역할)

| 조직 | 소속 | 등급 | 역할 | 01~10월 계획(MM) | 합계(계획) | 실적(집행월) |
|------|------|------|------|-----------------|-----------|------------|
| SI사업 2팀 | 당사 | 특급 | PM/PL | 1.00×10 | 10.00 | 1.00, 1.00, 0.80 (3월 차이) |
| SI사업 2팀 | 당사 | 고급 | 사업관리 | 1.00×10 | 10.00 | 1.00, 1.00, 1.00 |
| SI사업 2팀 | 외주 | 고급 | 업무분석/설계 | 3,3,3,3,3,3,3,2,2,2 | 28.00 | 3.00, 3.00, 2.50 (3월 차이) |
| SI사업 2팀 | 외주 | 중급 | 개발 | 2,2,4,4,4,4,4,4,2,2 | 32.00 | 2.00, 2.00, 4.00 |
| SI사업 2팀 | 외주 | 초급 | 개발 | 2,2,4,4,4,4,4,4,2,2 | 32.00 | 2.00, 2.00, 3.50 (3월 차이) |

### 9-5. 합계 행 (2행)

**전체 합계(계획) 행:**
- background: `#1a2540`
- colspan="4" 셀: `전체 합계 (계획)`, text-align right, color `#6b91ff`, font-weight 700, font-size 13px
- 월별 합계: color `#6b91ff`, font-weight 700
  (01: 9, 02: 9, 03: 13, 04~07: 13, 08: 12, 09: 10, 10: 8)
- 총합계: `112.00`, color `#f59e0b`, font-size 14px

**전체 합계(실적) 행:**
- background: `#152030`
- colspan="4" 셀: `전체 합계 (실적)`, text-align right, color `#22c55e`, font-weight 700, font-size 13px
- 실적 월: 01→9.00(green), 02→9.00(green), 03→11.80(red/차이), 04~10→`-`(tertiary)
- 총합계: `29.80`, color `#22c55e`, font-size 14px

### 9-6. 범례 (테이블 하단)

`display: flex; gap: 20px; margin-top: 12px; font-size: 12px`

| 색상 사각형 | 레이블 |
|------------|--------|
| `#e4e8f0` (10×10px, border-radius 2px) | 계획 (MM) |
| `#22c55e` | 실적 일치 |
| `#ef4444` | 실적 차이 발생 |
| `#5a6375` | 미집행 |

---

## 10. 섹션 4 — 월별 비용 계획 / 현황

카드: `animation-delay: 0.2s`

### 10-1. 섹션 헤더

제목과 단위 표시를 `flex, justify-content: space-between`으로 배치:
- 좌: `.section-title` (margin-bottom: 0) — `월별 비용 계획 / 현황`
- 우: `단위: 만원` (font-size 12px, color tertiary)

### 10-2. 테이블 컨테이너

```html
<div style="overflow-x:auto; border-radius:12px; border:1px solid var(--color-border);">
  <table style="width:100%; border-collapse:collapse; font-size:12px; min-width:900px;">
```

**colgroup 너비 설정:**

| 컬럼 | 너비 |
|------|------|
| 조직 | 80px |
| 소속 | 60px |
| 등급 | 60px |
| 역할 | 100px |
| 01~10월 (10개) | 60px each |
| 합계(만원) | 80px |
| 비고 | 64px |

### 10-3. 테이블 헤더 구조 (섹션 3과 동일, 연도 표기 차이)

1행의 연도 컬럼 내용: `2026 (만원)` (color `#6b91ff`, font-weight 700)
합계 컬럼 내용: `합계(만원)` (color `#f59e0b`, font-weight 700)

나머지 헤더 구조(rowspan, 스타일)는 섹션 3과 동일.

### 10-4. 데이터 행 구조 (계획/실적 쌍)

섹션 3과 동일한 행 구조(rowspan, 계획/실적 쌍, 등급 뱃지).
값은 MM 대신 만원 단위 금액. 금액은 `text-align: right`.

**계획 행 금액 스타일:** color `#e4e8f0`, font-weight 500
**실적 일치:** color `#22c55e`
**실적 차이:** color `#ef4444`
**미집행:** `-`, color `#5a6375`
**합계 셀 padding:** `padding: 6px 8px` (섹션3보다 우측 여백 추가)

### 10-5. 샘플 데이터 (5개 역할)

| 조직 | 소속 | 등급 | 역할 | 월 단가(계획) | 합계(계획) | 실적 집행 |
|------|------|------|------|-------------|-----------|----------|
| SI사업 2팀 | 당사 | 특급 | PM/PL | 900만원/월 | 9,000 | 920, 920, 736 (3월 차이) |
| SI사업 2팀 | 당사 | 고급 | 사업관리 | 750만원/월 | 7,500 | 780, 780, 780 |
| SI사업 2팀 | 외주 | 고급 | 업무분석/설계 | 2,250→1,500 (8월부터) | 21,000 | 2,340, 2,340, 1,950 (3월 차이) |
| SI사업 2팀 | 외주 | 중급 | 개발 | 1,100~2,200 (MM비례) | 17,600 | 1,120, 1,120, 2,240 |
| SI사업 2팀 | 외주 | 초급 | 개발 | 800~1,600 (MM비례) | 12,800 | 820, 820, 1,435 (3월 차이) |

### 10-6. 합계 행 (2행)

**전체 합계(계획) 행:**
- background: `#1a2540`
- 월별: 5,800 / 5,800 / 7,700 / 7,700 / 7,700 / 7,700 / 7,700 / 6,950 / 5,050 / 5,050
- 총합계: `67,900`, color `#f59e0b`, font-size 14px

**전체 합계(실적) 행:**
- background: `#152030`
- 01→5,980(green), 02→5,980(green), 03→7,141(red/차이), 04~10→`-`
- 총합계: `19,101`, color `#22c55e`, font-size 14px

### 10-7. 단가 범례

`display: flex; gap: 20px; margin-top: 12px; font-size: 12px; flex-wrap: wrap; align-items: center`

계획/실적 두 그룹으로 구분 (구분선: `width:1px; height:16px; background:var(--color-border)`):

**계획 단가:**

| 뱃지 | 단가 |
|------|------|
| 특급 (blue) | 900만원/MM |
| 고급 (purple) | 750만원/MM |
| 중급 (green) | 550만원/MM |
| 초급 (yellow) | 400만원/MM |

**실적 단가:**

| 뱃지 | 단가 |
|------|------|
| 특급 (blue) | 920만원/MM |
| 고급 (purple) | 780만원/MM |
| 중급 (green) | 560만원/MM |
| 초급 (yellow) | 410만원/MM |

뱃지 스타일: `padding: 1px 8px; border-radius: 4px; font-weight 600; font-size 11px` (섹션 3 등급 뱃지와 동일 색상)

### 10-8. 예산 요약 노트

```html
<div style="margin-top:10px; padding:10px 16px;
  background:rgba(79,124,255,0.05); border:1px solid rgba(79,124,255,0.2);
  border-radius:8px; font-size:12px; color:#5a6375;">
  ※ 총 계획 예산: <span style="color:#f59e0b;font-weight:700;">6억 7,900만원</span>
  | 실적 집행: <span style="color:#22c55e;font-weight:700;">1억 9,101만원</span>
  | 집행률: <span style="color:#6b91ff;font-weight:700;">28.1%</span>
  | 단가 차이로 인한 비용 증가: <span style="color:#ef4444;font-weight:700;">+556만원</span>
</div>
```

---

## 11. 데이터 연동 (JavaScript)

### 11-1. localStorage 데이터 구조 (키: `pgDetailData`)

```json
{
  "프로젝트_진행_상황_요약": {
    "프로젝트명": "",
    "고객사명": "",
    "계약_시작일": "YYYY-MM-DD",
    "계약_종료일": "YYYY-MM-DD",
    "우선순위": "High | Medium | Low",
    "전체_진행률": "65%",
    "경과_일수": "42일",
    "남은_일수": "261일",
    "담당자": ""
  },
  "마일스톤_진행현황": [
    {
      "단계_번호": 1,
      "단계명": "",
      "시작일": "YYYY-MM-DD",
      "종료일": "YYYY-MM-DD",
      "상태": "완료 | 진행중 | 대기",
      "작업_내용": ["항목1", "항목2", "항목3"],
      "단계_진행률": "65%"
    }
  ]
}
```

> **참고**: 섹션 3(인력 투입)·섹션 4(비용)는 현재 정적 데이터로 표시되며 localStorage 동적 연동 미구현.

### 11-2. 함수 목록

| 함수명 | 트리거 | 동작 |
|--------|--------|------|
| `loadProjectData()` | `DOMContentLoaded` | localStorage에서 JSON 파싱 → 섹션1 필드 업데이트 → 마일스톤 동적 렌더링 |
| `refreshData()` | `↻ 갱신` 버튼 클릭 | `loadProjectData()` 재호출 + 버튼 피드백 1.5초 ("✓ 갱신 완료") |

### 11-3. `loadProjectData()` 로직 흐름

```
1. localStorage.getItem('pgDetailData') → JSON.parse
2. 프로젝트_진행_상황_요약 처리:
   - getElementById(id).textContent = 값
   - pg-period: 날짜 "-" → "." 변환 후 "시작 ~ 종료" 형식
   - pg-priority: textContent + className 동시 변경
   - pg-progress-text: 퍼센트 문자열
   - pg-progress-bar: style.width = 퍼센트%
   - pg-manager-avatar: textContent = 이름 첫 글자
3. 마일스톤_진행현황 처리:
   - container.innerHTML = milestones.map(renderMilestoneItem).join('')
   - 각 아이템: 상태에 따라 도트·카드·뱃지 스타일 분기
   - 날짜 "-" → "." 변환
   - 단계_진행률는 진행중 상태일 때만 진행바 렌더링
```

---

## 12. 연관 파일

| 파일 | 역할 |
|------|------|
| `project-detail.html` | 이 명세의 대상 파일 (뷰) |
| `project-detail2.html` | 이 명세로 재생성된 파일 |
| `project-detail-input.html` | 진행 상황 데이터 입력 폼 (`pgDetailData` localStorage 저장) |
| `project-detail-data.json` | 데이터 구조 템플릿 (JSON 스키마 참고용) |
| `project-management-mockup.html` | 대시보드 (현재 링크만 존재) |
