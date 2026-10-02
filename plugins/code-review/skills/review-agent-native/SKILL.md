---
name: review-agent-native
description: Review a code change for gaps where a user can do something an agent cannot, where the agent lacks the context to act, or where agent tools encode workflows instead of primitives. Use when reviewing for agent-native architecture, action and context parity, agent tool design, or system prompt context.
---

# Review lens: Agent-Native

Review whether agents are first-class users of the application, with the same capabilities and context as a human user rather than a bolted-on feature.

## Scope

- **Action parity** Every UI action that matters has an equivalent agent tool. A new button, form or gesture with no matching tool is an orphan feature.
- **Context parity** The system prompt carries runtime state, not only static instructions: the resources the user can see, recent activity, what each tool does, and the app's domain vocabulary.
- **Shared workspace** Agents and users read and write the same data, with the same paths. Red flags are a separate agent sandbox, writes to `agent_output/` instead of the user's documents, a sync layer bridging two spaces, a UI that does not update when the agent mutates state, and agent-created artifacts users cannot inspect or edit.
- **Primitive tools** Tools read, write and store, take data rather than decisions, and return enough output for the agent to verify success. A tool that categorizes, prioritizes and decides to notify inside its body is a workflow tool; one that accepts a decision enum instead of raw data takes the choice away from the agent.
- **Discoverability** Users can find out what the agent can do, and every domain noun (feed, report, task, profile) is known to the agent, has a tool, and is documented in the system prompt.

## Method

First check whether the codebase has agent integration at all: tool definitions, system prompt construction, or LLM API calls. If it has none, that absence is the main finding.

Find where UI actions and agent tools are defined for the stack: `onClick`, `onSubmit`, form actions, `button_to`, `form_with` or Turbo and Stimulus actions for UI; `tool()` and the `tools` parameter of `streamText` or `generateText` (Vercel AI SDK), `@tool` and `StructuredTool` (LangChain), assistant `tools` arrays (OpenAI), MCP server definitions and `.mcp.json`, or `agents/*.md` and `skills/*/SKILL.md` for tools.

For a change, focus on new and modified code and whether it keeps existing parity; search outward only when it touches the tool registry, system prompt construction or the shared data layer. Cross-reference UI actions against tools, then make a second pass by domain noun.

## Threshold

Report missing parity for core domain create, read, update and delete, primary workflows, and actions that modify user data; report secondary features and filtered or sorted read-only views at lower priority. Report gaps traceable from the code alone; report one that depends on unseen context, such as a prompt assembled elsewhere, only when it is severe.

Do not report settings, onboarding, admin panels or cosmetic UI (animations, theme toggles, layout preferences) as more than observations. Do not report intentionally human-only flows (CAPTCHA, 2FA, OAuth consent, terms acceptance), authentication ceremony, or platform-imposed gates such as OS permission dialogs. If an action might be intentionally human-only, report it as an observation and say so.

A workflow tool that wraps a safety-critical atomic sequence (record, charge and receipt as one unit) or external orchestration the agent should not drive step by step, such as a deploy, is acceptable; note it for review without treating it as a defect.

## Reporting

- Name the UI action or domain noun, its location (`file:line`), and the missing or deficient tool or prompt context.
- State the priority of the capability: core, secondary, or low.
- State the fix: add the tool and document it in the prompt, inject resources and vocabulary, move to a shared store, or extract primitives and move orchestration into the prompt.
