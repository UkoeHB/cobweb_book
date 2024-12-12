# Custom marker component

In bevy its common to have marker components, that is components which contains no data,
but does mark the entity as having some user defined purpose there are at least two ways to do this.


Before looking at either approach we should set the a common goal and setup code.

- Add an exit button to the interface.
- Add a button to despawn the interface.
- On despawning add another button to respawn the interface.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Component)]
struct MainInterface;

fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.spawn(Camera2d);
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.edit("cell::text", |loaded_scene| {
                        loaded_scene.update_text(i.to_string());
                        loaded_scene.on_pressed(move|/* We write arbitary bevy parameters here*/|{
                            println!("You clicked {}",i);
                        });
                    });
                });
            }
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


```
#scenes
"main_scene"
    AbsoluteNode{left:40%,flex_direction:Column}
    "cell"
        Animated<BackgroundColor>{
            idle:#FF0000 // You can input colours in other formats
            hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
            press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
        }
        NodeShadow{color:#FF0000 spread_radius:10px blur_radius:5px }
        "text"
            TextLine{text:"Hello, World!, I am writing using cobweb "  } // change the textline here


"number_text"
    "cell"
        "text"
            TextLine{text:"placeholder"}
            TextLineColor(Hsla{hue:45 saturation:1.0 lightness:0.5 alpha:1.0}) //still can change at runtime
            Interactive //Loads listeners for user interaction


"exit_button"
    TextLine{text:"Exit"}
    Interactive
"despawn_button"
    TextLine{text:"Despawn"}
    Interactive
"respawn_button"
    TextLine{text:"Respawn"}
    Interactive
```

I could just have easily made exit and despawn child of `main_scene` there is no significance to my decision.

## Rust approach

let's look at the first way of doing this using what is likely to be a more familiar approach to you.



```rs
#[derive(Component)]
struct MainInterface;

fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.spawn(Camera2d);
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene.insert(MainInterface); //add this to the scene
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.edit("cell::text", |loaded_scene| {
                        loaded_scene.update_text(i.to_string());
                        loaded_scene.on_pressed(move|/* We write arbitary bevy parameters here*/|{
                            println!("You clicked {}",i);
                        });
                    });
                });
            }
        });
}
```
Now let's add despawning button, we can use `on_pressed` along with a normal bevy query.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Component)]
struct MainInterface;

fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.spawn(Camera2d);
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene.insert(MainInterface);
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.edit("cell::text", |loaded_scene| {
                        loaded_scene.update_text(i.to_string());
                        loaded_scene.on_pressed(move|/* We write arbitary bevy parameters here*/|{
                            println!("You clicked {}",i);
                        });
                    });
                });
            }
            loaded_scene.load_scene_and_edit(("main.cob", "despawn_button"), |loaded_scene| {
            //button here despawns notice this looks like a normal bevy query
                loaded_scene.on_pressed(
                    |interface_query: Query<Entity, With<MainInterface>>,
                     mut commands: Commands| {
                        //I usually use tiny_bail instead of all this unwrapping
                        commands
                            .get_entity(interface_query.get_single().unwrap())
                            .unwrap()
                            .despawn_recursive();
                    },
                );
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

Now let's add the exit button which will not any more difficult

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Component)]
struct MainInterface;

fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.spawn(Camera2d);
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene.insert(MainInterface);
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.edit("cell::text", |loaded_scene| {
                        loaded_scene.update_text(i.to_string());
                        loaded_scene.on_pressed(move|/* We write arbitary bevy parameters here*/|{
                            println!("You clicked {}",i);
                        });
                    });
                });
            }
            loaded_scene.load_scene_and_edit(("main.cob", "despawn_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |interface_query: Query<Entity, With<MainInterface>>,
                     mut commands: Commands| {
                        commands
                            .get_entity(interface_query.get_single().unwrap())
                            .unwrap()
                            .despawn_recursive();
                    },
                );
            });
            loaded_scene.load_scene_and_edit(("main.cob", "exit_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |mut commands: Commands, focused_windows: Query<Entity, With<Window>>| {
                        let window = focused_windows.get_single().unwrap();
                        commands.entity(window).despawn();
                    },
                );
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
### spawn other interface

let's add a function for spawning the new interface

```rs
fn spawn_other_interface(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.ui_root()
        .load_scene_and_edit(("main.cob", "respawn_button"), &mut s, |loaded_scene| {
            //TODO respawning main interface
        });
}
```

Now call it on despawn

```rs
            loaded_scene.load_scene_and_edit(("main.cob", "despawn_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |interface_query: Query<Entity, With<MainInterface>>,
                     mut commands: Commands| {
                        commands
                            .get_entity(interface_query.get_single().unwrap())
                            .unwrap()
                            .despawn_recursive();
                        commands.run_system_cached(spawn_other_interface);
                    },
                );
            });
```

`run_system_cached` is a convenient way to call a function arbitrarily when you don't need to supply data.
Consider observers if you do.

We can now despawn the main interface, another approach could have been using an observer with an entity code.

All there is to do now is fill in the respawn logic there is not much to discuss so will see the code below.

