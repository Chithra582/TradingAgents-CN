---
name: nanobot-agent-design
description: Design guide and prompt patterns for Nanobot multi-agent analyst workflows.
license: Apache-2.0
metadata:
  platform: "nanobot"
---
# Nanobot Agent Design

Use this skill when the user wants Nanobot to create or redesign an agent based on the resources already present in this repository.

Do not directly hand off the task to the old Agent Workshop flow unless the user explicitly asks to use that workflow.
The default path for Nanobot is a lighter workflow built around resource scouting, confirmation, and targeted generation.

This repository also supports v3.0 style configurable agents stored in MongoDB.
Nanobot should treat database-backed agent assets as first-class resources, not just the built-in Python agent registry.

Relevant design references:

1. `docs/design/v3.0/ai-workflow-generation.md`
2. `docs/design/v3.0/agent-workshop-interface-sequence.md`
3. `docs/design/v3.0/implementation-status-report.md`
4. `docs/architecture/AGENT_WORKSHOP_END_TO_END_DESIGN.md`

The architecture document is the source of truth for:

1. Nanobot-first end-to-end flow from demand intake to version publish
2. the real runtime call chain between Nanobot, AgentWorkshopService, prompt templates, runtime configs, and database assets
3. the distinction between `gap_report.suggested_tools`, `version.required_tools`, and `execution_plan.execution_steps[].suggested_tools`

This SKILL file is the source of truth for Nanobot behavior rules.
The architecture document is the source of truth for system structure and data flow.

## Core Principle

Nanobot should not start from a blank slate and should not jump straight into heavy multi-step workflow services.
It should first discover what already exists, confirm the reusable parts, and only then generate the minimum agent artifacts needed for the request.

## Numerical Calculation Principle (Hard Rule)

All key financial calculations must be performed through tools or Skills, never by the LLM itself.

This applies to:

1. DCF / absolute valuation
2. PE / PB / PEG / EV/EBITDA relative valuation
3. financial ratio computation (ROE, ROA, debt ratios, growth rates, etc.)
4. scoring, ranking, weighting, or any quantitative output
5. fair value, target price, or price range derivation
6. any numeric output that would be presented as a research conclusion

When a gap is discovered for a required calculation:

1. Nanobot must **not** suggest that the LLM can compute the value itself.
2. Nanobot must **not** accept "LLM 自己算也可以接受" as a valid fallback.
3. Nanobot must treat the missing calculation capability as a **blocking gap** and trigger Skill/tool creation to fill it.
4. If the user asks "是否需要 Skill 还是 LLM 自己算", Nanobot must answer: **必须通过 Skill 或工具计算，不能由 LLM 自己算。**

This rule is non-negotiable and applies to all agents generated through this system.

## Data Source Transparency Principle (Hard Rule)

All Skills and agents must prioritize system-provided data sources. Any external data source must be clearly labeled.

System-provided data sources include:
- tushare
- akshare
- QMT (迅投)
- Other data sources already registered in the project's tool registry

When creating or reviewing a Skill:

1. Nanobot must **prefer** system-provided data sources when generating Skill specs. If the same data can be obtained through tushare/akshare/QMT, use those first.
2. If a Skill requires data from an external source (e.g., direct web scraping from a specific website, third-party API, proprietary data feed), the Skill's `data_source` field must explicitly state this, and the Skill description must mention the external source.
3. Nanobot must **not** silently accept a Skill that scrapes external websites or calls external APIs without declaring the data source.
4. When presenting a Skill to the user, Nanobot must highlight any non-system data sources so the user can judge reliability and data provenance.
5. If a gap requires data that is not available through system-provided sources, Nanobot must inform the user before creating the Skill, explaining where the data will come from and the associated reliability trade-offs.

This rule ensures users can audit data provenance and make informed decisions about the reliability of their analysis pipeline.

## Resource Discovery Order

When the user asks for a new agent, inspect resources in this order:

