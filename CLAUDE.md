# CLAUDE.md — seyongeo.github.io

이 파일은 이 저장소에서 작업하는 AI 어시스턴트(Claude Code 등)를 위한 프로젝트 맥락 문서입니다.

---

## 프로젝트 개요

한국 기업용 **정적 HTML 프로젝트 관리 애플리케이션**입니다. 빌드 도구, 패키지 매니저, 백엔드 서버가 없으며, 모든 파일은 GitHub Pages를 통해 `main` 브랜치에서 그대로 배포됩니다.

- **배포 URL:** https://seyongeo.github.io
- **도메인 언어:** 한국어 (프로젝트 데이터, 라벨, UI 텍스트 전체)
- **렌더링 방식:** 브라우저에서 직접 실행 (서버 불필요)

---

## 디렉터리 구조

```
seyongeo.github.io/
│
├── index.html                      # 메인 랜딩 페이지 — "seyongeo Workspace"
│                                   #   Dashboard / Explorer 듀얼 뷰 토글
│
├── project-detail.html             # 프로젝트 상세 뷰 (기본)
├── project-detail2.html            # 디자인 변형 1
├── project-detail3.html            # 디자인 변형 2
├── project-detail-input.html       # 데이터 입력 폼 (localStorage에 저장)
├── project-detail-fields.html      # 폼 필드 컴포넌트 템플릿
│
├── project-management.html         # 대시보드 (레거시, 단순 스타일)
├── project-management-view.html    # 대시보드 변형
├── project-management-w2.html      # 대시보드 변형
│
├── project-detail-spec.md          # 상세 페이지 전체 디자인 명세서 (~26KB)
├── project-detail-data.json        # JSON 데이터 스키마 (한국어 필드명)
│
├── CLAUDE.md                       # 이 파일 — AI 어시스턴트용 문서
├── test_memo.txt                   # 스크래치 파일
├── files.zip                       # 이전 버전 아카이브
├── files/                          # 이전 버전 HTML 파일들
└── files 2/                        # 중복 아카이브
```

---

## 기술 스택

| 레이어 | 기술 |
|--------|------|
| 마크업 | HTML5 (템플릿·전처리기 없음) |
| 스타일 | CSS3 — 커스텀 프로퍼티, `<style>` 태그 인라인 |
| 스크립팅 | 바닐라 JavaScript — `<script>` 태그 인라인 |
| 폰트 | Google Fonts CDN — Instrument Sans + JetBrains Mono |
| 데이터 저장 | 브라우저 `localStorage` (키: `pgDetailData`) |
| 배포 | GitHub Pages (정적 파일 그대로) |
| 빌드 도구 | **없음** |
| 패키지 매니저 | **없음** |

---

## 로컬 개발 방법

### HTML 파일 직접 열기 (서버 불필요)

```bash
open index.html
open project-detail.html
```

### 간단한 정적 서버 (라이브 리로드가 필요한 경우)

```bash
python3 -m http.server 8080
# 접속: http://localhost:8080
```

### 데이터 흐름

1. `project-detail-input.html` — 사용자가 폼에 프로젝트 데이터 입력
2. 폼 제출 시 JSON을 `localStorage`의 `pgDetailData` 키에 저장
3. `project-detail.html` — 로드 시 `pgDetailData`를 읽어 동적으로 UI 렌더링

---

## 디자인 시스템

모든 페이지는 CSS 커스텀 프로퍼티로 정의된 일관된 **다크 테마** 디자인 시스템을 공유합니다.

### 컬러 팔레트

```css
--color-bg:               #0a0e1a   /* 가장 어두운 배경 */
--color-surface:          #141925   /* 카드 표면 */
--color-surface-elevated: #1c2331   /* 상위 요소 */
--color-border:           #252d3f   /* 테두리 */
--color-text-primary:     #e4e8f0   /* 기본 텍스트 */
--color-text-secondary:   #8b95a8   /* 보조 텍스트 */
--color-text-tertiary:    #5a6375   /* 흐린 텍스트 */
--color-accent:           #4f7cff   /* 파란색 강조 */
--color-accent-light:     #6b91ff
--color-success:          #22c55e   /* 초록색 */
--color-warning:          #f59e0b   /* 황색 */
--color-danger:           #ef4444   /* 빨간색 */
--color-purple:           #a855f7
--color-cyan:             #06b6d4
```

### 타이포그래피

- **기본 폰트:** `Instrument Sans` (굵기: 400, 500, 600, 700)
- **고정폭 폰트:** `JetBrains Mono` (굵기: 400, 500)

### 레이아웃 패턴

