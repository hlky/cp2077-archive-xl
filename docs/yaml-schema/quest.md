[Back to overview](../yaml-schema.md)

# `quest.phases`

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

## Fields

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
  - `intercept`: scalar parsed with `as<bool>()`.

## Connection formats

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

## Notes

- `connection` and `input` both write into the same input connection structure. If both are present, `input` is applied after `connection` and wins for overlapping fields.
- The map form assumes both `node` and `socket` exist; malformed maps can fail hard rather than being skipped.
- `parent` accepts only a single scalar even though the runtime stores parent paths in a set.
- `parent` is still resolved through the shared resource-scope registry, so one alias can target multiple concrete `.quest` resources.

```yaml
resource:
  scope:
    "my_mod.quest_roots":
      - "base\\quests\\main_quest.quest"
      - "mods\\bonus\\main_quest_variant.quest"

quest:
  phases:
    - path: "mods\\phases\\branch_intro.questphase"
      parent: "my_mod.quest_roots"
      intercept: true
```
