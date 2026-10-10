# agents-skills

A collection of skills for AI coding agents (OpenCode, Claude Code, and
other agents-runtime-compatible tools). Each skill is a self-contained
`SKILL.md` that instructs an agent how to perform a task correctly.

## Skills

| Skill | When to use |
|-------|-------------|
| [commit-code](commit-code/SKILL.md) | Creating a git commit — enforces Conventional Commits v1.0.0 format, diff analysis before composing the message, and a required `On-behalf-of` footer |

## Usage

Install all skills from this repo:

```bash
npx skills add icyzyy/agents-skills
```

Or copy a skill manually into your agent's skills folder:

```bash
cp -r commit-code ~/.config/opencode/skill/   # OpenCode
cp -r commit-code ~/.claude/skills/           # Claude Code
```

## Commit messages

This repo's own history follows the conventions defined in its
[commit-code skill](commit-code/SKILL.md): Conventional Commits v1.0.0 with
one `type(scope): description` line per concern and a required
`On-behalf-of` footer.

## License

[MIT](LICENSE)
