# Agent Skills

A collection of [agent skills](https://agentskills.io) for Claude Code.

Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. This repository follows the [agentskills.io specification](https://agentskills.io/specification).

## Available Skills

| Skill | Description |
|-------|-------------|
| [excalidraw](skills/excalidraw/) | Create and edit Excalidraw diagrams |

## Usage

Install skills via the Claude Code plugin marketplace:

```bash
claude install needcaffeine/skills
```

Or add individual skills to your project's `.claude/settings.json`:

```json
{
  "skills": ["needcaffeine/skills/skills/excalidraw"]
}
```

## Creating New Skills

Copy `template/SKILL.md` into a new directory under `skills/` and follow the [agentskills.io specification](https://agentskills.io/specification) for frontmatter and content guidelines.

## License

[MIT](LICENSE)
