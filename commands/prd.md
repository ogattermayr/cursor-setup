# Create PRD

Generate a Product Requirements Document for a new feature using the HowTheF* PRD system.

## Usage

```
/prd <feature description>
/prd <feature description> --target BE
/prd <feature description> --target FE
```

If no `--target` is specified, infer from the feature description. Default to backend.

## Step 1: Research Phase (MANDATORY)

Before writing anything, deeply understand the current state:

1. **Read the backend skill** at `.cursor/skills/backend/backend-rules/SKILL.md` if target is BE
2. **Read the frontend skill** at `.cursor/skills/frontend/frontend-rules/SKILL.md` if target is FE
3. **Explore the existing codebase** related to this feature:
   - Search for related services, agents, models, tables, endpoints
   - Check what already exists that can be leveraged or extended
   - Identify integration points with existing systems
4. **Check existing PRDs** in `{target}/AI_CONTEXT/PRDS/` for patterns and related features
5. **Check the database** — use Supabase MCP to understand existing tables that relate to this feature
6. **Understand the full current state** before proposing anything new

## Step 2: Determine Feature Size

Based on research, classify the feature:

| Size | Criteria | Documents Created |
|------|----------|-------------------|
| **Small** | Single service/component, few files, few integration points | `{feature}_prd.md` only |
| **Medium** | Multiple services/components, needs architecture decisions, moderate integration | `{feature}_strategy.md` → then `{feature}_prd.md` |
| **Large** | Multi-agent system, many integration points, needs phased rollout, high complexity | `{feature}_strategy.md` → `{feature}_prd.md` → `phases/PHASE_N_*.md` |

**Ask the user to confirm the size classification before proceeding.** Present what you found during research and your recommendation.

## Step 3: Create the PRD

### File Location

```
# Backend — folder if medium/large, single file if small
howthef-backend/AI_CONTEXT/PRDS/{feature_name}/{feature}_strategy.md
howthef-backend/AI_CONTEXT/PRDS/{feature_name}/{feature}_prd.md
howthef-backend/AI_CONTEXT/PRDS/{feature_name}/phases/PHASE_1_{feature_name}_*.md

# Backend — small feature, single file
howthef-backend/AI_CONTEXT/PRDS/{feature_name}_prd.md

# Frontend — same pattern
howthef-frontend/AI_CONTEXT/PRDS/{feature_name}/{feature}_strategy.md
howthef-frontend/AI_CONTEXT/PRDS/{feature_name}/{feature}_prd.md
```

---

## The Golden Rule: Specifications, Not Implementations

**The PRD defines WHAT to build and the interface contracts. It does NOT write the implementation code.**

Think of it like an architect's blueprint vs the actual construction:
- **YES:** Class signatures, method signatures with type hints and docstrings, TypedDict state, Pydantic models with fields, SQL schemas, ASCII architecture diagrams, graph flow diagrams, endpoint tables, config constants, example Slack messages, pipeline stage diagrams
- **NO:** Full method bodies, complete file contents, line-by-line implementations, copy-paste-ready code for every file

The AI agent implementing the PRD will write the actual code. The PRD gives it unambiguous specifications to build against.

### What Level of Code IS Appropriate

**Show interfaces and contracts:**
```python
class GTMExperimentService:
    async def create_experiment(
        self,
        name: str,
        hypothesis: str,
        test_variable: str,
        control_description: str,
        variant_description: str,
        target_sample_size: int = 100,
        **targeting_criteria
    ) -> GTMExperiment

    async def check_for_winner(self, experiment_id: str) -> Optional[ExperimentResult]
    async def declare_winner(self, experiment_id: str, winner: str, learnings: str) -> GTMExperiment
```

**Don't write the full implementation:**
```python
# DON'T DO THIS IN THE PRD:
async def create_experiment(self, name, hypothesis, ...):
    clean = {k: v for k, v in data.items() if v is not None}
    result = await upsert_record(
        table_name=TABLE_EXPERIMENTS,
        data=clean,
        conflict_column="name",
    )
    if not result:
        raise ValueError("Failed to create experiment")
    logger.info(f"Created experiment: {result[0]['id']}")
    return GTMExperiment(**result[0])
```

### Reference: CRO Agent PRDs

The CRO Agent PRDs (`howthef-backend/AI_CONTEXT/PRDS/cro_agent/`) are the gold standard. Study them for tone, structure, and level of detail. Key patterns:

**Strategy doc** (~1,600 lines): Vision, pipeline stages with ASCII diagrams, current state analysis, complete SQL schemas, sub-agent definitions with purpose/capabilities/key methods (signatures only), Slack message mockups, DSPy integration concepts, self-improvement loop architecture, implementation phases with deliverables.

