# Multi-section mod pack example

This example demonstrates how several top-level sections can work together in one config set for a larger mod pack.

Use this pattern when your mod needs to:

- register reusable resource aliases,
- patch or redirect assets,
- apply an animation set to multiple entities,
- load localization files,
- add streaming modifications for world content.

## Example

```yaml
resource:
  scope:
    my_mod.npc_templates.ent:
      - base\characters\npcs\gang_a.ent
      - base\characters\npcs\gang_b.ent
    my_mod.common_meshes.mesh:
      - base\environment\props\terminal_a.mesh
      - base\environment\props\terminal_b.mesh

  copy:
    base\environment\props\terminal_a.mesh: mods\environment\terminal_a_mod.mesh

  link:
    my_mod\environment\terminal_current.mesh:
      - mods\environment\terminal_a_mod.mesh

  patch:
    mods\patches\terminal_screen_patch.mesh:
      props: [appearances, renderResourceBlob]
      targets:
        - my_mod.common_meshes.mesh

animations:
  - entity: my_mod.npc_templates.ent
    set: my_mod_idle_variants
    priority: 180

localization:
  onscreens:
    en-us: mods\localization\onscreens_en-us.json
    pl-pl: mods\localization\onscreens_pl-pl.json
  subtitles:
    en-us: mods\localization\subtitles_en-us.json

streaming:
  sectors:
    - path: base\worlds\watson\sector_01.streamingsector
      expectedNodes: 42
      nodeMutations:
        - type: worldEntityNode
          index: 10
          entityTemplate: mods\entities\ambient_terminal.ent
          appearance: default
```

## Why this is useful

- `resource.scope` keeps the asset graph readable as the mod grows.
- `resource.copy`, `resource.link`, and `resource.patch` let you separate ownership, redirection, and data merge responsibilities.
- `animations`, `localization`, and `streaming` can all consume the same broader content pack without requiring separate config files per subsystem.

## Suggested layout

For larger mods, split the YAML into multiple files by concern even though the runtime allows multiple sections per file:

- one file for resource aliases and patching,
- one file for quest or animation logic,
- one file for localization,
- one file for streaming edits.

That keeps review and troubleshooting manageable while still preserving shared aliases across the mod.
