# skills

Agent skills (SKILL.md) for OpenCode, Claude Code & any skills-CLI agent.
面向 OpenCode、Claude Code 及任何支持 Agent Skills 规范的 AI 编程助手的技能集合。

## Skills / 技能列表

### [manage-project-skills](skills/manage-project-skills/SKILL.md)

Initialize, adjust, or audit a project's agent-skill set:
初始化、调整或审计项目的技能集：

- **T0** start-up inventory (strictly read-only) / 开工盘点（严格只读）
- **T1** gap-driven single install mid-task / 任务中缺口驱动单点安装
- **T2** batch install once the plan is settled / 方案确定后批量安装
- **Audit** library check: duplicates, conflicts, bloat, dead references / 技能库审计：重复、矛盾、膨胀、死引用

Local-first · approval-gated · four-layer verification (file / manifest / load / behavior)
本地优先 · 授权门控 · 四层验证（文件 / 清单 / 加载 / 行为）

## Install / 安装

```bash
# skills CLI (skills.sh ecosystem)
npx skills add CaseYwY/skills@manage-project-skills

# GitHub CLI
gh skill install CaseYwY/skills manage-project-skills --agent opencode
```

Works with OpenCode, Claude Code, Cursor, Codex and ~40 other agents.
支持 OpenCode、Claude Code、Cursor、Codex 等 40+ agent。

## Versioning / 版本

Releases are tagged (semver) — pin a version via `gh skill install CaseYwY/skills manage-project-skills@v1.0.0`.
发布以 semver 打 tag——用 `gh skill install CaseYwY/skills manage-project-skills@v1.0.0` 锁定版本。

## License

[MIT](LICENSE)
