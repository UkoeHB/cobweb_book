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
