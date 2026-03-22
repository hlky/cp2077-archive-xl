[Back to overview](../yaml-schema.md)

# `customizations`

Accepted shape:

```yaml
customizations:
  male: "base\\characters\\appearances\\male.app"
  female:
    - "base\\characters\\appearances\\female_01.app"
    - "base\\characters\\appearances\\female_02.app"
```

## Fields

- `customizations.male`: optional scalar or sequence of scalars.
- `customizations.female`: optional scalar or sequence of scalars.

Any other node type for `male` or `female` records `Bad format. Expected resource path or list of paths.`.
