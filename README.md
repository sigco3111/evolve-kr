# Evolve 한국어판 (Evolve KR)

> 원작: [pmotschmann/Evolve](https://github.com/pmotschmann/Evolve) (MPL-2.0)
> 한국어 fork: [sigco3111/evolve-kr](https://github.com/sigco3111/evolve-kr)

## 소개

**Evolve**는 원시 수프에서 우주 문명까지 문명을 진화시키는 브라우저 인크리멘털 게임입니다. 클릭커와 아이들러 요소를 결합했고, 많은 마이크로 매니지먼트가 포함되어 있습니다.

이 한국어 fork는 다음을 목표로 합니다:

- 🇰🇷 **한국어 번역 보완** — `strings/strings.ko-KR.json` (이미 약 10,166줄/803KB가 들어가 있음) 검토 및 개선
- 🎌 **한국 문화 콘텐츠 추가** — 한국적 컨셉, 이벤트, 항목 등 (작업 진행 중)
- 🚀 **Vercel 정적 호스팅** — `gh-pages` 대신 Vercel로 배포

## 플레이

- 원본 (영어): https://pmotschmann.github.io/Evolve/
- 한국어 fork (Vercel): https://evolve-kr.vercel.app

## 설치 & 실행

```bash
# 의존성 설치
npm install

# 빌드 (게임 번들 + 위키 번들 + LESS → CSS)
npm run build

# 로컬 서버 (localhost:4400)
npm run serve
```

`npm run serve`는 내부적으로 `servehere -c`를 실행하므로 `node_modules`가 설치된 상태여야 합니다.

## 한국어 번역 위치

| 역할 | 위치 |
|------|------|
| 영문 원본 | `strings/strings.json` (11,643줄) |
| 한국어 | `strings/strings.ko-KR.json` (10,166줄) |
| 언어 등록 | `src/locale.js` (`locales` 상수, L90~104) |

### 번역 기여 방법

1. `strings/strings.ko-KR.json` 파일을 편집
2. 키는 절대 변경하지 말 것 (예: `"tab_evolve": "진화"`)
3. 토큰 (`%0`, `%1`, …)은 그대로 유지하되 위치는 이동 가능
4. 누락된 키는 영어 원본으로 폴백됨 (`src/locale.js`의 `Object.assign`)

검증 도구: `strings/checkStrings.py`

## 디렉토리 구조

```
.
├── index.html          # 게임 진입점
├── save.html           # 세이브 매니저
├── wiki.html           # 위키 진입점
├── src/                # JS 모듈 (게임 + 위키)
│   ├── main.js         # 게임 진입점
│   ├── index.js
│   ├── locale.js       # 🔑 i18n 헬퍼 + locales 상수
│   ├── vars.js         # 전역 변수
│   ├── actions.js      # 액션 정의
│   ├── tech.js, races.js, jobs.js, resources.js, ...
│   └── wiki/           # 위키 모듈 (arpa.js, basics.js, ...)
├── strings/            # 🌐 다국어 번역
│   ├── strings.json    # 기본 (영문)
│   ├── strings.ko-KR.json
│   └── strings.<locale>.json
├── evolve/             # 빌드 산출물 (CSS 등)
├── lib/                # 외부 라이브러리 (jQuery, LZString, popper, ...)
├── font/               # 웹 폰트
├── wiki/               # 빌드 산출물
├── buildEvolve.js      # esbuild 스크립트 (게임)
├── buildWiki.js        # esbuild 스크립트 (위키)
└── package.json
```

## 빌드 명령

| 명령 | 설명 |
|------|------|
| `npm run build` | 게임+위키 번들 + LESS → CSS (Linux) |
| `npm run build-win` | Windows 빌드 |
| `npm run evolve` | esbuild로 `evolve.js` 생성 |
| `npm run evolve-less` | `src/evolve.less` → `evolve/evolve.css` |
| `npm run wiki` | esbuild로 `wiki.js` 생성 |
| `npm run wiki-less` | `src/wiki/wiki.less` → `wiki/wiki.css` |
| `npm run serve` | `servehere -c`로 localhost:4400 |
| `npm run deploy` | gh-pages 배포 (이 fork에서는 Vercel로 대체) |

## 라이선스

원본은 **Mozilla Public License 2.0 (MPL-2.0)**. `LICENSE` 파일을 보존했습니다.
이 한국어 fork도 동일 라이선스를 따릅니다.

원작자: Peter Motschmann
