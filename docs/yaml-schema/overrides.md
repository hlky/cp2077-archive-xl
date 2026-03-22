[Back to overview](../yaml-schema.md)

# `overrides.tags`

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

## Fields

- `overrides.tags`: required map when `overrides` is used.
- Each tag name maps to a component map.
- Each component name maps to one of:
  1. A scalar integer bitmask.
  2. A sequence of byte values.
  3. A map with exactly one supported operation:
     - `hide: [byte, ...]`
     - `show: [byte, ...]`

## Notes

- If `overrides.tags` is missing, the section is silently ignored.
- If `overrides.tags` is present but not a map, the parser records `Bad format. Expected list of tags.`.
- Invalid component definitions mark the whole section malformed, but successfully parsed tags are still retained.
