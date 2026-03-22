# Body type registration and dynamic appearance setup

This example combines a schema-backed body type registration with a typical dynamic appearance authoring pattern described in the project wiki.

Use this when you want a clothing mod to:

- register a custom body type through ArchiveXL,
- let the game select the proper refit automatically,
- keep the appearance definition compact by using dynamic placeholders instead of many duplicated appearance variants.

## ArchiveXL registration

```yaml
player:
  bodyTypes:
    - Gymfiend
```

The registered body type name should also be added as a visual tag on the relevant `.ent` or `.app` resources used by the body mod.

## TweakDB and appearance example

```yaml
Items.MyJacketGymfiend:
  $base: Items.GenericOuterChestClothing
  entityName: my_jacket
  appearanceName: my_jacket!black+logo_a
  appearanceSuffixes:
    - !append itemsFactoryAppearanceSuffix.BodyType
```

```yaml
name: my_jacket
appearanceResource: my_mod\appearances\my_jacket.app
appearanceName: my_jacket
```

```yaml
mesh: *my_mod\meshes\my_jacket_{gender}_{body}.mesh
meshAppearance: *{variant.1}
```

## Why this is useful

- `player.bodyTypes` makes the body type visible to ArchiveXL's body-type selection flow.
- Dynamic placeholders such as `{gender}`, `{body}`, and `{variant.1}` reduce the number of duplicated appearance definitions you need to maintain.
- Appending `itemsFactoryAppearanceSuffix.BodyType` lets the item request the correct refit when body mods are present.

## Recommended checks

- Make sure the body type name is spelled the same everywhere: YAML config, visual tags, and any related appearance logic.
- Keep at least one fallback mesh or appearance path for users who do not have the body mod installed.
- If you use composite variants like `black+logo_a`, keep a clear convention for what each variant segment controls.
