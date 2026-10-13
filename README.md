# A2A Skill Schemas extension

An [A2A](https://a2a-protocol.org) extension that declares, in the Agent Card, the **JSON Schema of the structured
data each skill takes and returns**.

```
URI: https://github.com/ive-spec/a2a-skill-schemas
```

## Why

An A2A skill says which media types it accepts and produces (`inputModes`, `outputModes`), for example
`application/json`, but not what that JSON looks like. A client that wants to send a `DataPart` to a skill, or read
the `DataPart` it returns, has to guess or read the documentation.

This extension gives each skill an input schema and an output schema, so a client can build the request, validate it,
generate a form or code, or check the response, before talking to the agent.

It is a **data-only extension**: it only adds information to the Agent Card. It does not change messages, tasks or
methods, so it needs no activation (`A2A-Extensions` header) and an agent or client that does not know it can ignore
it.

## Declaring it

The agent lists it in `capabilities.extensions` of its Agent Card, with `required: false`:

```json
{
  "name": "Checkout Agent",
  "capabilities": {
    "extensions": [
      {
        "uri": "https://github.com/ive-spec/a2a-skill-schemas",
        "description": "JSON Schema of the data parts of each skill.",
        "required": false,
        "params": {
          "skills": {
            "buyProduct": {
              "input": {
                "name": "PlaceOrderInput",
                "schema": {
                  "$schema": "https://json-schema.org/draft/2020-12/schema",
                  "type": "object",
                  "properties": {
                    "productId": { "type": "string" },
                    "quantity": { "type": "integer", "minimum": 1 }
                  },
                  "required": ["productId", "quantity"]
                }
              },
              "output": {
                "name": "Order",
                "schema": {
                  "$schema": "https://json-schema.org/draft/2020-12/schema",
                  "type": "object",
                  "properties": {
                    "orderId": { "type": "string" },
                    "total": { "type": "number" }
                  },
                  "required": ["orderId"]
                }
              }
            }
          }
        }
      }
    ]
  },
  "skills": [
    {
      "id": "buyProduct",
      "name": "Buy a product",
      "description": "Places an order.",
      "tags": ["shopping"],
      "inputModes": ["application/json"],
      "outputModes": ["application/json"]
    }
  ]
}
```

It works the same in A2A 1.0 and 0.3 cards: both have `capabilities.extensions` with `uri`, `description`,
`required` and `params`.

## params

| Field | Type | Description |
|---|---|---|
| `skills` | object | One entry per skill that has schemas. The key is the `id` of the skill in the same card. |
| `skills.<id>.input` | Schema entry | What the skill takes: the `data` of the `DataPart` the client sends. Optional. |
| `skills.<id>.output` | Schema entry | What the skill returns: the `data` of the `DataPart` in the artifacts or messages of the agent. Optional. |

A **schema entry** has:

| Field | Type | Description |
|---|---|---|
| `name` | string | Required. The name of the schema (the type or class it comes from). It lets a tool find the same type on its side, and it identifies the schema when it is omitted. |
| `schema` | object | Optional. A [JSON Schema 2020-12](https://json-schema.org/draft/2020-12) document. It should be self-contained: nested types in `$defs`, no `$ref` to other documents. |

Rules:

- A skill that is not in `skills` has no declared schema (it may still take or return JSON).
- A key that is not the `id` of a skill of the card is an error of the card; clients should ignore it.
- The schemas describe the `data` of `DataPart`s only. Text and file parts keep using `inputModes` and `outputModes`.
- The schemas are informative: the agent still validates what it receives and answers with A2A errors as usual.

## Schema of params

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "skills": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "properties": {
          "input": { "$ref": "#/$defs/entry" },
          "output": { "$ref": "#/$defs/entry" }
        },
        "additionalProperties": false
      }
    }
  },
  "required": ["skills"],
  "$defs": {
    "entry": {
      "type": "object",
      "properties": {
        "name": { "type": "string", "minLength": 1 },
        "schema": { "type": "object" }
      },
      "required": ["name"],
      "additionalProperties": false
    }
  }
}
```

## Versioning

This URI identifies version 1. Fields may be added in a compatible way (clients ignore what they do not know). An
incompatible change will get a new URI (`https://github.com/ive-spec/a2a-skill-schemas/v2`), so cards that declare
this one keep their meaning.

## Implementations

- **IA Agents** add-in for Sparx Enterprise Architect: generates the Agent Card of an agent modeled in EA, with this
  extension built from the input and output schemas of each skill (DataSchemas classes), and imports cards that have
  it back into the model.
