---
name: google-agents-cli-adk-code
description: >
  This skill should be used when the user wants to "write agent code",
  "add a tool", "create a callback", "define an agent", "use state management",
  or needs ADK Python API patterns and code examples.
  It provides a quick reference for agent types, tool definitions, orchestration
  patterns, callbacks, and state management.
  Do NOT use for creating new projects (use google-agents-cli-scaffold) or deployment
  (use google-agents-cli-deploy).
metadata:
  author: Google
  license: Apache-2.0
  version: 0.0.3
  requires:
    bins:
      - agents-cli
    install: "uv tool install google-agents-cli"
---

# ADK Cheatsheet

> **Python only for now.** This cheatsheet currently covers the Python ADK SDK.
> Support for other languages is coming soon.

## Quick Reference — Most Common Patterns

### Agent Creation

```python
from google.adk.agents import Agent

root_agent = Agent(
    name="my_agent",
    model="gemini-3-flash-preview",
    instruction="You are a helpful assistant that ...",
    tools=[my_tool],
)
```

> **NEVER change an existing agent's `model=` value unless the user explicitly asks.** If a Gemini model returns a 404, it's almost always a `GOOGLE_CLOUD_LOCATION` issue — run the listing command to verify availability before changing anything. For model docs, fetch `https://google.github.io/adk-docs/agents/models/google-gemini/index.md`.
> ```bash
> uv run --with google-genai python -c "
> from google import genai
> client = genai.Client(vertexai=True, location='global')
> for m in client.models.list(): print(m.name)
> "
> ```

### Basic Tool

```python
from google.adk.tools import FunctionTool

def get_weather(city: str) -> dict:
    """Get current weather for a city."""
    return {"city": city, "temp": "22°C", "condition": "sunny"}

weather_tool = FunctionTool(func=get_weather)
```

> **ADK built-in tool imports:** Import the tool instance, not the module.
> ```python
> from google.adk.tools.load_web_page import load_web_page  # CORRECT
> from google.adk.tools import load_web_page                 # WRONG
> ```

### Simple Callback

```python
from google.adk.agents.callback_context import CallbackContext

async def initialize_state(callback_context: CallbackContext) -> None:
    state = callback_context.state
    if "history" not in state:
        state["history"] = []

root_agent = Agent(
    name="my_agent",
    model="gemini-3-flash-preview",
    instruction="...",
    before_agent_callback=initialize_state,
)
```

---

## Reference Files

| File | Contents |
|------|----------|
| `references/adk-python.md` | Python ADK API quick reference — agents, tools, auth, orchestration, callbacks, plugins, state, artifacts, context caching/compaction, session rewind |
| `references/reference-implementations.md` | Production patterns from ADK samples — code executors, planners, grounding metadata, advanced callbacks |
Read `references/adk-python.md` for the full API quick reference.

For the ADK docs index (titles and URLs for fetching documentation pages), use `curl https://google.github.io/adk-docs/llms.txt`.

> **Creating a new agent project?** Use `/google-agents-cli-scaffold` instead — this skill is for writing code in existing projects.

---

## Related Skills

- `/google-agents-cli-workflow` — Development workflow, coding guidelines, and operational rules
- `/google-agents-cli-scaffold` — Project creation and enhancement with `agents-cli scaffold create` / `scaffold enhance`
- `/google-agents-cli-eval` — Evaluation methodology, evalset schema, and the eval-fix loop
- `/google-agents-cli-deploy` — Deployment targets, CI/CD pipelines, and production workflows
