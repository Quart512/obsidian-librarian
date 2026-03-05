# Obsidian Librarian 개발 로드맵

AI 기반 노트 요약/분류 및 토큰 효율적 자연어 검색으로 대규모 Obsidian 볼트의 정보 과부하 문제를 해결하는 플러그인.

## 개요

Obsidian Librarian은 수백~수천 개의 노트를 관리하는 지식 노동자 및 연구자를 위한 AI 사서 플러그인으로 다음 기능을 제공합니다:

- **노트 자동 요약 및 분류 (F001, F002)**: 볼트 전체 스캔 후 Claude API로 1~2문장 요약과 태그를 자동 생성
- **사용자 주도 검토 및 적용 (F003)**: AI 추천을 검토하고 개별/일괄 수락하여 frontmatter에 적용
- **토큰 효율적 자연어 검색 (F004, F005)**: 전체 본문 대신 요약 인덱스(800개 노트 기준 24K 토큰)를 활용한 의미 기반 검색
- **유연한 설정 관리 (F006)**: API 키, 모델 선택, 동시 요청 수 등 사용자 맞춤 설정
- **통합 사이드바 패널 (F007)**: 분류 탭과 검색 탭으로 구성된 직관적 UI

## 개발 워크플로우

1. **작업 계획**

   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - 새로운 작업을 포함하도록 `ROADMAP.md` 업데이트
   - 우선순위 작업은 마지막 완료된 작업 다음에 삽입

2. **작업 생성**

   - 기존 코드베이스를 학습하고 현재 상태를 파악
   - `/tasks` 디렉토리에 새 작업 파일 생성
   - 명명 형식: `XXX-description.md` (예: `001-setup.md`)
   - 고수준 명세서, 관련 파일, 수락 기준, 구현 단계 포함
   - API/비즈니스 로직 작업 시 "## 테스트 체크리스트" 섹션 필수 포함
   - 예시를 위해 `/tasks` 디렉토리의 마지막 완료된 작업 참조
   - 초기 상태의 샘플로 `000-sample.md` 참조

3. **작업 구현**

   - 작업 파일의 명세서를 따름
   - 기능과 기능성 구현
   - API 연동 및 비즈니스 로직 구현 시 Playwright MCP로 테스트 수행 필수
   - 각 단계 후 작업 파일 내 단계 진행 상황 업데이트
   - 구현 완료 후 E2E 테스트 실행 및 통과 확인
   - 각 단계 완료 후 중단하고 추가 지시를 기다림

4. **로드맵 업데이트**

   - 로드맵에서 완료된 작업을 완료 표시로 갱신

---

## 기술 스택

| 영역 | 기술 | 비고 |
|------|------|------|
| 언어 | TypeScript 5.6+ | 타입 안전성, Obsidian API 타입 통합 |
| 플러그인 프레임워크 | Obsidian Plugin API | ItemView, Modal, PluginSettingTab, vault.*, requestUrl() |
| 번들러 | esbuild | Obsidian 표준, 빠른 빌드 |
| AI | Anthropic Claude API (REST) | 기본 모델: claude-haiku-4-5-20251001 |
| HTTP | Obsidian requestUrl() | fetch() 사용 금지 (CORS 제한) |
| UI | 순수 TypeScript DOM + Obsidian CSS 변수 | 외부 UI 프레임워크 없음 |
| 데이터 저장 | Obsidian saveData/loadData | data.json 평문 저장 |
| 개발 환경 | Node.js 20+, npm, ESLint, Prettier | obsidian-typings 포함 |

---

## 개발 단계

### Phase 1: 애플리케이션 골격 구축

> **목표**: 프로젝트 초기 환경을 구성하고, 전체 플러그인의 구조와 타입 시스템을 확립한다.

- **Task 001: 프로젝트 초기화 및 개발 환경 구성** - 우선순위
  - Obsidian 플러그인 TypeScript + esbuild 빌드 환경 구성
  - manifest.json, package.json, tsconfig.json 설정
  - ESLint + Prettier 코드 품질 도구 설정
  - main.ts 플러그인 엔트리포인트 생성 (onload/onunload 골격)
  - 핫 리로드 개발 환경 구성

