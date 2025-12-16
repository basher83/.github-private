# GitHub Copilot custom agents: GitHub.com-native, VS Code-compatible

This repo uses custom agent profiles in `.github/agents/*.agent.md`.

Goal: author agent profiles that work best on **GitHub.com Copilot coding agent** today, while staying **forward-compatible with VS Code** (and other IDEs) as more properties become supported over time.

## Cheat sheet

- Default (GitHub.com-first, can edit files):

  ```yaml
  ---
  description: "<what this agent does>"
  target: github-copilot
  tools: ["read", "search", "edit"]
  ---
  ```

- Read-only agent (reviews / planning, no edits): set `tools: ["read", "search"]`.
- Needs to run commands/tests: add `execute` (example: `tools: ["read", "search", "edit", "execute"]`).
- Want maximum power (not least-privilege): omit `tools` entirely or set `tools: ["*"]`.
- Want zero tool usage (persona-only): set `tools: []`.

## Key compatibility facts

- GitHub.com and IDEs read the same `.agent.md` files, but **not all frontmatter keys behave the same** everywhere.
- Per GitHub Docs, **unrecognized tool names are ignored**. This makes it safe to include “future” tools as long as the important ones are present.
- Per GitHub Docs, the following VS Code/IDE agent properties are **ignored on GitHub.com** today:
  - `model`
  - `argument-hint`
  - `handoffs`

## Recommended frontmatter pattern

For GitHub.com-first agents, use:

- `target: github-copilot` so the intent is explicit.
- A least-privilege `tools:` allowlist using **tool aliases**.
- `infer: false` for specialized agents that should only be used when manually selected.

Example template:

```yaml
---
description: "Short, action-oriented description."
target: github-copilot
infer: false
tools: ["read", "search", "edit"]
---
```

Notes:

- If you omit `tools` entirely, GitHub.com defaults to **all tools**.
- `tools: ["*"]` also enables all available tools.
- `tools: []` disables all tools (usually not what you want for workflow/code generation).

## Choosing tools (least privilege)

Pick the smallest set that enables the agent’s mission.

Common minimal sets:

- **Read-only analysis/review**: `tools: ["read", "search"]`
- **Repo edits (files only)**: `tools: ["read", "search", "edit"]`
- **Repo edits + running commands/tests**: `tools: ["read", "search", "edit", "execute"]`

Guideline: only include `execute` if the agent must run commands (it increases blast radius).

## Tool aliases vs explicit tools

Prefer tool aliases (like `read`, `search`, `edit`) because they are stable abstractions across environments.

When you need a specific tool from a specific MCP server, GitHub Docs allow selecting tools by prefix:

- All tools from a server: `tools: ["some-mcp-server/*"]`
- One tool from a server: `tools: ["some-mcp-server/some-tool"]`

Repository-level agent profiles **cannot configure MCP servers** in the agent file itself. They can only use MCP servers already configured in that repo’s Copilot settings.

## VS Code-compatibility strategy

Even though GitHub.com ignores `model`/`handoffs` today, you can include them **without breaking GitHub.com**, and they may become useful later.

Two recommended approaches:

### Approach A: GitHub.com-first (recommended today)

Keep the frontmatter minimal and GitHub.com-native:

```yaml
---
description: "Generate a Fast PR Gate workflow using mise tasks."
target: github-copilot
tools: ["read", "search", "edit"]
---
```

### Approach B: Dual-target future-proofing

Add IDE-only niceties, knowing GitHub.com will ignore them:

```yaml
---
name: "Fast PR Gate Generator"
description: "Generate a Fast PR Gate workflow using mise tasks."
target: github-copilot
infer: false
tools: ["read", "search", "edit"]

# IDE-focused keys (currently ignored on GitHub.com)
model: "gpt-5.2-preview"
argument-hint: "Tell me the default branch and desired gate tasks"
handoffs:
  - label: "Review workflow"
    agent: "reviewer"
    prompt: "Review the generated workflow for correctness and minimal permissions."
    send: false
---
```

If you use this approach, keep IDE-only keys clearly grouped and ensure the GitHub.com-critical keys are present and correct.

## Guardrails to include in the body

Because `tools` can grant write/execute power, include explicit constraints in the Markdown body:

- Exactly which files may be created/edited (example: only `.github/workflows/fast-pr-gate.yml`).
- No extra jobs/matrices/services.
- Minimal permissions.
- “Do not guess tooling; verify from config files first.”

## Documentation hygiene

- Keep agent docs concise and task-oriented.
- Avoid including secrets or real interaction IDs.
- Prefer referencing `uv` commands when giving “how to run” instructions.

## Suggested references

- GitHub Docs: Custom agents configuration
  - <https://docs.github.com/en/copilot/reference/custom-agents-configuration>
- GitHub Docs: Creating custom agents
  - <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents>
