---
title: How to draw custom paths/lines in Android usign PathEffect
date: 2020-06-08 12:41:05
author: Ashish Jajoria
categories:
- ["Android"]
tags: 
- android
- kotlin
- canvas
- paint
- path
- PathDashPathEffect
- DashPathEffect
- CornerPathEffect
- ComposePathEffect
- draw custom path
- Ashish Jajoria
---

We can draw simple lines and shapes by using `path.lineTo`, `path.moveTo` etc. But sometimes we have requirements to draw a line in pattern, for example: simple dashed, two lines where 1 is continuous and other one is dashed etc.

![Lines Drawn By PathEffects](/blog/Android/drawing-custom-paths-in-android/lines.png)

## Getting started

First we'll have to create a small path that we want to repeat in the final path

```kotlin
// These are some sample methods to generate a single block of path
// which will be repeated in the final path that we'll draw on canvas

// This will draw a small line with width 10px and length 30px
private fun makeDefaultLinePath(): Path {
        val p = Path()
        p.moveTo(-15f, 5f)
        p.lineTo(15f, 5f)
        p.lineTo(15f, -5f)
        p.lineTo(-15f, -5f)
        return p
    }

// This will draw two small lines with width 4px and length 30px
// They'll have 4px space between them
private fun makeDoubleLinePath(): Path {
    val p = Path()
    p.moveTo(-15f, 6f)
    p.lineTo(15f, 6f)
    p.lineTo(15f, 2f)
    p.lineTo(-15f, 2f)
    p.close()
    p.moveTo(-15f, -6f)
    p.lineTo(15f, -6f)
    p.lineTo(15f, -2f)
    p.lineTo(-15f, -2f)
    return p
}

// This will draw two small lines
// One with width 4px and length 15px
// Other with width 4px and length 30px
// They'll have 4px space between them
private fun makeBrokenSolidLinePath(): Path {
    val p = Path()
    p.moveTo(-15f, 6f)
    p.lineTo(0f, 6f)
    p.lineTo(0f, 2f)
    p.lineTo(-15f, 2f)
    p.close()
    p.moveTo(-15f, -6f)
    p.lineTo(15f, -6f)
    p.lineTo(15f, -2f)
    p.lineTo(-15f, -2f)
    return p
}
```

We've create our building blocks of the final path. Now we'll be setting PathEffect to the paint that will draw these blocks.

```kotlin
// First setup your paint object
val paint = Paint()
paint.style = Paint.Style.STROKE
paint.strokeWidth = 10f
paint.color = Color.YELLOW

// Declare your pathDashPathEffect
var pathDashPathEffect: PathDashPathEffect? = null

// Define your pathDashPathEffect
pathDashPathEffect = PathDashPathEffect(makeDoubleLanePath(),           //Your building block
                                        45f,                            //At how much distance the next block should be drawn from the current block's starting point
                                        0f,                             //Phase value
                                        PathDashPathEffect.Style.MORPH) //EffectStyle

// Set your defined pathDashPathEffect to your paint object
pathDashPathEffect?.let { effect ->
    paint.pathEffect = effect
}
```

We have our paint object with pathEffect with us. Now we'll be drawing a path that we actaully want to draw using this paint object.

```kotlin
var lineX1 = 0f
var lineX2 = 0f
var lineY1 = 0f
var lineY2 = 0f
val path = Path()

override fun onDraw(canvas: Canvas?) {
    super.onDraw(canvas)
    path.reset()
    path.moveTo(lineX1, lineY1)
    path.lineTo(lineX2, lineY2)

    canvas?.drawPath(path,  //This final path that we are drawing now
                     paint) //The one that we created earlier
}

```

## References:-

1. [Custom path line style when drawing on canvas](https://stackoverflow.com/questions/10907386/custom-path-line-style-when-drawing-on-canvas)
2. [what does path mean in PathDashPathEffect constructor](https://stackoverflow.com/questions/20068803/what-does-path-mean-in-pathdashpatheffect-constructor)
3. [PathDashPathEffect example](http://android-coding.blogspot.com/2014/05/pathdashpatheffect-example.html)

## Some good reads you may like:-

1. [Paytm Gateway Integration](https://nayan.co/blog/Ruby-on-Rails/paytm-gateway-integration/)
1. [Android Testing Strategy](https://nayan.co/blog/Android/Android-Testing-Strategy/)
