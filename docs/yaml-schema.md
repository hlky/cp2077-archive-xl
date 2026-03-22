# YAML schema reference

This document describes the YAML accepted by the extension config parsers implemented in `src/App/Extensions/*/Config.cpp`.

> Notes
>
> - This reference is derived from the parser implementation, so it documents what the code currently accepts, including a few parser quirks.
> - Unless noted otherwise, omitted sections are optional.
> - Resource paths, appearance names, body types, language codes, and similar string values are expected to be YAML scalars.

## Top-level sections

The following top-level keys are recognized by the reviewed extension configs:

- `animations`
- `customizations`
- `factories`
- `journal`
- `localization`
- `overrides`
- `player`
- `quest`
- `resource`
- `streaming`

A single YAML file may define more than one section if multiple extensions consume it.

---

## `animations`

Accepted shape:

```yaml
animations:
  - entity: "path/to/entity.ent"
    set: "anim_set_name"
    priority: 128          # optional integer-like scalar
    component: "root"     # optional scalar, defaults to "root"
    vars:                  # optional sequence of scalars; see quirk below
      - "VarA"
      - "VarB"
```

### Fields

- `animations`: required sequence when present.
- Each item must be a map.
- `entity`: required scalar.
  - The implementation appears intended to also allow a sequence of entity paths, but the current code only accepts a scalar because the sequence branch is unreachable.
- `set`: required scalar.
- `priority`: optional scalar parsed as an integer. Default is `128`.
- `component`: optional scalar. Default is `root`.
- `vars`: optional sequence.
  - Parser quirk: the loop checks the container node instead of each item, so sequence entries are currently ignored and no variables are actually loaded from YAML.

If `animations` exists but is not a sequence, the parser records `Bad format. Expected list of anim entries.`.

---

## `customizations`

Accepted shape:

```yaml
customizations:
  male: "base\\characters\\appearances\\male.app"
  female:
    - "base\\characters\\appearances\\female_01.app"
    - "base\\characters\\appearances\\female_02.app"
```

### Fields

- `customizations.male`: optional scalar or sequence of scalars.
- `customizations.female`: optional scalar or sequence of scalars.

Any other YAML node type for `male` or `female` records `Bad format. Expected resource path or list of paths.`.

---

## `factories`

Accepted shape:

```yaml
factories:
  - "base\\factories\\items_01.entfactory"
  - "base\\factories\\items_02.entfactory"
```

Or a single scalar:

```yaml
factories: "base\\factories\\items_01.entfactory"
```

### Fields

- `factories`: scalar or sequence of scalars.

Any other node type records `Bad format. Expected resource path or list of paths.`.

---

## `journal`

Accepted shape:

```yaml
journal:
  - "base\\journal\\quest_a.journal"
  - "base\\journal\\quest_b.journal"
```

Intended alternate shape:

```yaml
journal: "base\\journal\\quest_a.journal"
```

### Fields

- `journal`: intended to accept a scalar or sequence of scalars.

### Parser quirk

The sequence form works as expected. The scalar branch reads `aNode.Scalar()` instead of `rootNode.Scalar()`, so a scalar `journal:` value is not loaded correctly by the current implementation.

Invalid types record `Bad format. Expected resource path or list of paths.`.

---

## `localization`

Accepted shape:

```yaml
localization:
  onscreens:
    en-us:
      - "localization\\onscreens_en-us.json"
    pl-pl: "localization\\onscreens_pl-pl.json"
  subtitles:
    en-us: "localization\\subtitles_en-us.json"
  lipmaps:
    - "localization\\lipmaps_en-us.json"
  vomaps: "localization\\vo_maps_en-us.json"
  extend: "SomeBaseMod"   # optional scalar
```

### Fields

Each of these keys is optional:

- `localization.onscreens`
- `localization.subtitles`
- `localization.lipmaps`
- `localization.vomaps`
- `localization.extend`

For `onscreens`, `subtitles`, `lipmaps`, and `vomaps`, the parser accepts:

1. A map of language code -> scalar or sequence of scalars.
2. A sequence of scalars.
3. A single scalar.

### Language handling

- In the map form, each key must be a known language code.
- Unknown language codes generate `Unknown language code "...".` and that entry is skipped.
- In the scalar and sequence shorthand forms, content is assigned to English (`en-us` in practice via `Language::English`).
- The first successfully loaded language becomes the fallback language.

If any localization group uses an unsupported node type, the parser records `Bad format. Expected resource path or list of paths.`.

---

## `overrides.tags`

Accepted shape:

```yaml
overrides:
  tags:
    hide_helmet:
      head:
        hide: [0, 1, 2]
      visor: 7
    show_collar:
      collar:
        show: [1, 3]
      straps: [0, 2, 4]
```

### Fields

