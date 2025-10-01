# Workflow Object MCP Documentation

## Overview
This document describes the structure and properties of a workflow object based on BPMN 2.0 standards with system-specific extensions.

## Root Properties

### `name` (required)
- **Type**: `string`
- **Description**: Describes the name of the workflow
- **Constraints**:
  - Cannot be null or undefined
  - No length limits
  - Does not need to be unique
- **Example**: `"The API Whisperer"`

### `id` (required)
- **Type**: `string`
- **Description**: Unique identifier for the workflow
- **Constraints**: Auto-generated once created
- **Example**: `"68dbed66cb40e3f639272753"`

### `organizationId` (required)
- **Type**: `string`
- **Description**: Identifier of the organization this workflow belongs to
- **Example**: `"584ec417d9383b68ed2cd560"`

## Activities Array

### `activities` (required)
- **Type**: `Array<Activity>`
- **Description**: Contains BPMN 2.0 activity elements that define the workflow steps
- **Constraints**:
  - Can be empty array
  - Cannot be null or undefined
  - Contains objects representing different activity types

#### Activity Types

#### StartEvent
Based on BPMN 2.0 standard for workflow initiation points.

```json
{
  "type": "startEvent",
  "name": "",
  "id": "auto-generated-id"
}
```

**Properties:**
- `type`: Always `"startEvent"`
- `name`: Optional string (recommended for clarity of starting point)
- `id`: Auto-generated unique identifier
- **Constraints**:
  - Can only have outgoing transitions, no incoming
  - Can only have one outgoing transition
- **Variants**: Basic start events, intermediate timer events, intermediate link events

#### UserTask
Represents human-performed activities based on BPMN 2.0.

```json
{
  "type": "userTask",
  "candidateIds": [],
  "candidateGroupIds": [],
  "candidateExpressions": [],
  "form": {
    "fields": [
      {
        "elementType": "fieldDefinition",
        "id": "auto-generated-id",
        "asButtons": true,
        "binding": {
          "expression": "variable-id"
        },
        "parameters": {}
      }
    ]
  },
  "escalateToIds": [],
  "escalateToGroupIds": [],
  "escalateToVariableExpressions": [],
  "name": "task-name",
  "id": "auto-generated-id"
}
```

**Properties:**
- `type`: Always `"userTask"`
- `name`: String name of the task
- `id`: Auto-generated unique identifier
- `candidateIds`: Array of user ID strings for assignment options (users must be known to organization)
- `candidateGroupIds`: Array of group ID strings that resolve to users for assignment
- `candidateExpressions`: String referencing a user variable from the variables array
- `form.fields`: Array of form field definitions
  - `elementType`: Type of form element (e.g., `"fieldDefinition"`)
  - `asButtons`: Boolean - when `true`, renders buttons at bottom of form (can only appear before gateways)
  - `binding.expression`: References a variable ID for data binding
  - `parameters`: Object for additional field configuration
- `escalateToIds`: Array for task reassignment after timeouts
- `escalateToGroupIds`: Array for group escalation after timeouts
- `escalateToVariableExpressions`: Array for variable-based escalation

**Note**: Multiple form field types available beyond choice/button fields (reminder: provide form field example later).

#### ExclusiveGateway
Based on BPMN 2.0 XOR gateway for routing decisions.

```json
{
  "type": "exclusiveGateway",
  "name": "gateway-name",
  "id": "auto-generated-id",
  "decisionType": "manual"
}
```

**Properties:**
- `type`: Always `"exclusiveGateway"`
- `name`: String name of the gateway
- `id`: Auto-generated unique identifier
- `decisionType`:
  - `"manual"`: User assigned to previous userTask decides by button click
  - `"automatic"`: Decision computed based on transition conditions
  - `"notset"`: (Skip for documentation purposes)

#### EndEvent
Based on BPMN 2.0 standard for workflow termination points.

```json
{
  "type": "endEvent",
  "name": "",
  "id": "auto-generated-id"
}
```

**Properties:**
- `type`: Always `"endEvent"`
- `name`: Optional string
- `id`: Auto-generated unique identifier
- **Variants**: Basic end events, intermediate timer events, intermediate link events

## Transitions Array

### `transitions` (required)
- **Type**: `Array<Transition>`
- **Description**: Defines the flow connections between activities
- **Constraints**: For logical workflows, all transitions require `fromId`, `toId`, and `id`

#### Basic Transition
```json
{
  "id": "auto-generated-id",
  "fromId": "source-activity-id",
  "toId": "target-activity-id"
}
```

