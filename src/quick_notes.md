# Quick notes

This section will mention notes that did not come up in the tutorial but may still be helpful for you.

## Misc Cob Syntax

### Splat

`splat` is a shorthand way for multiple fields to be set the same value, you may have already encountered this in bevy with `Vec2::splat` or `Vec3::splat`.

To use splat the data type should implement [splattable](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/loading/trait.Splattable.html#associatedtype.Splat).

For example
`Splat<Border>(15px)`


This should not be defined in a `FlexNode` or `AbsoluteNode` but as a seperate line.

## Loading Cob files

When you have a cob file that defines scenes, you must register with the app by using this syntax (or add it to a manifest section of another file):
`.load("main.cob")`

### Waiting for LoadState::Done

If your UI is to appear at the start of the game then you should wait to ensure cobweb has had a chance to read its scene data.
`.add_systems(OnEnter(LoadState::Done), build_ui)`

If your UI comes up in response to player actions on other UI, then calling it using observers or events should be fine as cob files would have been already read by cobweb.

## Other features not covered (at least for now!)

### Cobweb has reactive features.
Some examples can be found [here](https://github.com/UkoeHB/bevy_cobweb_ui/tree/main/examples).

You can use broadcasts to refresh your ui, `.reactor(broadcast::<MyArbitaryStruct>(), |/*bevy query*/| { });`
Send the event using `commands.react().broadcast(MyArbitaryStruct)`.

There are others as well.

### Other features
- Radio button widget: we hope to have an example later. It can also be useful for tabbing.
- [Here's](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/sickle_ext/index.html) a good starting point for animations, states, interactions.
- Localization: there is an example [here](https://github.com/UkoeHB/bevy_cobweb_ui/tree/main/examples/localization), with documentation [here](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/localization/index.html).
- Commands: cob files include a `#commands` section. TODO


## Cob documentation.
You can find more details about cob files [here](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/loading/index.html).

## Pulling existing node to edit
For creating nodes from scratch we used `commands.ui_builder(UiRoot)` (or `commands.ui_root()`). To modify an existing UI node, we can use the `ui_builder` extension with the entity that will be the parent of the newly-spawned scene:
`commands.ui_builder(parent_entity).spawn_scene_simple(..)`.

If you use `commands.get` you can end up with this error:

`WARN bevy_ui::layout: Node (233769v8) is in a non-UI entity hierarchy. You are using an entity with UI components as a child of an entity without UI components, your UI layout may be broken.
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/bevy_ui-0.15.0-rc.3/src/layout/mod.rs:267`