- `overrides.tags`: required map when `overrides` is used.
- Each tag name maps to a component map.
- Each component name maps to one of:
  1. A scalar integer bitmask.
  2. A sequence of byte values.
  3. A map with exactly one supported operation:
     - `hide: [byte, ...]`
     - `show: [byte, ...]`

### Notes

- If `overrides.tags` is missing, the section is silently ignored.
- If `overrides.tags` is present but not a map, the parser records `Bad format. Expected list of tags.`.
- Invalid component definitions mark the whole section malformed, but successfully parsed tags are still retained.

---

## `player.bodyTypes`

Accepted shape:

```yaml
player:
  bodyTypes:
    - "masc"
    - "fem"
```

Or:

```yaml
player:
  bodyTypes: "masc"
```

### Fields

- `player.bodyTypes`: optional scalar or sequence of scalars.

Any other node type records `Bad format. Expected body type name or list of names.`.

---

## `quest.phases`

Accepted shape:

```yaml
quest:
  phases:
    - path: "quests/main/phase_a"
      parent: "quests/main/parent_phase"
      connection:
        node: [0, 1, 2]
        socket: "Input"
      input: [0, 1, 2]      # alternative shorthand for the input connection
      output:
        node: [3, 0]
        socket: "Output"
      intercept: true
```

### Fields

- `quest.phases`: optional sequence.
- Each phase item must be a map.
- Required:
  - `path`: scalar.
  - `parent`: scalar.
- Optional connection fields:
  - `connection`
  - `input`
  - `output`
- Optional behavior flag:
  - `intercept`: scalar boolean-ish value parsed with `as<bool>()`.

### Connection shapes

For `connection`, `input`, and `output`, the parser accepts either:

1. A map:
   ```yaml
   input:
     node: [0, 1, 2]
     socket: "Input"
   ```
2. A sequence shorthand:
   ```yaml
   input: [0, 1, 2]
   ```

### Notes

- `connection` and `input` both write into the same input connection structure. If both are present, `input` is applied after `connection` and therefore wins for overlapping fields.
- The map form assumes both `node` and `socket` exist; malformed maps can fail hard rather than merely being skipped.
- `parent` accepts only a single scalar even though the runtime stores parent paths in a set.

---

## `resource`

The `resource` top-level section is shared by three config parsers: `ResourceLink`, `ResourceMeta`, and `ResourcePatch`.

### `resource.link`

Accepted shape:

```yaml
resource:
  link:
    "base\\target_a.mesh": "mods\\replacement_a.mesh"
    "base\\target_b.mesh":
      - "mods\\replacement_b1.mesh"
      - "mods\\replacement_b2.mesh"
```

- `resource.link`: map.
- Each key is a target path.
- Each value is either a scalar source path or a sequence of source paths.

### `resource.copy`

Accepted shape:

```yaml
resource:
  copy:
    "mods\\source_a.mesh": "mods\\copy_a.mesh"
    "mods\\source_b.mesh":
      - "mods\\copy_b1.mesh"
      - "mods\\copy_b2.mesh"
```

- `resource.copy`: map.
- Each key is a source path.
- Each value is either a scalar target path or a sequence of target paths.

### `resource.scope`

Accepted shape:

```yaml
resource:
  scope:
    "mods\\scope_a.archive": "base\\target_a.mesh"
    "mods\\scope_b.archive":
      - "base\\target_b.mesh"
      - "base\\target_c.mesh"
```

- `resource.scope`: map.
- Each key is a scope path.
- Each value is either a scalar target path or a sequence of target paths.

### `resource.fix`

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

- `resource.fix`: map.
- Each key is the target resource path to fix.
- Each value must be a map.
- Optional submaps:
  - `names`: map of old name -> new name.
  - `paths`: map of old resource path -> new resource path.
  - `context`: map of context parameter -> scalar value.

If any of `names`, `paths`, or `context` is present but not a map, that `fix` entry is skipped entirely.

### `resource.patch`

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

- `resource.patch`: map.
- Each key is the patch resource path.
- Each value may be:
  1. A scalar target path.
  2. A sequence of target paths.
  3. A map with:
     - `props`: sequence of property names.
     - `targets`: sequence of target paths.

### `resource.patch` tag behavior

- Sequence and map definitions honor the YAML tag `!exclude`; tagged targets are stored in the patch's exclusion set instead of the inclusion set.
- Scalar definitions do **not** honor `!exclude`; scalar targets always go into the inclusion set.
- Only patches with at least one included target are retained by the current implementation. A definition that contains only excluded targets is dropped.

---

## `streaming`

Accepted shape:

