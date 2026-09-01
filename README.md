# skills-something

A collection of [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) for Claude Code.

## Skills

- **[network-troubleshooting](network-troubleshooting/)** — a structured method for troubleshooting infrastructure, networks and systems: initial scoping, zero assumptions, IS/IS NOT (Kepner-Tregoe), the scientific method, bottom-up OSI, 5 whys, an incident file, time-boxing and post-incident review.

## Installing

Copy a skill folder into your skills directory:

```
~/.claude/skills/<skill-name>/        # personal, all projects
<project>/.claude/skills/<skill-name>/ # project-scoped
```

Each skill is a single `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the instructions. Claude loads it on demand when the task matches the description.

## License

MIT — see [LICENSE](LICENSE).
