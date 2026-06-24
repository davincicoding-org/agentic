# agentic

A Claude Code / Cowork plugin marketplace.

## Plugins

### requirements-analyst

Turns Claude into a requirements analyst for a Linear project. A non-technical
product owner describes what the product should do in plain language; the skill
interviews them, drafts **capabilities** (top-level issues) and their
**requirements** (sub-issues), and manages the full loop — discover, inquire,
create, refine, prioritize, approve — without anyone touching Linear directly.

The skill discovers the project, its modules (from the project description),
workflow states, and labels at runtime, so it works for any Linear project.

**Requires:** a connected Linear MCP. This plugin bundles the hosted Linear MCP
(`https://mcp.linear.app/sse`); on first use you'll be prompted to authorize your
Linear workspace. If you already have Linear connected, you can ignore or remove
the bundled `.mcp.json`.

## Install

```
/plugin marketplace add davincicoding-org/agentic
/plugin install requirements-analyst@agentic
```

Then authorize Linear when prompted, and start with a phrase like
"what's going on with the project?" or "I want to add a feature".

## Update

```
/plugin marketplace update agentic
```

New versions flow in after the maintainer pushes to this repo.
