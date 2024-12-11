# Errors

## Forgetting to add Interactive to nodes
If you node defined in the cob file has to be interacted with make sure you put in `Interactive`

## Odd lifetime errors, usually about static lifetimes
If you capture closures like in `on_pressed` make sure you use move and clone anything you need.

## Trying to load non top level scenes
Only the top level scenes can be loaded independently.

## Loading manifests in multiple files

If you have warnings about reparenting its likely because your have manifests defined in multiple places
`   WARN bevy_cobweb_ui::loading::cache::commands_buffer: reparenting file CobFile("ui/colour_scheme.cob") from Parent(CobFile("ui/panels/outliner.cob")) to Parent(CobFile("ui/moons.cob"))
    at /home/lyndonm/.cargo/git/checkouts/bevy_cobweb_ui-68d12fe85b5a400c/5b3a3aa/src/loading/cache/commands_buffer.rs:485`


## No using ui_builder to retrieve an interface

if you get an error like below.

Substitute UiRoot with the entity you want to work from
`commands.ui_builder(UiRoot)`

`WARN bevy_ui::layout: Node (233769v8) is in a non-UI entity hierarchy. You are using an entity with UI components as a child of an entity without UI components, your UI layout may be broken.
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/bevy_ui-0.15.0-rc.3/src/layout/mod.rs:267`
