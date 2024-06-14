---
layout: post
title: "Animate a Groovin' Raccoon with Jetpack Compose!"
description: "This tutorial guides you through creating a playful dancing raccoon animation using Jetpack Compose's Animation API."
categories: [Android, Jetpack Compose]
tags: [animation, pedro]
image: assets/images/animation-racoon.jpg
beforetoc: ""
toc: false
featured: false
hidden: false
---

Unleash your inner animator and bring your UI to life! This tutorial guides you through creating a playful dancing raccoon animation using Jetpack Compose's Animation API.

See the animation in action!

<video width="360px" height="360px" controls="controls" style="display:block; margin-left: auto; margin-right: auto; margin-bottom:40px;" autoplay muted>
<source src="{{site.baseurl}}/assets/videos/animation-pedro-demo.mp4" type="video/mp4" />
</video>

## Getting assets

We'll need an image to animate and I found an SVG raccoon on <a href="https://freepik.com">Freepik</a> in a <a href="https://www.freepik.com/free-vector/forest-animal-collection_3388708.htm">forest animal collection</a>. Since we want to move the raccoon's head separately, we'll need to split the SVG into two files: one for the head and one for the body.

<img src="{{site.baseurl}}/assets/images/animation-racoon-original.png" />

There are many tools for editing SVGs and I used Inkscape. If editing the SVG isn't your thing, no worries! I've already prepared both head and body SVG files you can download by clicking <a href="#">here</a>. This way you can jump right into the animation fun.

## Setting up a project

Use the Resource Manager in Android Studio to import the SVGs into your project.

<img src="{{site.baseurl}}/assets/images/animation-resource-manager.png" />

We will need Jetpack Compose dependencies in the project, which you can find in <a href="https://developer.android.com/develop/ui/compose/setup">the documentation</a>.
Once everything is done, we are ready to write code!

## Creating a component

Let's start by building a simple component that shows a raccoon in a fixed position. We'll then add a parameter to make its head move down by X pixels.

```kotlin
@Composable
fun Racoon(
    modifier: Modifier = Modifier,
    headDrop: Float,
) {
    Box(
        modifier = modifier
    ) {
        Image(
            modifier = Modifier
                .fillMaxSize(),
            painter = painterResource(id = R.drawable.body),
            contentDescription = "Body"
        )
        Image(
            modifier = Modifier
                .fillMaxSize()
                .graphicsLayer { translationY = headDrop },
            painter = painterResource(id = R.drawable.head),
            contentDescription = "Head"
        )
    }
}
```

The head's vertical movement will be controlled by the <a href="https://developer.android.com/reference/kotlin/androidx/compose/ui/graphics/package-summary#graphicsLayer">Modifier.graphicsLayer()</a>.

## The Circular Mask

Another component we need to create is the background... or the foreground. This component will essentially act like a mask, covering the entire screen except for a specific area in the middle. It's important to position this component on top of (higher in the composable hierarchy) the raccoon component. This ensures the raccoon can be hidden behind the mask when needed.

To achieve that, we'll use the use the `.drawBehind()` modifier and the `DrawScope`.
Then we need to set up a `radialGradient` in the way it instantly changes from Transparent to Black in a calculated position.

```kotlin
@Composable
fun CircularMask(
    modifier: Modifier = Modifier,
    radius: Dp,
) {
    val radiusPx = with(LocalDensity.current) { radius.toPx() }
    Box(
        modifier = Modifier
            .fillMaxSize()
            .drawBehind {
                val brushRadius = radiusPx / minOf(size.width, size.height)
                val brush = Brush.radialGradient(
                    brushRadius to Color.Transparent,
                    brushRadius to Color.Black,
                )
                drawRect(brush)
            }
            .then(modifier),
    )
}
```

## Animation

Android Documentation provides a decision tree to choose right API for a specific type of animation. You can find it <a href="https://developer.android.com/develop/ui/compose/animation/choose-api">here</a>

