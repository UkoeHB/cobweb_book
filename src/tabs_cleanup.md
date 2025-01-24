# Cleanup

## COB cleanup

Our cob file from last chapter looks like the below.

It serves it's purpose but we can make this more maintainable.

Ideally we would have done it at the start but the choice was made to focus on the actual concepts

```
#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh   }
    BackgroundColor(Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 })
    "title"
        FlexNode{justify_main:Center}
        "text"
            TextLine{ text: "Tabs education" }
            TextLineColor(Hsla{ hue:60 saturation:0.55 lightness:0.55 alpha:1.0 })

    //Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup //Stores RadioButton State
        "info"
            RadioButton //New Radio Button Code
            FlexNode{justify_main:Center}
            ControlRoot
            Multi<Animated<BackgroundColor>>[
                {
                    idle:Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover:Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }

                }
                {
                    state:[Selected]
                    idle:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]
            Multi<Animated<NodeShadow>>[
                {
                    idle:{
                        color:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }

                }
                {
                    state:[Selected]
                    idle:{
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
                        idle:Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
                        hover:Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }
                    }
                    {
                        state:[Selected]
                        idle:#FFFF00
                    }
                ]  

            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton //New Radio Button Code
            ControlRoot
            Multi<Animated<BackgroundColor>>[
                {
                    idle:Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
                    hover:Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }

                }
                {
                    state:[Selected]
                    idle:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                }
            ]  
            Multi<Animated<NodeShadow>>[
                {
                    idle:{
                        color:Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }

                }
                {
                    state:[Selected]
                    idle:{
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
                        idle:Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
                        hover:Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }
                    }
                    {
                        state:[Selected]
                        idle:#FFFF00
                    }
                ]  

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
    "text"
        TextLine{text:"You are in the info tab"}

"exit_tab"
    //Not implemented
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"Click me to quit"}
```

### def

We should start with defining colour names as we repeat many colours that are related.

At the top of the file.

```
#defs
$window_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }


#scenes
"main_scene"
    AbsoluteGridNode{left:30% width:40vw min_height:30vh   }
    BackgroundColor($window_colour)
    //snip
```

We will be repeating this pattern

I postfixed all the names with colour as you can save other data like widths as well.
```
#defs
$window_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
$title_text_colour =Hsla{ hue:60 saturation:0.55 lightness:0.55 alpha:1.0 }

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

    //Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup //Stores RadioButton State
        "info"
            RadioButton //New Radio Button Code
            FlexNode{justify_main:Center}
            ControlRoot
            Multi<Animated<BackgroundColor>>[
                {
                    idle:$tab_background_colour
                    hover:$tab_hover_colour

                }
                {
                    state:[Selected]
                    idle:$tab_selected_background_colour
                }
            ]
            Multi<Animated<NodeShadow>>[
                {
                    idle:{
                        color:$box_shadow_colour
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }

                }
                {
                    state:[Selected]
                    idle:{
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
                        idle:$text_line_idle_colour
                        hover:$text_line_hover_colour
                    }
                    {
                        state:[Selected]
                        idle:$text_selected_colour
                    }
                ]  

            
        "exit"
            FlexNode{justify_main:Center}
            RadioButton //New Radio Button Code
            ControlRoot
            Multi<Animated<BackgroundColor>>[
                {
                    idle:$tab_background_colour
                    hover:$tab_hover_colour

                }
                {
                    state:[Selected]
                    idle:$tab_selected_background_colour
                }
            ]  
            Multi<Animated<NodeShadow>>[
                {
                    idle:{
                        color:$box_shadow_colour
                        x_offset: 0px,
                        y_offset: 0px,
                        spread_radius: 1px,
                        blur_radius: 1px,
                    }

                }
                {
                    state:[Selected]
                    idle:{
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
                        idle:$text_line_idle_colour
                        hover:$text_line_hover_colour
                    }
                    {
                        state:[Selected]
                        idle:$text_selected_colour
                    }
                ]  

    //This is what changes based on menu selection
    "tab_content"
        GridNode{ height:25vh }
        BackgroundColor($tab_selected_background_colour)

    "footer_content"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "I don't change" }
                TextLineColor($footer_text_colour)
        


//tab content that will be spawned at runtime

"info_tab"
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"You are in the info tab"}

"exit_tab"
    //Not implemented
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"Click me to quit"}
```
### Scene macros

Our two tab titles have an almost identical structure that we can make share.

Changing the template is as simple as overwriting the value.

