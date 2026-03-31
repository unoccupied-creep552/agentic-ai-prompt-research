# Claude Code System Prompts

The complete, extracted collection of every internal system prompt, agent directive, and security classifier used by **Claude Code**, Anthropic's agentic command-line interface for software engineering.

All prompts were extracted from the publicly leaked source at [instructkr/claude-code](https://github.com/instructkr/claude-code) and organized into a structured reference.

## Overview

Claude Code uses a sophisticated multi-layered prompt architecture. The main system prompt is not a static string but is dynamically assembled at runtime from modular section-builder functions. A boundary marker splits it into a globally cacheable prefix and a session-specific suffix, enabling prompt caching across API calls.

Beyond the core identity prompt, the system includes specialized agent prompts, a multi-worker coordinator, a 2-stage security classifier for auto-approving tool calls, and a suite of utility prompts for memory selection, session search, and tool summarization.

## Prompt Catalog

### Core Identity

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 01 | [Main System Prompt](prompts/01_main_system_prompt.md) | `prompts.ts` | Dynamically assembled master prompt covering identity, behavior, tool guidance, tone, and efficiency |
| 02 | [Simple Mode](prompts/02_simple_mode.md) | `prompts.ts` | Minimal 4-line prompt activated by `CLAUDE_CODE_SIMPLE` |
| 03 | [Default Agent Prompt](prompts/03_default_agent_prompt.md) | `prompts.ts` | Base prompt inherited by all sub-agents |
| 04 | [Cyber Risk Instruction](prompts/04_cyber_risk_instruction.md) | `cyberRiskInstruction.ts` | Security boundaries for authorized vs. prohibited actions |

### Orchestration

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 05 | [Coordinator System Prompt](prompts/05_coordinator_system_prompt.md) | `coordinatorMode.ts` | Multi-worker orchestrator with 4-phase workflow and concurrency rules |
| 06 | [Teammate Prompt Addendum](prompts/06_teammate_prompt_addendum.md) | `teammatePromptAddendum.ts` | Communication protocol for swarm and team mode |

### Specialized Agents

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 07 | [Verification Agent](prompts/07_verification_agent.md) | `verificationAgent.ts` | Adversarial testing specialist that tries to break implementations |
| 08 | [Explore Agent](prompts/08_explore_agent.md) | `exploreAgent.ts` | Read-only codebase exploration with strict no-modify constraints |
| 09 | [Agent Creation Architect](prompts/09_agent_creation_architect.md) | `generateAgent.ts` | Designs new agent configurations from user requirements |
| 10 | [Status Line Setup Agent](prompts/10_statusline_setup_agent.md) | `statuslineSetup.ts` | Configures terminal status line across shell environments |

### Security and Permissions

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 11 | [Permission Explainer](prompts/11_permission_explainer.md) | `permissionExplainer.ts` | Explains tool risk levels before user approval |
| 12 | [Auto Mode Classifier](prompts/12_yolo_auto_mode_classifier.md) | `yoloClassifier.ts` | 2-stage security classifier for auto-approving tool calls |

### Tool Descriptions

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 13 | [Tool-Specific Prompts](prompts/13_tool_prompts.md) | `src/tools/*/prompt.ts` | All 30+ tool descriptions including Bash, Edit, Agent, and fork semantics |

### Utility and Helpers

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 14 | [Tool Use Summary](prompts/14_tool_use_summary.md) | `toolUseSummaryGenerator.ts` | Generates git-commit-style labels for completed tool batches |
| 15 | [Session Search](prompts/15_session_search.md) | `agenticSessionSearch.ts` | Semantic search across past conversation sessions |
| 16 | [Memory Selection](prompts/16_memory_selection.md) | `findRelevantMemories.ts` | Selects relevant memory files for query context |
| 17 | [Auto Mode Critique](prompts/17_auto_mode_critique.md) | `autoMode.ts` | Reviews user-written auto-mode classifier rules |
| 20 | [Session Title](prompts/20_session_title.md) | `sessionTitle.ts` | Haiku-powered 3-7 word session title generator |
| 29 | [Agent Summary](prompts/29_agent_summary.md) | `agentSummary.ts` | Periodic progress updates for sub-agents in coordinator mode |
| 30 | [Prompt Suggestion](prompts/30_prompt_suggestion.md) | `promptSuggestion.ts` | Predicts user follow-up commands for clickable suggestions |

### Context Window Management

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 21 | [Compact Service](prompts/21_compact_service.md) | `compact/prompt.ts` | Multi-variant conversation summarization with analysis/summary blocks |
| 22 | [Away Summary](prompts/22_away_summary.md) | `awaySummary.ts` | 1-3 sentence session recap for returning users |

### Dynamic Sections

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 18 | [Proactive Mode](prompts/18_proactive_mode.md) | `prompts.ts` | Autonomous agent with tick-based pacing and terminal focus awareness |
| 23 | [Chrome Browser Automation](prompts/23_chrome_browser_automation.md) | `claudeInChrome/prompt.ts` | Claude-in-Chrome extension: GIF recording, tab management, dialog handling |
| 24 | [Memory Instruction](prompts/24_memory_instruction.md) | `claudemd.ts` | CLAUDE.md loading, @include directives, and frontmatter globs |

### Bundled Skills

| # | Prompt | Source | Description |
|---|--------|--------|-------------|
| 19 | [Simplify Skill](prompts/19_simplify_skill.md) | Bundled binary | Three-agent parallel review for code reuse, quality, and efficiency |
| 25 | [Skillify Skill](prompts/25_skillify.md) | Bundled binary | Interview-based skill creation, generates SKILL.md files |
| 26 | [Stuck Skill](prompts/26_stuck_skill.md) | Bundled binary | Internal diagnostic for frozen or slow sessions |
| 27 | [Remember Skill](prompts/27_remember_skill.md) | Bundled binary | Promotes auto-memory entries into CLAUDE.md files |
| 28 | [Update Config Skill](prompts/28_update_config_skill.md) | Bundled binary | Manages settings.json, hooks, and permission arrays |

## Architecture

### Prompt Assembly

The main system prompt is built by `getSystemPrompt()` through a pipeline of section builders:

```
getSystemPrompt()
    |
    |   Static Prefix (globally cached)
    |-- getSimpleIntroSection()             Identity and Cyber Risk
    |-- getSimpleSystemSection()            Permission modes, hooks, reminders
    |-- getSimpleDoingTasksSection()        Code style, security, error handling
    |-- getActionsSection()                 Reversibility, blast radius
    |-- getUsingYourToolsSection()          Tool preferences, parallel calls
    |-- getSimpleToneAndStyleSection()      No emojis, concise references
    |-- getOutputEfficiencySection()        Inverted pyramid communication
    |
    |   SYSTEM_PROMPT_DYNAMIC_BOUNDARY
    |
    |   Dynamic Suffix (session-specific)
    |-- getSessionSpecificGuidanceSection() Agent tools, skills, verification
    |-- loadMemoryPrompt()                  MEMORY.md content
    |-- getAntModelOverrideSection()        Internal model overrides
    |-- computeSimpleEnvInfo()              CWD, OS, git state, model info
    |-- getLanguageSection()                Language preferences
    |-- getOutputStyleSection()             Custom output styles
    |-- getMcpInstructionsSection()         MCP server instructions
    |-- getScratchpadInstructions()         Temporary file directory
    |-- getFunctionResultClearingSection()  Context window management
    |-- SUMMARIZE_TOOL_RESULTS_SECTION      Persist important information
```

The `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` marker is critical for Anthropic's prompt caching system. Everything above it is identical across sessions and cached globally. Everything below it is regenerated per session.

### Auto Mode Classifier

The auto-approval system uses a separate classifier prompt assembled from:

1. **Base prompt** from `auto_mode_system_prompt.txt` with classifier instructions
2. **Default rules** from `permissions_external.txt` with allow, deny, and environment sections
3. **User overrides** from `settings.autoMode` that replace sections entirely
4. **2-stage classification**: Stage 1 runs fast, Stage 2 uses extended thinking if the initial result is uncertain

### Context Window Pipeline

```
User Message
    |
    v
[Micro-Compaction]  -- Cache-aware tool result deletion (time/count triggers)
    |
    v
[Compact Service]   -- Full/partial summarization with analysis blocks
    |
    v
[Prompt Suggestion] -- Predict next user command (async, non-blocking)
    |
    v
[Away Summary]      -- Session recap if user was idle
```

### Memory System

```
Memory Loading Order (first loaded = lowest priority):
    |
    |-- /etc/claude-code/CLAUDE.md          Managed (enterprise)
    |-- ~/.claude/CLAUDE.md                 User (global)
    |-- <project>/CLAUDE.md                 Project (shared)
    |-- <project>/.claude/CLAUDE.md         Project (shared)
    |-- <project>/.claude/rules/*.md        Project rules (shared)
    |-- <project>/CLAUDE.local.md           Local (private, git-ignored)
    |
    |   @include directives resolve transitively (max depth: 5)
    |   Frontmatter `paths:` field enables conditional injection
```

### Environment Variables

| Variable | Effect |
|----------|--------|
| `CLAUDE_CODE_SIMPLE` | Activates the minimal 4-line system prompt |
| `USER_TYPE=ant` | Enables Anthropic-internal sections and model overrides |
| Feature flags via `bun:bundle` | Gate proactive mode, verification agents, fork subagents, and more |

## Repository Structure

```
claude-code-system-prompts/
    README.md
    prompts/
        01_main_system_prompt.md
        02_simple_mode.md
        03_default_agent_prompt.md
        04_cyber_risk_instruction.md
        05_coordinator_system_prompt.md
        06_teammate_prompt_addendum.md
        07_verification_agent.md
        08_explore_agent.md
        09_agent_creation_architect.md
        10_statusline_setup_agent.md
        11_permission_explainer.md
        12_yolo_auto_mode_classifier.md
        13_tool_prompts.md
        14_tool_use_summary.md
        15_session_search.md
        16_memory_selection.md
        17_auto_mode_critique.md
        18_proactive_mode.md
        19_simplify_skill.md
        20_session_title.md
        21_compact_service.md
        22_away_summary.md
        23_chrome_browser_automation.md
        24_memory_instruction.md
        25_skillify.md
        26_stuck_skill.md
        27_remember_skill.md
        28_update_config_skill.md
        29_agent_summary.md
        30_prompt_suggestion.md
```

## Disclaimer

This repository exists for educational and research purposes only. All prompt content is the intellectual property of Anthropic. This project is not affiliated with, endorsed by, or connected to Anthropic in any way.

## License

No license is claimed over the original prompt content. This repository documents publicly available information extracted from leaked source code for research purposes.