**PRD doc** (~1,150 lines): Architecture overview with ASCII art, directory structure, service specifications with class signatures + method signatures (type hints + brief docstrings), graph flow diagrams, endpoint tables with router registration patterns, configuration constants, scheduled tasks with brief pseudocode, approval workflow, implementation order.

**Phase docs** (large features only): Step-by-step build guides. This is the ONLY place where more detailed code is appropriate — and even then, focus on the tricky parts, not boilerplate.

---

## Document Templates

### Strategy Document (Medium & Large features)

The strategy is the **vision and architecture document**. It answers "what are we building and why?" with enough technical depth to make architecture decisions. Write it in a direct, opinionated voice — not corporate fluff.

**Required sections:**

```markdown
# {Feature Name} — Strategy PRD

**Version:** 1.0
**Date:** {today}
**Author:** Oliver + AI
**Status:** Strategy Draft — Refine before implementation detail

---

> **Bold, opinionated 2-3 sentence summary of what this system does and why it matters.
> Written in second person. Be direct about the problem.**

---

## 1. The Problem

What's broken today. Be specific and blunt:
- What's manual that shouldn't be
- What data/capabilities are missing
- What's the cost of inaction
- Use bullet points, not paragraphs

## 2. Current State Analysis

### What Exists Today

| Asset | Status | Details |
|-------|--------|---------|

### What's Missing

- ❌ List what doesn't exist yet
- ❌ Be specific about gaps

## 3. The Vision / Solution

What we're building. Include:
- Core philosophy / design principles (numbered, opinionated)
- North star metric (if applicable)
- Key capabilities (numbered list)

## 4. The System (High Level)

ASCII architecture diagram showing the full system:
- Use box-drawing characters (┌ ─ ┐ │ └ ┘ ├ ┤ ┬ ┴ ┼ ▶ ▼)
- Show data flow with arrows
- Label all components with file names where known
- Show integration points with existing systems
- Show the sub-agents/services and their responsibilities

## 5. The Complete Pipeline (if applicable)

Walk through the full pipeline stage by stage:
- Each stage: ASCII flow → description → key tables → what happens
- Show data transforming through each stage
- Call out manual vs automated steps
- This is where you explain HOW the system works end-to-end

## 6. Architecture & Code Location

### Directory Structure

Full proposed file tree following the project conventions:
- Backend: Agent structure with api/, models/, prompts/, services/, tasks/
- Frontend: Feature structure with components/, services_and_hooks/
- Include every file that will be created
- Brief comment per file explaining purpose

### Naming Convention

All files prefixed with `{feature_name}_` to match project patterns.

## 7. Database Schema

Full SQL CREATE TABLE statements for new tables:
- Include all columns with types and constraints
- Include CHECK constraints for enums
- Include indexes
- Include RLS policies
- Include ALTER TABLE for existing table modifications
- This is the ONE place where complete code is required — schemas must be exact

## 8. Sub-Agent / Service Definitions (if multi-agent)

For each sub-agent or major service:
- **Purpose**: One sentence
- **Capabilities**: Bullet list of what it can do
- **Key methods**: Signatures only (name, params, return type)
- **Key data models**: Class names with field list (not full Pydantic definitions)

## 9. API Endpoints

Table of all endpoints:
- Method, path, description
- Group by domain
- Note which require auth/approval

## 10. Integration Points

How this connects to existing systems:
- Which existing services are used (with file paths)
- Which existing tables are read/written
- Which external APIs are called (with rate limits/costs if relevant)
- How this feeds into or consumes from other features

## 11. Interface Mockups (if applicable)

For Slack bots, dashboards, or user-facing features:
- Show example messages/screens
- Show interaction flows (commands, buttons, approvals)
- Show notification templates

## 12. Implementation Phases

High-level phase breakdown:
- Phase 1: MVP / Foundation — Goal, key tasks, deliverable
- Phase 2: Core feature — Goal, key tasks, deliverable
- Phase 3+: Enhancements, automation, optimization
- Each phase: checklist of tasks (- [ ])
- NEVER include time estimates or dates — just complexity and dependencies

## 13. Open Questions

Numbered list of decisions that need to be made:
- Technical decisions
- Architecture choices
- Business/product decisions
- Call out what's validated vs unvalidated
```

---

### PRD Document (All sizes)

The PRD is the **implementation specification**. It defines the interfaces, contracts, and architecture precisely enough for a developer (or AI agent) to build without ambiguity. It does NOT write implementation code — it defines WHAT each component does and its exact interface.

