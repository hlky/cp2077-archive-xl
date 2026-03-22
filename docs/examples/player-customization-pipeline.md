# Player customization patch pipeline

This example shows a common customization workflow for player hair or similar appearance-driven assets:

1. Define reusable aliases for the target appearance and mesh families.
2. Patch those targets with a shared resource patch.
3. Fix names or material context inside the affected meshes.

This pattern mirrors the bundled player customization resources, where `resource.scope` groups many appearance and mesh paths and `resource.patch` / `resource.fix` apply one change across the whole set.

## Example

```yaml
resource:
  scope:
    player_customization.app:
      - player_ma_hair.app
      - player_wa_hair.app
    player_ma_hair.app:
      - base\characters\head\player_base_heads\appearances\hairs\hh_000_pma__hairs_045.app
      - base\characters\head\player_base_heads\appearances\hairs\hh_001_pma__hairs_053.app
    player_ma_hair.mesh:
      - base\characters\common\hair\hh_006_ma__demo\hh_006_ma__demo.mesh
      - base\characters\common\hair\hh_007_ma__demo\hh_007_ma__demo.mesh

  patch:
    archive_xl\characters\common\hair\h1_base_color_patch.mesh:
      props: [appearances]
      targets:
        - player_ma_hair.mesh

  fix:
    base\characters\common\hair\hh_006_ma__demo\hh_006_ma__demo.mesh:
      names:
        phoenix_fire_cap: phoenix_fire@cap
        phoenix_fire_cards: phoenix_fire@dread
      context:
        DreadBaseMaterial: base\characters\common\hair\textures\hair_profiles\_master__dread_bright.mi
        CapBaseMaterial: archive_xl\characters\common\hair\textures\hair_profiles\hh_006_ma__demo_cap.mi
```

## Why this is useful

- `resource.scope` lets you patch one logical target instead of repeating every concrete path.
- `resource.patch` is a good fit when you want to merge a shared appearance or blob change into many existing resources.
- `resource.fix` is useful when the patched resource also needs internal name remaps or context rewrites.

## Variations

- Add `resource.copy` before `patch` when you want to patch mod-owned copies instead of original game resources.
- Add `resource.link` if other systems should reference a stable alias rather than the copied path directly.
- Split appearance and mesh aliases by gender, cyberware state, or other grouping that matches your asset set.
