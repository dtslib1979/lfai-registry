# LFAI Registry — GUI · ADB · 에이전트 자동화 전수 카탈로그

> 박씨 생태계에서 만든 모든 GUI/ADB/브라우저 자동화를 한곳에 정리.
> 각 항목: 대상 · 방식 · 상태 · 위치 · 핵심 기법.
> 작성 2026-10-21 · 기능인 박씨 / Claude

---

## A. ADB 기반 (안드로이드 실기기 제어)

### A1. 삼성 그리기 어시스트 — `parksy-gallery` MCP
- **대상:** 삼성 그리기 어시스트 (`com.samsung.android.app.sketchbook`)
- **방식:** **비루트 ADB** shell만으로 전 과정 (루팅 0 · 워런티 훼손 0)
- **핵심 기법:** `uiautomator dump` → XML 파싱 → 중심좌표 계산(좌표 하드코딩 X) · `input tap/text/keyevent` · `am force-stop/start`(매 호출 클린) · `parksy_stage` 격리 앨범(결정론적 선택) · `MEDIA_SCANNER` 갤러리 반영
- **하드웨어:** Galaxy Tab S9 5G (SM-X716N) · Tailscale `100.86.15.50:5900`
- **상태:** 빌드·검증 완료(2026-08-30), 이후 정지. MCP 7툴(generate/restyle/extract_character/generate_intro/list_styles/suggest_style/list_presets)
- **위치:** `parksy-image` · (태블릿본) `mcp-servers/parksy_gallery_mcp.py`

### A2. 네이버 로그인·발행 자동화
- **대상:** 네이버 블로그 로그인·발행
- **방식:** ADB 실터치 E2E (uiautomator dump + input tap)
- **이력:** 쿠키 추출 전투(2026-03-16) → ADB 실터치 E2E 완주(2026-08-27, `5da81da`)
- **위치:** 태블릿 레포 / `dtslib-papyrus`

### A3. ADB 워치독
- **대상:** ADB 직결 기기 상시 감시·재연결
- **방식:** cron 감시 + 자동 재연결
- **위치:** `bin/devbatch_adb_watchdog.sh`

### A4. 실기기 브라우저 캡처 — po-tutorial `device`
- **대상:** 태블릿/폰 실브라우저(장편 스크롤 캡처)
- **핵심 기법:** ADB forward → **CDP `scrollTop` 직접 밀기**(ADB `input swipe`는 위젯에서 멈춤 → CDP로 우회, 2~3시간 장편도 길이 제한 없음)
- **왜:** 메타AI·ChatGPT 공유 페이지가 **데스크톱·데이터센터 IP 차단** → 실기기는 통과
- **위치:** `parksy-image/tools/mcp_tutorial/urlintro.py`

---

## B. CDP 기반 (크롬 원격 제어)
### B1. 크롬 CDP 브릿지
- **대상:** 실기기/로컬 크롬을 CDP로 제어
- **방식:** ADB forward + Chrome DevTools Protocol
- **용도:** 데이터센터 IP 차단 우회, 스크롤/캡처
- **위치:** `bin/chrome_cdp.sh` · `bin/cdp_forward.py` · `bin/device-cdp.sh` · `bin/stage-chrome.sh`

---

## C. Playwright / 웹 자동화
### C1. 티스토리 자동 발행
- **대상:** 티스토리 25개 블로그(단행본 출판사)
- **방식:** Playwright 로그인 → **TinyMCE `setContent`**로 HTML 본문 주입 → 발행
- **발견:** 티스토리 HTML 모드가 `<script>/<style>/<div>` 지원 → 인디자인급 단행본 레이아웃 가능
- **삽질:** `accounts.json` slug ↔ 코드 slug 불일치 = 전부 404
- **위치:** `tools/tistory/post.py` · 개발일지 060 (2026-07-14)

### C2. 헤드리스 카드/교재 렌더
- **대상:** 슬라이드 포스터·교재 페이지 → PNG
- **방식:** 셀(`youthful_borg`) Playwright headless, device_scale 2
- **위치:** `~/deepfake-grind/*_cards/shoot.py`

---

## D. 비공식 클라이언트 API
### D1. 인스타그램 — instagrapi
- **대상:** 인스타그램 로그인·캐러셀 발행
- **방식:** instagrapi 로그인→세션 저장→`album_upload`(캐러셀)→permalink manifest
- **리스크:** instagrapi = 비공식 = **계정 ban 리스크**(세션 재사용도 탐지 안전 X)
- **위치:** `dtslib-papyrus-real/tools/instagram/login.py·post.py·renderer/`
- **참고:** 공식 Graph API(비루트 아님, 승인 토큰) 쪽은 `dtslib-papyrus/tools/meta/`(별개)

---

## E. 인프라 / 가드
### E1. mcp_jail · jail_guard
- **용도:** 에이전트 격리·위험 명령 차단(pkill/killall 등)
- **위치:** `parksy-image/tools/mcp_jail` · `bin/jail_guard_minipc.py`

### E2. mcp_phonerec
- **용도:** 폰 화면 녹화(삼성 QS 타일 토글로 시작/정지)
- **위치:** `parksy-image/tools/mcp_phonerec`

### E3. win-gui MCP (계획/부분)
- **용도:** Windows GUI 자동화 TCP 데몬(GUI 자동화 대안)
- **위치:** `dtslib-papyrus-real/docs/gui-automation-solution.md`

---

## 전자소송
- **현황:** `phoneparis` 전략/랜딩 문서에 **서비스 use-case로 언급**은 있으나, **작동하는 전자소송 자동화 코드는 미발견.**
- dev-log/폴더 지목 시 이 섹션에 추가 예정.

---

## 리스크 메모 (사실 기록)
- **GUI 자동화 ToS/봇탐지:** 메타·네이버 등 계정락 가능. 비루트 ADB = 워런티 유지. instagrapi = ban 리스크.
- **전자소송 '대행'(타인 대신 작성·제출):** 변호사법 §109 / 법무사법 리스크 → 안전형 = 셀프 자동화 툴/교재 또는 면허(법무사·변호사) 제휴.
