# skills-something

A collection of [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) for Claude Code.

## Skills

- **[network-troubleshooting](network-troubleshooting/)** — a structured method for troubleshooting infrastructure, networks and systems: initial scoping, zero assumptions, IS/IS NOT (Kepner-Tregoe), the scientific method, bottom-up OSI, 5 whys, an incident file, time-boxing and post-incident review.
- **[cicd-pipeline](cicd-pipeline/)** — method and checklists for shipping a containerized app with GitHub Actions, Docker and Kubernetes: build once, publish a signed immutable image, promote that exact digest through gated environments to production. Covers the inventory to gather first, the dev/infra boundary, build-once/promote-by-digest, supply-chain integrity, deployment strategies, secrets, health-gated rollout, GitHub Environments approvals, and rollback.
- **[writing-style](writing-style/)** — writing standard for technical documentation, replies and written communication: clarity, concision, objectivity and precision, cohesion, correctness, impersonality and completeness, with a pre-delivery review checklist.

## Installing

Copy a skill folder into your skills directory:

```
~/.claude/skills/<skill-name>/        # personal, all projects
<project>/.claude/skills/<skill-name>/ # project-scoped
```

Each skill is a single `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the instructions. Claude loads it on demand when the task matches the description.

## License

MIT — see [LICENSE](LICENSE).
