---
name: Copilot Studio Mapping
description: >
  Translation rules and templates for porting Claude Code plugins to Microsoft Copilot Studio.
  This skill should be used when the user mentions "port to copilot studio", "convert plugin to copilot",
  "migrate to copilot studio", "copilot studio mapping", "translate plugin to microsoft",
  "export plugin for copilot", "copilot studio YAML", "copilot studio topic", or any request
  involving converting Claude Code plugin components (agents, skills, commands, hooks, MCP servers)
  into their Microsoft Copilot Studio equivalents (agents, topics, flows, triggers, connectors).
version: 0.1.0
---

# Copilot Studio Mapping Rules

Port Claude Code plugins to Microsoft Copilot Studio by translating each component type using the rules below. Generate output in two formats: (1) exportable files for the VS Code extension workspace, and (2) step-by-step GUI instructions for manual entry.

## Component Translation Map

| Claude Code | Copilot Studio | Output Format |
|---|---|---|
| Agent (system prompt, tools) | Agent (instructions, tools, knowledge) | `agent.mcs.yaml` + GUI instructions |
| Skill (SKILL.md + references) | Topic with generative orchestration + Knowledge files | `.mcs.yaml` topic + knowledge docs |
| Command (frontmatter + instructions) | Topic with trigger phrases + flow actions | `.mcs.yaml` topic + `workflow.json` |
| Hook (event handler) | Event trigger topic or system topic override | `.mcs.yaml` trigger topic |
| MCP server (tool provider) | Custom connector (OpenAPI 2.0) or REST API tool | `apiDefinition.swagger.json` + `apiProperties.json` |

## Output Workspace Structure

Generate all output into a `copilot-studio-output/` directory using the VS Code extension workspace format:

```
copilot-studio-output/
├── agent.mcs.yaml
├── settings.mcs.yml
├── connectionreferences.mcs.yml
├── GUI-INSTRUCTIONS.md
├── topics/
│   ├── <skill-name>.mcs.yaml
│   └── <command-name>.mcs.yaml
├── trigger/
│   └── <hook-name>.mcs.yaml
├── workflows/
│   └── <flow-name>/
│       ├── metadata.yaml
│       └── workflow.json
├── actions/
│   └── <connector-name>.mcs.yml
└── knowledge/
    └── files/
        └── <skill-knowledge>.md
```

## Translation Rules by Component

### 1. Agent → Copilot Studio Agent

Read the source agent's markdown file. Extract:
- **System prompt** → map to `instructions` in `agent.mcs.yaml`
- **Tools list** → map to tools/connectors attached to the agent
- **Description / whenToUse** → map to agent `description`

Rewrite the system prompt for Copilot Studio style:
- Remove Claude-specific references (tool names like `Read`, `Write`, `Bash`)
- Replace with Copilot Studio equivalents ("use the connected flow", "query knowledge source")
- Keep behavioral instructions, persona, and domain knowledge intact
- Add explicit scope boundaries ("You should NOT answer questions about...")

### 2. Skill → Topic + Knowledge

Each skill becomes TWO outputs:

**a) Topic (`.mcs.yaml`):**
- Extract trigger phrases from the skill's `description` field
- The skill's core workflow becomes topic actions (message nodes, question nodes)
- If the skill has decision logic, use `ConditionGroup` nodes
- Reference knowledge sources for detailed content

**b) Knowledge file:**
- Export the skill's reference material as markdown files in `knowledge/files/`
- These become uploadable knowledge sources in Copilot Studio

### 3. Command → Topic + Optional Flow

Each command becomes a topic. If the command performs actions (writes files, runs commands), also generate a Power Automate flow.

**Topic (`.mcs.yaml`):**
- Command `name` → topic display name
- Command `description` → topic description (used by generative orchestration)
- Command `argument-hint` → `Question` node to collect the argument
- Command instructions → sequence of `SendActivity` and action nodes

**Flow (`workflow.json`):**
- If the command calls external tools or APIs, generate a Power Automate flow
- Use the Logic Apps workflow definition schema
- Include `Run a flow from Copilot` trigger and `Respond to Copilot` action

### 4. Hook → Trigger Topic

Map Claude hook events to Copilot Studio trigger types:

| Claude Hook Event | Copilot Studio Trigger |
|---|---|
| `SessionStart` | `OnConversationStart` system topic |
| `UserPromptSubmit` | `OnMessageReceived` trigger topic |
| `PreToolUse` | `OnAIGeneratedResponseAboutToBeSent` or custom topic with conditions |
| `PostToolUse` | `OnPlanCompletes` trigger topic |
| `Stop` | `OnEndOfConversation` system topic |
| `Notification` | Event trigger (SharePoint, Dataverse, or scheduled) |

For hooks with validation logic, translate the validation into `ConditionGroup` nodes.

### 5. MCP Server → Custom Connector or REST API Tool

Each MCP server becomes either a custom connector or a REST API tool definition.

**Custom Connector (`apiDefinition.swagger.json`):**
- Generate OpenAPI 2.0 (Swagger) specification
- Map each MCP tool to a path/operation
- Tool parameters → operation parameters
- Tool responses → response schema
- Authentication from MCP env vars → security definitions

**REST API Tool (alternative):**
- For simpler integrations, generate an OpenAPI v2/v3 spec
- Users upload this directly via Tools > Add a tool > REST API

## YAML Template Reference

For detailed YAML templates, AdaptiveDialog node types, workflow.json schema, and OpenAPI connector templates, see the reference files:

- `references/topic-templates.md` — AdaptiveDialog YAML templates for all node types
- `references/workflow-templates.md` — Power Automate flow JSON templates
- `references/connector-templates.md` — OpenAPI 2.0 connector templates
- `references/gui-instructions-template.md` — GUI instruction format and field mappings

## GUI Instructions Format

For the GUI instruction format and field mapping reference, see `references/gui-instructions-template.md`.

## Edge Cases and Fallbacks

When a Claude component has no direct Copilot Studio equivalent:
- Add a `# MANUAL REVIEW NEEDED` comment in the generated output explaining the gap
- Suggest the closest alternative or a manual workaround
- Common gaps: Bash tool execution (suggest Power Automate desktop flows), file system access (suggest SharePoint/OneDrive connectors), git operations (suggest Azure DevOps connectors)

## Important Copilot Studio Constraints

When generating output, enforce these rules:
- Agent names: max 42 characters, no angle brackets
- Topic YAML: 2-space indentation, unique IDs per node
- `Question` nodes require `interruptionPolicy: { allowInterruption: true }` and `init:` prefix on new variables
- Entity types: `StringPrebuiltEntity`, `BooleanPrebuiltEntity`, `NumberPrebuiltEntity`, `DateTimePrebuiltEntity`
- Variables referenced in activity text use `{Variable.Name}` syntax
- Power Fx expressions prefixed with `=`
- Custom connectors require OpenAPI 2.0 (not 3.0)
- Knowledge sources: max 500 objects per agent, files up to 512 MB
