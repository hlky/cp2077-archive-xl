# YAML schema reference

This overview documents the YAML consumed by the extension config parsers implemented in `src/App/Extensions/*/Config.cpp` and links to the detailed reference for each supported section.

> Notes
>
> - The section references are based on the current parser implementation, so they describe actual runtime behavior, including implementation quirks that may affect authoring.
> - Unless a section states otherwise, omitted sections and fields are optional.
> - Resource paths, appearance names, body types, language codes, and similar values are expected to be YAML scalars.

## How the YAML model fits together

Most top-level sections can be authored independently, but several features share a common resource-aliasing model centered on `resource.scope`:

- `resource.scope` defines reusable named lists of resources.
- `resource.copy`, `resource.link`, and `resource.patch` can target those named lists in addition to concrete depot paths.
- Other systems also resolve scoped names before acting on them. In practice, this allows one alias to expand into multiple real resources when used from fields such as `animations[*].entity` or `quest.phases[*].parent`.

A practical workflow is:

1. Define a logical resource name with `resource.scope`.
2. Reuse that name from other sections.
3. Let the runtime resolve it to the currently registered concrete resources.

Because scope expansion is string-based rather than file-extension specific, the same approach works across `.app`, `.mesh`, `.ent`, `.morphtarget`, `.quest`, and similar resource paths.

### Example: reusable resource aliases

```yaml
resource:
  scope:
    "my_mod.characters.hair.app":
      - "mods\\hair_a.app"
      - "mods\\hair_b.app"
```

This does not load or patch the files by itself. It registers a reusable symbolic name that other sections can reference.

Aliases can also point to quest resources and be reused for quest phase injection:

```yaml
resource:
  scope:
    "my_mod.quest":
      - "base\\quests\\ep1\\my_story.quest"
      - "mods\\bonus\\my_story_variant.quest"

quest:
  phases:
    - path: "mods\\phases\\extra_intro.questphase"
      parent: "my_mod.quest"
      input:
        node: [0]
        socket: "In"
      output:
        node: [12]
        socket: "Out"
```

Scope entries may also chain through other aliases. During configuration, the runtime flattens those chains transitively until it reaches concrete resources.

## Section index

The following top-level keys are recognized by the reviewed extension configs:

- [`animations`](yaml-schema/animations.md)
- [`customizations`](yaml-schema/customizations.md)
- [`factories`](yaml-schema/factories.md)
- [`journal`](yaml-schema/journal.md)
- [`localization`](yaml-schema/localization.md)
- [`overrides.tags`](yaml-schema/overrides.md)
- [`player.bodyTypes`](yaml-schema/player.md)
- [`quest.phases`](yaml-schema/quest.md)
- [`resource`](yaml-schema/resource.md)
- [`streaming`](yaml-schema/streaming.md)

A single YAML file may define more than one section.

Extensions that do not own a dedicated top-level section, but still consume YAML-authored data, are documented separately in [Extensions without top-level sections](yaml-schema/extensions.md).

## Practical examples

For end-to-end authoring patterns that combine multiple sections, see the examples in [docs/examples](examples/README.md).
