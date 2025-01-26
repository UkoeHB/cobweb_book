# Cleanup

## COB cleanup

Our cob file from last chapter looks like below.

It serves it's purpose but we can make this more maintainable.

Ideally we would have done it at the start but the choice was made to focus on the basic concepts.

```rs
#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh}
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
            ControlRoot
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
                        color:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  

            "text"
                TextLine{ text: "Info" }
                ControlMember
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
            ControlRoot
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
                        color:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  
            "text"
                ControlMember
                TextLine{ text: "Exit button" }
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

### `#defs`

We should start with defining colour names as we repeat many colours that are related.

At the top of the file.

```rs
#defs
$window_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }


#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh   }
    BackgroundColor($window_colour)
    //snip
```

We will be repeating this pattern.

I postfixed all the names with colour to disambiguate from other data we could add here like widths.
```rs
#defs
$window_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
$title_text_colour = Hsla{ hue:60 saturation:0.55 lightness:0.55 alpha:1.0 }

$tab_background_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }

$tab_selected_background_colour = Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }

$tab_hover_colour = Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }

$box_shadow_colour = Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }

$text_line_idle_colour = Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
$text_line_hover_colour =  Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }

$text_selected_colour = #FFFF00

$footer_text_colour = Hsla{hue:0 saturation:0.00 lightness:0.85 alpha:1.0}

#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh   }
    BackgroundColor($window_colour)
    "title"
        FlexNode{justify_main:Center}
        "text"
            TextLine{ text: "Tabs education" }
            TextLineColor($title_text_colour)

    // Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup
        "info"
            RadioButton
            FlexNode{justify_main:Center}
            ControlRoot
            Multi<Animated<BackgroundColor>>[
                {
                    idle: $tab_background_colour
                    hover: $tab_hover_colour
                }
                {
                    state: [Selected]
                    idle: $tab_selected_background_colour
                }
            ]
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color:$box_shadow_colour
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color:$box_shadow_colour
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  

            "text"
                TextLine{ text: "Info" }
                ControlMember
                Multi<Animated<TextLineColor>>[
                    {
                        idle: $text_line_idle_colour
                        hover: $text_line_hover_colour
                    }
                    {
                        state: [Selected]
                        idle: $text_selected_colour
                    }
                ]  

            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton
            ControlRoot
            Multi<Animated<BackgroundColor>>[
                {
                    idle: $tab_background_colour
                    hover: $tab_hover_colour
                }
                {
                    state: [Selected]
                    idle: $tab_selected_background_colour
                }
            ]  
            Multi<Animated<NodeShadow>>[
                {
                    idle: {
                        color:$box_shadow_colour
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }
                }
                {
                    state: [Selected]
                    idle: {
                        color:$box_shadow_colour
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 10px,
                        blur_radius: 10px,
                    }
                }
            ]  
            "text"
                ControlMember
                TextLine{ text: "Exit button" }
                Multi<Animated<TextLineColor>>[
                    {
                        idle: $text_line_idle_colour
                        hover: $text_line_hover_colour
                    }
                    {
                        state: [Selected]
                        idle: $text_selected_colour
                    }
                ]  

    // This is what changes based on menu selection
    "tab_content"
        GridNode{ height:25vh }
        BackgroundColor($tab_selected_background_colour)

    "footer_content"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "I don't change" }
                TextLineColor($footer_text_colour)
        


// Tab content that will be spawned at runtime

"info_tab"
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"You are in the info tab"}
"calm_tab" // <-- We will be adding this below

"exit_tab"
    // Not implemented
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"Click me to quit"}
```

### Scene macros

Our two tab titles have an almost identical structure that we can share using a scene macro.

Changing the template is as simple as overwriting parts of its structure.

Code is now much shorter and it's easier to change the colour scheme.

