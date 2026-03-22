[Back to overview](../yaml-schema.md)

# `resource`

The `resource` top-level section is shared by the `ResourceLink`, `ResourceMeta`, and `ResourcePatch` config parsers.

In practice, the examples under `bundle/source/resources/*.xl` use `resource` as a lightweight graph language:

- logical aliases point to other aliases or to concrete game resources,
- copies create mod-owned resources,
- links redirect lookups,
- patches merge targeted data,
- fixes rewrite nested names, paths, or context values inside matching resources.

The parser does not distinguish between built-in depot paths and mod-defined intermediate names. Any non-empty resource-path-like scalar can be used as a key or value.

## Resource authoring workflow

A common authoring pattern is:

1. Use `scope` to define the family of resources you want to work with.
2. Use `copy` to create mod-owned intermediates when you should not patch originals directly.
3. Use `patch` to apply targeted modifications.
4. Use `link` to expose a stable logical path to other systems.
5. Use `fix` when internal references inside the loaded resource also need to be rewritten.

## `resource.link`

Accepted shapes:

```yaml
resource:
  link:
    "base\\target_a.mesh":
      - "mods\\replacement_a.mesh"
      - "mods\\replacement_b.mesh"
```

```yaml
resource:
  link:
    "base\\target_a.mesh": "mods\\replacement_a.mesh"
```

### Fields

- `resource.link`: map.
- Each key is the target path.
- Each value is intended to be either:
  1. A scalar source path.
  2. A sequence of source paths.

### Notes

- The sequence form behaves consistently and is what the bundled examples use.
- In the current implementation, the scalar branch stores the mapping in reverse (`source -> target` instead of `target -> source`). Prefer the sequence form even when only one source is needed.

## `resource.copy`

Accepted shape:

```yaml
resource:
  copy:
    "mods\\source_a.mesh": "mods\\copy_a.mesh"
    "mods\\source_b.mesh":
      - "mods\\copy_b1.mesh"
      - "mods\\copy_b2.mesh"
```

### Fields

- `resource.copy`: map.
- Each key is a source path.
- Each value is either a scalar target path or a sequence of target paths.

Bundled examples commonly pair `copy` with `patch`: first copy a base resource into a mod-owned path, then patch the copied resource or use it as a reusable intermediate target.

## `resource.scope`

Accepted shape:

```yaml
resource:
  scope:
    "player_customization.app":
      - "player_ma_hair.app"
      - "player_wa_hair.app"
    "player_ma_hair.app":
      - "base\\characters\\head\\player_base_heads\\appearances\\hairs\\hh_000_pma__hairs_045.app"
      - "base\\characters\\head\\player_base_heads\\appearances\\hairs\\hh_001_pma__hairs_053.app"
    "cyberpunk2077.quest":
      - "cyberpunk2077_main.quest"
      - "cyberpunk2077_ep1.quest"
```

### Fields

- `resource.scope`: map.
- Each key is a scope path.
- Each value is either a scalar target path or a sequence of target paths.

### Notes

- Scope entries can form chains, with aliases expanding into other aliases and then into concrete resources.
- Scope is used across multiple resource types, including `.app`, `.mesh`, `.morphtarget`, `.ent`, and `.quest`.
- Empty YAML files are valid overall input and simply contribute no config.

## `resource.fix`

Accepted shape:

```yaml
resource:
  fix:
    "base\\broken.mesh":
      names:
        oldName: newName
      paths:
        "base\\old_path.mesh": "mods\\new_path.mesh"
      context:
        paramA: "value"
        paramB: "other"
```

### Fields

- `resource.fix`: map.
- Each key is the target resource path to fix.
- Each value must be a map.
- Optional submaps:
  - `names`: map of old name -> new name.
  - `paths`: map of old resource path -> new resource path.
  - `context`: map of context parameter -> scalar value.

If any of `names`, `paths`, or `context` is present but not a map, that `fix` entry is skipped entirely.

### Notes

- A fix entry may define any subset of `names`, `paths`, and `context`.
- `names` is commonly used to remap appearance or material names.
- `paths` is used to remap referenced resources inside another resource.
- `context` stores string values keyed by parameter name.
- YAML anchors and aliases work here because the parser operates on the resolved YAML tree.

## `resource.patch`

Accepted shapes:

```yaml
resource:
  patch:
    "mods\\patch_a.mesh": "base\\target_a.mesh"
    "mods\\patch_b.mesh":
      - "base\\target_b.mesh"
      - "base\\target_c.mesh"
    "mods\\patch_c.mesh": !exclude
      - "base\\target_d.mesh"
    "mods\\patch_d.mesh":
      props:
        - appearanceName
        - chunkMask
      targets:
        - "base\\target_e.mesh"
        - "base\\target_f.mesh"
```

### Fields

- `resource.patch`: map.
- Each key is the patch resource path.
- Each value may be:
  1. A scalar target path.
  2. A sequence of target paths.
  3. A map with:
     - `props`: sequence of property names.
     - `targets`: sequence of target paths.

### Tag behavior

- Sequence and map definitions honor the YAML tag `!exclude`; tagged targets are stored in the patch exclusion set instead of the inclusion set.
- Scalar definitions do not honor `!exclude`; scalar targets always go into the inclusion set.

### Notes

- Bundled examples usually use the map form so they can limit patching to specific properties such as `appearances`, `renderResourceBlob`, `blob`, `boundingBox`, `targets`, `baseTexture`, or `baseTextureParamName`.
- In the map form, the actual include or exclude list is defined under `targets`.
- A patch entry is retained only if it ends up with at least one included target. Entries that produce only excluded targets are discarded.

## Example: scoped quest registry with phase injection

```yaml
resource:
  scope:
    "story_bundle.quest":
      - "base\\quests\\prologue.quest"
      - "base\\quests\\ep1\\followup.quest"

quest:
  phases:
    - path: "mods\\phases\\story_gate.questphase"
      parent: "story_bundle.quest"
```

Here `story_bundle.quest` acts as a reusable registry entry. The `quest` section names one parent alias, and the runtime expands it to each concrete quest resource before applying the phase modification.