**The CRO Agent PRD is the reference.** Study `howthef-backend/AI_CONTEXT/PRDS/cro_agent/CRO_AGENT_prd.md` for the right level of detail.

**Required sections:**

```markdown
# {Feature Name} — PRD

**Version:** 1.0
**Date:** {today}
**Status:** Implementation Ready

---

## 1. Overview

Brief summary (3-5 sentences). What this builds, what problem it solves, how it fits into the system. Reference the strategy doc if one exists.

## 2. Architecture Overview

ASCII architecture diagram showing the full system. Include:
- All major components with their file names
- Data flow between components
- External service integrations
- Data layer (tables)

## 3. Core Principles & Architecture Decisions

Numbered design principles that guide implementation. Include WHY for each. Example:
1. **Config-driven** — Source list defined in Python, not DB. Mirrors existing pattern.
2. **One transcript = one LLM call** — No chunking. Modern context windows handle it.

## 4. Directory Structure

Exact file tree with every file to create:
- Brief comment per file explaining purpose
- Follow agent structure for backend (api/, models/, prompts/, services/)
- Follow feature structure for frontend (components/, services_and_hooks/)

## 5. Service & Agent Specifications

This is the CORE of the PRD. For each service/agent, provide:

### 5.X {Service Name}

**File:** `path/to/file.py`
**Class:** `ClassName`
**Type:** Main Agent / Sub-Agent Service / Integration Service / Storage Service
**Purpose:** One sentence.

**Responsibilities:** (bullet list)

**Methods:** (signatures with types and brief docstrings — NOT implementations)
```python
class MyService:
    async def do_thing(self, param: str, limit: int = 10) -> list[dict]
    async def process_item(self, item_id: str) -> Optional[Result]
```

**Key data models:** (Pydantic models with fields — just field names and types)
```python
class MyModel(BaseModel):
    name: str
    status: str  # 'active', 'paused', 'completed'
    metrics: dict
    created_at: datetime
```

**If LangGraph agent — Graph flow:**
```
[node_a] → check_errors → [node_b] → check_errors → [node_c] → END
```

**If LangGraph agent — State:**
```python
class MyState(TypedDict, total=False):
    job_id: str
    status: str
    # ... fields with brief comments
    errors: List[str]
```

## 6. API Endpoints

Router registration pattern with all endpoints:
```python
router = APIRouter(prefix="/v1/my-feature", tags=["My Feature"])

router.get("/status")(get_status_endpoint)
router.post("/run")(trigger_run_endpoint)
router.get("/results/{id}")(get_results_endpoint)
```

## 7. Database Schema

Full SQL (if not already in strategy doc). Or reference: "See strategy document Section 7."

## 8. Scheduled Tasks / Cron Jobs (if applicable)

For each task:
- File, schedule, function name
- Brief pseudocode (3-5 lines max) showing the flow, not the implementation

## 9. Configuration & Constants

All config values in one place:
```python
FEATURE_SOME_THRESHOLD = 100
FEATURE_SOME_INTERVAL_HOURS = 6
FEATURE_SOME_CHANNEL = "#my-channel"
```

## 10. Error Handling Strategy

How errors are handled at each layer. Brief — just the strategy, not the code.

## 11. Implementation Order

Numbered steps in the exact order to build:
1. Database migration
2. Models (Pydantic + State)
3. Storage service
4. Core service(s)
5. API endpoints + router registration
6. Integration (webhooks, Slack, cron)
7. Testing

## 12. Dependencies

- External packages needed (with install commands)
- Existing services used (with file paths)
- API keys / env vars required

## 13. Future Enhancements

What's explicitly NOT in scope but planned.
```

---

### Phase Documents (Large features only)

Phase docs are **build guides** for implementing one slice of the system. They're the ONE place where you can include more implementation detail — but focus on the non-obvious parts, not boilerplate.

