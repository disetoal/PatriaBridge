# PatriaBridge

PatriaBridge is a Geyser extension I made for PatriaCraft. I use it to connect custom Java-side entity/model information with the entity definitions Bedrock players need through Geyser.

I made this because some custom server content does not translate cleanly to Bedrock just by installing Geyser. The extension keeps a small mapping layer between my model ids, Java entity types and Bedrock identifiers.

## What it does

- registers custom Bedrock entity definitions during Geyser startup
- reads model mappings from `patriabridge-entities.json`
- reads live UUID -> model bindings from `runtime-entities.json`
- falls back to the Java entity type when there is no UUID binding
- reloads the runtime binding file only when it changes
- applies the right custom Geyser definition when an entity spawns
- leaves `furniture_*` item displays to the display/furniture path instead of forcing them through the normal entity mapping
- has its own small JSON reader so the extension does not need another JSON dependency for these files

## Version I currently use

- PatriaBridge: `3.0.0-beta.15`
- Geyser Extension API: `2.11.0`
- Java

The extension entry point is:

```text
net.patriacraft.bridge.geyser.PatriaBridgeExtension
```

## Code structure

The current build is intentionally small.

```text
net.patriacraft.bridge.geyser
├── PatriaBridgeExtension
│   └── EntityEntry
└── MiniJson
    └── Parser
```

`PatriaBridgeExtension` handles the Geyser lifecycle and entity mapping. `MiniJson` is a lightweight parser used for the two runtime JSON files.

A more detailed breakdown is in [docs/CODE_STRUCTURE.md](docs/CODE_STRUCTURE.md).

## Basic flow

```text
Geyser starts
   ↓
load patriabridge-entities.json
   ↓
register CustomEntityDefinition objects
   ↓
load runtime-entities.json
   ↓
Java entity spawns
   ↓
UUID binding found? ── yes ──> use mapped model id
       │
       no
       ↓
try Java entity type fallback
   ↓
find registered Bedrock definition
   ↓
apply definition to spawn event
```

## Files used by the extension

### `patriabridge-entities.json`

Defines the models that the extension knows about.

```json
{
  "entities": [
    {
      "model_id": "example_model",
      "bedrock_identifier": "patriacraft:example_model",
      "java_entity_type": "minecraft:zombie"
    }
  ]
}
```

### `runtime-entities.json`

Connects an entity UUID to one of those model ids while the server is running.

```json
{
  "bindings": {
    "00000000-0000-0000-0000-000000000000": "example_model"
  }
}
```

See [docs/FILE_FORMATS.md](docs/FILE_FORMATS.md) for the exact role of each field.

## About the source

I have not uploaded the full source or the compiled jar because this extension is part of my live server setup. This repository is a code/architecture showcase, not the distribution repository.

I included the class layout, lifecycle, file formats and simplified examples so the way I built it can still be reviewed without exposing the full implementation.
