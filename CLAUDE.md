# Agent Skills Repository

This repository contains agent skills following the [agentskills.io](https://agentskills.io) standard for use with Claude Code.

## Repository Structure

```
.claude-plugin/marketplace.json   # Plugin marketplace registration
template/SKILL.md                 # Starter template for new skills
skills/<name>/SKILL.md            # Skill definition (required)
skills/<name>/LICENSE.txt         # Skill license (recommended)
skills/<name>/scripts/            # Optional helper scripts
skills/<name>/references/         # Optional reference docs
skills/<name>/assets/             # Optional templates, images, etc.
```

## Skill Format

Each skill lives in `skills/<name>/` and must contain a `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name          # Must match the directory name, lowercase with hyphens
description: ...          # Keyword-rich description of what the skill does and when to use it
license: MIT              # License identifier
allowed-tools: [...]      # Tools the skill needs access to
---
```

The `description` field is critical for skill discovery — it determines when Claude triggers the skill. Include specific use cases, trigger phrases, and file types.

The body contains markdown instructions that Claude follows when the skill is activated.

## Adding a New Skill

1. Copy `template/SKILL.md` to `skills/<name>/SKILL.md`
2. Set `name` in frontmatter to match the directory name
3. Write a comprehensive `description` for discovery
4. Add instructions in the markdown body
5. Add `LICENSE.txt` to the skill directory
6. Register the skill path in `.claude-plugin/marketplace.json` under the appropriate plugin group

## Validation Checklist

- [ ] `name` in frontmatter matches the directory name
- [ ] `description` is keyword-rich and includes when-to-use guidance
- [ ] `name` is lowercase, uses hyphens (no spaces or underscores)
- [ ] Skill path is listed in `.claude-plugin/marketplace.json`
- [ ] `LICENSE.txt` exists in the skill directory
