[Back to overview](../yaml-schema.md)

# `localization`

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
  extend: "SomeBaseMod"
```

## Fields

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

## Language handling

- In the map form, each key must be a known language code.
- Unknown language codes generate `Unknown language code "...".` and the entry is skipped.
- In the scalar and sequence shorthand forms, content is assigned to English (`en-us` in practice via `Language::English`).
- The first successfully loaded language becomes the fallback language.

If any localization group uses an unsupported node type, the parser records `Bad format. Expected resource path or list of paths.`.