1. Use `search_project_capabilities` to find relevant tools, skills, MCP capabilities, and external skills.
2. Use `inspect_agent_blueprints` to find similar built-in agents.
3. Use `inspect_runtime_agent_configs` to find database-backed runtime agents and UniversalAgent candidates.
4. Use `inspect_agent_workshop_assets` to find reusable `agent_specs`, `agent_versions`, and prompt-template lineage.
5. Use `get_agent_blueprint_details` on the closest built-in candidate.
6. Only then read focused source files such as:
   - `core/agents/config.py`
   - `core/agents/universal.py`
   - `core/agents/factory.py`
   - the nearest adapter under `core/agents/adapters/`
   - `core/tools/config.py` or the specific tool implementation file
   - `skills/*/SKILL.md` if the task depends on a specialized workflow

Do not start by reading large service files or old workshop orchestration code unless the task specifically needs it.

## Resource Confirmation Rules

Before claiming a resource is usable, confirm at least one of these:

1. It appears in `search_project_capabilities` as bindable.
2. It appears in a registered agent blueprint.
3. It appears in `inspect_runtime_agent_configs` as an existing runtime agent definition.
4. It appears in `inspect_agent_workshop_assets` as an existing spec or built version.
5. Its implementation file and tool metadata both exist.

If a capability is only mentioned in old docs or old configs but is not bindable or not implemented, treat it as a gap, not an available dependency.

Nanobot must never collapse different resource layers into a single undifferentiated “existing agents” list.

It must keep these layers distinct in both reasoning and final user-facing output:

1. built-in blueprints from the Python agent registry
2. runtime-configured agents from `agent_configs` and `tool_agent_bindings`
3. workshop assets from `agent_specs` and `agent_versions`

If the user asks why something is not visible in Agent Workshop, Nanobot must explicitly state that the Agent Workshop UI reflects workshop assets, not every runtime-configured agent and not built-in blueprints.

If the user asks about whether an existing agent/version/spec still exists, or says something was deleted, Nanobot must not rely on prior conversation memory or earlier resource summaries.

Required behavior:

1. re-run live checks before answering existence/state questions
2. treat the newest tool output as authoritative over earlier conversation text
3. if `inspect_agent_workshop_assets` says `spec_count=0` and `version_count=0`, do not claim Workshop still has that agent/version
4. if `inspect_runtime_agent_configs` still shows a runtime candidate while Workshop assets are gone, explicitly call it a runtime residue or candidate config rather than a current Workshop version

## Prompt Template Resolution Rules

Nanobot must treat prompt-template questions as live state queries, not as loose design discussion.

Required tool preference:

1. use `inspect_prompt_templates` when the user wants to browse, search, filter, or confirm what templates exist in the template library
2. use `get_prompt_template_detail` when the user already has a template identity and wants the full template body or metadata
3. use `get_current_thread_effective_prompt_template` first when the user is asking which template is active in the current Nanobot thread or current workbench context
4. use `get_effective_prompt_template` when the user provides explicit resolution parameters or when current-thread context is unavailable

Required behavior:

1. do not infer the effective template from `agent_name` alone when debug mode, workflow context, node context, user preference, or explicit debug-template override may change the result
2. if the user asks “当前生效模板”, “现在用的是哪份模板”, “调试态命中的是哪份模板”, or any equivalent wording, prefer `get_current_thread_effective_prompt_template` first rather than browsing the template library
3. if the user asks why a specific template was selected, first resolve the effective template, then explain the result as a priority-resolution outcome rather than as a guess
4. if context such as `workflow_id`, `node_id`, `debug_template_id`, `preference_id`, or `user_id` is already available from the current session or thread context, Nanobot should pass that context into `get_effective_prompt_template`
5. if that context is missing, Nanobot may use the best currently available context, but it must say which resolution inputs were actually used and must not present the answer as stronger than the available context supports

Disallowed behavior:

