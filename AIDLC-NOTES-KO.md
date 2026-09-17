# AI-DLC 분석 노트 (한국어)

> 이 저장소(AI-DLC)를 처음 받아본 뒤 **"이게 뭐고, 어떻게 쓰고, 나한테 어떤 가치가 있는지"**
> 를 실제 코드와 문서를 직접 확인해 정리한 문서입니다.
> 작성일: 2026-09-17 · 기준 버전: v2.9.0

## 저장소 주소

| 구분 | 주소 |
| --- | --- |
| 원본 (AWS Labs) | <https://github.com/awslabs/aidlc-workflows> |
| 이 저장소 (포크) | <https://github.com/bmshin94/aidlc-workflows> |
| 문서 사이트 / 로드맵 | <https://awslabs.github.io/aidlc-workflows/roadmap.html> |
| AI-DLC 방법론 블로그 | <https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/> |
| 방법론 정의 문서 | <https://prod.d13rzhkk8cj2z0.amplifyapp.com/> |

라이선스: **MIT-0 (MIT No Attribution)** — 상업적 이용·수정·재배포·판매 모두 자유이며
저작자 표시 의무조차 없음. 단 AWS/Amazon **상표** 사용은 불가.

---

## 1. 한 줄 요약

AI-DLC(AI-Driven Development Life Cycle)는 **AI 코딩 어시스턴트를 "절차를 밟는 개발팀"으로
바꿔주는 프레임워크**다. 하나의 하네스-중립 코어(`core/`)를 빌드해 7개 CLI 하네스
(Claude Code / Kiro CLI / Kiro IDE / Codex CLI / Cursor / opencode / GitHub Copilot)에
동일하게 배포한다.

요리 비유: AI에게 "대충 맛있는 거 해줘"라고 시키는 대신, **정해진 순서표와 확인 절차**를
쥐여주는 것.

## 2. 핵심 구조 (실측)

| 항목 | 수치 | 위치 |
| --- | --- | --- |
| 페이즈 | 5 (Initialization / Ideation / Inception / Construction / Operation) | `core/aidlc-common/stages/` |
| 스테이지 | 33 | 같음 |
| 에이전트 | 14 (도메인 전문가 11 + 리뷰어 2 + 컴포저 1) | `core/agents/` |
| 워크플로 프로필(스코프) | 11 + 자동감지 | `core/scopes/` |
| 훅 | 17 | `core/hooks/` |
| CLI 도구 | 72 | `core/tools/` |
| 센서 | 6 | `core/sensors/` |
| 감사 이벤트 타입 | 99종 | `core/tools/aidlc-audit.ts` |
| 테스트 파일 | 508개 (smoke/unit/integration/e2e 4단계) | `tests/` |
| 코어 소스 크기 | 6.7MB (문서 2.1MB, 테스트 14MB) | — |

### 워크플로 프로필 (작업 크기별 코스 선택)

| 프로필 | 용도 | 스테이지 |
| --- | --- | --- |
| `express` | 요구사항이 이미 명확할 때 최단 코스 | 10 / 33 |
| `bugfix` | 버그 1건 + 회귀 테스트 | 9 / 33 |
| `poc` | 실현 가능성만 빨리 확인 | 8 / 33 |
| `refactor` | 동작 유지 개선 | 10 / 33 |
| `security-patch` | CVE·취약점 대응 | 10 / 33 |
| `infra` | 환경·IaC·배포 기반 | 13 / 33 |
| `classic` | v1 스타일 (엔진 기본값) | 18 / 33 |
| `mvp` | 운영 페이즈 제외 첫 제품 | 23 / 33 |
| `workshop` | 교육·다중팀 진행 | 26 / 33 |
| `feature` | 정식 기능 (전체 라이프사이클) | 33 / 33 |
| `enterprise` | 규제·고신뢰 (최고 깊이) | 33 / 33 |

