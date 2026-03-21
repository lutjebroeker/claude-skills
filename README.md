# claude-skills

Custom [Agent Skills](https://agentskills.io/specification) for Claude Code.

## Skills

| Skill | Description |
|-------|-------------|
| [prototype-builder](skills/prototype-builder) | Generate self-contained clickable HTML prototypes — no external APIs, no Figma. Pure HTML/CSS/JS. |
| [notebooklm-youtube](skills/notebooklm-youtube) | YouTube URL(s) → NotebookLM pipeline: transcript ophalen, analyses uitvoeren, infographic/audio/flashcards genereren. |

## Installation

### Claude Code (global)

```bash
npx skills add git@github.com:lutjebroeker/claude-skills.git
```

Or manually: copy the `skills/` directory into `~/.claude/skills/`.

### Per project

Copy into `.claude/skills/` at the root of your project. Claude Code auto-discovers them.
