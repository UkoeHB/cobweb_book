# Errors

## Forgetting to add Interactive to nodes
If your node defined in the cob file has to be interacted with (e.g. with `.on_pressed()`), make sure you put in `Interactive`.

## Odd lifetime errors, usually about static lifetimes
If you capture data in closures like `.on_pressed()`, make sure you use move and clone anything you need.

## Trying to load non-top-level scenes
Only the top level scenes can be loaded as scenes independently.

## Multiple manifests warning.

You may get a warning like below, it means you added a file to multiple manifests in different files.

`WARN bevy_cobweb_ui::loading::cache::commands_buffer: reparenting file CobFile("ui/colour_scheme.cob") from Parent(CobFile("ui/panels/outliner.cob")) to Parent(CobFile("ui/moons.cob"))
    at /home/lyndonm/.cargo/git/checkouts/bevy_cobweb_ui-68d12fe85b5a400c/5b3a3aa/src/loading/cache/commands_buffer.rs:485`
