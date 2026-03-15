# P-Review — Performance Review Skill for Claude Code

A Claude Code skill that generates structured performance reviews by analyzing your GitHub contributions (PRs, commits) across organizations.

## What it does

1. Auto-detects your GitHub username and organizations
2. Asks who will read the review (CEO, Tech Lead, or HR)
3. Scans all your PRs and commits for the review year in parallel
4. Groups contributions by project and asks for business context
5. Generates a formatted P-Review with:
   - **Q1**: Top 5 projects table (Product, Project, Key Task, Key Accomplishment, Priority)
   - **Q2**: Main accomplishments narrative with business impact

The output tone, technical depth, and project prioritization automatically adapt to the target audience.

## Installation

### Option 1: Symlink (recommended)

```bash
# Clone the repo
git clone https://github.com/thinhkhang97/p-review.git ~/.agents/skills/p-review

# Symlink into Claude Code skills
ln -s ~/.agents/skills/p-review ~/.claude/skills/p-review
```

### Option 2: Direct copy

```bash
# Create the skill directory
mkdir -p ~/.claude/skills/p-review

# Download the skill file
curl -o ~/.claude/skills/p-review/SKILL.md \
  https://raw.githubusercontent.com/thinhkhang97/p-review/main/SKILL.md
```

### Option 3: Project-level (for teams)

Copy the `SKILL.md` into your project's `.claude/skills/p-review/` directory:

```bash
mkdir -p .claude/skills/p-review
cp SKILL.md .claude/skills/p-review/SKILL.md
```

Every developer working in that repo will have the skill available.

## Usage

Start a Claude Code session and say any of:

- "Help me with my performance review"
- "I need to do my P-Review for 2025"
- "Generate my annual review"

Claude will guide you through the process step by step.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated
- Access to the GitHub organizations you want to include

## Target Audiences

| Audience | Focus | Priority |
|----------|-------|----------|
| **Non-technical Manager** (CEO, COO, PM) | Business impact, user growth, revenue, cost savings | Products driving growth > Cost reduction > New launches > Infrastructure |
| **Tech Lead** (EM, CTO, Senior Engineer) | Technical depth + business impact | Complex challenges > Architecture > Code quality > DevOps > Mentoring |
| **HR** | Role responsibilities, growth, collaboration | Org impact > Collaboration > Growth > Process > Initiative |

## License

MIT
