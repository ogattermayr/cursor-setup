# Cursor Setup

Reusable Cursor AI skills, subagents, and commands for web development projects.

## Installation

### Option 1: Copy to Global (Recommended)

Copy to your user-level Cursor folder for all projects:

```bash
# Clone the repo
git clone https://github.com/ogattermayr/cursor-setup.git

# Copy to global Cursor folders
cp -r cursor-setup/skills/* ~/.cursor/skills/
cp -r cursor-setup/agents/* ~/.cursor/agents/
```

Skills and subagents will be available in ALL Cursor projects.

### Option 2: Copy to Project

Copy to a specific project's `.cursor/` folder for project-specific usage.

### Note on Remote Rules

Remote Rules (GitHub import) only work for `.cursor/rules/` files, not skills or agents. Use the copy method above for skills.

---

## Contents

### Skills

#### Security (Trail of Bits)
- `security/security-audit/` - Security vulnerability detection checklist
- `security/sharp-edges/` - API design footguns and dangerous patterns
- `security/differential-review/` - Security-focused PR/diff review
- `security/semgrep/` - Static analysis scanning

#### Frontend
- `frontend/react-best-practices/` - Vercel's 40+ React/Next.js performance rules
- `frontend/browser-testing/` - Browser automation with agent-browser

### Subagents

- `agents/verifier.md` - Validates completed work is functional
- `agents/security-reviewer.md` - Proactive security checks on sensitive code

---

## Tech Stack Compatibility

These skills are designed for:
- **Backend**: Python, FastAPI, any REST API
- **Frontend**: React, Next.js, TypeScript
- **General**: Any web application

---

## License

MIT
