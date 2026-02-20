# Create Commit

Create a well-formatted git commit for staged files with conventional commits format and emojis.

## Usage

```
/commit BE        → Commit to howthef-backend
/commit FE        → Commit to howthef-frontend  
/commit COMPANY   → Commit to howthef-company
/commit all       → Show status of all 3 repos
```

## Aliases

| Alias | Repo |
|-------|------|
| `BE`, `backend`, `back` | howthef-backend |
| `FE`, `frontend`, `front` | howthef-frontend |
| `COMPANY`, `company`, `co` | howthef-company |

## Workflow

1. **Parse target repo** from user input (BE/FE/COMPANY)
2. **Run git commands in that folder**: `git -C howthef-backend/ ...`
3. **Get staged files**: `git -C <repo> diff --cached --name-only`
4. **If nothing staged**: Show unstaged changes and ask if user wants to stage all
5. **Quick security scan**: Check staged files for obvious issues
6. **Generate commit message**: Based on changes
7. **Commit immediately**: `git -C <repo> commit -m "message"` — no approval prompt needed

## Commit Message Format

```
<emoji> <type>(<scope>): <subject>

<body>
```

### Emojis by Type

| Type | Emoji | When to Use |
|------|-------|-------------|
| feat | ✨ | New feature |
| fix | 🐛 | Bug fix |
| docs | 📝 | Documentation |
| style | 💄 | Formatting, UI styling |
| refactor | ♻️ | Code refactor (no feature/fix) |
| perf | ⚡ | Performance improvement |
| test | ✅ | Adding/updating tests |
| chore | 🔧 | Build, config, dependencies |
| security | 🔒 | Security fix |
| wip | 🚧 | Work in progress |
| remove | 🗑️ | Removing code/files |
| hotfix | 🚑 | Critical hotfix |

### Scope (based on changed files)

| Path Pattern | Scope |
|--------------|-------|
| `howthef-backend/` | `backend` |
| `howthef-frontend/` | `frontend` |
| `howthef-company/` | `company` |
| `*/agents/` | `agent` |
| `*/api/` | `api` |
| `*/components/` | `ui` |
| `.cursor/` | `cursor` |

### Subject Rules

- Use imperative mood: "add" not "added"
- Lowercase first letter
- No period at end
- Max 50 characters

## Examples

### Simple feature
```
✨ feat(frontend): add organization settings page
```

### Bug fix
```
🐛 fix(backend): correct user permission check in API

The previous implementation allowed users to access
other users' data by manipulating the user_id parameter.
```

### Security fix
```
🔒 security(api): add rate limiting to auth endpoints
```

### Multiple changes
```
♻️ refactor(agent): restructure personalized use case nodes

- Extract step logic to service
- Add proper error handling
- Update state management
```

### Agent/LangGraph specific
```
✨ feat(agent): add industry opportunity generator

New LangGraph agent for generating industry-specific
AI opportunities based on company context.
```

## Pre-Commit Quick Check

Before generating the commit message, do a quick scan of staged files for:

1. **Hardcoded secrets**: API keys, passwords, tokens
2. **Debug code**: `console.log`, `print()` statements
3. **TODO comments**: Unfinished work being committed
4. **Large files**: Binary files or data files

If issues found, warn before proceeding.

## Command Output

**For `/commit BE` with staged files — auto-commit, no approval prompt:**
```
📍 Repo: howthef-backend

📋 Staged Files (3):
- features/agents/new_agent/api/new_agent_api.py
- features/agents/new_agent/services/new_agent.py
- features/agents/new_agent/models/new_agent_state.py

🔍 Quick Check: ✅ No issues found

✨ feat(agent): add new industry analysis agent
abc1234 — 3 files changed, +250 / -10
```

**For `/commit FE` with nothing staged:**
```
📍 Repo: howthef-frontend

⚠️ No staged files.

📝 Unstaged changes (5 files):
- app/features/settings/components/SettingsForm.tsx
- app/features/settings/services_and_hooks/useSettingsQuery.ts
- ...

Stage all and commit? (y/n)
```

**For `/commit all` (status check):**
```
📊 Monorepo Status:

howthef-backend/
  ✅ 3 staged, 0 unstaged

howthef-frontend/
  ⚠️ 0 staged, 5 unstaged

howthef-company/
  ✅ Clean (nothing to commit)

---
Which repo to commit? (BE/FE/COMPANY)
```

## Usage Notes

- Works from anywhere in the monorepo (uses `git -C <path>`)
- If files already staged in GitKraken → analyzes those
- If nothing staged → offers to stage all changes
- Auto-commits without approval — never ask for confirmation, just commit directly

## Scope Detection (auto)

| Repo | Default Scope | Sub-scopes |
|------|---------------|------------|
| howthef-backend | `backend` | `agent`, `api`, `service` |
| howthef-frontend | `frontend` | `ui`, `hook`, `feature` |
| howthef-company | `company` | `gtm`, `ops`, `sales` |
