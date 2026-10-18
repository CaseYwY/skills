---
name: manage-project-skills
license: MIT
description: Manage a project's agent skills across four modes — T0 start-up inventory (read-only), T1 gap-driven single install mid-task, T2 batch install after the plan is settled, and a library audit mode (duplicates, conflicts, bloat, dead references). Local-first. Use when the user says "初始化项目技能/管理项目技能", "给这个项目配技能/装技能", "审计技能库", "audit skills", or a task hits a durable capability gap. NOT for authoring skills (use skill-creator) or one-off answer lookups (search docs or `npx skills find` instead, install nothing).
---

# manage-project-skills — project skill selection & audit

## 0. Rule hierarchy (read first)

1. **Authorization (highest, no exceptions)**: no install/update/remove, no project file writes without explicit user approval; exception: approved ledger writes. approval = specific skill + source + scope; "manage my skills" authorizes nothing individual.
2. **Process**: follow the selected mode (T0/T1/T2/audit); "forbidden" steps never skipped for efficiency.
3. **Guidance**: local-first, restraint — deviations allowed with a stated reason.

Core: **missing beats mis-installed** — pull-back ≈ 1 min; wrong install = permanent context + trigger competition. restraint, mandatory justification, local-first.

## 1. Modes

Requirements unclear ⇒ full needs unknowable ⇒ no mass-install at kickoff.

- **T0 · kickoff inventory (READ-ONLY)**: steps 1-2 + chat report ("covered / not installing"). zero installs/writes; ledger init = separate approved write. coverage matrix = stack-inferable needs only. fast path: ledger exists && no skill talk ⇒ read ledger only. re-inventory if: skill dirs / AGENTS.md / lockfile / agent changed ∨ ledger ≠ disk ∨ user asks.
- **T1 · gap-driven (mid-task)**: classify first. one-off answer (API detail, single command) ⇒ look up, no install. reusable (workflows/standards/toolchains this repo hits again) ⇒ pull one skill ⇒ record. entry relies on AGENTS.md rules (see MAINTENANCE.md), not model self-assessment.
- **T2 · plan settled**: batch install, each candidate justified by the confirmed plan. full steps 1-7.
- **Audit**: user asks review/trim/clean ⇒ see end.

All modes share the project ledger `.agents/skills-rationale.md` (if approved); read it first.

## 2. Flow

### 1. Profile the project

Read if present (don't guess): manifests (package.json / pyproject.toml / go.mod / Cargo.toml / *.csproj); layout + monorepo signals (workspaces, turbo.json); infra (.github/workflows, Dockerfile, vercel.json, wrangler.toml); README head, docs/agents/.

### 2. Inventory coverage (local-first; dedupe before pulling)

Order: project-installed → global → registry. Two classes:

- **personal skills**: global, cross-project. don't duplicate into the project; record target agent. global presence ≠ declared project dependency.
- **project-critical**: workflow/delivery depends on it. record source + version/commit + restore path — even if globally installed; new machines don't inherit globals.

Scan: project dirs (.agents/skills, .opencode/skills, .claude/skills = common examples, confirm per agent docs; unreachable ⇒ "unchecked", never "no skills"); global via `npx skills ls -g`; source manifest & skills-lock.json (sync if present).
Output: coverage matrix (need → satisfied by / gap).

### 3. Fill gaps from reliable sources

Trust tiers: vendor-official (anthropics/skills, openai/skills, supabase/agent-skills) > original-author (mattpocock/skills, cursor/plugins) > community > unverifiable (raise scrutiny). else `npx skills find <kw>` — install count = reference, not trust. nothing fits ⇒ say so + propose skill-creator; no near-miss hacks.

### 4. Budget: no hard cap, need-driven

- each candidate needs one concrete reason ("project has X ⇒ Y"); no reason ⇒ drop.
- >~10 candidates = analysis bug ⇒ re-audit steps 1-2.
- marginal ⇒ "deferred" list (pull-back is cheap).
- dedup ≠ auto-prune: description overlap ⇒ candidate group ⇒ compare triggers/IO/side-effects/deps/versions ⇒ merge/remove only with replacement evidence + user approval; else keep both, mark to-compare.

### 5. Security gate: before install, user-approved

Review the whole package, not just SKILL.md:

1. read SKILL.md fully + all referenced files/scripts/config/deps; per script check: exec / network / writes / credentials.
2. record scan evidence: scanner + verdict + matched version + date. no scan = unknown, not safe. scanner disagreement ⇒ list, don't average.
3. private source without scans ⇒ manual full-file review replaces gate, marked "unscanned".
4. med+ risk or unexplained behavior ⇒ user decides; nothing installs while unclear.

### 6. Install

```bash
# auto-detects the agent (report detection to user); multi-agent machines: pass --agent
npx skills add <owner/repo>@<skill> --copy -y
```

- channel 2 — version-pinned / npm-unreachable fallback: `gh skill install <owner/repo> <skill[@version]> --agent opencode` (preview; copies files, no skills-lock update ⇒ ledger records source+version; always pass --agent — non-interactive default is github-copilot). one channel per skill — don't mix.
- `--copy` = snapshot isolation, NOT version pinning: no upstream sync, no version identity, reinstall changes it. reproducibility ⇒ ledger records: source + path + version/commit (+hash).
- project-critical workflow skills: `--copy`, never symlink (symlink tracks upstream).
- `skills-lock.json` produced ⇒ commit (restore: `npx skills experimental_install`); ensure `.agents/` not gitignored.
- effectiveness varies: OpenCode hot-reloads (verified); other agents may need restart ⇒ prompt user to restart + re-verify.
- GitHub unreachable ⇒ report + propose local cache; never skip silently.

### 7. Verify & record (four layers; missing layer = "unverified")

- file: exists, path correct, references complete
- manifest: `npx skills ls` matches lockfile & disk (visible ≠ loadable)
- load: target agent actually loads the skill
- behavior: one representative positive task works
- write decisions (what/why/deferred/verified-to) → `.agents/skills-rationale.md`
- remove: prefer `npx skills remove <name> -y` (params per `npx skills --help`); never just delete the dir (lockfile desync); record source+version before delete; back up if not recoverable; re-check four layers after.

## 3. Audit mode

Includes this skill itself. report → user approval → execute. checks:

1. duplicates: cluster descriptions ⇒ candidate groups ⇒ compare triggers/IO/side-effects/deps/versions; replacement evidence + approval ⇒ merge/remove; else keep both (to-compare). never delete on similarity.
2. conflicts: same-scenario contradictions ⇒ record quotes + scope ⇒ user adjudicates.
3. bloat: no usage telemetry — "never triggered" = user-sourced. measurable: description bloat vs function, overlap w/ higher-frequency skills. downgrade ⇒ state restore path.
4. dead refs: required input (missing = blocking) / optional dep (absent = skip) / expected output (runtime-created; absence ≠ error). dead ref ⇔ required input missing + no recovery.
report: evidence → recommendation → impact → authorization. pre-delete: confirm recoverability (`npx skills find`); non-recoverable ⇒ back up + ask. findings → ledger.

## 4. Maintenance & enforcement

AGENTS.md rule templates, trigger-failure diagnosis loop, 15-scenario test set, release flow → **MAINTENANCE.md** — run before/after editing this skill.
