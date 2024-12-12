# Quick notes

This section will mention notes that did not come up in the tutorial but may still be helpful for you.

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

You can use broadcasts to refresh your ui, `update_on(broadcast::<MyArbitaryStruct>(),|/*bevy query*/|{});`
Call it using `commands.react().broadcast(MyArbitaryStruct)`.

There are others as well.

### Other features
- Radio button widget: we hope to have an example later. It can also be useful for tabbing.
- [Here's](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/sickle_ext/index.html) a good starting point for animations, states, interactions.
- Localization: there is an example [here](https://github.com/UkoeHB/bevy_cobweb_ui/tree/main/examples/localization), with documentation [here](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/localization/index.html).
- Commands: cob files include a `#commands` section. TODO


## Cob documentation.
You can find more details about cob files [here](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/loading/index.html).
