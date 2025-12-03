# Custom Agents Repository

This repository contains custom Copilot Coding Agents for the organization. These agents provide specialized assistance for common development tasks.

## Available Agents

### PR Review Agent (`pr-review-agent.md`)
Expert code reviewer for pull requests that provides thorough, constructive feedback and helps address review comments. Use this agent to:
- Review pull requests for code quality, security, and best practices
- Get help addressing PR review comments
- Understand the rationale behind code review feedback

### Documentation Maintenance Agent (`documentation-maintenance-agent.md`)
Expert documentation maintainer that keeps project documentation accurate, consistent, and up-to-date. Use this agent to:
- Update documentation when code changes
- Identify outdated or missing documentation
- Write clear, user-focused technical documentation
- Maintain README files and API documentation

### Python Code Review Agent (`python-code-review-agent.md`)
Expert Python code reviewer specializing in Python best practices, PEP compliance, and Pythonic patterns. Use this agent to:
- Review Python code for PEP 8 compliance
- Get suggestions for more Pythonic implementations
- Ensure proper type hints and annotations
- Review pytest tests and improve test quality

### Test Specialist Agent (`example-agent.md`)
Testing specialist focused on test coverage, quality, and testing best practices. Use this agent to:
- Analyze existing tests and identify coverage gaps
- Write unit, integration, and end-to-end tests
- Improve test maintainability and reliability

## Using Custom Agents

To use these agents with GitHub Copilot Coding Agent, reference them by name in your prompts. For example:
- "Use the PR Review Agent to review this pull request"
- "Ask the Python Code Review Agent to check this file for PEP 8 compliance"

The agents will be automatically available when working within repositories in this organization.

For more information about custom agents, see:
- [About Custom Agents](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-custom-agents)
- [Creating Custom Agents](https://docs.github.com/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
- [Awesome Copilot Agent Examples](https://github.com/github/awesome-copilot/tree/main/agents)

## Contributing

To add a new custom agent:

1. Create a new `.md` file in the `agents` directory
2. Include the required YAML frontmatter with `description` and `tools`
3. Document the agent's purpose, capabilities, and guidelines
4. Update this README to include the new agent

### Agent File Structure

Each agent file should follow this structure:

```markdown
---
description: "Brief description of what the agent does"
tools:
  [
    "codebase",
    "editFiles",
    "search",
    # ... other tools as needed
  ]
---

# Agent Name

[Detailed description of the agent's purpose, capabilities, and guidelines]
```

## Guidelines for Custom Agents

When creating custom agents:

- **Be Specific**: Define clear responsibilities and boundaries
- **Include Examples**: Provide code examples and response formats
- **Document Tools**: Only include tools the agent actually needs
- **Follow Patterns**: Look at existing agents for structure guidance
- **Test Thoroughly**: Verify agents work as expected before sharing
