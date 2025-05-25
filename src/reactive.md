# Reactivity

This section will be using `bevy_cobweb` note that this is a companion crate of `bevy_cobweb_ui`.
Ensure that `bevy_cobweb` is added as a dependency.

## Broadcasts

Perhaps the simplest general purpose way to ensure that your UI is up to date.
Broadcasting tells your ui when it is time to update.

Lets get straight to it.

```rust
use std::time::Duration;

use bevy::{prelude::*, time::common_conditions};
use bevy_cobweb::prelude::*;
use bevy_cobweb_ui::prelude::*;

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
        .add_systems(
            Update,
            update_ui.run_if(common_conditions::on_timer(Duration::from_millis(500))),
        )
        .run();
}

//dummy entity data
#[derive(Component)]
struct Score(usize);

//see the broadcast below
struct ScoreUpdate;

fn build_ui(mut c: Commands, mut s: SceneBuilder) {
    //Just going to initialise score here for this example
    c.spawn(Score(0));

    c.spawn(Camera2d);
    c.ui_root()
        .spawn_scene(("main.cob", "main_scene"), &mut s, |scene_handle| {
            scene_handle.update_on(
                broadcast::<ScoreUpdate>(), // This is what listens to the update, it will run first time as well
                //TargetId is compulsory rest of the parametrs is just like any other system
                |id: TargetId, mut editor: TextEditor, score: Query<&Score>| {
                    let score = score.single()?;
                    write_text!(editor, *id, "{}", score.0);

                    DONE //Allows cobweb to handle results
                },
            );
        });
}

//New logic to increment the score then let cob know to update
fn update_ui(mut score: Query<&mut Score>, mut cmd: Commands) -> Result {
    let mut score = score.single_mut()?;
    score.0 += 1;

    //Tell the UI to update
    cmd.react().broadcast(ScoreUpdate);

    println!("{}", score.0);
    Ok(())
}
```

main.cob

```rust
#scenes
"main_scene"
    TextLine{ text: "Hello, World!" } //this text will be gone soon


```

This program just updates a number every 500 milliseconds, cobweb updates the ui at the time of the changes.

There is not much more to add other then you can have unlimited listeners and you may think of other non-ui purposes for broadcasts.


## TODO ReactResource
For resources you can use `ReactResource`.

TODO rework the example
