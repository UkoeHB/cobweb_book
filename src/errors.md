# Errors

## Odd lifetime errors, usually about static lifetimes
If you capture data in closures like `.on_pressed()`, make sure you use move and clone anything you need.

## Trying to load non-top-level scenes
Only the top level scenes can be loaded as scenes independently.

## Multiple manifests warning.

You may get a warning like below, it means you added a file to multiple manifests in different files.

`WARN bevy_cobweb_ui::loading::cache::commands_buffer: reparenting file CobFile("ui/colour_scheme.cob") from Parent(CobFile("ui/panels/outliner.cob")) to Parent(CobFile("ui/moons.cob"))
    at /home/lyndonm/.cargo/git/checkouts/bevy_cobweb_ui-68d12fe85b5a400c/5b3a3aa/src/loading/cache/commands_buffer.rs:485`

## Failing to use `ui_builder` to load a scene
If you get an error like below when loading a scene inside another scene, then use the `ui_builder` extension with the entity you want to be the parent of the newly-spawned scene:
`commands.ui_builder(parent_entity).spawn_scene_simple(..)`
```
WARN bevy_ui::layout: Node (233769v8) is in a non-UI entity hierarchy. You are using an entity with UI components as a child of an entity without UI components, your UI layout may be broken.
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/bevy_ui-0.15.0-rc.3/src/layout/mod.rs:267
```