- 고정 사이드바: 너비 280px, `slideInLeft 0.5s ease-out` 애니메이션
- 고정 헤더: `slideInDown 0.5s ease-out`
- 콘텐츠 카드: `fadeInUp 0.5s ease-out` (딜레이 단계적 적용)
- 카드 그리드: CSS Grid / 행 레이아웃: Flexbox

### 애니메이션

```css
slideInLeft   0.5s ease-out   /* 사이드바 */
slideInDown   0.5s ease-out   /* 헤더 */
fadeInUp      0.5s ease-out   /* 카드 (animation-delay로 순차 등장) */
```

---

## 주요 컨벤션

### HTML

- CSS는 각 HTML 파일의 `<style>` 태그에 인라인으로 포함 — 외부 스타일시트 없음.
- JavaScript는 `<script>` 태그에 인라인 — 외부 `.js` 파일 없음.
- 각 HTML 파일은 독립적으로 배포 가능한 자체 완결형 구조.
- 공유 에셋 파일을 만드는 것보다 파일 간 스타일/스크립트를 복제하는 기존 패턴을 유지.

### CSS

- 모든 색상과 일관된 값은 CSS 커스텀 프로퍼티(변수)를 사용.
- 유틸리티 클래스 프레임워크(Tailwind, Bootstrap 등) 도입 금지.
- 다크 테마를 유지 — 밝은 배경 도입 금지.

### JavaScript

- 바닐라 JS만 사용 — 프레임워크(React, Vue 등) 금지.
- DOM 조작은 `querySelector`/`querySelectorAll`.
- 데이터 저장은 반드시 `localStorage` 키 `pgDetailData` 사용.
- 외부 API `fetch` 호출 없음.

### 한국어 규칙

- 모든 UI 라벨, 데이터 필드, 사용자 표시 텍스트는 한국어.
- JSON 데이터 스키마는 한국어 키 사용 (예: `프로젝트명`, `담당자`, `마일스톤_진행현황`).
- 상태값: `완료` / `진행중` / `대기`.
- 인력 등급: `특급` / `고급` / `중급` / `초급`.

### 파일 명명 규칙

- 변형 페이지는 숫자 접미사 사용: `project-detail.html`, `project-detail2.html`, `project-detail3.html`.
- 파일명 전체 케밥케이스(kebab-case).

---

## 데이터 스키마

`localStorage`에 저장되는 기본 데이터 구조:

```json
{
  "프로젝트_진행_상황_요약": {
    "프로젝트명": "",
    "고객사명": "",
    "계약_시작일": "",
    "계약_종료일": "",
    "우선순위": "",
    "전체_진행률": "",
    "경과_일수": "",
    "남은_일수": "",
    "담당자": ""
  },
  "마일스톤_진행현황": [
    {
      "단계_번호": 1,
      "단계명": "",
      "시작일": "",
      "종료일": "",
      "상태": "",
      "작업_내용": [],
      "단계_진행률": ""
    }
  ],
  "월별_인력_투입_계획_및_실적": [],
  "월별_비용_집행_계획_및_실적": []
}
```

---

## 프로젝트 상세 페이지 콘텐츠 섹션

`project-detail.html`의 4개 주요 섹션:

1. **프로젝트 진행 상황 요약** — 상태, 기간, 담당자, 진행률
2. **마일스톤 진행 현황** — 4단계 타임라인 (완료율 포함)
3. **월별 인력 투입 계획 및 실적** — 등급별 인원 표
4. **월별 비용 집행 계획 및 실적** — 월별 예산 추적

---

## Git 워크플로우

- **메인 브랜치:** `main`
- **기능 브랜치:** `claude/<설명>` 또는 `feature/<설명>` 형식
- CI/CD 파이프라인 없음 — `main` 푸시 시 GitHub Pages 자동 배포 (보통 1분 이내)
- 커밋 메시지: 명령형, 간결한 설명 (예: "Replace dashboard table with card grid layout")

---

## 참고 문서

- **`project-detail-spec.md`** — `project-detail.html`의 시각적·기능적 명세서 전체. 상세 페이지 레이아웃, 섹션, 데이터 렌더링 로직 변경 전 반드시 참고.
- **`project-detail-data.json`** — 정식 JSON 스키마. 모든 필드명과 데이터 구조의 기준.

---

## 도입 금지 항목

아래 항목은 명시적 지시 없이 이 프로젝트에 도입하지 않습니다:

- 빌드 도구 (webpack, vite, rollup, esbuild)
- 패키지 매니저 (npm, yarn, pnpm)
- CSS 전처리기 (Sass, Less)
- JavaScript 프레임워크 (React, Vue, Svelte 등)
- TypeScript
- 백엔드 서버 또는 데이터베이스
- 테스트 프레임워크
- CI/CD 파이프라인