Code is now much shorter and it's easier to change colour scheme at one location.

```
#defs
$window_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }
$title_text_colour =Hsla{ hue:60 saturation:0.55 lightness:0.55 alpha:1.0 }

$tab_background_colour = Hsla{ hue:221 saturation:0.5 lightness:0.15 alpha:0.5 }

$tab_selected_background_colour = Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }

$tab_hover_colour = Hsla{ hue:24 saturation:0.5 lightness:0.50 alpha:1.0 }

$box_shadow_colour = Hsla{ hue:221 saturation:0.5 lightness:0.20 alpha:0.5 }

$text_line_idle_colour = Hsla{ hue:60 saturation:0.85 lightness:0.90 alpha:1.0 }
$text_line_hover_colour =  Hsla{ hue:221 saturation:0.0 lightness:1.0 alpha:1.0 }

$text_selected_colour = #FFFF00

$footer_text_colour = Hsla{hue:0 saturation:0.00 lightness:0.85 alpha:1.0}


+tab_selector = \
    RadioButton //New Radio Button Code
    FlexNode{justify_main:Center}
    ControlRoot
    Multi<Animated<BackgroundColor>>[
        {
            idle:$tab_background_colour
            hover:$tab_hover_colour

        }
        {
            state:[Selected]
            idle:$tab_selected_background_colour
        }
    ]
    Multi<Animated<NodeShadow>>[
        {
            idle:{
                color:$box_shadow_colour
                x_offset: 0px,
                y_offset: 0px,
                spread_radius: 1px,
                blur_radius: 1px,
            }

        }
        {
            state:[Selected]
            idle:{
                color:$box_shadow_colour
                x_offset: 0px,
                y_offset: 0px,
                spread_radius: 10px,
                blur_radius: 10px,
            }
        }
    ]

    "text"
        TextLine{ text: "PlaceHolder" }
        ControlMember
        Multi<Animated<TextLineColor>>[
            {
                idle:$text_line_idle_colour
                hover:$text_line_hover_colour
            }
            {
                state:[Selected]
                idle:$text_selected_colour
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

    //Actual tab menu
    "tab_menu"
        GridNode{grid_auto_flow:Column}
        RadioGroup //Stores RadioButton State
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

    //This is what changes based on menu selection
    "tab_content"
        GridNode{ height:25vh }
        BackgroundColor($tab_selected_background_colour)

    "footer_content"
            FlexNode{justify_main:Center}
            "text"
                TextLine{ text: "I don't change" }
                TextLineColor($footer_text_colour)
        


//tab content that will be spawned at runtime

"info_tab"
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"You are in the info tab"}

"exit_tab"
    //Not implemented
    FlexNode{justify_main:Center}
    "text"
        TextLine{text:"Click me to quit"}
```

## Adding one more tab

With our fancy new scene macro it would be nice to add a new tab before we clean up the rust code.

```
    //Actual tab menu
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
            //Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            let tab_menu_entity = scene_handle.get("tab_menu").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                let info_button_entity = scene_handle.id();

                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    if let Some(mut tab_commands) = c.get_entity(tab_content_entity) {
                        tab_commands.despawn_descendants();
                    };

                    //Radio button select
                    c.ui_builder(tab_menu_entity)
                        .react()
                        .entity_event(info_button_entity, Select);
                    //Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "info_tab"), &mut s);
                });
            });
            //handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                let exit_button_entity = scene_handle.id();
                scene_handle.on_pressed(move |mut c: Commands, mut s: SceneBuilder| {
                    if let Some(mut tab_commands) = c.get_entity(tab_content_entity) {
                        tab_commands.despawn_descendants();
                    };
                    //Radio button select
                    c.ui_builder(tab_menu_entity)
                        .react()
                        .entity_event(exit_button_entity, Select);
                    //Use this instead of c.get_entity()
                    c.ui_builder(tab_content_entity)
                        .spawn_scene_simple(("main.cob", "exit_tab"), &mut s);
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

### Default tab
We can consolodate our tab change code, first lets get the information needed to load tab context.

```rs
#[derive(Event)]
struct TabChange {
    tab_menu_entity: Entity,
    tab_content_entity: Entity,
    button_entity: Entity,
    scene_name: String,
}

