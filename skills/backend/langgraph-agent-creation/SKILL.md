---
name: langgraph-agent-creation
description: Creates new LangGraph agents with the HowTheF* folder structure and patterns. Use when creating a new agent, adding agent functionality, or scaffolding agent files.
---

# LangGraph Agent Creation

Use this skill when creating new LangGraph agents in the HowTheF* backend.

## Agent Location Decision

- **Customer-facing agents** → `features/agents/{agent_name}/`
- **Internal operations agents** → `internal_agents/{agent_name}/`

## Required Folder Structure

Every agent MUST have this exact structure:

```
{agent_name}/
├── __init__.py
├── api/
│   ├── __init__.py
│   ├── {agent_name}_api.py          # Endpoints
│   └── {agent_name}_router.py       # Router registration
├── models/
│   ├── __init__.py
│   ├── {agent_name}_models.py       # Pydantic I/O models
│   └── {agent_name}_state.py        # LangGraph TypedDict state
├── prompts/
│   ├── __init__.py
│   └── {agent_name}_prompts.py      # Prompt templates
└── services/
    ├── __init__.py
    ├── {agent_name}_agent.py        # Main agent class
    ├── {agent_name}_nodes.py        # Graph node functions
    ├── {agent_name}_steps_service.py    # Core step logic
    └── {agent_name}_storage_service.py  # DB operations
```

## File Templates

### 1. State File (`models/{agent_name}_state.py`)

```python
# models/{agent_name}_state.py
from typing import TypedDict, Optional, List, Any
from datetime import datetime

class {AgentName}State(TypedDict, total=False):
    """LangGraph state for {AgentName} agent."""
    # Required fields
    job_id: str
    user_id: str
    org_id: str
    status: str
    
    # Context
    context: dict
    
    # Step outputs (add as needed)
    step_1_output: Optional[dict]
    step_2_output: Optional[dict]
    
    # Metadata
    errors: List[str]
    created_at: Optional[datetime]
    completed_at: Optional[datetime]
    generation_model_configs: dict
```

### 2. Main Agent (`services/{agent_name}_agent.py`)

```python
# services/{agent_name}_agent.py
from typing import Optional, Dict, Any
from langgraph.graph import StateGraph, END

from features.agents_setup.base.agent_setup_base_agent import BaseHowthefAgent
from features.agents_setup.graph.agent_setup_nodes import load_context_node, handle_error_node
from features.agents_setup.graph.agent_setup_edges import check_for_errors

from ..models.{agent_name}_state import {AgentName}State
from .{agent_name}_nodes import (
    init_job_node,
    step_1_node,
    step_2_node,
    store_data_node,
)


class {AgentName}Agent(BaseHowthefAgent):
    """
    {AgentName} agent using LangGraph.
    
    Graph flow:
    init_job -> load_context -> step_1 -> step_2 -> store_data -> END
    """
    
    def __init__(self):
        super().__init__()
        self.graph = self._build_graph()
    
    def _build_graph(self) -> StateGraph:
        """Build the LangGraph state machine."""
        graph = StateGraph({AgentName}State)
        
        # Add nodes
        graph.add_node("init_job", init_job_node)
        graph.add_node("load_context", load_context_node)
        graph.add_node("step_1", step_1_node)
        graph.add_node("step_2", step_2_node)
        graph.add_node("store_data", store_data_node)
        graph.add_node("handle_error", handle_error_node)
        
        # Set entry point
        graph.set_entry_point("init_job")
        
        # Add edges
        graph.add_conditional_edges("init_job", check_for_errors, {
            "continue": "load_context",
            "handle_error": "handle_error"
        })
        graph.add_conditional_edges("load_context", check_for_errors, {
            "continue": "step_1",
            "handle_error": "handle_error"
        })
        graph.add_conditional_edges("step_1", check_for_errors, {
            "continue": "step_2",
            "handle_error": "handle_error"
        })
        graph.add_conditional_edges("step_2", check_for_errors, {
            "continue": "store_data",
            "handle_error": "handle_error"
        })
        graph.add_edge("store_data", END)
        graph.add_edge("handle_error", END)
        
        return graph.compile()
    
    async def run(
        self,
        user_id: str,
        org_id: str,
        job_id_override: Optional[str] = None,
        initial_state_override: Optional[Dict[str, Any]] = None,
    ) -> Dict[str, Any]:
        """Execute the agent."""
        import uuid
        
        job_id = job_id_override or str(uuid.uuid4())
        
        initial_state: {AgentName}State = initial_state_override or {
            "job_id": job_id,
            "user_id": user_id,
            "org_id": org_id,
            "status": "starting",
            "errors": [],
            "generation_model_configs": {},
        }
        
        config = {"configurable": {"thread_id": job_id}}
        final_state = await self.graph.ainvoke(initial_state, config)
        
        return {
            "job_id": job_id,
            "status": final_state.get("status", "unknown"),
            "errors": final_state.get("errors", []),
        }


# Singleton
{agent_name}_agent = {AgentName}Agent()
```

### 3. Nodes (`services/{agent_name}_nodes.py`)

