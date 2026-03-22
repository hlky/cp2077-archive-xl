# Quest phase injection across multiple quest roots

This example shows an advanced but common pattern for quest mods: define one reusable quest alias, then inject the same phase into multiple quest resources.

It follows the same model used by the bundled quest scope resource, where one logical quest name expands into the main-game and expansion quest files.

## Example

```yaml
resource:
  scope:
    my_story_bundle.quest:
      - my_story_main.quest
      - my_story_ep1.quest
    my_story_main.quest:
      - base\quest\my_story.quest
    my_story_ep1.quest:
      - ep1\quest\my_story_ep1.quest

quest:
  phases:
    - path: mods\quests\my_story\phases\intro_gate.questphase
      parent: my_story_bundle.quest
      input:
        node: [0]
        socket: In
      output:
        node: [12]
        socket: Out
      intercept: true
```

## Why this is useful

- You author the phase injection once, even if the quest exists in multiple roots.
- The alias remains readable and reusable throughout the rest of your config.
- Future variants can be added by extending the scope instead of rewriting the `quest.phases` entry.

## Recommended checks

- Keep the alias names descriptive so it is obvious which quest family they represent.
- Validate the node paths and socket names against the actual quest assets before using `intercept: true`.
- If you need different behavior per quest root, split the alias and add separate phase entries instead of overloading one shared patch.
