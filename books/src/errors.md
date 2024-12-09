# errors

## Forgetting to add Interactive to nodes
If you node defined in the cob file has to be interacted with make sure you put in `Interactive`

## Odd lifetime errors, usually about static lifetimes
If you capture closures like in `on_pressed` make sure you use move and clone anything you need.

## Trying to load non top level scenes
Only the top level scenes can be loaded independently.
