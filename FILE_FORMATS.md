# File formats

PatriaBridge uses two JSON files in its Geyser extension data folder.

## `patriabridge-entities.json`

This is the static mapping manifest.

```json
{
  "entities": [
    {
      "model_id": "minero_zombie",
      "bedrock_identifier": "patriacraft:minero_zombie",
      "java_entity_type": "minecraft:zombie"
    }
  ]
}
```

Fields:

| Field | Use |
| --- | --- |
| `model_id` | Internal id used by PatriaBridge and runtime bindings |
| `bedrock_identifier` | Bedrock/Geyser custom entity identifier |
| `java_entity_type` | Optional Java type fallback |

Entries with an empty model id or Bedrock identifier are ignored.

The first mapping registered for a Java entity type is used as that type's fallback.

## `runtime-entities.json`

This file is generated/updated by the server-side bridge part and maps live UUIDs to model ids.

```json
{
  "bindings": {
    "d67aef61-d6e4-4f98-a20c-e95ce3a47af7": "minero_zombie"
  }
}
```

Invalid UUID keys are ignored instead of stopping the whole load.

PatriaBridge checks the file modification time before reparsing it. This matters because the spawn event can fire a lot and reading the JSON from disk every time would be unnecessary work.
