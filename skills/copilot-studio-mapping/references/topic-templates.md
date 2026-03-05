# Copilot Studio Topic YAML Templates

All topics use the AdaptiveDialog YAML format. Files use `.mcs.yaml` extension.

## Minimal Topic (Generative Orchestration)

```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Topic Name
    description: When to invoke this topic — used by the orchestrator to decide
  actions:
    - kind: SendActivity
      id: sendActivity_intro
      activity: "Hello! I can help you with [topic purpose]."
```

## Topic with Trigger Phrases (Classic Orchestration)

```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Topic Name
    triggerQueries:
      - "trigger phrase one"
      - "trigger phrase two"
      - "trigger phrase three"
      - "trigger phrase four"
      - "trigger phrase five"
  actions:
    - kind: SendActivity
      id: sendActivity_intro
      activity: "Welcome to [topic name]."
```

## Question Node — Collect User Input

```yaml
- kind: Question
  id: question_getUserInput
  interruptionPolicy:
    allowInterruption: true
  variable: init:Topic.UserInput
  entity: StringPrebuiltEntity
  prompt: "Please provide the input value:"
```

### Entity Types

| Type | Use For |
|---|---|
| `StringPrebuiltEntity` | Free text input |
| `BooleanPrebuiltEntity` | Yes/no confirmation |
| `NumberPrebuiltEntity` | Numeric values |
| `DateTimePrebuiltEntity` | Dates and times |
| `ChoicePrebuiltEntity` | Selection from options |

## Question with Multiple Choice

```yaml
- kind: Question
  id: question_selectOption
  interruptionPolicy:
    allowInterruption: true
  variable: init:Topic.SelectedOption
  entity:
    kind: EmbeddedEntity
    definition:
      kind: ClosedListEntity
      items:
        - id: option1
          displayName: Option One
        - id: option2
          displayName: Option Two
        - id: option3
          displayName: Option Three
  prompt: "Which option would you like?"
```

## Condition Group — Branching Logic

```yaml
- kind: ConditionGroup
  id: conditionGroup_checkValue
  conditions:
    - id: condition_isHigh
      condition: =Topic.Score > 80
      actions:
        - kind: SendActivity
          id: sendActivity_highScore
          activity: "Great score! You passed with {Topic.Score} points."
    - id: condition_isMedium
      condition: =Topic.Score > 50
      actions:
        - kind: SendActivity
          id: sendActivity_mediumScore
          activity: "You scored {Topic.Score}. Room for improvement."
  defaultActions:
    - kind: SendActivity
      id: sendActivity_lowScore
      activity: "Score is {Topic.Score}. Let's work on improving that."
```

## Set Variable — Assign Values

```yaml
- kind: SetVariable
  id: setVariable_buildMessage
  variable: Topic.OutputMessage
  value: =Concatenate("Result for ", Topic.UserInput, ": ", Topic.Result)
```

## Call a Tool / Flow

```yaml
- kind: InvokeFlowAction
  id: invokeFlow_processData
  input:
    binding:
      inputParam: =Topic.UserInput
  output:
    binding:
      outputParam: Topic.FlowResult
  flowId: <flow-id-placeholder>
```

## Generative Answers Node

```yaml
- kind: SearchAndSummarizeContent
  id: searchAndSummarize_knowledge
  userMessage: =Topic.UserQuestion
```

## HTTP Request (Advanced)

```yaml
- kind: HttpRequest
  id: httpRequest_callApi
  method: GET
  url: "https://api.example.com/data"
  headers:
    Authorization: "Bearer {Global.ApiToken}"
  responseVariable: Topic.ApiResponse
```

## Redirect to Another Topic

```yaml
- kind: GotoTopic
  id: gotoTopic_escalate
  topicId: <target-topic-id>
  input:
    binding:
      paramName: =Topic.CurrentValue
```

## End Topic / Conversation

```yaml
- kind: EndDialog
  id: endDialog_finish
  clearTopicQueue: true
```

## Topic with Input/Output Parameters

```yaml
kind: AdaptiveDialog
modelDescription: Process a user request and return results
inputParameters:
  - name: InputData
    dataType: String
    description: The data to process
outputParameters:
  - name: ProcessedResult
    dataType: String
    description: The processed output
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Process Data
    description: Processes input data and returns results
  actions:
    - kind: SetVariable
      id: setVariable_process
      variable: Topic.ProcessedResult
      value: =Concatenate("Processed: ", Topic.InputData)
```

## Event Trigger Topic

```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnActivity
  id: main
  activityType: event
  eventName: "custom_event_name"
  actions:
    - kind: SendActivity
      id: sendActivity_eventReceived
      activity: "Event received and processed."
```

## Conversation Start Topic

```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnConversationStart
  id: main
  actions:
    - kind: SendActivity
      id: sendActivity_greeting
      activity: "Hello! I'm your assistant. How can I help you today?"
```

## Full Example — Command-to-Topic Translation

Source Claude command: `deploy` with argument `environment`

```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Deploy Application
    description: Deploy the application to a target environment
    triggerQueries:
      - "deploy application"
      - "deploy to production"
      - "deploy to staging"
      - "start deployment"
      - "release to environment"
  actions:
    - kind: SendActivity
      id: sendActivity_intro
      activity: "I'll help you deploy the application."

    - kind: Question
      id: question_environment
      interruptionPolicy:
        allowInterruption: true
      variable: init:Topic.Environment
      entity:
        kind: EmbeddedEntity
        definition:
          kind: ClosedListEntity
          items:
            - id: staging
              displayName: Staging
            - id: production
              displayName: Production
      prompt: "Which environment should I deploy to?"

    - kind: ConditionGroup
      id: conditionGroup_confirmProd
      conditions:
        - id: condition_isProd
          condition: =Topic.Environment = "production"
          actions:
            - kind: Question
              id: question_confirmProd
              interruptionPolicy:
                allowInterruption: true
              variable: init:Topic.Confirmed
              entity: BooleanPrebuiltEntity
              prompt: "You selected Production. Are you sure you want to proceed?"
            - kind: ConditionGroup
              id: conditionGroup_confirmed
              conditions:
                - id: condition_notConfirmed
                  condition: =Topic.Confirmed = false
                  actions:
                    - kind: SendActivity
                      id: sendActivity_cancelled
                      activity: "Deployment cancelled."
                    - kind: EndDialog
                      id: endDialog_cancelled
                      clearTopicQueue: true

    - kind: InvokeFlowAction
      id: invokeFlow_deploy
      input:
        binding:
          environment: =Topic.Environment
      output:
        binding:
          deploymentStatus: Topic.DeployStatus
      flowId: <deploy-flow-id>

    - kind: SendActivity
      id: sendActivity_result
      activity: "Deployment to {Topic.Environment} completed. Status: {Topic.DeployStatus}"
```