```
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Component)]
struct MainInterface;

fn build_ui(mut c: Commands) {
    c.spawn(Camera2d);
    c.run_system_cached(spawn_main_interface);
}

fn spawn_main_interface(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene.insert(MainInterface);
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.edit("cell::text", |loaded_scene| {
                        loaded_scene.update_text(i.to_string());
                        loaded_scene.on_pressed(move|/* We write arbitary bevy parameters here*/|{
                            println!("You clicked {}",i);
                        });
                    });
                });
            }
            loaded_scene.load_scene_and_edit(("main.cob", "despawn_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |interface_query: Query<Entity, With<MainInterface>>,
                     mut commands: Commands| {
                        commands
                            .get_entity(interface_query.get_single().unwrap())
                            .unwrap()
                            .despawn_recursive();
                        commands.run_system_cached(spawn_other_interface);
                    },
                );
            });
            loaded_scene.load_scene_and_edit(("main.cob", "exit_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |mut commands: Commands, focused_windows: Query<Entity, With<Window>>| {
                        let window = focused_windows.get_single().unwrap();
                        commands.entity(window).despawn();
                    },
                );
            });
        });
}

fn spawn_other_interface(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.ui_root()
        .load_scene_and_edit(("main.cob", "respawn_button"), &mut s, |loaded_scene| {
            let entity = loaded_scene.id();
            loaded_scene.on_pressed(move |mut commands: Commands| {
                commands.get_entity(entity).unwrap().despawn_recursive();
                commands.run_system_cached(spawn_main_interface);
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
In spawn other interface we do get the entity using the `id` method.

## Moving MainInterface to a cob file

We first need to implement the `Instruction` trait for this item and derive a few more traits.

```rs

#[derive(Component, Default, PartialEq, Reflect)]
struct MainInterface;

impl Instruction for MainInterface {
    fn apply(self, _entity: Entity, _world: &mut World) {}

    fn revert(_entity: Entity, _world: &mut World) {}
}
```
We don't need the arguments in this case so we can ignore them.
now let's remove `insert code` and register the component type.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Component, Default, PartialEq, Reflect)]
struct MainInterface;

impl Instruction for MainInterface {
    fn apply(self, _entity: Entity, _world: &mut World) {}

    fn revert(_entity: Entity, _world: &mut World) {}
}

fn build_ui(mut c: Commands) {
    c.spawn(Camera2d);
    c.run_system_cached(spawn_main_interface);
}

fn spawn_main_interface(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
        //No longer have insert here
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.edit("cell::text", |loaded_scene| {
                        loaded_scene.update_text(i.to_string());
                        loaded_scene.on_pressed(move|/* We write arbitary bevy parameters here*/|{
                            println!("You clicked {}",i);
                        });
                    });
                });
            }
            loaded_scene.load_scene_and_edit(("main.cob", "despawn_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |interface_query: Query<Entity, With<MainInterface>>,
                     mut commands: Commands| {
                        commands
                            .get_entity(interface_query.get_single().unwrap())
                            .unwrap()
                            .despawn_recursive();
                        commands.run_system_cached(spawn_other_interface);
                    },
                );
            });
            loaded_scene.load_scene_and_edit(("main.cob", "exit_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |mut commands: Commands, focused_windows: Query<Entity, With<Window>>| {
                        let window = focused_windows.get_single().unwrap();
                        commands.entity(window).despawn();
                    },
                );
            });
        });
}

fn spawn_other_interface(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.ui_root()
        .load_scene_and_edit(("main.cob", "respawn_button"), &mut s, |loaded_scene| {
            let entity = loaded_scene.id();
            loaded_scene.on_pressed(move |mut commands: Commands| {
                commands.get_entity(entity).unwrap().despawn_recursive();
                commands.run_system_cached(spawn_main_interface);
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
        .register_component_type::<MainInterface>() //This is what allows cob to load this item
        .load("main.cob")
        .add_systems(OnEnter(LoadState::Done), build_ui)
        .run();
}
```

let's now add this to our COB file.

```
#scenes
"main_scene"
    MainInterface
    AbsoluteNode{left:40%,flex_direction:Column}
    "cell"
        Animated<BackgroundColor>{
            idle:#FF0000 // You can input colours in other formats
            hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
            press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
        }
        NodeShadow{color:#FF0000 spread_radius:10px blur_radius:5px }
        "text"
            TextLine{text:"Hello, World!, I am writing using cobweb "  } // change the textline here


"number_text"
    "cell"
        "text"
            TextLine{text:"placeholder"}
            TextLineColor(Hsla{hue:45 saturation:1.0 lightness:0.5 alpha:1.0}) //still can change at runtime
            Interactive //Loads listeners for user interaction


"exit_button"
    TextLine{text:"Exit"}
    Interactive //Loads listeners for user interaction
"despawn_button"
    TextLine{text:"Despawn"}
    Interactive //Loads listeners for user interaction
"respawn_button"
    TextLine{text:"Respawn"}
    Interactive //Loads listeners for user interaction
```

We are now done.
