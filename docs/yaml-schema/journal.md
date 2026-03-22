[Back to overview](../yaml-schema.md)

# `journal`

Accepted shapes:

```yaml
journal:
  - "base\\journal\\quest_a.journal"
  - "base\\journal\\quest_b.journal"
```

```yaml
journal: "base\\journal\\quest_a.journal"
```

## Fields

- `journal`: intended to accept a scalar or sequence of scalars.

## Notes

- The sequence form works as expected.
- In the current implementation, the scalar branch reads `aNode.Scalar()` instead of `rootNode.Scalar()`, so scalar `journal:` values are not loaded correctly.
- Invalid node types record `Bad format. Expected resource path or list of paths.`.
