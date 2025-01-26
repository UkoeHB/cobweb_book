# Cob files
Cob allows you to separate your UI from your code, in addition to enabling hot reloading of changes. Cob is whitespace-sensitive.

## Sections
Cob files are made up of sections. For now, we will explore one type of section.

### Scenes

Scenes are declared following the `#scenes` keyword. Scenes are given a name. In our example, we use `"main_scene"`... extremely creative. We use scene root nodes to load scenes Cobweb. We will need to recompile to add new scenes to our app.


#### Extending our example.

First, rerun the program if you have closed it.  We want to see the hot reloading in action.

Let's change our cob file to be something like this:

```rs
#scenes
"main_scene"
    TextLine{ text: "Hello, World!, I am writing using cobweb " }
```

#### Loadables

Loadables are Rust types that can be added to the scene nodes. For now, we will only explore one type of loadable:

Types that implement the [instruction trait](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/loading/trait.Instruction.html).

`TextLine` is an example of a loadable.

*Loadables should not have space between their name and the opening `{`.*

let's add another called `AbsoluteNode`:

```rs
#scenes
"main_scene"
    AbsoluteNode{left:40% top:30vh}
    TextLine{ text: "Hello, World!, I am writing using cobweb" }

```
We have now moved the text around. You can also experiment with other units such as `40px 40vw`.

Separate fields are not comma separated.


let's try another loadable `BackgroundColor`:

```rs
#scenes
"main_scene"
    AbsoluteNode{left:40%}
    BackgroundColor(#FFFF00)
    TextLine{ text: "Hello, World!, I am writing using cobweb " }

```
We can see this does not contrast well with the text without recompiling!

Hex values convert to `Srgba` colours.

#### Animations 

Let's add hovering effects. We will start off by changing the background colours based on user hovering.

##### Hovering and pressing

```rs
#scenes
"main_scene"
    AbsoluteNode{left:40%}
    TextLine{ text: "Hello, World!, I am writing using cobweb " }
    
    Animated<BackgroundColor>{
        idle:#FF0000 // You can also input colours in other formats
        hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
        press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
    }
```

`Animated` is a loadable that works with implementers of the [AnimatableAttribute trait](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/sickle_ext/trait.AnimatableAttribute.html). One example implementer is `BackgroundColor`.


#### Next 

Next we will look at how we know what loadables we can use based on the documentation, and what fields are available in each loadable.
