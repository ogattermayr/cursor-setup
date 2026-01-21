---
name: backend-rules
description: Backend development rules for HowTheF* FastAPI + LangGraph API. Use when working in howthef-backend/ or discussing backend architecture, agents, CRUD operations, or Python code.
---

# Backend Rules

## Running
- `make dev` - Port **8040**
- NEVER `python main.py` - use UV/Makefile

## Architecture
```
howthef-backend/
├── shared/services/crud_service.py  # ALL DB ops
├── features/                        # Building blocks (context, llm, prompts, tools)
├── features/agents/                 # Customer-facing agents
├── internal_agents/                 # Internal operations (SDR, company_ops)
└── features/agents_setup/           # Base LangGraph components
```

## Agent Structure
```
{agent}/
├── api/
│   ├── {agent}_api.py      # Endpoints with _endpoint suffix
│   └── {agent}_router.py   # Router registration (NO decorators)
├── models/
│   ├── {agent}_models.py   # Pydantic models
│   └── {agent}_state.py    # LangGraph TypedDict state
├── prompts/                # Agent-specific prompts
└── services/
    ├── {agent}.py          # Main agent (inherits BaseHowthefAgent)
    ├── {agent}_nodes.py    # LangGraph node functions
    ├── {agent}_service.py  # Core business logic
    └── {agent}_storage_service.py  # DB ops via crud_service
```

## CRUD (use `shared/services/crud_service.py`)
```python
await read_record(table_name="x", filter_string=f"id=eq.{id}")
await create_record(table_name="x", data={...})
await update_record(table_name="x", data={...}, filter_string=f"id=eq.{id}")
await upsert_record(table_name="x", data={...}, conflict_column="id")
```

## LLM Service
```python
from features.llm.services.llm_service import LLMService
llm_service = LLMService()
response = await llm_service.generate_structured_output(
    prompt="...", response_model=PydanticModel, provider="anthropic"
)
```

## Code Patterns
- **API endpoints**: Minimal, `_endpoint` suffix, call services only
- **Services**: Business logic, use crud_service
- **Routers**: `router.post("/path")(endpoint_function)` - NO decorators
- **Agents**: Inherit `BaseHowthefAgent`, `_build_graph()`, `run()`
- **State**: Use TypedDict for LangGraph state schemas

## Critical Rules
- NEVER raw Supabase queries - use crud_service
- NEVER fallback functions with placeholder data
- NEVER decorator-based routes - register in router files
- Mark changes: `### CODE CHANGES START/END`