**중요:** 항상 33단계를 밟는 게 아니다. 작은 일은 8~10단계로 끝난다.

## 3. 설치 및 사용법

### 설치 (2가지 경로)

```bash
# 경로 A: 네이티브 설치 (Bun/Node 불필요) — 권장
curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh | sh

# Windows PowerShell
irm https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.ps1 | iex
```

경로 B: Bun 설치 후 release의 `aidlc-copy-runtime-X.Y.Z.tar.gz`에서
`runtime/<harness>/`를 프로젝트에 복사 (네이티브 `aidlc` 명령 없이 동작).

### 프로젝트 설정

```bash
cd /내/실제/프로젝트
aidlc config --dry-run --json   # 무엇이 바뀌는지 먼저 확인 (한 바이트도 안 씀)
aidlc config                    # 대화형 위저드 (하네스/프로바이더/모델/플러그인/MCP/설정레이어)
aidlc doctor                    # 진단
```

설정 시 생성되는 것: `.claude/`(또는 `.kiro/`, `.codex/` 등), `aidlc/` 워크스페이스,
`.gitignore` 관리 블록, `.mcp.json`(선택), 프로젝션 스탬프.

### 사용

```text
/aidlc 재고관리 API 만들어줘      # 설명 → 스코프 자동 감지
/aidlc bugfix                     # 스코프 직접 지정
/aidlc compose "이 작업"           # 맞춤 워크플로 설계
/aidlc --status                   # 진행 상황
/aidlc intent                     # 작업(intent) 목록/전환
/aidlc --jump --stage <번호>       # 단계 점프
/aidlc knowledge onboard          # 사내 문서 학습
/aidlc --help
```

> Codex CLI만 `/aidlc` 대신 `$aidlc`.

**주의:** 이 저장소 자체는 **프레임워크 소스**(`package.json` 이름이 `aidlc-workflows-dev`)이다.
여기서 `/aidlc`를 쓰는 게 아니라, 실제 개발 프로젝트에 설치해서 쓴다.
소스를 직접 수정할 때는 `core/`와 `harness/`만 고치고 `dist*`는 건드리지 않는다.

```bash
bun install --frozen-lockfile
bun scripts/package.ts        # dist/<harness>/ 생성
bun scripts/package.ts --check # 두 번 빌드 후 바이트 비교(결정론 보장)
bun tests/run-tests.ts --ci
```

## 4. 정체: 플러그인? 스킬? MCP?

**정답: "스킬 + 훅" 조합**이며 MCP는 선택적 곁들임이다.

| 정체 | 실제 파일 | 역할 |
| --- | --- | --- |
| 스킬 (본체) | `.claude/skills/aidlc/SKILL.md` | `/aidlc` 명령의 오케스트레이터 |
| **훅 (핵심)** | `.claude/settings.json` (17개) | 규칙을 물리적으로 강제 |
| 서브에이전트 | `.claude/agents/*.md` (14개) | 역할별 페르소나 |
| CLI 도구 | `.claude/tools/*.ts` (72개) | bun으로 도는 결정론적 엔진 |
| MCP (선택) | `.mcp.json` | `aws-mcp`, `aws-iac`, `aws-pricing`, `aws-serverless`, `context7` |

### 훅이 핵심인 이유

`harness/claude/settings.json` 기준, 11개 이벤트에 훅이 걸려 있다.

```text
PreToolUse   → state-transition-guard  (상태 몰래 변경 차단)
             → plan-approval-guard     (승인 없이 코드 생성 차단)
             → reviewer-scope          (리뷰어 읽기 범위 제한)
             → review-freeze           (리뷰 중 쓰기 동결)
PostToolUse  → write-audit-log         (모든 수정 감사 기록)
             → run-sensors             (린터/타입체크 자동 실행)
Stop         → continue-workflow       (임의 종료 방지)
그 외: SessionStart/End, PreCompact, SubagentStop, UserPromptSubmit
```

