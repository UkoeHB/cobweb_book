# Getting started


## Commands 
let's make an empty project to test this out

`cargo new cobweb_test`

`cd cobweb_test/`

This book won't be making any distinction between cobweb and cobweb_ui
`cargo add bevy_cobweb`


`cargo add bevy_cobweb_ui -F hot_reload`

We are definitely adding hot reloading, but you can remove on your release version.


## Syntax Highlighting
you can optionally install syntax highlighting for the cob files we will be using

[instructions here](https://github.com/UkoeHB/bevy_cobweb_ui)



## Rust Code

set your main to be as below.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>)
{
    c.spawn(Camera2d);
    c.ui_root().load_scene(("main.cob", "main_scene"), &mut s);
}

//-------------------------------------------------------------------------------------------------------------------

fn main()
{
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

This will add systems to handle loading of your files and other plumbing
`.add_plugins(CobwebUiPlugin)`

This tells cobweb to load this file
`.load("main.cob")`

When all the cob files are loaded this will call our setup ui
`.add_systems(OnEnter(LoadState::Done), build_ui)`

make a new ui with its own root node
`c.ui_root().load_scene(("main.cob", "main_scene"), &mut s);`


## COB code
Create a new folder called `assets`

Create a new file called `main.cob`

add in the following code
```
#scenes
"main_scene"
    TextLine{ text: "Hello, World!" }
```


Now let's run the program we have our first bevy cobweb program, next chapter we can start making changes *Without recompiling*
