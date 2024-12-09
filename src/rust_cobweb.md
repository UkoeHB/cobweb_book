# Using rust to update text

We will be using rust to modify the contents of text at runtime (no hot reloading).

## Modify the cob file
Lets setup the cob file as below.

```
"scene"
    AbsoluteNode{left:40%}
    "cell"
        Animated<BackgroundColor>{
            idle:#FF0000 // You can input colours in other formats
            hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
            press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
        }
        NodeShadow{color:#FF0000 spread_radius:10px blur_radius:5px }
        "text"
            TextLine{text:"Hello, World!, I am writing using cobweb "  } // change the textline here

```
We split position logic into a child node called `cell` which holds most of the positioning and styling logic.
`cell` has a child called `text`. Text is a minimal node responsible for just text stuff.

### Why we split text from styling
This is more of a html/CSS pattern then anything particular with cobweb but it is worth mentioning here.

It just turns out to be easier to position nodes then it is to position then text.


## Rust

<div class="warning">

At the time of writing the latest release does not have all the functions.

you may need to follow the main branch as below.

```
bevy_cobweb_ui = { git = "https://github.com/UkoeHB/bevy_cobweb_ui.git", features = [
  "hot_reload",
] }
```
</div>

### Updating text at runtime
Lets change the rust code to be as below.

```rs
fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.spawn(Camera2d);
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");
        });
}
```

We have changed `load_scene` to be `load_scene_and_edit`.

When we load `"main_scene"` in the cob file we automatically load all the child nodes recursively,
The second argument is a closure where we can use loaded scenes similar to commands along with
cobwebs extended commands. 

Inside the closure we call `get(cell::text)` which is basically a path syntax to go straight to the 
text node, it also possible to call `edit` on cell then call `update_text` inside the resulting closure.

Recompile and run the program you will see your text has changed to reflect the rust code.

### Spawning new nodes

Cobweb can also spawn multiple top level scenes lets start with an example.

Below we have our new scene called `number_text`.

If the concept of scenes was a bit confusing before this should clarify it a bit more.

```
#scenes
"scene"
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
```

Now lets change our rust code to spawn some scenes.

```rs
fn build_ui(mut c: Commands, mut s: ResMut<SceneLoader>) {
    c.spawn(Camera2d);
    c.ui_root()
        .load_scene_and_edit(("main.cob", "main_scene"), &mut s, |loaded_scene| {
            loaded_scene
                .get("cell::text")
                .update_text("My runtime text");

            //Spawning new ui nodes
            for i in (0..=10).into_iter() {
                loaded_scene.load_scene_and_edit(("main.cob", "number_text"), |loaded_scene| {
                    loaded_scene.get("cell::text").update_text(i.to_string());
                });
            }
        });
}
```

We now have some numbers that appear based on your run code.
We can still modify the cob files and change styling

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
```

#### Making nodes interactive

Setting our UI to react on user interfaces is essential, and easy first we need to load the interactive components.

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

```

Now we can use `on_pressed` in closures.

```rs
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
                        loaded_scene.on_pressed(move|/* We can write arbitrary bevy parameters here*/|{
                            println!("You clicked {}",i);
                        });
                    });
                });
            }
        });
}
 ```