즉 프롬프트로 "부탁"하는 게 아니라 **시스템이 도구 호출 자체를 거부**한다.
다른 프레임워크와의 결정적 차이.

### MCP 여부

AI-DLC 자체는 MCP 서버가 **아니다**. 위 5개 MCP 서버는 옵션이고
`aidlc config project --mcp none`으로 전부 제거해도 정상 동작한다.

### 별도의 "AIDLC 플러그인" 시스템

새 스테이지·에이전트·스코프·센서·규칙을 추가하는 확장팩 메커니즘이 정식 지원된다
(`docs/reference/18-plugin-mechanism.md`, 저작 가이드
`docs/harness-engineering/10-authoring-a-plugin.md`, 참고 구현체 `plugins/test-pro/`).

설계 원칙:

- **오직 추가만, 덮어쓰기 없음** (합집합이라 여러 플러그인 동시 사용 안전)
- **끄면 완전 복원** (모든 플러그인 비활성 시 bare core와 바이트 동일)
- **배포 인프라 불필요** — 각자 git repo + semver 태그 + `marketplace.json`
- 빌드 시 호스트별 실제 플러그인(Claude `.claude-plugin/`, Codex `.codex-plugin/` 등)으로 변환

## 5. API 토큰 / 비용

**AI-DLC 자체는 토큰을 쓰지 않는다.** 모델 접근은 하네스 책임.

| 하네스 | 인증 |
| --- | --- |
| Claude Code | 배포 기본값이 **Amazon Bedrock** (AWS 자격증명 필요) |
| Codex CLI | 기본 Bedrock |
| Kiro CLI / IDE | 별도 불필요 (모델 포함) |
| Cursor / opencode | 각자 설정한 프로바이더 |
| GitHub Copilot | GitHub 로그인 또는 BYOK |

### 함정과 해결

설치하면 `.claude/settings.json`에 아래가 박힌다.

```json
"CLAUDE_CODE_USE_BEDROCK": "1",
"AWS_REGION": "us-east-1",
"ANTHROPIC_DEFAULT_OPUS_MODEL": "global.anthropic.claude-opus-4-8[1m]"
```

→ Bedrock 계정이 없으면 동작하지 않는다.
일반 구독/API 키로 쓰려면 `CLAUDE_CODE_USE_BEDROCK`과 `ANTHROPIC_DEFAULT_*` 항목을
제거하고 하네스의 기본 인증을 사용한다 (`docs/guide/01-getting-started.md:146`).
자격증명·개인 오버라이드는 `.claude/settings.local.json`에 둔다.

### 비용 관리 장치

- `fold-usage` 훅이 토큰 사용 장부를 지속 기록
- `/aidlc-session-cost` 스킬로 세션 집계 확인 (읽기 전용)
- **모델 정책**: 에이전트를 3그룹으로 나눠 티어 배분
  - 판단 그룹(9명) → 고성능 / 리뷰 그룹(2명) → 중간 / 문서작성 그룹(3명) → 저비용
  - `minimal` / `balanced`(기본) / `thorough` 프리셋

**실전 팁:** 처음엔 `express` 또는 `bugfix`로 시작해 비용 감각을 잡는다.

## 6. 왜 주목받는가

1. **AWS 공식(awslabs)** — AWS가 발표한 AI-DLC 방법론의 공식 구현체
2. **MIT-0** — 저작자 표시조차 불필요, 기업 도입 장벽이 사실상 0
3. **"하나의 코어, 7개 하네스"** 라는 아키텍처적 야심
4. **규모가 진지함** — 테스트 508개, CHANGELOG 632KB, 도구 72개 (주말 프로젝트가 아님)
5. **시장 타이밍** — "AI가 대충 하고 다 됐다고 함 / 맥락 잃음 / 검증 안 됨"이라는
   현재 최대 불만을 프롬프트가 아닌 **시스템**으로 해결한다고 주장
