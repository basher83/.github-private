# Custom agents configuration

Reference notes for configuring custom agents. For the canonical documentation, see: <https://docs.github.com/en/copilot/reference/custom-agents-configuration>.

## YAML frontmatter properties

The YAML frontmatter controls how an agent is identified and what it can do. When multiple agent profiles share the same filename (ignoring the `.md` / `.agent.md` suffix), Copilot deduplicates by name and prefers the most specific scope (for example, repository-level over organization-level).

### Agent profile structure

An agent profile is a Markdown file with YAML frontmatter followed by the prompt text:

```yaml
---
name: example-agent
description: A short description of what this agent does
tools: ["read", "search"]
---

Write the agent instructions here.
```

| Property | Type | Purpose | Repository | Organization / enterprise | IDEs (VS Code, JetBrains, Eclipse, Xcode) |
| --- | --- | --- | --- | --- | --- |
| `name` | string | Display name for the custom agent (optional). | Yes | Yes | Yes |
| `description` | string (required) | Describes what the agent is for and how it should behave. | Yes | Yes | Yes |
| `target` | string | Target environment/context (for example `vscode` or `github-copilot`). If omitted, applies to both. | Yes | Yes | Yes |
| `tools` | string or string[] | Tool allow-list for the agent. Supports a comma-separated string or a YAML array. If omitted, all tools are allowed. See [Tools](#tools). | Yes | Yes | Yes |
| `infer` | boolean | Whether Copilot coding agent may auto-select this agent based on context. If `false`, it must be chosen explicitly. Defaults to `true` if omitted. | Yes | Yes | Yes |
| `mcp-servers` | object | Additional MCP servers and tool settings for the agent. | No | Yes | Varies |
| `metadata` | object (string→string map) | Arbitrary key/value annotations for the agent. | Yes | Yes | Yes |

The prompt content (Markdown) goes below the frontmatter. Keep prompts within the platform limits (for example, 30,000 characters).

> **Note**
> Some IDE-specific frontmatter fields (for example `model`, `argument-hint`, `handoffs`) may be ignored by Copilot coding agent on GitHub.com for compatibility.

## Tools

The `tools` frontmatter property controls which tools are available to the agent, including tools surfaced via MCP servers.

Repository-level custom agents can’t define MCP servers inside the agent file, but they can still use MCP tools that are configured at the repository level in settings.

Common patterns:

- **Enable everything**: omit `tools` or set `tools: ["*"]`.
- **Enable a specific set**: `tools: ["read", "edit", "search"]`.
- **Restrict to specific MCP tools**: namespace tools as `server-name/tool-name` (for example `some-mcp/some-tool`).
- **Enable all tools from an MCP server**: `some-mcp/*`.
- **Disable all tools**: `tools: []`.

Unrecognized tool names are ignored, which allows you to include tools that only exist in some environments.

### Tool aliases

Tool aliases are case-insensitive.

| Primary alias | Compatible aliases | Coding agent mapping | Purpose |
| --- | --- | --- | --- |
| `execute` | `shell`, `bash`, `powershell` | Shell tools (`bash` / PowerShell) | Runs a command in the OS-appropriate shell. |
| `read` | `Read`, `NotebookRead` | `view` | Reads file contents. |
| `edit` | `Edit`, `MultiEdit`, `Write`, `NotebookEdit` | Edit tools (implementation varies) | Allows the agent to modify files. |
| `search` | `Grep`, `Glob` | `search` | Finds files or text in files. |
| `agent` | `custom-agent`, `Task` | Custom agent tools | Allows handoff to another custom agent. |
| `web` | `WebSearch`, `WebFetch` | Not applicable to coding agent today | Web fetch/search (environment dependent). |
| `todo` | `TodoWrite` | Not applicable to coding agent today | Structured task lists (environment dependent). |

### Tool names for built-in MCP servers

Copilot coding agent includes some MCP servers out of the box and supports namespacing:

| MCP server name | Notes |
| --- | --- |
| `github` | Read-only GitHub tools. Token is scoped to the source repository by default. Use `github/*` or `github/<tool>`. |
| `playwright` | Playwright tools (typically restricted to localhost). Use `playwright/*` or `playwright/<tool>`. |

## MCP server configuration details

> **Note**
> MCP servers can only be defined directly in agent profiles at the organization/enterprise level. Repository-level agent profiles can still *use* MCP tools configured in repo settings.

Example org-level agent profile with an MCP server configuration:

```yaml
---
name: my-custom-agent-with-mcp
description: Custom agent description
tools: ['tool-a', 'tool-b', 'custom-mcp/tool-1']
mcp-servers:
  custom-mcp:
    type: local
    command: some-command
    args: ['--arg1', '--arg2']
    tools: ['*']
    env:
      ENV_VAR_NAME: $
---
```

### MCP server type

Some ecosystems use `stdio`; Copilot coding agent maps that concept to its `local` server type.

### MCP server environment variables and secrets

> **Note**
> If your MCP server requires secrets/env vars, configure them in the Copilot environment for each repository where the agent is used.

Variable/secret substitution supports common patterns used in GitHub Actions and some local-agent ecosystems. Examples you may see:

- `COPILOT_MCP_ENV_VAR_VALUE` (env value only)
- `$COPILOT_MCP_ENV_VAR_VALUE` (env value + header)
- `${COPILOT_MCP_ENV_VAR_VALUE}` (env value + header)
- `${{ secrets.COPILOT_MCP_ENV_VAR_VALUE }}`
- `${{ var.COPILOT_MCP_ENV_VAR_VALUE }}`

## Example agent profiles

These examples illustrate typical agent definitions.

### Testing specialist

Enables all tools by omitting the `tools` property.

```yaml
---
name: test-specialist
description: Focuses on test coverage and test quality without changing production code
---
```

Prompt content example:

- Analyze existing tests and identify coverage gaps
- Add unit/integration/e2e tests using best practices
- Keep tests deterministic and isolated
- Avoid modifying production code unless explicitly requested

### Implementation planner

Enables only a subset of tools.

```yaml
---
name: implementation-planner
description: Creates implementation plans and technical specs in Markdown
tools: ["read", "search", "edit"]
---
```

Prompt content example:

- Break requirements into actionable tasks
- Draft technical specs, APIs, and data model notes
- Call out dependencies, risks, testing, and rollout considerations

## Processing of custom agents

### Custom agent names

If agent profiles conflict by name, the more specific scope wins (repository overrides organization, organization overrides enterprise).

### Versioning

Agent versions effectively follow the Git commit SHA of the agent profile file. When an agent is assigned to a task, the latest version on that branch is used, and PR interactions stay consistent with that version.

### Tools processing

`tools` filters the tool set available to the agent:

- If `tools` is omitted: all tools are allowed.
- If `tools: []`: no tools are allowed.
- If `tools: [...]`: only the listed tools are allowed.

### MCP server configurations

MCP configuration is applied in an override order from broad → specific (built-in defaults first, then org/enterprise agent configuration, then repository-level MCP settings), so later layers can refine earlier ones.
