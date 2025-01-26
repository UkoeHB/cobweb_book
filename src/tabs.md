# Tabs

## Goals

- Create a window with tabs that can be selected from.
- Learn about radio buttons.
- Learn about `ControlRoot` and `ControlMembers` for multi-entity interactions and states.

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

`FlexNode` is used to center the text on some grid column titles.

Note we could add abstractions to tidy up the code, but we will stick to simplicity at the expense of verbosity.
Once we have the functionality sorted we look at some abstractions.

Hopefully this code is also mostly familiar from previous chapters.

```rs
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
        

// Tab content that will be spawned at runtime

"info_tab"
    FlexNode{justify_main:Center}
    TextLine{text:"You are in the info tab"}

"exit_tab"
    //Not implemented
    FlexNode{justify_main:Center}
    TextLine{text:"Click me to quit"}
```

### Filling in `tab_content`

Let's run this and see our ui.

- Our starting code has an empty lightly coloured square box where our tab content should go.
- Two buttons are above the square box. These will be our tab menus.
- A title and footer are there to serve as normal constant values regardless of tab.

First thing to do now is spawn `info_tab` when `info` is clicked.

```rs
    //Snip
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            // Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "info_tab"), &mut s);
                });
            });
        });
```

Now lets do the same for the exit tab.

```rs
        //Snip    
        c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            // Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "info_tab"), &mut s);
                });
            });
            // handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "exit_tab"), &mut s);
                });
            });
        });

```

We also need to clear the tab contents upon selecting a new tab. Note that we use `?` syntax and end the closure with `DONE`.

```rs
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            // Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    c.get_entity(tab_content_entity).result()?.despawn_descendants();

                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "info_tab"), &mut s);
                    DONE
                });
            });
            // handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    c.get_entity(tab_content_entity).result()?.despawn_descendants();

                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "exit_tab"), &mut s);
                    DONE
                });
            });
        });
```

Ok we got it working, but its not really fun or intuitive. We still have some problems to solve.
- The tab selection doesn't communicate which tab is active.
- It should start with one tab selected.

We will start with communication.

### Styling our tab headers

#### Initial Cob Changes

The plan is to distinguish between selected and deselected tabs by changing the background colour.

To do this we need to use radio buttons.

Adding this as below.

```rs
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup //<--- Sets up state for a group of RadioButtons
        "info"
            RadioButton //<--- New Radio Button
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "Info" }
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })
            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton //<---- New Radio Button
            "text"
                TextLine{ text: "Exit button" }
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })

```

These changes by themselves will not have any noticable impact. However, one thing we should change is switching from `on_pressed` to `on_select` since the radio button widget will be selecting tabs for us on press.

```rs
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            // Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                scene_handle.on_select(move |mut c: Commands, mut s: SceneBuilder| { //<--- now on_select
                    c.get_entity(tab_content_entity).result()?.despawn_descendants();

                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "info_tab"), &mut s);
                    DONE
                });
            });
            // handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                scene_handle.on_select(move |mut c: Commands, mut s: SceneBuilder| { //<--- now on_select
                    c.get_entity(tab_content_entity).result()?.despawn_descendants();

                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "exit_tab"), &mut s);
                    DONE
                });
            });
        });
```

Next we will use pseudo states provided by the radio button widget to show tab selection.

#### Next COB Changes

Lets start with the code to add colours.

```rs
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup
        "info"
            RadioButton
            FlexNode{justify_main:Center}
            // New colour logic
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]  
            "text"
                TextLine{ text: "Info" }
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })
            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton
            // New colour logic
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]  
            "text"
                TextLine{ text: "Exit button" }
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })

```
Let's explain this code.

`Animated<BackgroundColor>` allows you to define values based on the three css interaction modes `idle`/`hover`/`press`.

By encapsulating with `Multi` we can specify multiple animation sets tied to the entity's pseudostates.

```rs
Multi<Animated<BackgroundColor>>[
    {
        idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
    }
    {
        state: [Selected]
        idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
    }
]  
```
Now when our tabs change, we know which tab we have open!


### Further styling, and sharing state

I have added some more stying without incident.

```rs
#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh   }
    BackgroundColor(Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 })
    "title"
        FlexNode{justify_main:Center}
        "text"
            TextLine{ text: "Tabs education" }
            TextLineColor(Hsla{ hue:60 saturation:0.55 lightness:0.55 alpha:1.0 })

    // Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup
        "info"
            RadioButton
            FlexNode{justify_main:Center}
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover: Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  

            "text"
                TextLine{ text: "Info" }
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })
            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla { hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover: Hsla { hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]  
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  
            "text"
                TextLine{ text: "Exit button" }
                TextLineColor(Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 })

    // This is what changes based on menu selection
    "tab_content"
        GridNode{ height:25vh }
        BackgroundColor(Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 })

    "footer_content"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "I don't change" }
                TextLineColor(Hsla{hue:0 saturation:0.00 lightness:0.85 alpha:1.0})
        

// Tab content that will be spawned at runtime

"info_tab"
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"You are in the info tab"}

"exit_tab"
    // Not implemented
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"Click me to quit"}
```

Next bit of styling I want to add is text changing colour on hover.

```rs
    // Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup
        "info"
            RadioButton
            FlexNode{justify_main:Center}
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover: Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  

            "text"
                TextLine{ text: "Info" }
                // New Text Colour logic
                Multi<Animated<TextLineColor>>[
                    {
                        idle: Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
                        hover: Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }
                    }
                    {
                        state: [Selected]
                        idle: #FFFF00
                    }
                ]  

            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover: Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]  
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  
            "text"
                TextLine{ text: "Exit button" }
                // New Text Colour logic
                Multi<Animated<TextLineColor>>[
                    {
                        idle: Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
                        hover: Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }
                    }
                    {
                        state: [Selected]
                        idle:#FFFF00
                    }
                ]  
```

We have a problem. The text only changes colour when you hover on the text directly, but we want it change when any part of the button is hovered.

This is a completely unforseen situation that I did not just invent for an example!

#### ControlGroup

`ControlMember` lets nodes respond to state information from the `ControlRoot`.

```rs
    // Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup
        "info"
            RadioButton
            FlexNode{justify_main:Center}
            ControlRoot //<--- Shares hover state
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover: Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  

            "text"
                TextLine{ text: "Info" }
                ControlMember //<--- reads hover state
                Multi<Animated<TextLineColor>>[
                    {
                        idle: Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
                        hover: Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }
                    }
                    {
                        state: [Selected]
                        idle: #FFFF00
                    }
                ]  

            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton 
            ControlRoot //<--- Shares hover state
            Multi<Animated<BackgroundColor>>[
                {
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover: Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }
                }
                {
                    state: [Selected]
                    idle: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]  
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color: Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  
            "text"
                ControlMember //<--- reads hover state
                TextLine{ text: "Exit button" }
                Multi<Animated<TextLineColor>>[
                    {
                        idle: Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
                        hover: Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }
                    }
                    {
                        state: [Selected]
                        idle:#FFFF00
                    }
                ]  
```

## Abstractions: default selection and cleanup

Next chapter we will introduce new abstractions to make our COB code more managable.

We will also handle selecting the starting tab.
