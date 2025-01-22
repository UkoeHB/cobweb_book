# Getting started


## Commands 

let's make an empty project to test it out:

```
cargo new cobweb_test
cd cobweb_test
cargo add bevy
```

This book won't be making any distinction between `bevy_cobweb` and `bevy_cobweb_ui`. `bevy_cobweb` is a reactivity library that `bevy_cobweb_ui` uses for convenience methods like `.on_pressed`.
```
cargo add bevy_cobweb
cargo add bevy_cobweb_ui -F hot_reload
```

We are definitely adding hot reloading, but you can remove it for your release version.


## Syntax Highlighting

You can optionally install syntax highlighting for the cob files we will be using.

[Instructions here](https://github.com/UkoeHB/bevy_cobweb_ui).



## Rust Code

Set your `main.rs` to be as below.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

fn build_ui(mut c: Commands, mut s: SceneBuilder) {
    c.spawn(Camera2d);
    c.ui_root().spawn_scene(("main.cob", "main_scene"), &mut s);
}

//-------------------------------------------------------------------------------------------------------------------

fn main() {
    App::new()
        .add_plugins(bevy::DefaultPlugins.set(WindowPlugin {
            primary_window: Some(Window {
                window_theme: Some(bevy::window::WindowTheme::Dark),
                ..default()
            }),
            ..default()
        }))
        .add_plugins(CobwebUiPlugin)
        .load("main.cob")
        .add_systems(OnEnter(LoadState::Done), build_ui)
        .run();
}
```

This will add systems to handle loading of your files and other plumbing:
`.add_plugins(CobwebUiPlugin)`

This tells cobweb to load this cob file:
`.load("main.cob")`

When all the cob files are loaded this will call our UI setup system:
`.add_systems(OnEnter(LoadState::Done), build_ui)`

This makes a new UI hierarchy with its own root node:
`c.ui_root().spawn_scene(("main.cob", "main_scene"), &mut s);`


## COB code
Create a new folder called `assets`.

Create a new file called `main.cob`.

Add in the following:
```
#scenes
"main_scene"
    TextLine{ text: "Hello, World!" }
```

Now let's run the program.

We have our first cobweb UI program. Next chapter we can start making changes *without recompiling*.