```rs
#defs
$window_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
$title_text_colour = Hsla{ hue:60 saturation:0.55 lightness:0.55 alpha:1.0 }

$tab_background_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }

$tab_selected_background_colour = Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }

$tab_hover_colour = Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }

$box_shadow_colour = Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }

$text_line_idle_colour = Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
$text_line_hover_colour =  Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }

$text_selected_colour = #FFFF00

$footer_text_colour = Hsla{hue:0 saturation:0.00 lightness:0.85 alpha:1.0}


+tab_selector = \
    RadioButton
    FlexNode{justify_main:Center}
    ControlRoot
    Multi<Animated<BackgroundColor>>[
        {
            idle: $tab_background_colour
            hover: $tab_hover_colour
        }
        {
            state: [Selected]
            idle: $tab_selected_background_colour
        }
    ]
    Multi<Animated<NodeShadow>>[
        {
            idle: {
                color:$box_shadow_colour
                x_offset: 0px,
                y_offset: 0px,
                spread_radius: 1px,
                blur_radius: 1px,
            }
        }
        {
            state: [Selected]
            idle: {
                color:$box_shadow_colour
                x_offset: 0px,
                y_offset: 0px,
                spread_radius: 10px,
                blur_radius: 10px,
            }
        }
    ]

    "text"
        TextLine // Placeholder
        ControlMember
        Multi<Animated<TextLineColor>>[
            {
                idle: $text_line_idle_colour
                hover: $text_line_hover_colour
            }
            {
                state: [Selected]
                idle: $text_selected_colour
            }
        ]  
\

#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh   }
    BackgroundColor($window_colour)
    "title"
        FlexNode{justify_main:Center}
        "text"
            TextLine{ text: "Tabs education" }
            TextLineColor($title_text_colour)

    // Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup
        "info"
            +tab_selector{
                "text"
                    TextLine{text:"Info"}
            }
        "exit"
            +tab_selector{
                "text"
                    TextLine{text:"Exit"}
            }

    // This is what changes based on menu selection
    "tab_content"
        GridNode{ height:25vh }
        BackgroundColor($tab_selected_background_colour)

    "footer_content"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "I don't change" }
                TextLineColor($footer_text_colour)
        


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

## Adding one more tab

With our fancy new scene macro it would be nice to add a new tab before we clean up the rust code.

```rs
    // Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup //Stores RadioButton State
        "info"
            +tab_selector{
                "text"
                    TextLine{text:"Info"}
            }
        "calm"
            +tab_selector{
                "text"
                    TextLine{text:"Calm"}
            }
        "exit"
            +tab_selector{
                "text"
                    TextLine{text:"Exit"}
            }
```

Calm will just be empty

## Rust code

This is our rust code from the previous chapter.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

fn build_ui(mut c: Commands, mut s: SceneBuilder) {
    c.spawn(Camera2d);
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            // Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                scene_handle.on_select(move |mut c: Commands, mut s: SceneBuilder| {
                    c.get_entity(tab_content_entity).result()?.despawn_descendants();

                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "info_tab"), &mut s);
                    DONE
                });
            });
            // handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                scene_handle.on_select(move |mut c: Commands, mut s: SceneBuilder| {
                    c.get_entity(tab_content_entity).result()?.despawn_descendants();

                    // Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "exit_tab"), &mut s);
                    DONE
                });
            });
        });
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

### Starting tab

We can consolodate our tab change code into a function.

```rs
fn setup_tab_content(h: &mut UiSceneHandle, content_entity: Entity, scene: &'static str) {
    h.on_select(move |mut c: Commands, mut s: SceneBuilder| {
        c.get_entity(content_entity).result()?.despawn_descendants();
        c.ui_builder(content_entity)
            .spawn_scene_simple(("main.cob", scene), &mut s);
        DONE
    });
}
```

Now the code for using that function and setting up the starting tab.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

fn setup_tab_content(h: &mut UiSceneHandle, content_entity: Entity, scene: &'static str) {
    h.on_select(move |mut c: Commands, mut s: SceneBuilder| {
        c.get_entity(content_entity).result()?.despawn_descendants();
        c.ui_builder(content_entity)
            .spawn_scene_simple(("main.cob", scene), &mut s);
        DONE
    });
}

//-------------------------------------------------------------------------------------------------------------------

fn build_ui(mut c: Commands, mut s: SceneBuilder) {
    c.spawn(Camera2d);
    c.ui_root()
        .spawn_scene_simple(("main.cob", "main_scene"), &mut s, |scene_handle| {
            // Get entity to place our tab scenes in.
            let tab_content_entity = scene_handle.get("tab_content").id();

            scene_handle.edit("tab_menu::info", |scene_handle| {
                setup_tab_content(scene_handle, tab_content_entity, "info_tab");

                // Set this up as the starting tab.
                let id = scene_handle.id();
                scene_handle.react().entity_event(id, Select);
            });
            // calm tab
            scene_handle.edit("tab_menu::calm", |scene_handle| {
                setup_tab_content(scene_handle, tab_content_entity, "calm_tab");
            });
            // handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                setup_tab_content(scene_handle, tab_content_entity, "exit_tab");
            });
        });
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