1. answering “当前模板是 X” only because a template exists for the same `agent_name`
2. using `inspect_prompt_templates` as a substitute for effective-template resolution when the user is asking about the template that is active right now
3. claiming a debug template is active without passing or verifying debug-mode context

## Default Design Workflow

For a new agent request, Nanobot should follow this sequence:

1. If the user's request is still high-level, call `prepare_agent_requirement_intake` first.
2. Restate the target job of the agent in one sentence.
3. Identify the minimum capabilities the agent truly needs.
4. Search project capabilities for those needs.
5. Check whether an existing runtime-configured agent or published version already covers the need.
6. Find the nearest existing built-in agent blueprint.
7. Decide the build path:
   - reuse an existing agent with light adaptation
   - publish or refine an existing configurable agent asset
   - create a new prompt/template-driven agent
   - create a new database-configured UniversalAgent-backed agent
   - create a new tool-bound agent
   - create a new workflow node only if a standalone agent is not enough
8. Generate only the minimal artifacts needed.

Prefer a narrow and composable agent over a large all-in-one design.

## Internal Phase Model

Nanobot currently uses an implicit phase model rather than a single explicit phase enum.

Required interpretation:

1. `requirement-intake` means the request is still missing key dimensions such as single responsibility, output shape, non-goals, preferred tools, overwrite/iteration mode, valuation method scope, or test strategy
2. `confirmation` means the request is already specific enough to produce a confirmation brief, but no database write should happen yet
3. `generation` means the user has explicitly confirmed and Nanobot may write candidate artifacts
4. `governance` means Nanobot is operating on Agent Workshop Draft / Version artifacts such as gap analysis, tooling plan, version build, debug, evaluate, iterate, or publish

Nanobot must not pretend these phases come from a hidden backend state field. They are inferred from:

1. the current user request
2. the current thread context
3. the currently available tool set
4. the structured outputs of the agent_builder tools

## Intake Gate Rules

`prepare_agent_requirement_intake` is the default gate for deciding whether Nanobot is still in the requirement-intake phase.

Required behavior:

1. The tool returns a `proposed_plan` with smart defaults already filled in. Nanobot's job is to **present this plan as a proposal with reasoning**, not to interrogate the user dimension by dimension.
2. If `critical_questions` is non-empty (typically 0-2 items), ask only those questions. For each question, **explain the background, trade-offs, and recommend a direction with reasons** — don't just throw questions at the user.
3. Multi-round conversations are fine. The key is that **every round should have substance**: explain distinctions, present options with pros/cons, give your expert recommendation.
4. Follow `nanobot_guidance` from the tool output — it contains the exact conversational strategy.
5. If `can_proceed_to_confirmation=true`, move directly to confirmation. The user only needs to say "可以" or "没问题" to proceed.
6. Do not present 5-8 decision points. The user is talking to an expert who should design the solution; the user's role is lightweight confirmation, not specification writing.

The core principle: **Nanobot educates, proposes, and the user confirms.** Smart defaults cover 90%+ of dimensions. Nanobot explains the remaining 10% with domain expertise.

## Confirmation Gate Rules

`prepare_agent_generation_confirmation` is the default confirmation gate.

Required behavior:

1. do not write any database artifact before confirmation unless the user explicitly asked for a draft sync action
2. if `prepare_agent_generation_confirmation` returns `needs_requirement_intake`, return to intake rather than forcing generation
3. if it reports `missing_tools`, treat that as a hard stop rather than a soft suggestion
4. only move to generation after the user gives an explicit confirmation compatible with the suggested confirmation template

## Generation Gate Rules

`generate_confirmed_candidate_agent` is the default generation gate.

Required behavior:

1. generation requires `confirmed=true`
2. when overwrite risk exists, generation additionally requires `allow_overwrite=true`
3. if `sync_to_workshop_draft=true`, Nanobot should treat the resulting Draft as an asset-layer sync, not as a final accepted release
4. if `auto_build_workshop_version=true`, Nanobot should treat the resulting version as testing-ready, not as already accepted

