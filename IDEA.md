# IaC AutoFix — Project Plan

## 0. Purpose of This Document
This file is the single source of truth for the project. It serves two purposes:
1. A personal checklist to track progress step by step.
2. Context to feed to an AI assistant while building, so it can recommend accurate next steps based on the actual plan (not guesses).

---

## 1. Why This Project Exists (Intentions & Priorities)

- This is a **CS/portfolio project** meant to be a **marquee project on my resume**.
- Two goals, in priority order:
  1. **Learning** — primarily deepen backend/infra skills (already have some experience via internship: K8s migration, AWS cross-account IAM, GHA), and secondarily learn **Generative AI / agentic AI engineering** (not classical ML). GenAI is a "keep up with the industry" skill, not the main focus.
  2. **Resume signal** — a project that reads as a genuine B2B-style tool, not a tutorial clone, with staying power (should still make sense to talk about in interviews a year from now).
- **Target role:** Backend / Infrastructure engineering. Not frontend-focused. Not primarily targeting ML/AI research roles.
- **Constraints:**
  - Must be **entirely free** to build and run — no paid APIs, no paid cloud tiers. I'm a college student with no budget.
  - **No real business relationship / no real company data** — everything must work using public repos, self-made test repos, or forked repos.
  - Ongoing / open-ended timeline — no fixed deadline, meant to grow over time.
- **Explicit non-goals:**
  - Not doing classical ML (no model training/fine-tuning).
  - Not trying to solve a dramatic "production outage" problem — this is a **DevSecOps / auto-remediation toil-reduction tool**, a legitimate and respected category, not a firefighting tool. Should be pitched honestly as such.
  - Not using React (comfortable with plain HTML/CSS/JS instead).

---

## 2. The Idea

**An agentic AI tool that scans Infrastructure-as-Code (Kubernetes YAML / Terraform) for security misconfigurations and automatically fixes them — not just reports them.**

### The real-world problem it addresses
Tools like Checkov already detect IaC misconfigurations for free, and many teams already run them in CI. But detection alone just creates a backlog — a human still has to read the cryptic finding, understand why it matters, write the correct fix, re-verify it worked, and open a PR. That manual "glue work" after detection is the actual toil this tool removes. This category is real and growing — sometimes called **auto-remediation** / **policy-as-code with autofix**.

### Honest positioning
- NOT "smarter than pasting into a chatbot" — the LLM's raw answer quality is comparable to asking Claude/ChatGPT directly.
- The actual value is the **automation, orchestration, and verification pipeline** around the LLM call — turning a manual, one-off chat interaction into something that runs unattended, at scale, and never blindly trusts the model's output.
- This is also the accurate definition of what "agentic AI engineering" jobs actually value: not "getting a better answer," but "wiring a model into a system that takes real actions and checks its own work."