6. **결정론적 빌드** — `--check`로 두 번 빌드해 바이트 비교

## 7. 직접 에이전트를 만들 때 배울 패턴 5개

1. **"부탁" 대신 "훅으로 강제"** — 지켜야 할 규칙은 프롬프트가 아니라 훅으로 옮긴다
   (`plan-approval-guard`가 승인 전 Write 호출을 거부)
2. **상태를 파일에 둔다** — `aidlc-state.md` + `PreCompact` 훅 + `recovery.md` 브레드크럼으로
   대화 압축·세션 종료를 넘어 작업이 이어진다
3. **센서 = 기계 검증 레이어** — AI의 "했어요"를 믿지 않고 린터/타입체크/필수섹션/
   근거(claim-sources)/추적성을 자동 채점
4. **서브에이전트 토폴로지 선택** — 33개 중 29개는 inline(혼자), 2개 subagent(hub),
   1개 pipeline, 1개 mob. **과한 분산은 손해**라는 걸 설계로 보여준다
   (문서: 좁은 전문가 수십 명 = 워터폴 재현)
5. **학습 루프** — 사용자 지적 → 승인 게이트 확인 → `team.md`/`project.md`에 규칙 저장
   → org → team → project → phase 5계층으로 해석

보너스: `tests/harness/sdk-drive.ts` — 에이전트를 실제 구동해 E2E 테스트하는 구현 사례.

## 8. 수익화 분석

### 코드에서 발견한 "비어 있는 자리" 3곳

이 세 곳이 수익 지점의 근거다.

#### 구멍 1 — 메트릭 엔드포인트가 비어 있음

`core/tools/aidlc-metrics.ts` 주석:

> OPT-IN and DISABLED by default: it emits ONLY when `AIDLC_METRICS_ENDPOINT` is set.
> **No endpoint is shipped in any harness's settings.**

- 감사 이벤트 99종을 StatsD 형식으로 HTTP 전송하는 기능이 **이미 구현되어 있다**
- `STAGE_COMPLETED` / `WORKFLOW_COMPLETED`에는 **토큰·비용 수치까지 포함**
- 그런데 받아줄 서버가 없다 (AWS가 만들지 않았다)

```bash
export AIDLC_METRICS_ENDPOINT=https://my-service.example.com/ingest
export AIDLC_METRICS_PREFIX=mycompany
```

→ **엔진 코드 수정 0줄**로 데이터 수집이 시작된다.

#### 구멍 2 — 신뢰 앵커(trust anchor)가 없음

`core/tools/aidlc-attest.ts`는 커밋 단위 출처 검증을 제공하면서 스스로 한계를 명시한다.

> resolve answers an INTEGRITY question ... It does NOT answer an AUTHENTICITY question ...
> A report is therefore informational **unless the caller supplies a trust anchor that the
> author of the change cannot forge.**

지원 앵커는 `--record-ref`(검증자가 통제하는 ref)와 `--require-trust signed`(서명 커밋 요구)
둘뿐이고, **둘 다 외부에서 운영해줘야 한다.**
경로 상태 분류(`verified` / `drifted` / `unattested` / `unverifiable` / `indeterminate` /
`excluded`)는 이미 구현되어 있어 감사 보고서 양식이 사실상 완성 상태다.

#### 구멍 3 — UI가 전혀 없음

산출물이 전부 마크다운 + JSON이다. 안에 담긴 것:
감사 이벤트 99종(`GATE_APPROVED`/`GATE_REJECTED`/`DECISION_RECORDED`/`HUMAN_TURN` 등),
`runtime-graph.json` 실행 텔레메트리, 토큰·비용 장부, 추적성 매트릭스, 리뷰 지문.
경영진·PM이 볼 화면은 0개.

### 아이디어 6개 (현실성 순)

