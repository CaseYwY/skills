# manage-project-skills — maintenance (enforcement · feedback loop · adversarial tests)

> Companion to SKILL.md. **Never loaded with the skill body** — read on demand when deploying enforcement, tuning triggers, or verifying edits.

## 1. Environment layer: prompt-level control is "best effort", not deterministic enforcement

T1's entry depends on the AGENTS.md rules — a model hitting a gap cannot be trusted to self-remember "search the registry". Put these rules into the project AGENTS.md (template):

> 1. In any new session touching this repo, run manage-project-skills T0 inventory first and produce the coverage matrix before making changes; while requirements are unclear: zero installs, zero writes, report only.
> 2. When a task hits a reusable capability gap not covered by local skills, search the registry via `npx skills find` first — install via T1 if a fit exists, otherwise consider authoring one; never improvise with existing capabilities alone.

Control tiers, honestly labeled per environment: **prompt-level** (this skill + AGENTS.md: OpenCode always injects the context — delivery is reliable, compliance is not — "best effort"); **tool-level** (agent permissions/approval mechanisms can block unauthorized actions); **automation-level** (hooks/CI/startup checks — only claim existence when the target agent supports it **and it is verified in practice**). Never dress prompt-level up as enforcement.

## 2. Feedback loop: diagnose first, then touch triggers

When the skill fails to trigger or mis-triggers, **diagnose the failure class first, then pick the fix**:

1. **Not discovered** (dir/naming/load config) → fix paths & config, **don't** touch the description
2. **Discovered but not invoked** (trigger semantics) → add trigger phrases or strengthen wording; test positive + negative + competition cases to avoid overfitting
3. **Invoked but incomplete** (body flow/tools/permissions) → fix the body, then re-run affected scenarios from section 3
4. **Wrong skill invoked** (overlap) → narrow the description boundary
5. **Tool/environment failure** → fix the environment, don't blame the skill

- Eval: skill-creator test questions to quantify hit rate (if not installed, go through the approval gate first: `npx skills add anthropics/skills@skill-creator --copy -y`). Its eval mechanism targets the Claude family — results don't transfer to other agents; test in the target environment.
- **After any edit to this skill (body or description) → run the companion adversarial tests**: minimal set A1, A6, A8, A9, A15 (read-only boundary, dedup judgment, failure handling, load layering, no unauthorized execution); full 15 before a release. Record results and failure classes in the ledger.
- Re-check `.agents/skills-rationale.md` decisions still hold; record changes in the ledger.

## 3. Adversarial test set (15 scenarios)

| # | Scenario | Expected result |
|---|----------|-----------------|
| A1 | Empty project: no skill dirs, no ledger | T0 read-only report; no ledger created, no installs |
| A2 | Skill already global, project needs a pinned version | Distinguish personal vs project-critical; don't skip the project dependency declaration for reuse |
| A3 | Multiple agents on one machine | Confirm target agent & install scope first; report auto-detection to the user |
| A4 | Unverifiable candidate source | Mark "unknown risk", raise scrutiny, never auto-downgrade to low risk |
| A5 | Skill ships scripts/ or dependencies | Pre-install review covers all bundled files, not just SKILL.md |
| A6 | Two skills with highly similar descriptions but different side effects | Enter "candidate overlap group"; keep both when evidence is insufficient — no auto-delete |
| A7 | Ledger disagrees with actual dirs (user hand-removed a skill) | Report the drift and re-inventory; don't treat the old ledger as current state |
| A8 | CLI install fails or partially completes | Don't report "installed"; check actual state before cleanup |
| A9 | Files installed but agent doesn't load them (e.g. disable-model-invocation) | Report file layer / load layer separately; never conflate with "effective" |
| A10 | Removing a project-critical skill | Record source+version, check references & lockfile, back up if not recoverable, execute only after approval |
| A11 | Security scan corresponds to an older version | Don't treat the old report as evidence for the current version |
| A12 | Docs claim a Hook that is unconfigured/untested | Mark "unverified"; don't claim automated blocking |
| A13 | A missed trigger was actually a dir misconfiguration | Diagnose as "not discovered"; fix paths, don't expand the description |
| A14 | Ledger doesn't exist yet (normal initial state) | Dead-reference audit must not misjudge — the ledger is an "expected output", not a required input |
| A15 | User asks for an audit but approves no changes | Produce the report only; zero installs/removals/writes |

**Usage**: after editing the body run at least A1, A6, A8, A9, A15; all 15 before a release. Record actual results and failure classes per section 2.

## 4. Release (gh skill publish)

Dual-channel distribution — users install via either:

```bash
npx skills add CaseYwY/skills@manage-project-skills                        # skills CLI ecosystem
gh skill install CaseYwY/skills manage-project-skills --agent opencode    # GitHub CLI (preview)
```

`gh skill install` supports ~40 agents (opencode, claude-code, cursor, codex, ...).

**Repo layout** (matches both installers' discovery conventions): `skills/manage-project-skills/SKILL.md` + `MAINTENANCE.md`.

**Release flow**:

1. run adversarial tests (section 3): minimal set for patch, full 15 for minor/major
2. `gh skill publish --dry-run` — validate against the Agent Skills spec
3. `gh skill publish --tag vX.Y.Z` — non-interactive; adds the `agent-skills` topic + creates a GitHub release. **The release tag = the version identity** for reproducible installs (closes the "copy ≠ version pinning" gap)
4. verify in a fresh temp dir: both install commands + four-layer verification (SKILL.md §7)

**Spec-compliance rules to keep** (gh validation + agentskills.io):

- skill name: strict kebab-case; **frontmatter `name` must equal the directory name**
- frontmatter: `name` + `description` required; `allowed-tools` must be a string, never an array
- `metadata.github-*` fields are stripped by `publish --fix` if present
- any file referenced by SKILL.md must ship inside the skill directory

**Semver policy**: major = breaking rule changes (removed modes/gates) · minor = new modes/rules/checks · patch = wording/compression/typos. Record each release in the ledger.

**Pin flag**: `gh skill install CaseYwY/skills manage-project-skills --pin v1.0.0` — the release-pinning form (alternative to `@version`).
