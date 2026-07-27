# evolve-kr Vercel 배포 가이드

> 작성: Hermes (subagent) · 2026-07-27
> 이유: 헤드리스 에이전트 환경에서 Vercel 인증(`vercel login`)을 인터랙티브하게 진행할 수 없어 CLI 자동 배포가 막힘 → 웹 콘솔 가이드로 fallback

---

## 환경 진단 결과

| 항목 | 결과 |
|---|---|
| vercel CLI | ✅ 설치됨 (`/opt/homebrew/bin/vercel`, v54.18.2) |
| `vercel whoami` | ❌ `No existing credentials found` |
| `~/.vercel` | ❌ 없음 |
| `VERCEL_TOKEN` 환경변수 | ❌ 없음 |
| 헤드리스 자동 `vercel login` | ❌ 인터랙티브 프롬프트 필요 (브라우저/이메일 인증) |
| 로컬 저장소 `/Users/mac/work/evolve-kr` | ✅ 존재, main 브랜치, clean tree, origin=sigco3111/evolve-kr |

→ CLI로 바로 배포하면 무조건 "No credentials"로 실패. **Vercel 웹에서 직접 Import** 해야 함.

---

## 1단계: Vercel 접속

<https://vercel.com/new> 이동 → GitHub 계정으로 로그인 (이미 로그인되어 있으면 2단계로).

---

## 2단계: 저장소 Import

1. **"Import Git Repository"** 검색창에 `sigco3111/evolve-kr` 입력
2. `sigco3111/evolve-kr` 리포 선택 → **Import** 클릭
3. Vercel이 자동으로 분석 시도 → **Project Configure** 화면 진입

---

## 3단계: 빌드 설정 (Project Configuration)

기본으로 잡힌 자동 감지 값을 모두 **수동으로 덮어쓰기**:

| 필드 | 값 |
|---|---|
| **Project Name** | `evolve-kr` (희정님 의도) |
| **Framework Preset** | `Other` (자동감지 ≠ Vite/Next.js이므로 강제) |
| **Root Directory** | `.` (기본값 유지) |
| **Build Command** | `npm run build` (← `package.json`에 이미 정의됨; 4단계로 대체 가능) |
| **Output Directory** | `.` ← ⚠️ **중요: 비워두거나 `.` 으로**. `dist/` 아님! |
| **Install Command** | `npm install --ignore-scripts` |
| **Node.js Version** | 22.x (Vercel 기본 22 사용 가능) |

### 왜 이런 설정이 필요한가

`evolve-kr`은 진정한 정적 사이트이지만 npm 빌드 스텝을 거칩니다:

- `npm run build` = `npm run evolve && npm run evolve-less && npm run wiki && npm run wiki-less`
  - `node buildEvolve.js` → `esbuild`로 `src/evolve.js`를 `lib/evolve.js`(번들)로 빌드
  - `lessc src/evolve.less evolve/evolve-unminified.css && csso ...` → CSS 압축
  - `node buildWiki.js` → esbuild로 위키 빌드
  - `lessc src/wiki/wiki.less ...` → 위키 CSS 압축
- 결과물이 `lib/`, `evolve/`, `wiki/` 에 떨어짐 → HTML(`index.html`)이 그대로 그 경로를 참조
- `servehere`(serve), `gh-pages`는 **devDependencies**이지만 빌드에는 불필요 → `--ignore-scripts` 안전