| 순위 | 아이디어 | 근거 | 난이도 | 가격 모델 |
| --- | --- | --- | --- | --- |
| 1 | **AI 개발 비용·감사 대시보드 SaaS** | 구멍 1 + 3 | 낮음 | $20~40/개발자/월 |
| 2 | 한국형 컴플라이언스 플러그인 팩 | 플러그인 시스템 | 중간 | 연 300~1,000만원/기업 |
| 3 | 도입 컨설팅 + 워크숍 | `workshop` 스코프 | 낮음 | 300만~3,000만원 |
| 4 | AI 코드 공증(Attestation) 서비스 | 구멍 2 | 높음 | 연 5,000만~2억 |
| 5 | 스택별 관례 플러그인 (소액 다수) | 플러그인 시스템 | 낮음 | $10~30 |
| 6 | 사내 도구화 → 레퍼런스 확보 | — | 낮음 | 간접 |

#### 1순위 상세: 대시보드 SaaS

화면과 데이터 출처:

| 화면 | 출처 |
| --- | --- |
| 토큰·비용 실시간 | `WORKFLOW_COMPLETED` 이벤트의 비용 수치 |
| 승인 게이트 현황 | `GATE_APPROVED` / `GATE_REJECTED` 비율 |
| 진행 중 작업 칸반 | `aidlc-state.md` + 스테이지 포인터 |
| 추적성 그래프 | 요구사항 → 코드 → 테스트 연결 |
| 팀 학습 현황 | `team.md` 규칙 증가 추이 |
| 드리프트 경고 | attest의 `drifted` / `unattested` 경로 |

권장 아키텍처 — **재구현이 아니라 감싸기(wrapping)**:

```text
[ 기존 AI-DLC 엔진 ]  ← 수정하지 않음 (508개 테스트 자산 재활용)
        ↓ AIDLC_METRICS_ENDPOINT
[ 수집 API (Laravel 또는 Node) ]
        ↓
[ React 대시보드 ]  ← 제품화 지점
```

#### 2순위 상세: 한국형 컴플라이언스 팩

금융(전자금융감독규정·망분리), 의료(식약처 의료기기 SW·의료법),
공공(조달청 요건·산출물 표준), ISMS-P, 개인정보보호법/PIA.
**AWS가 만들 수 없는 영역**이라 경쟁이 없고, 규제가 계속 바뀌므로 구독 매출이 된다.

#### 3순위 상세: 컨설팅 + 워크숍

`docs/guide/workshop-mode.md`에 **교육·다중팀 운영이 이미 설계되어 있다.**

- 역할 정의: 통합 리드 / 유닛 팀 / 리뷰 그룹
- 유닛 클레임: `aidlc unit claim payments --team "결제팀"`
- 두 가지 운영 방식: 팀별 clone / 형제 worktree
- `workshop` 스코프 = 26스테이지, 테스트는 Minimal 기준선

자동 산출물 2종이 컨설팅 결과물을 대신한다.

- `/aidlc-outcomes-pack` → `OUTCOMES.md` 인수인계 문서 자동 생성
- `/aidlc-replay` → "그 자리에 없던 이해관계자용" 세션 서술 자동 생성

현재 최대 진입장벽은 문서 2.1MB가 전부 영어라는 점이며, 이는 곧 **한국어화 차별화 기회**다.

### 리스크와 대응

| 리스크 | 대응 |
| --- | --- |
| AWS가 대시보드를 직접 만들 수 있음 | AWS가 안 할 영역(한국 규제·한국어·UI)에 집중 |
| 토큰 비용 부담이 도입을 막음 | "비용 절감 컨설팅"으로 역이용 (모델 정책 튜닝) |
| AI-DLC 자체가 확산되지 않을 수 있음 | 대시보드를 AI-DLC 전용으로 만들지 말고 일반 에이전트 로그도 수용 |
| v2.9.0, 변화가 잦음 | 버전 핀 고정 + 마이그레이션을 유료 서비스로 |
| 1인 개발 한계 | 1·3순위에 집중, 4순위는 팀 구성 후 |

