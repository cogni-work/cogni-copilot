# cogni-copilot

A Claude Code plugin that ports Claude Code plugins to Microsoft Copilot Studio. It reads your plugin's agents, skills, commands, hooks, and MCP servers, then generates equivalent Copilot Studio agent configs, topic YAML, Power Automate flow JSON, OpenAPI 2.0 connector definitions, and step-by-step GUI instructions.

## Installation

```bash
git clone https://github.com/anthropics/cogni-copilot.git
claude plugin add /path/to/cogni-copilot
```

## Usage

```
/port-plugin /path/to/your-claude-plugin
```

This analyzes the source plugin and generates a `copilot-studio-output/` directory in your current working directory.

## Output Structure

```
copilot-studio-output/
├── agent.mcs.yaml              # Agent identity and instructions
├── settings.mcs.yml            # Agent settings
├── connectionreferences.mcs.yml
├── GUI-INSTRUCTIONS.md         # Step-by-step manual setup guide
├── topics/                     # Conversation topics (.mcs.yaml)
│   ├── <skill-name>.mcs.yaml
│   └── <command-name>.mcs.yaml
├── trigger/                    # Event trigger topics
│   └── <hook-name>.mcs.yaml
├── workflows/                  # Power Automate flows
│   └── <flow-name>/
│       ├── metadata.yaml
│       └── workflow.json
├── actions/                    # Custom connectors (OpenAPI 2.0)
│   └── <connector-name>.mcs.yml
└── knowledge/files/            # Uploadable knowledge documents
```

## Component Mapping

| Claude Code | Copilot Studio |
|---|---|
| Agent (system prompt, tools) | Agent (instructions, tools, knowledge) |
| Skill (SKILL.md + references) | Topic + Knowledge files |
| Command (frontmatter + instructions) | Topic + optional Power Automate flow |
| Hook (event handler) | Event trigger topic |
| MCP server (tool provider) | Custom connector (OpenAPI 2.0) |

## License

AGPL-3.0 — see [LICENSE](LICENSE).
