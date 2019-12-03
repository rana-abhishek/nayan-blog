---
title: Drawing Lines on Images
date: 2019-12-02 19:27:37
author: Diwakar Singh
tags:
category: Android
---

You can draw different types of line on Canvas in Android. You can change it’s color, stroke, effect, etc. You can redo or undo lines , can select and delete them. Here we will see the basics of drawing line on Canvas.

## Let’s get started

### Step -1: Add dependencies

```
// PhotoView
    implementation 'com.github.chrisbanes:PhotoView:2.3.0'
//Glide
    implementation 'com.github.bumptech.glide:glide:4.9.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.9.0'
// Timber for logging
    implementation "com.jakewharton.timber:timber:${rootProject.ext.timber_version}"
// Coroutines
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:${rootProject.ext.coroutines_version}"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:${rootProject.ext.coroutines_version}"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:${rootProject.ext.lifecycle_extensions_version}"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:${rootProject.ext.lifecycle_extensions_version}"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:${rootProject.ext.lifecycle_extensions_version}"
```

### Step -2: Create bitmap of Image and canvas to draw

The first step is to create a bitmap of Image on which you want to draw lines. We are using [Glide](https://github.com/bumptech/glide "Glide") to download image and to get bitmap from it.

```
private suspend fun getOriginalBitmapFromUrl(): Bitmap =
    withContext(Dispatchers.IO) {
        Glide.with(this@MainActivity)
            .asBitmap()
            .load(Constants.IMAGE_URL)
            .submit().get()
    }
```

### Step -3: Create event listener

Create an interface to handle onTouch events of photoviewattacher.

```
fun touchPoints(x: Float, y: Float)
fun closeStroke()
fun discardStroke()
```

### Step -4: Create your overlay view attacher

Now we create overlayview attacher, where we define onTouch events.

```
MotionEvent.ACTION_UP -> {
    if (multiTouch) {
        onPencilDrawListener.discardStroke()
        multiTouch = false
    } else {
        onPencilDrawListener.closeStroke()
    }
}

MotionEvent.ACTION_DOWN -> {
    if (!multiTouch && isSelectModeEnabled) {
        sendEventToListener(ev)
    } else {
        super.onTouch(v, ev)
    }
}

MotionEvent.ACTION_MOVE -> {
    if (!multiTouch) {
        sendEventToListener(ev)
    } else {
        super.onTouch(v, ev)
    }
}
```
### Step -5: Create your overlay photo view

Create your own imageview by extending it from photoview and added some properties

#### a. For selecting and deleting lines:

If the angle at touch point is an obtuse angle and area of triangle formed is under the threshold limit, then the current line is found to be selected

#### To calculate area:

```
abs((touchX * (y1 — y2) + x1 * (y2 — touchY) + x2 * (touchY — y1)) / 2)
```

where (x1,y1) and (x2,y2) are endpoints of selected line.

#### To calculate angle:

```
private fun calculateAngle(
    tapPoint: List<Float>,
    linePoint1: List<Float>,
    linePoint2: List<Float>
): Float {
    // Square of lengths be a2, b2, c2
    val a2 = lengthSquare(linePoint1, linePoint2)
    val b2 = lengthSquare(tapPoint, linePoint2)
    val c2 = lengthSquare(tapPoint, linePoint1)

    // length of sides be b, c
    val b = sqrt(b2)
    val c = sqrt(c2)

    // From Cosine law
    var alpha = acos((b2 + c2 - a2) / (2f * b * c))

    // Converting to degree
    alpha = (alpha * 180 / PI).toFloat()

    return alpha
}
```

#### b. For redo operation:

If last undo operation list is not empty, add the first line points from last undo operation list to overlay data list and remove it from there.

```
fun redo() {
    if (lastUndoOperation != null) {
        if (lastUndoOperation?.isDeleteOperation == true) {
            val ids = lastUndoOperation?.overlayDataList?.map { overlayData -> overlayData.id }
            if (ids != null) {
                for (data in overlayDataList) {
                    if (data.id in ids) {
                        data.isDeleted = true
                    }
                }
            }
        } else {
            lastUndoOperation?.overlayDataList?.first()?.let { overlayDataList.add(it) }
        }
        lastUndoOperation?.let { overlayDataOperations.add(it) }
        lastUndoOperation = null
        invalidate()
    }
}
```

#### c. For undo operation:

If there is any point present in overlay data list then delete it from there and added it in last undo operation list.

```
fun undo() {
    if (overlayDataOperations.size > 0) {
        val index = overlayDataOperations.lastIndex
        lastUndoOperation = overlayDataOperations[index]
        overlayDataOperations.removeAt(index)

        if (lastUndoOperation?.isDeleteOperation == true) {
            val ids = lastUndoOperation?.overlayDataList?.map { overlayData -> overlayData.id }
            if (ids != null) {
                for (data in overlayDataList) {
                    if (data.id in ids) {
                        data.isDeleted = false
                    }
                }
            }
        } else {
            val id = lastUndoOperation?.overlayDataList?.first()?.id
            overlayDataList.removeAt(overlayDataList.indexOfFirst { overlayData -> overlayData.id == id })
        }
        invalidate()
    }
}
```

For more sample code , see the [Draw-Line](https://github.com/diwakarsinghdiwakar/Draw-Line "Draw Line")

## And we’re done!

#### Screenshots:

{% asset_img SC1.jpg Image_1 %}
{% asset_img SC2.jpg Image_2 %}
{% asset_img SC3.jpg Image_3 %}
{% asset_img SC4.jpg Image_4 %}