**When to create phase docs:**
- The feature has 6+ services/files with complex interdependencies
- Different phases depend on each other (can't build Phase 3 without Phase 1 done)
- The feature spans multiple systems (webhooks + Slack + cron + DB + external APIs)

**Required sections:**

```markdown
# {Feature} — Phase {N}: {Phase Name}

**Objective:** One sentence.
**Prerequisites:** What phases/work must be done first.

---

## 1. Files to Create

Directory tree of new files in this phase only.

## 2. What to Build

For each file in this phase:
- File path
- Purpose
- Key class/method signatures (same level as PRD)
- Any TRICKY implementation details that aren't obvious from the signature
  (e.g., "Use upsert with conflict on (podcast_id, title)", "Call yt-dlp with --write-auto-subs --skip-download")
- Don't write boilerplate — the implementing agent knows CRUD patterns, LangGraph patterns, etc.

## 3. Database Migration (if applicable)

SQL to run for this phase.

## 4. Integration Points

Which existing files need modification (imports, router registration, etc.)

## 5. Phase Completion Checklist

- [ ] File created: path/to/file.py
- [ ] Migration run
- [ ] Endpoints tested
- [ ] Integration verified

**Next:** `PHASE_{N+1}_{feature_name}_{name}.md` - Brief description
```

---

## Writing Style Rules

1. **Be direct and opinionated.** Not "we could potentially consider" — say "we will" or "this does X"
2. **Use tables** for any comparison, list of assets, or structured data
3. **Use ASCII diagrams** for architecture — box-drawing characters, arrows, clear labels
4. **Use checkboxes** (- [ ]) for checklists and completion tracking
5. **Use emoji sparingly** — ✅ ❌ for status indicators only
6. **Write in second person** for problem statements ("You're terrible at X" / "Your pipeline is broken")
7. **Include "What Exists" before "What's New"** — always ground in current state
8. **SQL schemas are complete** — include constraints, indexes, RLS, comments. This is the ONE exception to "don't write full code."
9. **Directory structures are exact** — every file, correct naming conventions
10. **No fluff sections** — every section earns its place with actionable content
11. **Show the pipeline end-to-end** — walk through the full data flow, not just individual components
12. **Mockup real interactions** — for Slack bots, show actual message examples; for dashboards, show the UI flow

## Code Level Guidelines

| What | Level of Detail | Example |
|------|----------------|---------|
| SQL schemas | **Complete** — every column, constraint, index, RLS | Full CREATE TABLE statements |
| Pydantic models | **Fields + types** — no validators unless critical | `name: str`, `status: str  # 'active', 'paused'` |
| TypedDict state | **Fields + comments** | `job_id: str`, `errors: List[str]` |
| Service classes | **Method signatures + types + brief docstring** | `async def create(self, data: dict) -> Optional[dict]` |
| LangGraph agents | **Graph flow diagram + node names + edge routing** | `[init] → [process] → check_errors → [store] → END` |
| Config constants | **Complete** — all values with comments | `FEATURE_THRESHOLD = 100  # Per variant` |
| API endpoints | **Router registration pattern** | `router.post("/run")(trigger_run_endpoint)` |
| Prompts | **Structure + key instructions** — not the full prompt text | "System prompt covers: persona, rules, output format" |
| Cron tasks | **Schedule + 3-5 line pseudocode** | "Pull metrics → update DB → check experiments → notify" |
| Slack messages | **Full mockup** — these define the UX contract | Actual formatted message blocks |
| Webhook handlers | **Event table + what each updates** — not handler code | Table: event → handler → tables updated |

## Backend-Specific Rules

- Follow agent structure: `api/`, `models/`, `prompts/`, `services/`, `tasks/`
- All files prefixed with `{feature_name}_`
- Services use `crud_service.py` for DB operations
- Agents inherit `BaseHowthefAgent`
- State uses `TypedDict`
- Routers use function registration, NOT decorators
- Include Supabase table schemas with RLS

## Frontend-Specific Rules

- Follow feature structure: `components/`, `services_and_hooks/`
- Components: `PascalCaseComponent.tsx`
- Services: `camelCaseService.ts`
- Hooks: `useSomethingHook.ts`
- All files start with file path comment
- Use TanStack Query for server state
- Use existing UI components (shadcn/ui, HTF design system)
- Include component tree diagram
- Include data flow diagrams

## Workflow Summary

```
1. User runs /prd with feature description
2. Agent researches codebase (MANDATORY — read code, check DB, find related features)
3. Agent classifies feature size (small/medium/large)
4. Agent presents findings + size recommendation → user confirms
5. Agent writes strategy doc first (if medium/large)
6. Agent presents strategy → user reviews and refines
7. Agent writes PRD (specifications, NOT implementations)
8. Agent writes phase docs (if large) — one at a time, user confirms before next
```

## Important

- NEVER skip the research phase. The PRD quality depends on understanding what exists.
- NEVER write generic/placeholder content. Every section must be specific to THIS feature in THIS codebase.
- NEVER write full implementation code in the PRD. Signatures, types, and docstrings — not method bodies.
- ALWAYS use Supabase MCP to check existing tables before proposing schema changes.
- ALWAYS reference existing services, models, and patterns from the actual codebase.
- ALWAYS study the CRO Agent PRDs as the reference for quality and detail level.
- The user will iterate on the documents with you. Write a solid first draft, expect revisions.