- **Task 002: 타입 정의 및 데이터 모델 설계**
  - NoteIndex 인터페이스 정의 (path, title, summary, contentHash, existingTags, lastModified, indexedAt)
  - ClassificationRecommendation 인터페이스 정의 (notePath, recommendedTags, reason, status, createdAt)
  - SearchResult 인터페이스 정의 (notePath, title, relevanceExplanation, summaryExcerpt, rank)
  - PluginSettings 인터페이스 및 기본값 정의 (claudeApiKey, claudeModel, excludedFolders, maxConcurrentRequests)
  - Claude API 요청/응답 타입 정의

- **Task 003: 기본 플러그인 구조 및 라우팅 구성**
  - 사이드바 ItemView 클래스 골격 생성 (분류 탭 / 검색 탭 전환 구조)
  - PluginSettingTab 클래스 골격 생성
  - 추천 검토 Modal 클래스 골격 생성
  - 리본 아이콘 등록 및 사이드바 뷰 활성화 로직
  - saveData/loadData 기반 설정 및 인덱스 영속화 골격

### Phase 2: UI/UX 완성 (더미 데이터 활용)

> **목표**: 하드코딩된 더미 데이터를 사용하여 모든 화면의 UI를 완성하고, 사용자 플로우를 검증한다.

- **Task 004: 설정 탭 UI 구현 (F006)**
  - Claude API 키 입력 필드 (마스킹 처리, 평문 저장 고지 안내 텍스트)
  - 모델 선택 드롭다운 (claude-haiku-4-5-20251001, claude-sonnet-4-6, claude-opus-4-6)
  - 스캔 제외 폴더 입력 (쉼표 구분)
  - 최대 동시 API 요청 수 슬라이더 (1~3, 기본값 1)
  - API 연결 테스트 버튼 (더미 응답으로 UI 동작 확인)
  - saveData() 자동 저장 연동

- **Task 005: 사이드바 패널 - 분류 탭 UI 구현 (F007)**
  - 분류/검색 탭 전환 UI 구현
  - "전체 볼트 스캔" 버튼 UI
  - 스캔 진행률 바 및 현재 처리 파일명 표시 UI
  - 추천 카드 목록 UI (더미 데이터): 노트 제목, AI 요약, 기존 태그, 추천 태그, 추천 이유
  - 카드별 수락/거절 버튼 UI
  - "전체 수락" 버튼 + 확인 다이얼로그 UI
  - Obsidian CSS 변수 활용 테마 대응

- **Task 006: 사이드바 패널 - 검색 탭 UI 구현 (F007)**
  - 자연어 질의 입력창 (placeholder 텍스트, Enter/버튼 실행)
  - 로딩 인디케이터 UI
  - 검색 결과 카드 목록 UI (더미 데이터): 노트 제목, 요약 미리보기, 관련도 설명
  - 인덱스 미존재 시 안내 메시지 ("먼저 볼트를 스캔해 주세요")
  - 결과 없음 상태 메시지 UI
  - 결과 카드 클릭 이벤트 골격

- **Task 007: 추천 검토 모달 UI 구현 (F003)**
  - 노트 제목, AI 생성 요약 (1~2문장), 본문 미리보기 (첫 200자) 표시
  - 기존 태그 목록 표시
  - 추천 태그 편집 가능 칩 UI (태그 추가/삭제)
  - AI 추천 이유 텍스트 표시
  - "적용" / "취소" 버튼 UI
  - 분류 탭 카드 클릭 시 모달 열기 연동

### Phase 3: 핵심 기능 구현

> **목표**: Claude API 연동, 실제 볼트 데이터 처리, 핵심 비즈니스 로직을 구현하여 더미 데이터를 실제 동작으로 교체한다.

- **Task 008: Claude API 연동 모듈 구현** - 우선순위
  - Obsidian requestUrl() 기반 Claude API 호출 래퍼 구현
  - API 키 인증 헤더 설정
  - Rate Limit 대응: 429 응답 시 지수 백오프 재시도 (1초->2초->4초, 최대 3회)
  - Semaphore 패턴 동시 요청 제한 (기본 1, 최대 3)
  - API 연결 테스트 기능 (설정 탭 연동)
  - 네트워크 오류, API 키 오류 등 에러 핸들링
  - Playwright MCP를 활용한 API 호출 래퍼 단위 테스트