```yaml
streaming:
  blocks:
    - "base\\worlds\\block_01.streamingblock"
  sectors:
    - path: "base\\worlds\\watson\\sector_01.streamingsector"
      expectedNodes: 42
      nodeDeletions:
        - type: "worldEntityNode"
          index: 7
          expectedActors: 4
          actorDeletions: [1, 3]
        - type: "worldInstancedMeshNode"
          index: 8
          expectedInstances: 12
          instanceDeletions: [0, 2, 5]
      nodeMutations:
        - type: "worldEntityNode"
          index: 10
          position: [1.0, 2.0, 3.0]
          orientation: [0.0, 0.0, 0.0, 1.0]
          scale: [1.0, 1.0, 1.0]
          nbNodesUnderProxyDiff: 2
          resource: "mods\\replacement.ent"
          appearance: "default"
          recordID: "Items.SomeRecord"
          expectedActors: 2
          actorMutations:
            - index: 1
              position: [1.0, 2.0, 3.0, 0.0]
              orientation: [0.0, 0.0, 0.0, 1.0]
              scale: [1.0, 1.0, 1.0]
```

### `streaming.blocks`

- `streaming.blocks`: optional sequence of scalar resource paths.
- A scalar shorthand is **not** supported here.

### `streaming.sectors`

- `streaming.sectors`: optional sequence.
- Each sector must be a map.
- Required per sector:
  - `path`: non-empty scalar.
  - `expectedNodes`: scalar integer greater than `0`.
- A sector is only retained if it produces at least one valid deletion or mutation.

### `nodeDeletions`

Accepted shape:

```yaml
nodeDeletions:
  - type: "worldEntityNode"
    index: 5
    expectedActors: 3
    actorDeletions: [0, 2]
```

Or with instance subnodes:

```yaml
nodeDeletions:
  - type: "worldInstancedMeshNode"
    index: 5
    expectedInstances: 8
    instanceDeletions: [1, 4]
```

Fields:

- `type`: required scalar.
- `index`: required scalar integer within `[0, expectedNodes)`.
- Optional actor subnode deletion pair:
  - `expectedActors`: required if `actorDeletions` is used.
  - `actorDeletions`: sequence of integer indices.
- Optional instance subnode deletion pair:
  - `expectedInstances`: required if `instanceDeletions` is used.
  - `instanceDeletions`: sequence of integer indices.

### `nodeMutations`

Accepted shape:

```yaml
nodeMutations:
  - type: "worldEntityNode"
    index: 10
    position: [1.0, 2.0, 3.0]
    orientation: [0.0, 0.0, 0.0, 1.0]
    scale: [1.0, 1.0, 1.0]
    nbNodesUnderProxyDiff: 2
    resource: "mods\\replacement.ent"
    appearance: "default"
    recordID: "Items.SomeRecord"
    expectedActors: 2
    actorMutations:
      - index: 1
        position: [1.0, 2.0, 3.0, 0.0]
```

Fields:

- `type`: required scalar.
- `index`: required scalar integer within `[0, expectedNodes)`.
- Optional transform fields:
  - `position`: sequence of `3` or `4` floats. The fourth value is ignored; stored W is forced to `0`.
  - `orientation`: sequence of `4` floats.
  - `scale`: sequence of `3` floats.
- Optional proxy field:
  - `nbNodesUnderProxyDiff`: scalar integer.
- Optional resource aliases, all writing the same resource-path field:
  - `resource`
  - `mesh`
  - `meshRef`
  - `material`
  - `effect`
  - `entityTemplate`
- Optional appearance aliases, all writing the same appearance field:
  - `appearance`
  - `appearanceName`
  - `meshAppearance`
- Optional record aliases, all writing the same record field:
  - `recordID`
  - `recordId`
  - `objectRecordId`
- Optional actor subnode mutation pair:
  - `expectedActors`
  - `actorMutations`
- Optional instance subnode mutation pair:
  - `expectedInstances`
  - `instanceMutations`

### Subnode mutations

Accepted shape:

```yaml
actorMutations:
  - index: 0
    position: [1.0, 2.0, 3.0, 0.0]
    orientation: [0.0, 0.0, 0.0, 1.0]
    scale: [1.0, 1.0, 1.0]
```

Fields:

- `index`: required scalar integer within `[0, expectedActors)` or `[0, expectedInstances)`.
- Optional:
  - `position`: sequence of exactly `4` floats; again, W is stored as `0`.
  - `orientation`: sequence of exactly `4` floats.
  - `scale`: sequence of exactly `3` floats.

A subnode mutation is kept only if at least one of `position`, `orientation`, or `scale` is valid.

### Notes

- The parser silently skips many malformed deletion/mutation entries instead of reporting issues.
- `WorldSectorMod` has an `autoUpdateNodesUnderProxies` field in C++, but this parser does not populate it from YAML.

---

## Summary of notable parser quirks

These are worth knowing because this document intentionally reflects the current implementation:

- `animations.entity` is effectively scalar-only even though the code appears to intend sequence support.
- `animations.vars` sequence items are currently ignored.
- Scalar `journal:` entries do not load because the parser reads the wrong node.
- `quest.connection`/`input` map parsing assumes both `node` and `socket` exist.
- `resource.patch: !exclude <scalar>` is not treated as an exclusion.
- `resource.patch` entries with only excluded targets are discarded.
