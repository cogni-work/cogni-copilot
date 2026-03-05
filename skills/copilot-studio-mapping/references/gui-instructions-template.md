# GUI Instructions Template

Use this format when generating step-by-step GUI instructions for `GUI-INSTRUCTIONS.md`.

## GUI-INSTRUCTIONS.md Structure

```markdown
# Copilot Studio Porting Guide

**Source plugin**: [plugin name] (`/path/to/source`)
**Generated**: [date]
**Components**: [count] topics, [count] flows, [count] connectors

---

## Prerequisites

1. Access to Microsoft Copilot Studio (https://copilotstudio.microsoft.com)
2. An existing agent OR create a new one:
   - Go to **Agents** > **New agent**
   - Name: [suggested name from source plugin]
   - Description: [from plugin.json description]
3. [Any additional prerequisites like Power Automate license, connectors]

---

## Step 1: Configure Agent Instructions

**Source**: Agent system prompt from `agents/[name].md`
**Target**: Copilot Studio > **Overview** > **Instructions**

### Steps:
1. Open your agent in Copilot Studio
2. Click the **Overview** tab
3. Find the **Instructions** text area
4. Paste the following instructions:

### Content:
\```
[Translated agent instructions — Claude-specific references removed,
Copilot Studio terminology used]
\```

### Verify:
- Open the **Test your agent** panel (bottom-right)
- Send a greeting message
- Confirm the agent responds according to instructions

---

## Step 2: Add Knowledge Sources

**Source**: Skill reference files from `skills/[name]/references/`
**Target**: Copilot Studio > **Knowledge** tab

### Steps:
1. Navigate to the **Knowledge** tab
2. Click **Add knowledge** > **Upload files**
3. Upload the following files from `knowledge/files/`:
   - `[filename1].md` — [description]
   - `[filename2].md` — [description]
4. Wait for indexing to complete (may take a few minutes)

### Verify:
- In the test panel, ask a question that requires knowledge from the uploaded files
- Confirm the agent references the knowledge correctly

---

## Step 3: Create Topics

**Source**: Skills/commands from `skills/` and `commands/`
**Target**: Copilot Studio > **Topics** tab

### For each topic:

#### Topic: [Topic Name]
**Source**: `[source component path]`

**Option A — Code Editor (recommended):**
1. Navigate to **Topics** tab
2. Click **+ New topic** > **From blank**
3. Click the **⋯** menu (top-right of canvas) > **Open code editor**
4. Replace all content with the YAML below
5. Click **Save**

**Option B — VS Code Extension:**
1. Clone your agent with the Copilot Studio VS Code extension
2. Copy `topics/[name].mcs.yaml` into the `topics/` folder
3. Sync changes back to Copilot Studio

**YAML content:**
\```yaml
[Full .mcs.yaml content for this topic]
\```

### Verify:
- In the test panel, type a trigger phrase
- Confirm the topic activates and flows correctly

---

## Step 4: Set Up Flows (if applicable)

**Source**: Commands with tool usage from `commands/`
**Target**: Copilot Studio > **Tools** tab > Power Automate flows

### Flow: [Flow Name]
**Source**: `[source component path]`

### Steps:
1. Navigate to the **Tools** tab
2. Click **Add a tool** > **Power Automate flow** > **New flow**
3. In the flow designer:
   a. The trigger "Run a flow from Copilot" is auto-added
   b. Add input parameters: [list params]
   c. Add the actions described below
   d. Add "Respond to Copilot" action with output parameters
4. Save and publish the flow
5. Return to Copilot Studio — the flow appears as a tool

### Flow definition (for reference):
\```json
[workflow.json content]
\```

---

## Step 5: Add Custom Connectors (if applicable)

**Source**: MCP servers from `.mcp.json`
**Target**: Copilot Studio > **Tools** tab > Custom connectors

### Connector: [Connector Name]
**Source**: MCP server `[server-name]`

**Option A — Via Copilot Studio UI:**
1. Navigate to **Tools** tab
2. Click **Add a tool** > **New tool** > **Custom connector**
3. Choose **Import from file**
4. Upload `actions/[name]/apiDefinition.swagger.json`
5. Configure authentication:
   - Type: [API Key / OAuth / Basic]
   - [Specific auth configuration steps]
6. Click **Create connector**

**Option B — Via paconn CLI:**
\```bash
pip install paconn
cd actions/[name]
paconn validate --api-def apiDefinition.swagger.json
paconn create --api-prop apiProperties.json --api-def apiDefinition.swagger.json
\```

### Verify:
- In the **Tools** tab, confirm the connector appears
- Test an operation from the connector details page

---

## Step 6: Configure Triggers (if applicable)

**Source**: Hooks from `hooks/hooks.json`
**Target**: Copilot Studio > **Topics** tab (system topics or trigger topics)

### [Trigger description]
1. Navigate to **Topics** tab
2. [For system topics]: Find and edit the relevant system topic
3. [For event triggers]: Click **+ New topic** > **From blank** > change trigger type
4. Apply the YAML from `trigger/[name].mcs.yaml`

---

## Step 7: Test End-to-End

1. Open the **Test your agent** panel
2. Run through each scenario:
   - [ ] [Test scenario 1 — description]
   - [ ] [Test scenario 2 — description]
   - [ ] [Test scenario 3 — description]
3. Check **Analytics** tab for any errors

## Step 8: Publish

1. Click **Publish** (top-right)
2. Review the publish summary
3. Confirm publication
4. Configure channels if needed (**Channels** tab)
```

## Copilot Studio GUI Field Reference

### Overview Tab
| Field | Description | Maps From |
|---|---|---|
| Name | Agent display name (max 42 chars) | plugin.json `name` |
| Description | Agent purpose | plugin.json `description` |
| Instructions | Natural language behavior instructions | Agent system prompt |
| Suggested prompts | Up to 10 starter prompts | Command descriptions |

### Topics Tab
| Element | Description | Maps From |
|---|---|---|
| Topic name | Display name for the topic | Skill/command `name` |
| Description | Used by generative orchestrator | Skill/command `description` |
| Trigger phrases | 5-10 phrases (classic mode) | Skill description keywords |
| Code editor | YAML view of topic | Generated `.mcs.yaml` |

### Tools Tab
| Element | Description | Maps From |
|---|---|---|
| Power Automate flows | Cloud flows callable by agent | Commands with side effects |
| Custom connectors | OpenAPI-based integrations | MCP servers |
| REST API tools | Direct API connections | MCP servers (simple) |

### Knowledge Tab
| Element | Description | Maps From |
|---|---|---|
| Uploaded files | PDF, DOCX, MD up to 512 MB | Skill reference files |
| Websites | Bing-indexed public sites | N/A |
| SharePoint | SharePoint sites/libraries | N/A |