빌드 산출물이 dist가 아니라 **레포 루트 + lib/ + evolve/ + wiki/** 그대로이므로 Output Directory는 `.` 이어야 함. Vercel이 자동으로 `index.html`을 찾아 정적 서빙.

---

## 4단계: 환경변수 (선택)

`.hermes-evolve-*-ko.json` 파일이 빌드에 필요하다면 (실제로는 런타임 fetch일 가능성 높음) 검토:

- 빌드 도중 `fs.readFileSync`로 읽는 게 있다면 → 환경변수보다 그냥 디스크에 있으므로 OK
- 따로 추가할 env 없음 (이 레포는 API 키 안 씀)

만약 첫 배포에서 `Cannot find module` 류 에러가 빌드 로그에 보이면, Install Command를 다음으로 바꿈:

```
npm install --ignore-scripts && npm install esbuild@^0.25.0 less@^3.13.0 csso-cli@^4.0.2
```

(이미 devDependencies에 있지만 `--ignore-scripts`로 native 빌드가 안 돌아 의존성 트리가 꼬이면 명시 설치)

---

## 5단계: Deploy

- **Deploy** 버튼 클릭
- 첫 빌드는 30~90초 (esbuild + less + csso)
- 성공 시 `https://evolve-kr.vercel.app` 형태의 URL 발급 (자동). 도메인 변경 가능

---

## 6단계: 라이브 URL 확보 후

1. Vercel 대시보드에서 **Visit** 버튼 → `https://evolve-kr.vercel.app/` 접속
2. `index.html`이 뜨고 한국어 게임 UI(`evolve/evolve.css`, `wiki/wiki.css`)가 정상 렌더링되는지 확인
3. 위키 페이지(`/wiki/...`)도 작동하는지 클릭 테스트
4. 이후 README에 라이브 URL 배지 추가 가능

---

## 실패 시 체크리스트

| 증상 | 원천 | 해결 |
|---|---|---|
| `Cannot find module 'esbuild'` | `--ignore-scripts`로 native deps 누락 | 4단계의 명시 install로 대체 |
| `lessc: command not found` | bin PATH | Install Command에 `npm install -g less` 추가 또는 devDep 재확인 |
| `Output Directory not found` | 빈 출력 디렉터리를 원함 | Output Directory를 `.` 로 (위 3단계) |
| 페이지 흰 화면 (CSS 없음) | `evolve/evolve.css` 미생성 | 로컬에서 `npm run evolve-less`만 실행 후 실패 로그 확인 |
| 404 on `/wiki/*` | Vercel 정적 라우팅이 wiki 폴더 안 잡음 | 기본 정적 서빙이면 OK. 안되면 `vercel.json` rewrites 추가 |

### `vercel.json` 강제 설정이 필요해지면

루트에 `vercel.json` 생성:

```json
{
  "cleanUrls": true,
  "trailingSlash": false
}
```

이 정도면 충분 (SPA가 아니라 진정한 MP 정적 사이트).

---

## CLI로 다시 시도하고 싶다면 (향후)

희정님 macOS에서 직접:

```bash
# 1) 로그인 (브라우저/이메일 인증 필요 — 본인이 직접)
cd /Users/mac/work/evolve-kr
vercel login

# 2) 인증 확인
vercel whoami

# 3) 프로덕션 배포 (현재 디렉터리 사용)
vercel --prod --yes --name evolve-kr

# 4) 또는 토큰 방식
export VERCEL_TOKEN=<Settings → Tokens 에서 발급>
vercel --prod --yes --name evolve-kr --token "$VERCEL_TOKEN"
```

토큰은 <https://vercel.com/account/tokens> 에서 발급 가능 (CI용 토큰 권장 — 만료 기간 지정 가능).

---

## 보고 상태

- **CLI 시도**: 시도했음. `vercel login` 단계에서 헤드리스 환경 인터랙티브 요구로 자동 진행 불가 → **실패로 간주**.
- **fabricate 없음**: 라이브 URL / 빌드 로그를 지어내지 않음. 실제 배포는 사용자가 **<https://vercel.com/new>** 에서 직접 해야 함.
- **산출물**: 본 가이드 파일(`/Users/mac/work/evolve-kr/VERCEL_DEPLOY_GUIDE.md`) + 위 설정값.

---

## 2차 세션 (2026-07-27 후속) 추가 메모

희정님이 vars.js 1줄 패치 (`en-US` → `ko-KR`, 라인 1316 + 1490-1491) 후 redeploy 위임.

**한 일**:
- `src/vars.js` 두 곳 패치 → 로컬에서 4 builds (`npm run evolve`/`evolve-less`/`wiki`/`wiki-less`) 모두 성공 (esbuild 2.2MB/2.3MB, less→csso 통과)
- 로컬 `evolve/main.js` 검증: `locale:"ko-KR"` 1회, `locale:"en-US"` 0회 ✅
- `git add src/vars.js evolve/main.js wiki/wiki.js` → 커밋 `77a3a3f7 feat(i18n): ko-KR를 기본 로케일로 설정 (vars.js)` → `git push origin main` 성공

**못 한 일 (솔직히)**:
- `vercel --prod --yes --non-interactive --token "$VERCEL_TOKEN" --name evolve-kr` → **실패**. 이 서브쉘에는 `$VERCEL_TOKEN` 환경변수가 비어있음 (부모 셸에는 있어도 서브프로세스로 전파 안 됨), `~/.vercel/` 인증도 없음 → `Error: No existing credentials found` 또는 `--token` missing value.
- 라이브 URL `https://evolve-kr.vercel.app/evolve/main.js` 라이브 grep 결과: 여전허 `locale:"en-US"` 1회, `locale:"ko-KR"` 0회. `last-modified: Mon, 27 Jul 2026 07:53:57 GMT` (이전 배포 시각) + `x-vercel-cache: HIT` + etag 동일 → 새 배포 안 됨.
- GitHub repo webhook 검사: `vercel.com` hook 없음 → push만으로는 자동 배포 트리거 안 됨.

**다음 시도 시 필요한 한 가지**: Vercel 토큰을 **이 서브쉘에서 읽을 수 있는 위치**에 둠. 옵션:
1. `vercel login` (인터랙티브) — 희정님이 직접
2. `~/.zshenv`에 `export VERCEL_TOKEN=...` 추가 후 새 세션
3. 1Password/vault에서 토큰을 export해 이 subagent에 주입
4. Vercel 웹 UI에서 "Import" 또는 GitHub 통합 활성화 → 그 다음부터는 push가 자동 배포

**이 세션이 바꾼 것들**:
- `src/vars.js` (커밋됨)
- `evolve/main.js`, `wiki/wiki.js` (재빌드 + 커밋됨)
- `VERCEL_DEPLOY_GUIDE.md` (이 노트 추가)

다음 subagent는 이 가이드 + 토큰 주입만 받으면 1분 안에 redeploy + 라이브 검증 가능.

</content>
</invoke>