```python
# services/{agent_name}_nodes.py
from typing import Dict, Any
from shared.configurations.logging_config import get_logger
from ..models.{agent_name}_state import {AgentName}State
from .{agent_name}_steps_service import {agent_name}_steps_service
from .{agent_name}_storage_service import {agent_name}_storage_service

logger = get_logger()


async def init_job_node(state: {AgentName}State) -> Dict[str, Any]:
    """Initialize the job and create DB record."""
    logger.info(f"[{AgentName}] Initializing job: {state['job_id']}")
    
    await {agent_name}_storage_service.create_initial_record(
        job_id=state["job_id"],
        user_id=state["user_id"],
        org_id=state["org_id"],
    )
    
    return {"status": "context_loading"}


async def step_1_node(state: {AgentName}State) -> Dict[str, Any]:
    """Execute step 1."""
    logger.info(f"[{AgentName}] Executing step 1")
    
    try:
        result, model_config = await {agent_name}_steps_service.execute_step_1(
            context=state.get("context", {}),
        )
        
        return {
            "step_1_output": result,
            "generation_model_configs": {
                **state.get("generation_model_configs", {}),
                "step_1": model_config,
            },
            "status": "step_2_processing",
        }
    except Exception as e:
        logger.error(f"[{AgentName}] Step 1 failed: {e}")
        return {"errors": state.get("errors", []) + [str(e)]}


async def step_2_node(state: {AgentName}State) -> Dict[str, Any]:
    """Execute step 2."""
    logger.info(f"[{AgentName}] Executing step 2")
    
    try:
        result, model_config = await {agent_name}_steps_service.execute_step_2(
            step_1_output=state.get("step_1_output"),
        )
        
        return {
            "step_2_output": result,
            "generation_model_configs": {
                **state.get("generation_model_configs", {}),
                "step_2": model_config,
            },
            "status": "storing_data",
        }
    except Exception as e:
        logger.error(f"[{AgentName}] Step 2 failed: {e}")
        return {"errors": state.get("errors", []) + [str(e)]}


async def store_data_node(state: {AgentName}State) -> Dict[str, Any]:
    """Store final results to database."""
    logger.info(f"[{AgentName}] Storing results")
    
    await {agent_name}_storage_service.mark_complete(
        job_id=state["job_id"],
        final_output={
            "step_1": state.get("step_1_output"),
            "step_2": state.get("step_2_output"),
        },
        model_configs=state.get("generation_model_configs", {}),
    )
    
    return {"status": "completed"}
```

### 4. Router (`api/{agent_name}_router.py`)

```python
# api/{agent_name}_router.py
from fastapi import APIRouter
from .{agent_name}_api import (
    generate_{agent_name}_endpoint,
    get_{agent_name}_status_endpoint,
)

router = APIRouter(prefix="/v1/{agent-name}", tags=["{AgentName}"])

# Register routes - NO DECORATORS
router.post("/generate/org/{org_id}/user/{user_id}")(generate_{agent_name}_endpoint)
router.get("/status/{job_id}")(get_{agent_name}_status_endpoint)
```

### 5. API (`api/{agent_name}_api.py`)

```python
# api/{agent_name}_api.py
from fastapi import BackgroundTasks, HTTPException
from typing import Optional
import uuid

from ..services.{agent_name}_agent import {agent_name}_agent
from ..services.{agent_name}_storage_service import {agent_name}_storage_service
from ..models.{agent_name}_models import {AgentName}Request, {AgentName}Response


async def generate_{agent_name}_endpoint(
    org_id: str,
    user_id: str,
    request: {AgentName}Request,
    background_tasks: BackgroundTasks,
) -> {AgentName}Response:
    """Trigger {agent_name} generation."""
    job_id = str(uuid.uuid4())
    
    background_tasks.add_task(
        {agent_name}_agent.run,
        user_id=user_id,
        org_id=org_id,
        job_id_override=job_id,
    )
    
    return {AgentName}Response(
        job_id=job_id,
        status="accepted",
        message="Generation started",
    )


async def get_{agent_name}_status_endpoint(job_id: str):
    """Get generation status."""
    record = await {agent_name}_storage_service.get_job_details(job_id)
    
    if not record:
        raise HTTPException(status_code=404, detail="Job not found")
    
    return record
```

## Checklist

When creating a new agent, verify:

- [ ] Folder structure matches template exactly
- [ ] State TypedDict defined with all required fields
- [ ] Agent inherits `BaseHowthefAgent`
- [ ] `_build_graph()` method defines graph structure
- [ ] `run()` method handles execution
- [ ] All nodes return state patches (dicts)
- [ ] Error handling uses `check_for_errors` edge
- [ ] Storage service uses `crud_service` (never raw Supabase)
- [ ] Router uses function registration (NO decorators)
- [ ] Router added to `main.py` includes
- [ ] Endpoints use `_endpoint` suffix

## Base Components Reference

Located in `features/agents_setup/`:

- `base/agent_setup_base_agent.py` - BaseHowthefAgent class
- `state/agent_setup_state.py` - AgentState TypedDict
- `graph/agent_setup_nodes.py` - Reusable nodes (load_context, handle_error)
- `graph/agent_setup_edges.py` - Reusable edges (check_for_errors)
- `graph/base_graph.py` - Graph creation utilities
