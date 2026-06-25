# agentic

A Claude Code / Cowork plugin marketplace.

## Plugins

### davinci

A davinci coding toolkit bundling two skills.

**spec** — turns Claude into a requirements analyst for a Linear project. A
non-technical product owner describes what the product should do in plain
language; the skill interviews them, drafts **capabilities** (top-level issues)
with their **requirements** written into the issue description, and manages the
full loop — discover, inquire, create, refine, review, prioritize, approve —
without anyone touching Linear directly. It discovers the project, its modules
(from the project description), the Creator/Expert roles, workflow states, and
labels at runtime, so it works for any Linear project.

**layer-cake** — helps decide which architectural layer or namespace a piece of
code belongs in for projects organized as the Layer Cake: a one-directional
dependency spine (config → storage → models → procedures → store → app → ui) plus
an orthogonal axis of cross-cutting code (infra + tools).

**Requires (spec):** a connected Linear MCP. This plugin bundles
the hosted Linear MCP (`https://mcp.linear.app/sse`); on first use you'll be
prompted to authorize your Linear workspace. If you already have Linear connected,
you can ignore or remove the bundled `.mcp.json`.

## Install

```
/plugin marketplace add davincicoding-org/agentic
/plugin install davinci@agentic
```

Then authorize Linear when prompted, and start with a phrase like
"what's going on with the project?" or "I want to add a feature".

## Update

```
/plugin marketplace update agentic
```

New versions flow in after the maintainer pushes to this repo.
