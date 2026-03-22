[Back to overview](../yaml-schema.md)

# `animations`

Accepted shape:

```yaml
animations:
  - entity: "path/to/entity.ent"
    set: "anim_set_name"
    priority: 128      # optional integer-like scalar
    component: "root" # optional scalar, defaults to "root"
    vars:              # optional sequence of scalars; see notes below
      - "VarA"
      - "VarB"
```

## Fields

- `animations`: required sequence when the section is present.
- Each item must be a map.
- `entity`: required scalar.
  - The implementation appears intended to also accept a sequence of entity paths, but the current code only accepts a scalar because the sequence branch is unreachable.
- `set`: required scalar.
- `priority`: optional scalar parsed as an integer. Default: `128`.
- `component`: optional scalar. Default: `root`.
- `vars`: optional sequence.
  - Current parser behavior ignores sequence entries because the loop checks the container node instead of each item.

## Usage notes

`entity` is resolved through the shared resource-scope registry, so it may be either a literal `.ent` path or a scoped alias.

```yaml
resource:
  scope:
    "my_mod.npc_templates.ent":
      - "base\\characters\\npc_a.ent"
      - "base\\characters\\npc_b.ent"

animations:
  - entity: "my_mod.npc_templates.ent"
    set: "my_anim_set"
```

If `animations` is present but is not a sequence, the parser records `Bad format. Expected list of anim entries.`.