## Tool Discovery Rules

Before each run, Nanobot already performs runtime tool discovery through `EmbeddedNanobotRuntime._build_scoped_tools`.

Required interpretation:

1. this is a real dynamic tool-selection step, not a static fixed tool list
2. because `nanobot-agent-design` is passed in `skill_names`, Nanobot should expect the full `AGENT_BUILDER_TOOL_IDS` set to be eligible
3. dynamic tool selection still ranks tools by query terms, categories, metadata text, and related tools before exposing them to the model
4. explicit resource-discovery tools like `search_project_capabilities`, `inspect_agent_workshop_assets`, and `inspect_runtime_agent_configs` are still recommended when the user is designing an agent or asking what is reusable

Nanobot must not claim that a capability was "discovered" merely because it was mentioned in chat. It is only discovered if it was surfaced from the real registered tool/capability layers.

## Prompt Relationship Rules

Nanobot must keep three prompt layers distinct:

1. Nanobot system prompt: generated by `ContextBuilder.build_system_prompt`; this governs Nanobot's own orchestration behavior
2. runtime ephemeral guidance: dynamically injected by runtime when query analysis suggests a special execution policy such as agent-first execution
3. target-agent prompt template: the template written for the generated agent or version; this governs the generated agent's runtime behavior, not Nanobot's orchestration behavior

Disallowed behavior:

1. mixing Nanobot orchestration rules with the target agent's runtime prompt body
2. treating the target agent's prompt template as if it explains Nanobot's phase decisions
3. claiming that one prompt layer fully explains the whole flow

## Nanobot-first Governance Rule

For agent creation and iteration tasks, Nanobot should remain the primary orchestration layer even when artifact persistence is delegated to `AgentWorkshopService`.

Required behavior:

1. do not describe the current system as "back to the old Agent Workshop flow"
2. describe it as "Nanobot-first orchestration with AgentWorkshopService as the artifact layer"
3. when debugging design issues, separate orchestration problems from asset-generation problems

## Minimal Generation Chain

When the user asks Nanobot to actually generate a new configurable agent now, prefer this direct chain:

1. Use `prepare_agent_generation_confirmation` to generate a short confirmation brief before any write.
2. Wait for explicit user confirmation if responsibility, output shape, tool set, or overwrite risk could change the intended agent shape.
3. After confirmation, use `generate_confirmed_candidate_agent` as the default one-shot path.
4. If that generation is synced to Agent Workshop Draft, it should normally auto-create one testing-ready version so the user can move directly into real-data testing.

If the task specifically needs low-level control, the internal write chain is:

1. `create_or_update_runtime_agent_config`
2. `create_or_update_universal_agent_prompt_template`
3. `bind_agent_tools_and_validate_candidate`

This is the default generation path for a new lightweight candidate.
Do not detour into the old workshop flow unless the user explicitly asks for spec/version/publish governance.

When the user only cares whether the agent is "generated and ready for formal testing", Nanobot should not ask the user to manually perform the "generate version" step if the system can do it automatically after confirmation.
It should auto-create the testing version when possible, then clearly state that the remaining decisive step is a real-data test pass/fail rather than version creation itself.

## Acceptance Chain

For Agent Workshop-backed agents, test and acceptance must follow the designated lifecycle chain rather than temporary ad hoc probing.

Required chain:

1. finish configuration generation and dry-run validation
2. create or auto-create the testing version
3. treat `testing` as "entered real-data testing", not as "already passed"
4. run a real-data test and record the evaluation result through the official Workshop evaluation path
5. only treat the version as acceptance-ready when the latest recorded evaluation decision is `pass`
6. only treat the version as formally released after `publish_version` promotes it to `active`

Preferred execution tools for the testing loop:

1. use `debug_agent_workshop_version` when the user wants only a sampl


<!-- Truncated for OpenGAP token limits -->
