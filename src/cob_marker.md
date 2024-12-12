# Custom marker component

In bevy it's common to have marker components. That is, components with no data that mark the entity as having some user-defined purpose. There are at least two ways to do this.


Before looking at either approach we should set some goals.

Goals:
- Add an exit button to the interface.
- Add a button to despawn the interface.
- On despawning add another button to respawn the interface.

Here is our code that we've built so far, adding in a `MainInterface` marker component.
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
                            println!("You clicked {}", i);
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

And here is the cob file we've built, adding in some small button scenes:
```rust
#scenes
"main_scene"
    AbsoluteNode{left:40% flex_direction:Column}
    "cell"
        Animated<BackgroundColor>{
            idle:#FF0000
            hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
            press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
        }
        NodeShadow{color:#FF0000 spread_radius:10px blur_radius:5px}
        "text"
            TextLine{text:"Hello, World!, I am writing using cobweb "}


"number_text"
    "cell"
        "text"
            TextLine{text:"placeholder"}
            TextLineColor(Hsla{hue:45 saturation:1.0 lightness:0.5 alpha:1.0})
            Interactive


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

The exit and despawn buttons could just as easily be added as children of `main_scene`.

## Rust approach

Let's look at the first way of doing this using what is likely to be a more familiar approach to you.

```rs
#[derive(Component)]
struct MainInterface;

fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.spawn(Camera2d);
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene.insert(MainInterface); // <-- add the marker component
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.edit("cell::text", |loaded_scene| {
                        loaded_scene.update_text(i.to_string());
                        loaded_scene.on_pressed(move|/* We write arbitary bevy parameters here*/|{
                            println!("You clicked {}", i);
                        });
                    });
                });
            }
        });
}
```
Now let's add the despawning button. We can use `on_pressed` along with a normal bevy query.

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

            // NEW: despawn button
            loaded_scene.load_scene_and_edit(("main.cob", "despawn_button"), |loaded_scene| {
                // Despawn the main interface on press.
                // Notice this looks like a normal bevy query.
                loaded_scene.on_pressed(
                    |interface_query: Query<Entity, With<MainInterface>>,
                     mut commands: Commands| {
                        // Cobweb callbacks can use `?` if you return `OK` or `DONE`.
                        // The .result() converts Options to Results.
                        commands
                            .get_entity(interface_query.get_single()?)
                            .result()?
                            .despawn_recursive();
                        OK
                    },
                );
            });
        });
}
```

Now let's add the exit button, which will not by any more difficult:

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
                            println!("You clicked {}", i);
                        });
                    });
                });
            }
            loaded_scene.load_scene_and_edit(("main.cob", "despawn_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |interface_query: Query<Entity, With<MainInterface>>,
                     mut commands: Commands| {
                        commands
                            .get_entity(interface_query.get_single()?)
                            .result()?
                            .despawn_recursive();
                        OK
                    },
                );
            });

            // NEW: exit button
            loaded_scene.load_scene_and_edit(("main.cob", "exit_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |mut commands: Commands, focused_windows: Query<Entity, With<Window>>| {
                        let window = focused_windows.get_single()?;
                        commands.get_entity(window).result()?.despawn();
                        OK
                    },
                );
            });
        });
}
```

### Spawn new interface

Let's add a system for making respawn buttons. The respawn button will spawn the main interface.

```rs
fn spawn_respawn_button(mut c: Commands, mut s: ResMut<SceneLoader>) {
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
                            .get_entity(interface_query.get_single()?)
                            .result()?
                            .despawn_recursive();
                        // NEW: spawn the respawn button
                        commands.run_system_cached(spawn_respawn_button);
                    },
                );
            });
```

`run_system_cached` is a convenient way to call a function arbitrarily when you don't need to supply data. Consider observers if you do. You can also use `syscall` from `bevy_cobweb`.

We can now despawn the main interface. Another approach could have been writing `run_system_cached` as an observer, and sending an event to trigger it in our `"despawn_button"` callback.

All there is to do now is fill in the respawn logic. There is not much to discuss so will see the updated code below.

```rs
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
                            .get_entity(interface_query.get_single()?)
                            .result()?
                            .despawn_recursive();
                        commands.run_system_cached(spawn_other_interface);
                        OK
                    },
                );
            });
            loaded_scene.load_scene_and_edit(("main.cob", "exit_button"), |loaded_scene| {
                loaded_scene.on_pressed(
                    |mut commands: Commands, focused_windows: Query<Entity, With<Window>>| {
                        let window = focused_windows.get_single()?;
                        commands.get_entity(window).result()?.despawn();
                        OK
                    },
                );
            });
        });
}

fn spawn_respawn_button(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.ui_root()
        .load_scene_and_edit(("main.cob", "respawn_button"), &mut s, |loaded_scene| {
            let entity = loaded_scene.id();
            loaded_scene.on_pressed(move |mut commands: Commands| {
                commands.get_entity(entity).result()?.despawn_recursive();
                commands.run_system_cached(spawn_main_interface);
                OK
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

To despawn the current scene we can get the entity using `.id()`.

## Moving MainInterface to a cob file

Now let's try out another way to add marker components via cob files.

We need to add some derives:

```rs

#[derive(Component, Default, PartialEq, Reflect)]
struct MainInterface;
```

Now let's remove the `.insert(MainInterface)` and register the component type.

```rs
use bevy::prelude::*;
use bevy_cobweb_ui::prelude::*;

//-------------------------------------------------------------------------------------------------------------------

#[derive(Component, Default, PartialEq, Reflect)]
struct MainInterface;

fn build_ui(mut c: Commands) {
    c.spawn(Camera2d);
    c.run_system_cached(spawn_main_interface);
}

fn spawn_main_interface(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            // <-- We no longer have insert here
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            // ...
        });
}

fn spawn_respawn_button(mut c: Commands, mut s: ResMut<SceneLoader>) {
    // ...
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
        .register_component_type::<MainInterface>() // <-- This allows cob to load this type
        .load("main.cob")
        .add_systems(OnEnter(LoadState::Done), build_ui)
        .run();
}
```

Let's now add `MainInterface` to our cob file.

```rs
#scenes
"main_scene"
    MainInterface // <-- NEW
    AbsoluteNode{left:40%,flex_direction:Column}
    "cell"
        Animated<BackgroundColor>{
            idle:#FF0000
            hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
            press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
        }
        NodeShadow{color:#FF0000 spread_radius:10px blur_radius:5px}
        "text"
            TextLine{text:"Hello, World!, I am writing using cobweb "}


"number_text"
    "cell"
        "text"
            TextLine{text:"placeholder"}
            TextLineColor(Hsla{hue:45 saturation:1.0 lightness:0.5 alpha:1.0})
            Interactive


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

We are now done.
