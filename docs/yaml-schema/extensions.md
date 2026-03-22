[Back to overview](../yaml-schema.md)

# Extensions without top-level sections

The following behaviors are not driven by a dedicated top-level section such as `animations:` or `resource:`. Instead, the extension reads TweakDB fields or string conventions that are commonly authored in YAML elsewhere.

## `Attachment`

`AttachmentExtension` reacts to attachment-slot records and reads two flats during slot initialization:

- `parentSlot`
- `dependencySlots`

Accepted shape:

```yaml
AttachmentSlots.MyExtraHeadSlot:
  $type: gamedataAttachmentSlot_Record
  parentSlot: AttachmentSlots.Head
  dependencySlots:
    - AttachmentSlots.Torso
    - AttachmentSlots.Chest
```

### Fields

- `parentSlot`: optional scalar `TweakDBID`.
  - When present, the current slot becomes an extra child slot of `parentSlot`.
  - Child slots inherit the base slot's spawning checks, third-person affected-slot handling, and visual-tag lookups.
- `dependencySlots`: optional sequence of `TweakDBID` values.
  - Each listed slot is treated as a prerequisite for the current slot.
  - Internally, the extension stores the inverse relationship so the listed slot can discover which slots depend on it.

### Notes

- Slots under `AttachmentSlots.Head` and `AttachmentSlots.Eyes` are treated as third-person appearance-affecting slots once they declare `parentSlot`.
- Visual-tag checks performed for `AttachmentSlots.Head`, `AttachmentSlots.Torso`, and `AttachmentSlots.Feet` also search child slots discovered through `parentSlot`.
- The extension always adds two built-in dependencies even if YAML does not declare them:
  - `AttachmentSlots.Torso` -> `AttachmentSlots.Chest`
  - `AttachmentSlots.Feet` -> `AttachmentSlots.Legs`
- Omitting both fields is valid; the slot is simply ignored by this extension.
- `parentSlot` is read as a single flat value, so list or map shapes are not used.
- `dependencySlots` is read only as an array flat; non-array values are ignored.

## `InkSpawner`

`InkSpawnerExtension` does not parse a dedicated top-level YAML section. Instead, it extends a string convention that can be embedded anywhere a widget library item name is authored as a YAML scalar or passed through scripts.

Accepted item-name format:

```yaml
itemName: "WidgetItemName[:ControllerClassName]"
```

Examples:

```yaml
itemName: "PhoneMessageItem"
itemName: "PhoneMessageItem:MyGameController"
itemName: "PhoneMessageItem:MyLogicController"
```

### Encoded fields

- `WidgetItemName`: required scalar prefix before the first `:`.
  - This is the actual widget library item name that will be spawned.
- `ControllerClassName`: optional scalar suffix after the first `:`.
  - When present, the extension attempts to resolve the class by name and inject it into the spawned widget instance.

### Controller rules

- If the resolved class derives from `inkIWidgetController`, it replaces the instance's `gameController`.
- If the resolved class derives from `inkWidgetLogicController`, it replaces the root widget's `logicController`.
- Existing controller properties with matching names and types are copied to the new controller instance.

### External-library shape

For spawn APIs that also take an external library path, the effective YAML-facing shape is:

```yaml
library: "base\\gameplay\\gui\\widgets.inkwidget"
itemName: "WidgetItemName[:ControllerClassName]"
```

- `library`: scalar resource path to an external widget library.
- `itemName`: scalar using the format above.

### Notes

- The `:ControllerClassName` suffix is recognized for synchronous and asynchronous local spawns.
- External spawns automatically inject `library` into the widget library dependency list before the original spawn call runs.
- Only the first `:` is treated as the separator.
- If the item name does not contain `:`, the extension falls back to the original spawn behavior with no controller injection.
- If `ControllerClassName` cannot be resolved, spawning still succeeds but no controller replacement occurs.

## `Transmog`

`TransmogExtension` does not define a standalone config section. It reacts to fields on TweakDB item and visual records that are commonly authored in YAML.

The extension reads:

- `entityName` from the visual record used by the appearance-change request
- `appearanceName` from the item record
- `visualTags` from the item record

Accepted shapes:

```yaml
Items.MyTransmogVisual:
  $type: gamedataItemVisual_Record
  entityName: MyFactoryEntityName

Items.MyTransmogItem:
  $type: gamedataItem_Record
  appearanceName: my_custom_appearance
  visualTags:
    - my_fallback_appearance
```

### Fields

- `entityName`
  - Location: visual record.
  - Shape: optional scalar `CName`.
  - Behavior:
    - The extension asks the Factory Index to resolve `entityName` into an entity template resource path.
    - If resolution succeeds, that resource path temporarily replaces the target entity template while the appearance-change request loads its template.
- `appearanceName`
  - Location: item record.
  - Shape: optional scalar `CName`.
  - Behavior:
    - Used only when the appearance-change request did not already specify an appearance.
    - The value must match an appearance name that exists in the loaded appearance resource; otherwise the normal game selection logic continues.
- `visualTags`
  - Location: item record.
  - Shape: optional sequence of `CName` values.
  - Behavior:
    - This is only a fallback after `appearanceName`.
    - The fallback is used only when the sequence contains exactly one entry.
    - That single tag is treated as an appearance name and selected only if it matches an appearance defined in the loaded appearance resource.

### Notes

- `entityName` depends on Factory Index data being available for the named factory entry.
- `appearanceName` takes precedence over the single-tag `visualTags` fallback.
- `visualTags` with zero entries or more than one entry are ignored by this extension's appearance fallback logic.
- Missing fields are allowed; the extension simply falls back to vanilla behavior.
- If `entityName` cannot be resolved to a resource path, no template override is applied.
- If `appearanceName` or the single `visualTags` entry does not exist in the appearance resource, the original appearance-selection function handles the request.