- **Task 009: 노트 스캔 및 인덱싱 구현 (F001)**
  - vault.getMarkdownFiles()로 전체 마크다운 파일 수집
  - excludedFolders 설정 기반 필터링
  - vault.read()로 본문 읽기 + metadataCache.getFileCache()로 frontmatter 추출
  - NoteIndex 생성 및 saveData()로 영속화
  - lastModified 비교로 변경된 파일만 재스캔하는 증분 로직
  - 스캔 진행률 UI 연동 (100ms 스로틀링)
  - Playwright MCP를 활용한 스캔 플로우 테스트

- **Task 010: AI 요약 및 분류/태그 추천 구현 (F002)**
  - 단일 API 호출 프롬프트 설계: 요약(1~2문장) + 태그 + 이유 동시 생성
  - JSON 구조화 응답 파싱 ({summary, recommendedTags, reason})
  - NoteIndex.summary 필드에 요약 저장 및 영속화
  - ClassificationRecommendation 메모리 생성
  - 진행률 UI에 "API 속도 제한 대기 중..." 메시지 표시
  - 구체적 개념/인명/기술명/날짜 보존하는 요약 품질 검증
  - Playwright MCP를 활용한 요약/분류 결과 검증 테스트

- **Task 011: 추천 검토 및 적용 로직 구현 (F003)**
  - 개별 수락: 선택된 추천 태그를 fileManager.processFrontMatter()로 적용
  - 개별 거절: 추천 목록에서 해당 항목 제거
  - 전체 수락: 확인 다이얼로그 후 일괄 적용
  - 모달에서 태그 편집 후 적용 시 편집된 태그 반영
  - 수락 후 NoteIndex.existingTags 메모리 업데이트
  - 완료 알림 Notice 표시 ("N개 노트에 태그 적용됨")
  - Playwright MCP를 활용한 추천 적용 E2E 테스트

- **Task 012: 자연어 검색 구현 (F004, F005)**
  - NoteIndex.summary 배열 수집 (전체 본문 제외)
  - 검색 프롬프트 설계: 요약 배열 + 질의 텍스트 -> 관련 노트 목록 + 관련도 설명
  - Claude API 응답 파싱하여 SearchResult 생성
  - 검색 결과 카드 UI에 실제 데이터 바인딩
  - 결과 카드 클릭 시 workspace.openLinkText()로 노트 열기
  - 인덱스 미존재 경고 처리
  - Playwright MCP를 활용한 검색 플로우 E2E 테스트

- **Task 012-1: 핵심 기능 통합 테스트**
  - Playwright MCP를 사용한 전체 사용자 플로우 테스트 (스캔 -> 추천 검토 -> 적용)
  - 검색 플로우 E2E 테스트 (검색 -> 결과 확인 -> 노트 오픈)
  - API 오류 시나리오 테스트 (네트워크 오류, 키 오류, Rate Limit)
  - 에러 핸들링 및 엣지 케이스 검증

### Phase 4: 안정화, 성능 검증 및 배포

> **목표**: 통합 테스트를 통해 품질을 보증하고, 대규모 볼트에서 성능을 검증한 후 커뮤니티 배포를 준비한다.

- **Task 013: 통합 테스트 및 안정화**
  - 엔드투엔드 시나리오: 전체 스캔 -> 추천 검토 -> 수락 적용 -> frontmatter 확인
  - 엔드투엔드 시나리오: 자연어 검색 -> 결과 확인 -> 노트 오픈
  - API 오류 처리 및 사용자 피드백 메시지 완성
  - requestUrl() 기반 HTTP 호출 엣지 케이스 검증
  - 다양한 노트 형식 호환성 검증 (frontmatter 없는 노트, 빈 노트, 대용량 노트)

- **Task 014: 성능 검증 및 최적화**
  - 100개 이상 노트 볼트에서 전체 스캔 성능 검증 (100개당 60초 이내)
  - 500개 이상 볼트에서 요약 기반 검색 토큰 사용량 검증 (200K 토큰 이내)
  - 검색 응답 시간 검증 (질의 후 5초 이내 결과 표시)
  - saveData/loadData 인덱스 저장/복원 성능 확인
  - 노트 1개당 API 비용 $0.001 이하 확인 (claude-haiku 기준)

- **Task 015: 배포 준비 및 문서화**
  - README.md 작성 (설치 가이드, 사용법, 스크린샷)
  - manifest.json 최종 검증 (id, version, minAppVersion 등)
  - LICENSE 파일 추가
  - CHANGELOG.md 초기 버전 작성
  - 커뮤니티 플러그인 제출 요구사항 충족 확인
  - GitHub 릴리즈 준비

---

## Phase 2 Post-MVP

