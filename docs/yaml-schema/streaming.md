[Back to overview](../yaml-schema.md)

# `streaming`

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

## `streaming.blocks`

- `streaming.blocks`: optional sequence of scalar resource paths.
- Scalar shorthand is not supported.

## `streaming.sectors`

- `streaming.sectors`: optional sequence.
- Each sector must be a map.
- Required per sector:
  - `path`: non-empty scalar.
  - `expectedNodes`: scalar integer greater than `0`.
- A sector is retained only if it produces at least one valid deletion or mutation.

## `nodeDeletions`

Accepted shapes:

```yaml
nodeDeletions:
  - type: "worldEntityNode"
    index: 5
    expectedActors: 3
    actorDeletions: [0, 2]
```

```yaml
nodeDeletions:
  - type: "worldInstancedMeshNode"
    index: 5
    expectedInstances: 8
    instanceDeletions: [1, 4]
```

### Fields

- `type`: required scalar.
- `index`: required scalar integer within `[0, expectedNodes)`.
- Optional actor subnode deletion pair:
  - `expectedActors`: required if `actorDeletions` is used.
  - `actorDeletions`: sequence of integer indices.
- Optional instance subnode deletion pair:
  - `expectedInstances`: required if `instanceDeletions` is used.
  - `instanceDeletions`: sequence of integer indices.

## `nodeMutations`

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

### Fields

- `type`: required scalar.
- `index`: required scalar integer within `[0, expectedNodes)`.
- Optional transform fields:
  - `position`: sequence of `3` or `4` floats. The fourth value is ignored; stored `W` is forced to `0`.
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

## Subnode mutations

Accepted shape:

```yaml
actorMutations:
  - index: 0
    position: [1.0, 2.0, 3.0, 0.0]
    orientation: [0.0, 0.0, 0.0, 1.0]
    scale: [1.0, 1.0, 1.0]
```

### Fields

- `index`: required scalar integer within `[0, expectedActors)` or `[0, expectedInstances)`.
- Optional:
  - `position`: sequence of exactly `4` floats; `W` is still stored as `0`.
  - `orientation`: sequence of exactly `4` floats.
  - `scale`: sequence of exactly `3` floats.

A subnode mutation is retained only if at least one of `position`, `orientation`, or `scale` is valid.

## Notes

- The parser silently skips many malformed deletion and mutation entries instead of reporting issues.
- `WorldSectorMod` has an `autoUpdateNodesUnderProxies` field in C++, but this parser does not populate it from YAML.
