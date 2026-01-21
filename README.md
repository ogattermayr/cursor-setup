# Cursor Setup

Reusable Cursor AI skills, subagents, and commands for web development projects.

## Installation

### Option 1: Remote Rule (Recommended)

In any Cursor project:
1. Open **Cursor Settings** (Cmd+Shift+J)
2. Navigate to **Rules**
3. Click **Add Rule** → **Remote Rule (GitHub)**
4. Enter: `https://github.com/ogattermayr/cursor-setup`

Skills will auto-sync when the repo updates.

### Option 2: Copy to Global

Copy to your user-level Cursor folder for all projects:

```bash
cp -r skills/* ~/.cursor/skills/
cp -r agents/* ~/.cursor/agents/
```

### Option 3: Copy to Project

Copy to a specific project's `.cursor/` folder.

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