### Ideas considered and rejected (for reference)
- Support Ticket Triage & Auto-Responder — good idea, but needs realistic fake data (a business I don't have), less infra-story fit.
- GitHub PR Review Bot — good but narrower/less deep than the chosen idea.
- Internal Docs Q&A Assistant (RAG) — deepest "textbook RAG" learning but weakest backend/infra story, and needs synthetic company docs.
- Contract/Invoice Analyzer — strong demo but least backend depth.
- Incident/Alert Triage — strong infra story but needs synthetic alert/log data which feels less authentic than real IaC repos.
- **Chosen: IaC Auto-Remediation Agent** — wins because it needs zero synthetic/business data (real public IaC repos exist for free), stays free indefinitely (few LLM calls per run), and is the strongest fit for a backend/infra resume since most of the hard problems are infra problems (git/GitHub orchestration, verification logic, multi-step automation), with GenAI as a bounded, well-guarded component.

---

## 3. Prerequisites / Setup

- [x] **Create a dedicated bot GitHub account** (separate from personal account) so automated commits/PRs are clearly distinguishable from manual work — standard real-world practice (like Dependabot/Renovate).
  - **Bot Gmail:** `korvelabot@gmail.com`
  - **Bot GitHub username:** `korvela-bot`
  - This bot account is reusable across future projects (not tied only to this one).
- [ ] **Generate a Personal Access Token (PAT)** for `korvela-bot`, scoped to repo write access. Store as an environment variable / secret — never hardcode or log it. Use a separate token per project so a leak only affects one project.
- [ ] **Fork the test target repo** (e.g. `bridgecrewio/terragoat`) into **my own account** (`GauravChanda7/terragoat`) — NOT into the bot account. This makes "my" account the stand-in for a real client/company that owns the repo.
- [ ] **Add `korvela-bot` as a collaborator** on `GauravChanda7/terragoat` with write access (accept the invite from the bot account). This correctly simulates the real-world scenario: the bot has been *granted* access to a repo it doesn't own, rather than just acting on its own property.
- [ ] **Set up free LLM API access** — Groq (free tier) or Google Gemini (free tier). No credit card required.
- [ ] **Install Checkov** locally (free, open-source, `pip install checkov`).
- [x] **Project name finalized: "IaC AutoFix"** — repo created under my own account: `github.com/GauravChanda7/iac-autofix`. (Considered and rejected: ConfigGuard — collides with an existing real product at configguard.com; also considered RemediAI, SecureIaC, GuardRail, Patchwright.)
- [x] **License: MIT** — chosen for maximum simplicity and lowest friction for anyone (including companies) to view, clone, or build on the project. Apache 2.0 was considered (patent grant, change-tracking) but its extra protections don't meaningfully apply here; GPL/AGPL avoided since copyleft can discourage company adoption of a dev tool.

---

## 4. Tech Stack

| Layer | Tech | Purpose |
|---|---|---|
| Backend language | **Python** | Orchestrates the whole pipeline: clone → scan → agent loop → git/GitHub actions. Already comfortable here. |
| Backend framework | **FastAPI** | Exposes API (e.g. `POST /scan`) for the frontend to call; async-friendly for slow operations (clone, LLM calls, subprocess calls). |
| Frontend | **Plain HTML + CSS + JS** | Single page: paste repo URL(s) → click "Scan" → see findings/fix status. No React (explicit preference). |
| IaC scanning | **Checkov** (open-source, free) | Finds K8s/Terraform misconfigurations. Called as a subprocess, output parsed as JSON. Do not reimplement detection logic. |
| AI / GenAI | **Groq or Gemini API (free tier)** | The "planning" brain of the agent — given a finding, proposes a fix. Called via HTTP/SDK. |
| Version control ops | **Git CLI (via subprocess) + GitHub REST API** | Git handles clone/branch/commit/push. GitHub API (with PAT) handles opening the actual Pull Request, since pushing a branch alone does not create a PR. |
| Database (optional, later) | **PostgreSQL** (e.g. free NeonDB tier, already used in ElBalon) | Stores scan history — which repos scanned, findings, fixes, PR links. Needed for a future dashboard, not required for MVP. |
| Containerization | **Docker** | Packages backend (and DB, if used) into portable containers; required step before K8s deployment. |
| Orchestration/deployment | **Kubernetes (via Minikube)** | Runs the Dockerized backend locally, completely free, zero cloud cost/risk. Real showcase of actual K8s experience. |
| CI/CD | **GitHub Actions (GHA)** | Automates testing/building of *this project's own repo* — separate concern from the tool's function. |
| Optional / stretch | **Go** | See Section 6 below — not required for MVP. |

---

## 5. Core Application Flow (MVP)

```
User pastes repo URL, clicks "Scan"
  → Backend clones the repo (git clone via subprocess)
  → Backend runs Checkov on the cloned folder → list of findings (JSON)
  → For each finding:
        Plan  → send finding to LLM (free tier), get proposed fix
        Act   → apply fix by editing the real file on disk
        Verify → check three things, in order:
                   1. File still parses as valid YAML (reject if not, retry)
                   2. Re-run Checkov: is this specific finding now resolved?
                   3. Did any previously-passing check break? (regression check)
                 If verification fails → send failure back to LLM, retry (capped attempts)
                 If verification passes → accept the fix
  → Once fixes are accepted:
        Create a new branch (git checkout -b)
        Commit accepted changes (git add, git commit)
        Push branch using bot PAT embedded in the push URL
        Call GitHub REST API (same PAT) to open a real Pull Request
  → Backend returns summary to frontend: findings, what got fixed, what failed, PR link
  → Frontend displays results to user
  → Backend cleans up the temporary cloned folder
```

Notes:
- If testing against a repo I don't own (e.g. terragoat), the whole flow happens on **my fork** of it, and the PR is opened within that fork — not submitted upstream to the original project.
- Helm (templates repo + values repo) support is a **later addition**, deferred for now to keep MVP simple. When added: clone both repos, run `helm template <templates-folder> -f <values-file>` to render a final YAML before handing it to Checkov; everything downstream is unchanged.

---

## 6. Later / Optional Additions

### Go (optional, learn-if-time)
Not required for MVP. Add only after the Python pipeline fully works end-to-end. Best entry points, in order of recommended priority:
1. **Go CLI / GitHub Action** — rewrite the "scan + report" piece as a single compiled Go binary distributed as a GitHub Action, so teams can drop it into their own CI with zero Python/pip setup overhead (matches how real tools like kubectl, Terraform, Trivy are built). This is the most natural, most visible use of Go here.
2. **Verification microservice in Go** — pull the verify step (YAML validity + re-scan comparison + regression check) into its own Go service called by the Python orchestrator over HTTP — demonstrates real polyglot microservice design, not an arbitrary language choice.
3. **Parallel scanning worker in Go** — only if scanning many files/repos becomes an actual measured bottleneck; use goroutines to scan in parallel. Save for last — add concurrency because it's needed, not for its own sake.

### Database / Dashboard (optional)
- Add PostgreSQL once there's a reason to persist history (not needed for a single live demo).
- Build a small dashboard showing past scans, fixes, and success rate over time.

### Distribution as a GitHub Action (Role B, stretch goal)
- Package the scanner as an installable GitHub Action so teams can run it automatically in their own CI on every PR, instead of only via the website.
- This is where the Go CLI (above) pays off — instant startup, no dependency installation.
- Consider a proper **GitHub App** (separate bot identity, installable by other users/orgs without sharing a personal token) only if this tool is ever meant to be used by people other than me.

### RAG over past fixes (future stretch)
- Once there's a history of past fixes, retrieve similar past findings/fixes to inform new ones — deepens the GenAI-engineering learning surface if desired later.

---

## 7. What I'm Actually Learning (AI-Specific Skills)

Concrete GenAI/agentic skills this project is meant to build:
1. **Prompt engineering for machine-usable output** — structuring prompts so responses are directly usable by code (e.g. valid YAML snippets), not just human-readable text.
2. **Tool use / function calling** — wiring the LLM to trigger real actions (scanner, file edits, git commands) instead of just returning chat text. This is the core "agentic AI" skill.
3. **The agent loop pattern (plan → act → verify → retry)** — building this loop by hand (rather than relying blindly on a pre-built framework) to actually understand how agents work underneath.
4. **Grounding / not trusting AI output blindly** — the verification layer exists specifically so the system never accepts an LLM's claim without checking it against ground truth (re-scan, YAML validity, regression check).
5. **Working within free-tier constraints** — rate limits, model selection for cost/speed tradeoffs, graceful error handling.

Explicitly NOT covered by this project: classical ML/model training, prompt-injection security research, or reliance on heavy agent frameworks (LangGraph/CrewAI) — hand-rolling the loop is intentional, to understand fundamentals first.

---

## 8. Build Order (Suggested Sequence)

1. Set up bot GitHub account + PAT + fork of test repo.
2. Hand-write 3–5 small, intentionally-broken K8s YAML files as an initial, predictable test repo.
3. Build and test: clone → run Checkov → parse/print findings. (No AI yet.)
4. Add the agent loop: plan (LLM call) → act (file edit) → verify (3-layer check) → retry logic. Test against the hand-written files first, since expected findings are known in advance.
5. Add git branch/commit/push + GitHub API PR creation, tested against the bot account's own repo/fork.
6. Validate the full pipeline against a real-world repo (e.g. forked `terragoat`) to prove it holds up on messier, real code.
7. Build the simple HTML/JS frontend + FastAPI endpoints to tie it together into a usable demo.
8. Dockerize the backend; deploy via Minikube.
9. Set up GitHub Actions CI for the project's own repo (tests/build).
10. (Optional, later) Add Helm support, PostgreSQL history/dashboard, Go components, GitHub Action distribution, RAG over past fixes — in whatever order makes sense once the core is solid.

---

## 9. Open Decisions
- [x] Final project name: **IaC AutoFix**
- [x] Final bot account: **korvela-bot** (korvelabot@gmail.com)
- [x] License: **MIT**
- [ ] Whether/when to introduce PostgreSQL
- [ ] Whether/when to introduce Go, and which of the three entry points to use

## 10. Repo Ownership Map (for clarity)
- **Main project repo** (this tool's source code — backend, frontend, IDEA.md): `github.com/GauravChanda7/iac-autofix` — owned by me, normal personal commit history.
- **Test target repo** (the forked "victim" repo the tool scans/fixes): `github.com/GauravChanda7/terragoat` — owned by me (stand-in for a real client/company), with `korvela-bot` added as a collaborator with write access.
- **Bot identity** (`korvela-bot`) only ever acts *on repos it's been granted access to* (like the forked terragoat) — it never owns the target repo itself. This mirrors the real-world deployment model: a company grants the tool's bot access to their repo; the tool never needs to own or fork anything in production.