#### Conditional Transition
```json
{
  "id": "auto-generated-id",
  "fromId": "gateway-id",
  "toId": "target-activity-id",
  "condition": {
    "type": "equals",
    "left": {
      "expression": "variable-id"
    },
    "right": {
      "type": {
        "name": "choice",
        "options": [
          {
            "id": "option-id",
            "name": "option-name",
            "deleted": false
          }
        ]
      },
      "value": "option-id"
    }
  }
}
```

**Properties:**
- `id`: Auto-generated unique identifier
- `fromId`: Source activity ID
- `toId`: Target activity ID
- `condition`: Optional condition object for exclusive gateways
  - `type`: Condition type (e.g., `"equals"`)
  - `left.expression`: References a variable ID
  - `right.value`: References a choice option ID
  - `right.type`: Contains full choice definition with options

**Behavior:**
- Conditions evaluate if variable value equals specified choice option
- If no condition matches on exclusive gateway, workflow stops
- Multiple condition types may be available beyond "equals"

## Variables Array

### `variables` (required)
- **Type**: `Array<Variable>`
- **Description**: Defines workflow variables used in conditions and forms

#### Choice Variable
```json
{
  "id": "auto-generated-id",
  "name": "variable-name",
  "type": {
    "name": "choice",
    "options": [
      {
        "id": "auto-generated-id",
        "name": "option-name",
        "deleted": false
      }
    ]
  },
  "isManualDecision": true
}
```

**Properties:**
- `id`: Auto-generated unique identifier
- `name`: Variable name
- `type.name`: Type of variable (e.g., `"choice"`)
- `type.options`: Array of choice options with ID, name, and deletion status
- `isManualDecision`: Boolean indicating if this is used for manual gateway decisions

## Metadata Properties

### Timestamps and Ownership
- `lastUpdated`: ISO timestamp of last modification
- `editorId`: ID of user who last edited
- `editorLock`: ISO timestamp of edit lock
- `ownerId`: ID of workflow owner
- `changed`: Boolean indicating if workflow has unsaved changes
- `published`: Boolean indicating publication status

### User Objects
- `editor`: User object with `id`, `firstName`, `lastName`, `emailAddress`, `groupIds`, `organizationIds`
- `owner`: User object with same structure as editor

### Organization and Permissions
- `involvedUserIds`: Array of user IDs involved in workflow
- `involvedGroupIds`: Array of group IDs involved in workflow
- `enableCases`: Boolean for case management features
- `labelIds`: Array of label identifiers
- `labels`: Array of label objects

### Case Management
- `caseColumns`: Array of case column definitions with binding expressions
  - Standard bindings include: `"case.creatorId"`, `"case.milestone"`, `"case.dueDate"`

### Core Information
- `coreInformation.fields`: Array of core information fields (structure varies)

## Diagram Object

### `diagram` (required)
- **Type**: `Object`
- **Description**: Contains visual layout information for workflow diagram
- **Properties**:
  - `version`: Version number of diagram format
  - `canvas`: Canvas bounds and child element positioning
  - `edges`: Visual edge routing information between activities

#### Canvas Structure
```json
{
  "canvas": {
    "bounds": {
      "upperLeft": {"x": 40.0, "y": 108.0},
      "lowerRight": {"x": 881.03125, "y": 348.0}
    },
    "children": [
      {
        "id": "visual-element-id",
        "bounds": {
          "upperLeft": {"x": 40.0, "y": 213.0},
          "lowerRight": {"x": 70.0, "y": 243.0}
        },
        "children": [],
        "elementId": "activity-id"
      }
    ]
  }
}
```

#### Edge Routing
```json
{
  "edges": [
    {
      "id": "visual-edge-id",
      "dockers": [
        {"x": 15.0, "y": 15.0},
        {"x": 50.0, "y": 40.0}
      ],
      "fromId": "source-visual-element-id",
      "toId": "target-visual-element-id",
      "transitionId": "logical-transition-id"
    }
  ]
}
```

## Schema Constraints

- All ID fields are auto-generated upon creation
- Arrays can be empty but not null/undefined
- Required string fields cannot be null/undefined
- Visual diagram coordinates use floating-point precision
- ISO timestamps follow standard format
- Email addresses in user objects should be valid format

## Workflow Execution Behavior

- Follows BPMN 2.0 token-based execution semantics
- Start events initiate workflow execution
- User tasks pause execution until completion
- Exclusive gateways route based on conditions or manual decisions
- End events terminate workflow execution
- Failed condition matching stops workflow execution
