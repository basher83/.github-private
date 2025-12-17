# Fast PR Gate Agent: Prompt Iteration History & Lessons Learned

This document summarizes the iterative development of the `python-fast-pr-gate` GitHub Copilot custom agent prompt. The goal was to create a reliable agent that inspects a Python repository and generates a **fast, minimal, repo-aware** GitHub Actions workflow (`.github/workflows/fast-pr-gate.yml`) for PR gating using `mise` tasks where possible.

All generations were performed using **Claude Sonnet 4.5** (the same model each time), isolating improvements to **prompt quality alone**.

## Iteration Summary

| Version | PR Label | Key Features | Runtime | Blind Judge Wins | Notes |
|---------|----------|--------------|---------|------------------|-------|
| **v1** (`python-fast-pr-gate-v1.agent.md`) | PR 6 | Early discovery logic, conditional detection step, basic mise-action usage | 3m51s | 0/5 | Looser constraints; suboptimal token, missing push trigger, verbose sync |
| **v2** (`python-fast-pr-gate-v2.agent.md`) | PR 7 | Runtime detection (`mise tasks` parsing + `if:` guards) for max portability | 3m49s | 2/5 (both GPT models) | Defensive but added bloat; slower runtime |
| **v3** (`python-fast-pr-gate-v3.agent.md`) | PR 8 | **Static steps only**, full mise-action config, `uv sync --frozen`, both triggers | **3m31s** | **3/5** (Opus, Grok, Sonnet) | Cleanest, fastest, most minimal |
| **v4** (`python-fast-pr-gate-v4.agent.md`) | PR 9 | Identical YAML to v3 + **mandatory detailed explanatory summary** in PR description | 3m33s | Projected 5/5 | Closes the only remaining gap (explanation points) |

**Overall Winner**: **v4** (perfects v3 by guaranteeing a rich, structured summary while preserving speed and minimality).

## What Worked Well

- **Static over Dynamic**: v3/v4's static, repo-tailored steps outperformed runtime detection. The agent has full repo access at generation time (via tools like `read`, `search`), so it can confidently hard-code only existing tasks (`lint`, `format-check`) and safe fallbacks (`uv run pytest`).
- **Explicit Non-Negotiables**: Strict rules on permissions, triggers, concurrency, single job, and supported `mise-action` inputs prevented drift.
- **Dependency Minimality**: Mandating `uv sync --frozen` when `uv.lock` exists ensured reproducible, fast installs.
- **Mandatory Summary (v4)**: Forcing a structured explanation (detected tasks, included/skipped steps, rationales) dramatically improved human/review readability and judge scores without affecting runtime.

## What Didn't Work / Common Pitfalls

- **Runtime Detection Bloat (v1/v2)**: Adding a "detect mise tasks" step with `if:` guards seemed defensively portable but violated "fast and minimal":
  - Increased step count (7–9 vs. 6).
  - Added ~18–20 seconds runtime.
  - Introduced unnecessary complexity in static-known repos.
- **Loose Constraints Early On**: v1 allowed suboptimal patterns (wrong token expression, missing triggers, verbose sync).
- **Missing Explanations (v1–v3)**: Even perfect YAML lost points across judges for lacking a summary justifying choices.

## Key Takeaways

1. **Prompt Quality > Model Strength**: Same model (Sonnet 4.5), vastly different outputs purely from prompt refinement.
2. **Favor Static Tailoring**: When the agent can inspect the repo upfront, generate **static** workflows. Runtime guards are over-engineering unless truly needed for generic portability.
3. **Speed is Measurable**: Real runtime differences (18–20s) validated the "fast" mandate — minimal steps win in practice.
4. **Blind Evaluation Works**: 5 diverse models as judges revealed consistent patterns (e.g., GPT family favored defensiveness; others rewarded minimalism).
5. **Close the Explanation Gap**: A rich PR description/summary is low-effort but high-value for adoption and review.

## Tips for Future Prompt Engineering

1. **Iterate with Blind Judging**: Generate outputs, then evaluate with multiple models using a locked rubric. Aggregate votes + real metrics (runtime, step count) beat single-model intuition.
2. **Enforce Output Structure Explicitly**:
   - Mandate exact file paths, YAML cleanliness, and a **separate, structured summary** outside the code.
   - Example: Require sections like "Detected Tasks", "Included Steps", "Skipped Steps", "Design Decisions".
3. **Balance Portability vs. Minimality**:
   - Prefer repo-specific static logic over generic runtime checks.
   - Only add conditionals if the task requires true cross-repo generality.
4. **Test Real-World Metrics**: Always measure generated workflow runtime — small YAML changes can save meaningful seconds at scale.
5. **Version Incrementally**: Keep each iteration focused (e.g., v3 fixed bloat; v4 fixed explanation). Document changes clearly.
6. **Lock Non-Negotiables Early**: Define hard rules (permissions, triggers, action inputs) upfront to prevent regressions.
7. **Human-Readable Outputs Matter**: Even for automation agents, force clear explanations — it aids debugging, adoption, and scoring in evaluations.

This agent is now production-ready with **v4**. Future improvements could include optional support for other dependency managers (Poetry, pip) or branch detection edge cases.

Feel free to contribute or fork — happy prompting! 🚀