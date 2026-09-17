# Code structure

This document describes how the current `3.0.0-beta.15` build is split internally.

## `PatriaBridgeExtension`

Main Geyser extension class.

It keeps four main maps:

```text
entriesByModel       model id -> entity mapping entry
fallbackByJavaType   Java entity identifier -> model id
definitions          model id -> registered CustomEntityDefinition
runtimeEntities      entity UUID -> model id
```

It also stores the last modification time of `runtime-entities.json`. That lets the spawn path avoid reparsing the same file if nothing changed.

### Lifecycle methods

```java
onPreInitialize(GeyserPreInitializeEvent event)
onDefineEntities(GeyserDefineEntitiesEvent event)
onSpawn(ServerSpawnEntityEvent event)
onReload(GeyserPostReloadEvent event)
```

### `onPreInitialize`

The extension data folder is created first. Then it loads the static manifest and the runtime bindings.

Simplified logic:

```java
createDataFolder();
loadManifest();
reloadRuntime(true);
```

The real source contains the error handling and logging around these operations.

### `onDefineEntities`

This is where every configured Bedrock identifier becomes a Geyser `CustomEntityDefinition`.

Conceptually:

```java
for (EntityEntry entry : entriesByModel.values()) {
    Identifier id = Identifier.of(entry.bedrockIdentifier());
    CustomEntityDefinition definition = CustomEntityDefinition.of(id);
    event.register(definition);
    definitions.put(entry.modelId(), definition);
}
```

The exact implementation is kept private, but this shows the responsibility of the method.

### `onSpawn`

When Geyser sees a Java entity spawn, PatriaBridge first checks whether that UUID has a runtime model binding. If it does not, it tries the Java entity identifier as a fallback.

```text
UUID -> runtime model
        OR
Java entity type -> fallback model
        ↓
model id -> Geyser custom definition
```

If a definition exists, the spawn event is switched to that definition.

There is also a special case for model ids beginning with `furniture_` when the Java entity is `minecraft:item_display`. Those are intentionally delegated to the display/furniture system instead of being replaced here.

### Reload behavior

`onReload` reloads the mapping files. Existing mapping data can be refreshed, but new entity types still need a Geyser restart because entity definitions are registered during the lifecycle definition phase.

## `EntityEntry`

Small immutable record used by the main extension.

```text
modelId
bedrockIdentifier
javaEntityType
```

The Java entity type is optional as a fallback. `modelId` and `bedrockIdentifier` are the important values for an actual custom mapping.

## `MiniJson`

I wrote a small internal JSON reader for this extension instead of pulling a larger dependency just to read two simple configuration files.

The public-to-package helpers are basically:

```java
read(Path path)
object(Object value)
array(Object value)
string(Map<String, Object> map, String key)
```

The nested parser handles objects, arrays, strings, numbers, booleans and null plus normal JSON escaping.

This is intentionally not meant to be a general JSON library. It only exists to keep the extension small and self-contained.
