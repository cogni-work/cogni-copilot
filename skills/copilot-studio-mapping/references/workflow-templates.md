# Power Automate Workflow JSON Templates

Flows use the Azure Logic Apps workflow definition schema. Each flow lives in `workflows/<FlowName>/` with `metadata.yaml` and `workflow.json`.

## Metadata File

```yaml
# workflows/<FlowName>/metadata.yaml
displayName: Flow Display Name
description: What this flow does
```

## Minimal Flow — Copilot Studio Trigger + Response

```json
{
  "properties": {
    "definition": {
      "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "triggers": {
        "Run_a_flow_from_Copilot": {
          "type": "Request",
          "kind": "ApiConnection",
          "inputs": {
            "schema": {
              "type": "object",
              "properties": {
                "inputParam": {
                  "type": "string",
                  "description": "Input from the agent"
                }
              },
              "required": ["inputParam"]
            }
          }
        }
      },
      "actions": {
        "Respond_to_Copilot": {
          "type": "Response",
          "kind": "ApiConnection",
          "inputs": {
            "statusCode": 200,
            "body": {
              "outputParam": "Result value"
            },
            "schema": {
              "type": "object",
              "properties": {
                "outputParam": {
                  "type": "string",
                  "description": "Output back to the agent"
                }
              }
            }
          },
          "runAfter": {}
        }
      }
    }
  }
}
```

## Flow with HTTP Action (API Call)

Use this when a Claude command calls external APIs.

```json
{
  "properties": {
    "definition": {
      "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "triggers": {
        "Run_a_flow_from_Copilot": {
          "type": "Request",
          "kind": "ApiConnection",
          "inputs": {
            "schema": {
              "type": "object",
              "properties": {
                "query": {
                  "type": "string",
                  "description": "The search query or input"
                }
              },
              "required": ["query"]
            }
          }
        }
      },
      "actions": {
        "Call_External_API": {
          "type": "Http",
          "inputs": {
            "method": "GET",
            "uri": "https://api.example.com/search",
            "queries": {
              "q": "@triggerBody()?['query']"
            },
            "headers": {
              "Authorization": "Bearer @{parameters('apiKey')}"
            }
          },
          "runAfter": {}
        },
        "Parse_Response": {
          "type": "ParseJson",
          "inputs": {
            "content": "@body('Call_External_API')",
            "schema": {
              "type": "object",
              "properties": {
                "results": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "title": { "type": "string" },
                      "value": { "type": "string" }
                    }
                  }
                }
              }
            }
          },
          "runAfter": {
            "Call_External_API": ["Succeeded"]
          }
        },
        "Respond_to_Copilot": {
          "type": "Response",
          "kind": "ApiConnection",
          "inputs": {
            "statusCode": 200,
            "body": {
              "result": "@body('Parse_Response')?['results']"
            },
            "schema": {
              "type": "object",
              "properties": {
                "result": {
                  "type": "string",
                  "description": "API results"
                }
              }
            }
          },
          "runAfter": {
            "Parse_Response": ["Succeeded"]
          }
        }
      }
    }
  }
}
```

## Flow with Condition (Branching)

Use when a Claude command has conditional logic.

```json
{
  "properties": {
    "definition": {
      "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "triggers": {
        "Run_a_flow_from_Copilot": {
          "type": "Request",
          "kind": "ApiConnection",
          "inputs": {
            "schema": {
              "type": "object",
              "properties": {
                "action": { "type": "string" },
                "target": { "type": "string" }
              },
              "required": ["action"]
            }
          }
        }
      },
      "actions": {
        "Check_Action_Type": {
          "type": "If",
          "expression": {
            "and": [
              {
                "equals": [
                  "@triggerBody()?['action']",
                  "deploy"
                ]
              }
            ]
          },
          "actions": {
            "Deploy_Action": {
              "type": "Http",
              "inputs": {
                "method": "POST",
                "uri": "https://api.example.com/deploy",
                "body": {
                  "target": "@triggerBody()?['target']"
                }
              },
              "runAfter": {}
            }
          },
          "else": {
            "actions": {
              "Default_Action": {
                "type": "Compose",
                "inputs": "Unknown action: @{triggerBody()?['action']}",
                "runAfter": {}
              }
            }
          },
          "runAfter": {}
        },
        "Respond_to_Copilot": {
          "type": "Response",
          "kind": "ApiConnection",
          "inputs": {
            "statusCode": 200,
            "body": {
              "status": "completed"
            },
            "schema": {
              "type": "object",
              "properties": {
                "status": { "type": "string" }
              }
            }
          },
          "runAfter": {
            "Check_Action_Type": ["Succeeded"]
          }
        }
      }
    }
  }
}
```

## Flow with Loop (For Each)

Use when a Claude command processes multiple items.

```json
{
  "properties": {
    "definition": {
      "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "triggers": {
        "Run_a_flow_from_Copilot": {
          "type": "Request",
          "kind": "ApiConnection",
          "inputs": {
            "schema": {
              "type": "object",
              "properties": {
                "items": {
                  "type": "array",
                  "items": { "type": "string" }
                }
              },
              "required": ["items"]
            }
          }
        }
      },
      "actions": {
        "Initialize_Results": {
          "type": "InitializeVariable",
          "inputs": {
            "variables": [
              {
                "name": "results",
                "type": "array",
                "value": []
              }
            ]
          },
          "runAfter": {}
        },
        "Process_Each_Item": {
          "type": "Foreach",
          "foreach": "@triggerBody()?['items']",
          "actions": {
            "Append_Result": {
              "type": "AppendToArrayVariable",
              "inputs": {
                "name": "results",
                "value": "Processed: @{items('Process_Each_Item')}"
              }
            }
          },
          "runAfter": {
            "Initialize_Results": ["Succeeded"]
          }
        },
        "Respond_to_Copilot": {
          "type": "Response",
          "kind": "ApiConnection",
          "inputs": {
            "statusCode": 200,
            "body": {
              "processedItems": "@variables('results')"
            },
            "schema": {
              "type": "object",
              "properties": {
                "processedItems": { "type": "array" }
              }
            }
          },
          "runAfter": {
            "Process_Each_Item": ["Succeeded"]
          }
        }
      }
    }
  }
}
```

## Key Rules for Valid Flows

1. **Every flow for Copilot Studio must have**: `Run_a_flow_from_Copilot` trigger + `Respond_to_Copilot` action
2. **Execution order**: Determined by `runAfter` property — NOT array position
3. **Expression syntax**: Use `@triggerBody()`, `@body('ActionName')`, `@variables('varName')`
4. **Actions are an object**: Each action is a named key, not an array element
5. **Nested actions**: `If`, `Foreach`, `Scope` contain nested `actions` objects