> MVP 출시 후 사용자 피드백을 반영하여 고도화하는 기능들.

### F101: 점진적 증분 분석

- vault.on('modify') 이벤트 리스닝으로 변경 파일 실시간 감지
- 변경된 노트만 재분석하여 NoteIndex.summary 자동 갱신
- 마지막 인덱싱 시각 추적으로 중복 분석 방지
- API 비용 절감 효과 측정 및 최적화

### F102: 검색 결과 순위/필터링

- 날짜 범위 필터 (수정일 기준)
- 태그 기반 필터 (특정 태그를 가진 노트만 검색)
- 폴더 기반 범위 제한 검색
- 관련도 점수 정렬 및 표시

### F103: 분류 규칙 커스터마이징

- 사용자 정의 태그 어휘 목록 입력 (설정 탭 확장)
- AI 프롬프트에 사용자 태그 어휘 주입으로 일관성 향상
- 특정 폴더에 자동 적용할 태그 규칙 설정

### F104: 노트 링크 관계 분석

- 위키링크 ([[노트명]]) 파싱으로 링크 그래프 구성
- 링크 관계를 AI 프롬프트 컨텍스트에 포함하여 추천 품질 향상
- 관련 노트 클러스터 시각화 (사이드바 추가 탭)

---

## 주요 마일스톤

| 마일스톤 | 목표 시점 | 완료 기준 |
|---------|----------|----------|
| M1: 개발 환경 및 구조 완성 | 1주차 완료 | Task 001~003 완료, 빌드 성공, 빈 사이드바 표시 |
| M2: 전체 UI 완성 | 2주차 완료 | Task 004~007 완료, 더미 데이터로 모든 화면 동작 |
| M3: API 연동 및 스캔 동작 | 3주차 완료 | Task 008~010 완료, 실제 볼트 스캔 및 요약 생성 |
| M4: 핵심 기능 완성 | 4주차 완료 | Task 011~012-1 완료, 추천 적용 및 검색 동작 |
| M5: 안정화 및 성능 검증 | 5주차 완료 | Task 013~014 완료, 100개+ 노트 성능 기준 충족 |
| M6: MVP 출시 | 6주차 완료 | Task 015 완료, 커뮤니티 플러그인 제출 준비 |

---

## 위험 요소 및 대응 전략

### 기술적 위험

| 위험 요소 | 영향도 | 대응 전략 |
|----------|--------|----------|
| Claude API Rate Limit | 높음 | 지수 백오프 재시도 (1s->2s->4s), 동시 요청 기본 1개, UI 피드백 |
| requestUrl() CORS/호환성 이슈 | 높음 | fetch() 대신 requestUrl() 전용 사용, 모바일/데스크톱 교차 검증 |
| 대규모 볼트 성능 저하 | 중간 | lastModified 증분 스캔, 요약 기반 검색으로 토큰 97% 절감 |
| data.json 저장 크기 초과 | 낮음 | content 필드 미저장, summary만 영속화 (노트당 약 100 토큰) |

### 일정 위험

| 위험 요소 | 영향도 | 대응 전략 |
|----------|--------|----------|
| AI 프롬프트 품질 튜닝 지연 | 중간 | 2주차에 조기 프롬프트 테스트, 반복 개선 |
| Obsidian API 동작 차이 (모바일 vs 데스크톱) | 중간 | 5주차 통합 테스트에서 교차 플랫폼 검증 |
| 예상 외 API 비용 초과 | 낮음 | claude-haiku 기본 사용, 동시 요청 1개 제한, 비용 모니터링 |

---

## 성공 지표

### 정확도

- 태그 추천 수락률 70% 이상 (수정 없이 수락하는 비율)
- 검색 결과 상위 3개 내 원하는 노트 포함률 85% 이상

### 성능

- 전체 스캔 속도: 노트 100개당 60초 이내 (동시 요청 1개 기준)
- 검색 응답 시간: 질의 후 결과 표시까지 5초 이내
- 검색 토큰 사용량: 1,000개 노트 기준 200K 토큰 이내

### 비용

- 노트 1개당 평균 API 비용: $0.001 이하 (claude-haiku 기준)
- 검색 1회당 평균 API 비용: 800개 노트 기준 $0.03~$0.04

### 사용자 경험

- 스캔->검토->적용 작업 완료율 80% 이상
- 주 1회 이상 능동적 사용률 60% 이상
