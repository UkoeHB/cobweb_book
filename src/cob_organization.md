# Cob file organization

## #defs section

Cob files can have constants that let you define common values to be used all over your cob files. This will allow you to make changes in one place to impact multiple nodes.

Let's do an example.

```rs
// #defs is another type of section. So far we have only been doing #scenes.
#defs
$text_colour = Hsla{hue:45 saturation:1.0 lightness:0.5 alpha:1.0}

#scenes
"main_scene"
    MainInterface
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
            TextLineColor($text_colour) // <-- now uses a constant
            Interactive


"exit_button"
    TextLine{text:"Exit"}
    TextLineColor($text_colour) // <-- now uses a constant
    Interactive
"despawn_button"
    TextLine{text:"Despawn"}
    TextLineColor($text_colour) // <-- now uses a constant
    Interactive
"respawn_button"
    TextLine{text:"Respawn"}
    TextLineColor($text_colour) // <-- now uses a constant
    Interactive
```

## #manifest and #import

You can further extend this over many files using `#manifest` and `#import` sections.

### #manifest

Manifests let you load many files recursively. This way you only need to write `.load("manifest.cob")` once in your app.

Make a central manifest file at `assets/manifest.cob`:

```
#manifest
"main.cob" as main
"ui/colour_scheme.cob" as cs
```

The `as main` defines a manifest key for the file `main.cob`.

Now replace the `.load("main.cob")` in your app with `.load("manifest.cob")`. You can also simplify `.load_scene_and_edit(("main.cob", "main_scene"), ...)` to `.load_scene_and_edit(("main", "main_scene"), ...)`, using the main file's manifest key.

### #import

Imports let you bring in `#defs` from other files.

In any file where you want to import defs you can do as below.
```
#import
cs as colours
```

Defs from `cs` are scoped to the import alias `colours`:

```
BackgroundColor($colours::background_colour)
```