To manage the animation effectively, we'll be using the <a href="https://developer.android.com/develop/ui/compose/animation/value-based#rememberinfinitetransition">InfiniteTransition</a> API.

We can create an instance of `InfiniteTransition` using the `rememberInfiniteTransition` function.
Once created, we can specify the value we want to animate, such as position or size, using the `animateFloat` function.
To create a continuously repeating animation, we'll utilize the `infiniteRepeatable` animation spec.
Finally, we can pass the animated value to our `Racoon` component to control its behavior based on the animation.

```kotlin
@Composable
fun RacoonAnimation(modifier: Modifier = Modifier) {
    val infiniteTransition = rememberInfiniteTransition(label = "Racoon")

    val headDrop by infiniteTransition.animateFloat(
        initialValue = 100f,
        targetValue = 150f,
        animationSpec = infiniteRepeatable(
            animation = tween(durationMillis = 250),
            repeatMode = RepeatMode.Reverse
        ),
        label = "Head Drop"
    )

    Racoon(headDrop = headDrop)
}
```

We also need to add 2 more animations: rotation and the vertical movement of the racoon.

```kotlin
val rotate by infiniteTransition.animateFloat(
    initialValue = 359f,
    targetValue = 0f,
    animationSpec = infiniteRepeatable(
        animation = tween(durationMillis = 6000, easing = LinearEasing),
        repeatMode = RepeatMode.Restart
    ),
    label = "Rotate"
)

val verticalMovement by infiniteTransition.animateFloat(
    initialValue = 600f,
    targetValue = 100f,
    animationSpec = infiniteRepeatable(
        animation = tween(durationMillis = 9000, easing = LinearEasing),
        repeatMode = RepeatMode.Reverse
    ),
    label = "Vertical Movement"
)
```

Now, let's bring everything together!

```kotlin
@Composable
fun RacoonAnimation(modifier: Modifier = Modifier) {
    val infiniteTransition = rememberInfiniteTransition(label = "Racoon")

    val headDrop by infiniteTransition.animateFloat(
        initialValue = 100f,
        targetValue = 150f,
        animationSpec = infiniteRepeatable(
            animation = tween(durationMillis = 250),
            repeatMode = RepeatMode.Reverse
        ),
        label = "Head Drop"
    )

    val rotate by infiniteTransition.animateFloat(
        initialValue = 359f,
        targetValue = 0f,
        animationSpec = infiniteRepeatable(
            animation = tween(durationMillis = 6000, easing = LinearEasing),
            repeatMode = RepeatMode.Restart
        ),
        label = "Rotate"
    )

    val verticalMovement by infiniteTransition.animateFloat(
        initialValue = 600f,
        targetValue = 100f,
        animationSpec = infiniteRepeatable(
            animation = tween(durationMillis = 9000, easing = LinearEasing),
            repeatMode = RepeatMode.Reverse
        ),
        label = "Vertical Movement"
    )

    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(Color.White)
            .graphicsLayer {
                rotationZ = rotate
            },
        contentAlignment = Alignment.Center,
    ) {
        Racoon(
            modifier = Modifier
                .size(300.dp)
                .graphicsLayer {
                    this.translationY = verticalMovement
                },
            headDrop = headDrop
        )
    }
    CircularMask(radius = 310.dp)
}
```

## Final thoughts

Congratulations! With the concepts covered in this tutorial, you've successfully created a basic animation in Jetpack Compose. Remember, practice makes perfect. Experiment with different animation types and values to create unique and engaging experiences for your users.

To see the complete code implementation, check out the link to my repository in the Useful Links section below.

## Useful links

- <a href="https://gist.github.com/GregKluska/9887c1f11a0eb41298cae290c3d426ee" target="_blank">Source Code - Greg Kluska GitHub
- <a href="https://developer.android.com/jetpack/androidx/releases/compose-animation" target="_blank">Compose Animation Documentation</a>
- <a href="https://developer.android.com/develop/ui/compose/animation/choose-api" target="_blank">Choose right animation API</a>