### 권장 로드맵

```text
[1~2주]  직접 사용 — 사이드 프로젝트에 설치, /aidlc bugfix 완주,
         aidlc/ 산출물 확인, AIDLC_METRICS_ENDPOINT를 로컬 서버로 걸어 데이터 관찰
[3~6주]  대시보드 MVP — 수집 API + React 화면 3개(비용·게이트·진행상황)
[2~3개월] 콘텐츠로 시장 반응 확인 — 한국어 가이드 공개, 문의 시 컨설팅 즉시 착수
[6개월+] 반응에 따라 선택 — 기업 문의 → 컴플라이언스 팩 / 금융·공공 → 공증 서비스
```

핵심 전략: **콘텐츠 → 컨설팅(즉시 현금) → 그 자금으로 제품 개발**.

## 9. React / PHP 관련

### 해석 A: AI-DLC로 React/PHP 앱을 개발할 수 있는가

가능하다. AI-DLC는 방법론 프레임워크이므로 결과물 언어와 무관하다.
`run-sensors` 훅은 프로젝트에 이미 설정된 도구를 호출하는 방식이라

- React → ESLint, `tsc`가 자연스럽게 연동
- PHP → PHPStan, PHP-CS-Fixer 연결 설정 필요
- 프로젝트 관례는 `project.md`에 적어두면 에이전트가 따른다

### 해석 B: AI-DLC 자체를 React/PHP로 재구현할 수 있는가

| 대상 | 가능성 | 설명 |
| --- | --- | --- |
| PHP로 엔진 재구현 | 기술적으로 가능 | 훅은 단순 셸 명령 실행이라 `php aidlc.php hook xxx`로 대체 가능. 다만 도구 72 + 훅 17 + 테스트 508을 이식하는 것은 1인 프로젝트로 비현실적 |
| React로 엔진 재구현 | 부적합 | 파일시스템 접근·프로세스 실행이 필요해 브라우저 런타임과 맞지 않음 |
| **React로 대시보드/뷰어** | **최적** | 산출물이 마크다운 + JSON이라 읽어서 시각화하기 좋음 |
| PHP(Laravel)로 수집 백엔드 | 적합 | 감사로그 수집·팀 대시보드·보고서 서버 |

**결론: 재구현이 아니라 감싸기.** 8절의 1순위 아키텍처와 동일하다.

## 10. 다음 단계 체크리스트

- [ ] 사이드 프로젝트에 설치 (Bedrock 대신 일반 인증으로 설정)
- [ ] `/aidlc bugfix` 1건 완주
- [ ] `aidlc/spaces/default/intents/<날짜>-<라벨>/` 산출물 전부 확인
- [ ] 로컬 수집 서버를 띄우고 `AIDLC_METRICS_ENDPOINT`로 실제 이벤트 관찰
- [ ] 대시보드 MVP 화면 3개 설계
- [ ] `plugins/test-pro/` 기반으로 플러그인 1개 실습
- [ ] 한국어 가이드 콘텐츠 1편 작성

## 참고 문서 (이 저장소 내부)

| 문서 | 내용 |
| --- | --- |
| `docs/guide/00-introduction.md` | 전체 개요 (첫 독서 추천) |
| `docs/guide/workflow-profiles.md` | 프로필(스코프) 선택 기준 |
| `docs/guide/16-worked-examples.md` | 버그픽스·기능 개발 실제 대화 전문 |
| `docs/guide/18-install-and-lifecycle.md` | 설치·설정·업그레이드·모델 정책 |
| `docs/guide/workshop-mode.md` | 다중팀·워크숍 운영 |
| `docs/reference/18-plugin-mechanism.md` | 플러그인 메커니즘 설계 |
| `docs/harness-engineering/10-authoring-a-plugin.md` | 플러그인 저작 실습 |
| `docs/reference/20-commit-provenance.md` | 커밋 출처 검증 위협 모델 |
