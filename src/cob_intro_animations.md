# Animations and State

Rerun your program built in the previous chapters if you have closed it already.

To start off let's add hovering changing background colours based on user hovering.

## Hovering and pressing

```rs
#scenes
"scene"
    AbsoluteNode{left:40% }
    TextLine{ text: "Hello, World!, I am writing using cobweb " }
    
    Animated<BackgroundColor>{
        idle:#FF0000 // You can input colours in other formats
        hover:Hsla{ hue:120 saturation:1.0 lightness:0.50 alpha:1.0 }
        press:Hsla{ hue:240 saturation:1.0 lightness:0.50 alpha:1.0 }
    }
```

Animated is a loadable that works with implementers of the [AnimatableAttribute trait](https://docs.rs/bevy_cobweb_ui/latest/bevy_cobweb_ui/sickle_ext/trait.AnimatableAttribute.html)
One example implementer is background colour.
