# Loadables and their fields

So far you have just been given the types to modify. This page aims to explain the different data types in cob files.
When you read the documentation of each loadable hopefully you will be able to easily implement it.

Rerun your program if you have closed it.


## Node shadow

let's start with adding a `NodeShadow` to our example. The documentation can be found [here](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/ui_bevy/struct.NodeShadow.html).

At the time of writing it looked like this:
```rust
pub struct NodeShadow {
    pub color: Color,
    pub x_offset: Val,
    pub y_offset: Val,
    pub spread_radius: Val,
    pub blur_radius: Val,
}
````

Let's start with `Val` before color as it is slightly simpler.

### Val

`Val` variants can be written with special units (`px, %, vw, vh, vmin, vmax`) and the keyword `auto`. For example, `10px` is equivalent to `Px(10)`.

### Colour

In our previous examples we have loaded colour using both RGB and HSLA.

#### Hex

Hex colours are a special data type in cob and can just be written as `#FF00FF` with an implied alpha of `FF`. You can also add an explicit alpha by adding in extra digits `#FF00FFEE`.

#### NewType collapsing

Loadable newtypes and newtype enum variants use newtype collapsing to simplify what you write. Newtypes are collapsed by discarding 'outer layers'.

An example we have used for HSLA. This is written in rust:

```rust
Color::Hsla(Hsla {
    hue: 240.0,
    saturation:1.0,
    lightness: 0.5,
    alpha:1.0,
})
```

And this is written in cob:

`Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }`

The rule for enums inside loadables is `A::B(C{ .. }) -> B{ .. }`. Newtype collapsing can occur for loadables like `A(vec![]) -> A[]`.

### Defaults

Not all fields in loadables need to be filled out. Every field left blank we will be defaulted. Only fields annotated with `#[reflect(default)]` can be skipped.

### Adding NodeShadow

With the above information we have enough to create our node shadow:

```rust
#scenes
"main_scene"
    AbsoluteNode{left:40%}
    TextLine{ text: "Hello, World!, I am writing using cobweb " }
    NodeShadow{color:#FF0000 spread_radius:10px blur_radius:5px} // <-- our new node shadow
    Animated<BackgroundColor>{
        idle:#FF0000 // You can also input colours in other formats
        hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
        press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
    }
    
```

Done

### Floats

Floats are written similar to how they are written in rust.

- Scientific notation: `1.2e3` or `1.2E3`.
- Integer-to-float conversion: `1` can be written instead of `1.0`.
- Keywords `inf`/`-inf`/`nan`: infinity, negative infinity, `NaN`.

Let's go ahead with an example using `size` in `TextLine`:

```rust
#scenes
"main_scene"
    AbsoluteNode{left:40%}
    TextLine{ text: "Hello, World!, I am writing using cobweb " size:150 } // <-- add the size here
    NodeShadow{color:#FF0000 spread_radius:10px blur_radius:5px}
    Animated<BackgroundColor>{
        idle:#FF0000 // You can input colours in other formats
        hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
        press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
    }

````

### Strings

Strings are handled similar to how rust string literals are handled.

- Enclosed by double quotes (e.g. `"Hello, World!"`).
- Escape sequences: standard ASCII escape sequences are supported (`\n`, `\t`, `\r`, `\f`, `\"`, `\\`), in addition to Unicode code points (`\u{..1-6 digit hex..}`).
- Multi-line strings: a string segment that ends in `\` followed by a newline will be concatenated with the next non-space character on the next line.
- Can contain raw Unicode characters.
