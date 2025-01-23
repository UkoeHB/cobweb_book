# Tabs (WIP)

## Goals
- Create a window with tabs that can be selected from
- Learn about radio buttons
- Learn about controlRoot and ControlMembers for transferring state

## Method

Tabs are pretty much radio buttons, so we will be using these to implement them in cobweb_ui.
We will start with a basic window that should approximate what this book has taught so far.


### Starting rust code.

Hopefully this is familiar from previous chapters.

```rs

use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

fn build_ui(mut c: Commands, mut s: SceneBuilder) {
    c.spawn(Camera2d);
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {});
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

### Starting COB file

`GridNode` and `AbsoluteGridNode` are used to create an orderly scaled layout.

`FlexNode` is used to center the text on some titles.

Note we can do some abstractions to tidy up the code, we will stick to simplicity at the expense of verbosity.
Once we have the functionality sorted we look at some abstractions.

Hopefully this code is also mostly familiar from previous chapters.

```
#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh   }
    BackgroundColor(Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 })
    "title"
        FlexNode{justify_main:Center}
        "text"
            TextLine{ text: "Tabs education"}
            TextLineColor(Hsla{ hue:60 saturation:0.55 lightness:0.55 alpha:1.0 })

    //Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        "info"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "Info" }
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })
            
        "exit"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "Exit button"}
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })

    //This is what changes based on menu selection
    "tab_content"
        GridNode{ height:25vh }
        BackgroundColor(Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 })

    "footer_content"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "I don't change" }
                TextLineColor(Hsla{hue:0 saturation:0.00 lightness:0.85 alpha:1.0})
        


//tab content that will be spawned at runtime

"info_tab"
    FlexNode{justify_main:Center}
    TextLine{text:"You are in the info tab"}

"exit_tab"
    //Not implemented
    FlexNode{justify_main:Center}
    TextLine{text:"Click me to quit"}
```

### Filling in tab_content

Let's run this and see our ui.

- Our starting code has an empty lightly coloured square box where our tab content should go.
- Two buttons are above the square box these will be our tab menus.
- A title and footer they are there to serve as normal constant values regardless of tab.

First thing to do now is spawn `info_tab` when `info` is clicked.

```rs
    //Snip
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            //Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    //Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "info_tab"), &mut s);
                });
            });
        });
```