```

We can define an observer that will despawn other tab content, select our tab and spawn this tabs content.

```rs
fn tab_change(trigger: Trigger<TabChange>, mut c: Commands, mut s: SceneBuilder) {
    if let Some(mut tab_commands) = c.get_entity(trigger.tab_content_entity) {
        tab_commands.despawn_descendants();
    };

    c.ui_builder(trigger.tab_menu_entity)
        .react()
        .entity_event(trigger.button_entity, Select);

    c.ui_builder(trigger.tab_content_entity)
        .spawn_scene_simple(("main.cob", trigger.scene_name.clone()), &mut s);
}
```

Don't forget to add it.
`.add_observer(tab_change).


Now the code for calling the observers and setting up the initial tab

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Event)]
struct TabChange {
    tab_menu_entity: Entity,
    tab_content_entity: Entity,
    button_entity: Entity,
    scene_name: String,
}

fn build_ui(mut c: Commands, mut s: SceneBuilder) {
    c.spawn(Camera2d);
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            //Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            let tab_menu_entity = scene_handle.get("tab_menu").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                let info_button_entity = scene_handle.id();

                scene_handle.on_pressed(move |mut c: Commands| {
                    c.trigger(TabChange {
                        tab_menu_entity,
                        tab_content_entity,
                        button_entity: info_button_entity,
                        scene_name: "info_tab".to_string(),
                    });
                });

                //Set it up as initial tab
                scene_handle.commands().trigger(TabChange {
                    tab_menu_entity,
                    tab_content_entity,
                    button_entity: info_button_entity,
                    scene_name: "info_tab".to_string(),
                });
            });
            //calm tab

            scene_handle.edit("tab_menu::calm", |scene_handle| {
                let calm_button_entity = scene_handle.id();
                scene_handle.on_pressed(move |mut c: Commands| {
                    c.trigger(TabChange {
                        tab_menu_entity,
                        tab_content_entity,
                        button_entity: calm_button_entity,
                        scene_name: "calm_tab".to_string(),
                    });
                });
            });
            //handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                let exit_button_entity = scene_handle.id();
                scene_handle.on_pressed(move |mut c: Commands| {
                    c.trigger(TabChange {
                        tab_menu_entity,
                        tab_content_entity,
                        button_entity: exit_button_entity,
                        scene_name: "exit_tab".to_string(),
                    });
                });
            });
        });
}

fn tab_change(trigger: Trigger<TabChange>, mut c: Commands, mut s: SceneBuilder) {
    if let Some(mut tab_commands) = c.get_entity(trigger.tab_content_entity) {
        tab_commands.despawn_descendants();
    };

    c.ui_builder(trigger.tab_menu_entity)
        .react()
        .entity_event(trigger.button_entity, Select);

    c.ui_builder(trigger.tab_content_entity)
        .spawn_scene_simple(("main.cob", trigger.scene_name.clone()), &mut s);
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
        .add_observer(tab_change)
        .run();
}
```

TODO noticed redundant code final result should be this

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Event)]
struct TabChange {
    tab_content_entity: Entity,
    scene_name: String,
}

fn build_ui(mut c: Commands, mut s: SceneBuilder) {
    c.spawn(Camera2d);
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            //Get entity to place in our scene
            let tab_content_entity = scene_handle.get("tab_content").id();
            scene_handle.edit("tab_menu::info", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands| {
                    c.trigger(TabChange {
                        tab_content_entity,
                        scene_name: "info_tab".to_string(),
                    });
                });

                //Set it up as initial tab
                scene_handle.commands().trigger(TabChange {
                    tab_content_entity,
                    scene_name: "info_tab".to_string(),
                });
            });
            //calm tab

            scene_handle.edit("tab_menu::calm", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands| {
                    c.trigger(TabChange {
                        tab_content_entity,
                        scene_name: "calm_tab".to_string(),
                    });
                });
            });
            //handling exit tab
            scene_handle.edit("tab_menu::exit", |scene_handle| {
                scene_handle.on_pressed(move |mut c: Commands| {
                    c.trigger(TabChange {
                        tab_content_entity,
                        scene_name: "exit_tab".to_string(),
                    });
                });
            });
        });
}

fn tab_change(trigger: Trigger<TabChange>, mut c: Commands, mut s: SceneBuilder) {
    if let Some(mut tab_commands) = c.get_entity(trigger.tab_content_entity) {
        tab_commands.despawn_descendants();
    };

    c.ui_builder(trigger.tab_content_entity)
        .spawn_scene_simple(("main.cob", trigger.scene_name.clone()), &mut s);
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
        .add_observer(tab_change)
        .run();
}
```
