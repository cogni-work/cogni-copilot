---
name: port-plugin
description: Port a Claude Code plugin to Microsoft Copilot Studio — reads plugin structure and generates agent config, topic YAML, flow JSON, connector definitions, and step-by-step GUI instructions
argument-hint: /path/to/claude-plugin
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Bash
  - Agent
---

# Port Plugin to Copilot Studio

You are porting a Claude Code plugin to Microsoft Copilot Studio. Follow these steps precisely.

## Input

The user provides a path to a Claude Code plugin directory as an argument. If no path is provided, ask for one.

## Step 1: Discover Source Plugin

Read the source plugin to understand its components:

1. Read `.claude-plugin/plugin.json` for metadata
2. Glob for `commands/*.md` to find commands
3. Glob for `agents/*.md` to find agents
4. Glob for `skills/*/SKILL.md` to find skills
5. Check for `hooks/hooks.json` for hooks
6. Check for `.mcp.json` for MCP servers

Read each discovered component file to understand its purpose, triggers, and behavior.

## Step 2: Create Output Directory

Create `copilot-studio-output/` in the CURRENT WORKING DIRECTORY (not inside the source plugin) with subdirectories:

```
copilot-studio-output/
├── topics/
├── trigger/
├── workflows/
├── actions/
└── knowledge/files/
```

## Step 3: Generate Agent Configuration

Create `copilot-studio-output/agent.mcs.yaml` by:
- Combining all agent system prompts into unified instructions
- Including plugin description as agent description
- Removing Claude-specific tool references (Read, Write, Bash, Glob, Grep)
- Replacing with Copilot Studio equivalents (flows, connectors, knowledge sources)
- Adding scope boundaries

## Step 4: Generate Topics

For each skill and command, generate a `.mcs.yaml` topic file in `copilot-studio-output/topics/`:

**From skills:**
- Extract trigger phrases from the skill description
- Create a topic with generative orchestration (use `description` field, not trigger phrases, unless classic orchestration is more appropriate)
- Convert skill workflow into message nodes, question nodes, and condition groups
- Export reference material as knowledge files to `knowledge/files/`

**From commands:**
- Map command name to topic display name
- Map command argument-hint to a Question node
- Convert command instructions into topic action flow
- If the command uses tools/APIs, also generate a workflow in `workflows/`

## Step 5: Generate Flows

For commands that perform actions (file operations, API calls, external integrations):
- Create `workflows/<FlowName>/metadata.yaml` and `workflows/<FlowName>/workflow.json`
- Use the Logic Apps workflow definition schema
- Include `Run_a_flow_from_Copilot` trigger with input parameters
- Include `Respond_to_Copilot` action with output parameters
- Map command parameters to flow inputs/outputs

## Step 6: Generate Connectors

For each MCP server in `.mcp.json`:
- Create `actions/<connector-name>/apiDefinition.swagger.json` (OpenAPI 2.0)
- Create `actions/<connector-name>/apiProperties.json`
- Map each MCP tool to an API operation
- Map environment variables to security definitions

## Step 7: Generate Trigger Topics

For hooks in `hooks/hooks.json`:
- Map each hook event to a Copilot Studio trigger type
- Generate `.mcs.yaml` files in `trigger/`
- Translate validation logic into ConditionGroup nodes

## Step 8: Generate GUI Instructions

Create `copilot-studio-output/GUI-INSTRUCTIONS.md` with step-by-step instructions for every generated component. Follow the template from the copilot-studio-mapping skill's `references/gui-instructions-template.md`. Include:
- Prerequisites
- Agent instruction setup
- Knowledge source upload steps
- Topic creation (with YAML to paste into code editor)
- Flow setup in Power Automate
- Connector import steps
- End-to-end test checklist

## Step 9: Summary

Present a summary to the user:
- Number of components generated per type
- List of output files with their purposes
- Any components that could NOT be directly translated (with explanation)
- Recommended manual steps

## Important Rules

- ALWAYS use generative orchestration style (topic `description`) unless the source explicitly uses keyword matching
- Generate unique `id` values for every YAML node (use descriptive names like `sendActivity_greeting`, `question_getInput`)
- Enforce all Copilot Studio constraints (42-char names, 2-space YAML indent, `init:` prefix on new variables, `interruptionPolicy` on Question nodes)
- For MCP connectors, MUST use OpenAPI 2.0 (not 3.0)
- Include inline comments in the GUI instructions explaining WHY each mapping was made
