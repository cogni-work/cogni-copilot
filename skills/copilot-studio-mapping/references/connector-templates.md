# Custom Connector Templates (OpenAPI 2.0)

Generate these files for each MCP server that needs to become a Copilot Studio connector.

## apiDefinition.swagger.json — Full Template

```json
{
  "swagger": "2.0",
  "info": {
    "title": "Connector Name",
    "description": "Description of what this connector does",
    "version": "1.0.0"
  },
  "host": "api.example.com",
  "basePath": "/v1",
  "schemes": ["https"],
  "consumes": ["application/json"],
  "produces": ["application/json"],
  "paths": {
    "/resource": {
      "get": {
        "operationId": "GetResource",
        "summary": "Get a resource",
        "description": "Retrieves a resource by query",
        "parameters": [
          {
            "name": "query",
            "in": "query",
            "required": true,
            "type": "string",
            "description": "The search query"
          }
        ],
        "responses": {
          "200": {
            "description": "Success",
            "schema": {
              "$ref": "#/definitions/ResourceResponse"
            }
          },
          "400": {
            "description": "Bad request"
          }
        }
      },
      "post": {
        "operationId": "CreateResource",
        "summary": "Create a resource",
        "description": "Creates a new resource",
        "parameters": [
          {
            "name": "body",
            "in": "body",
            "required": true,
            "schema": {
              "$ref": "#/definitions/CreateResourceRequest"
            }
          }
        ],
        "responses": {
          "201": {
            "description": "Created",
            "schema": {
              "$ref": "#/definitions/ResourceResponse"
            }
          }
        }
      }
    },
    "/resource/{id}": {
      "get": {
        "operationId": "GetResourceById",
        "summary": "Get resource by ID",
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "type": "string",
            "description": "Resource ID"
          }
        ],
        "responses": {
          "200": {
            "description": "Success",
            "schema": {
              "$ref": "#/definitions/ResourceResponse"
            }
          }
        }
      }
    }
  },
  "definitions": {
    "ResourceResponse": {
      "type": "object",
      "properties": {
        "id": { "type": "string" },
        "name": { "type": "string" },
        "data": { "type": "object" }
      }
    },
    "CreateResourceRequest": {
      "type": "object",
      "required": ["name"],
      "properties": {
        "name": { "type": "string" },
        "data": { "type": "object" }
      }
    }
  },
  "securityDefinitions": {
    "api_key": {
      "type": "apiKey",
      "in": "header",
      "name": "X-API-Key"
    }
  },
  "security": [
    { "api_key": [] }
  ]
}
```

## apiProperties.json — Full Template

```json
{
  "properties": {
    "capabilities": [],
    "connectionParameters": {
      "api_key": {
        "type": "securestring",
        "uiDefinition": {
          "constraints": {
            "clearText": false,
            "required": "true",
            "tabIndex": 2
          },
          "description": "The API key for this service",
          "displayName": "API Key",
          "tooltip": "Provide your API key"
        }
      }
    },
    "iconBrandColor": "#007EE6",
    "policyTemplateInstances": []
  }
}
```

## MCP Tool → OpenAPI Operation Mapping

For each MCP tool, create one path + operation:

| MCP Tool Property | OpenAPI Field |
|---|---|
| Tool name | `operationId` (PascalCase) |
| Tool description | `summary` + `description` |
| Input schema properties | `parameters` (query/body) |
| Output schema | `responses.200.schema` |

### Translation Rules

1. **Tool name** → `operationId` in PascalCase: `get_user` → `GetUser`
2. **Required inputs** → `parameters` with `required: true`
3. **Optional inputs** → `parameters` with `required: false`
4. **Complex input objects** → `in: body` with `$ref` to definitions
5. **Simple string/number inputs** → `in: query` parameters
6. **Tool output** → `responses.200.schema` with `$ref` to definitions

### Authentication Mapping

| MCP Env Var Pattern | OpenAPI Security |
|---|---|
| `*_API_KEY` | `apiKey` in header |
| `*_TOKEN` | `apiKey` in header (as Bearer token) |
| `*_USERNAME` + `*_PASSWORD` | `basic` authentication |
| OAuth-related vars | `oauth2` with appropriate flow |

## Important Constraints

- **Must be OpenAPI 2.0** (Swagger) — OpenAPI 3.0 is NOT supported by Power Platform custom connectors
- `host` must be a publicly accessible HTTPS endpoint
- Max 500 operations per connector
- Operation IDs must be unique within the connector
- Use `paconn validate` to verify before importing
