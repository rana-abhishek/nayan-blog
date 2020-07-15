---
title: How to implement Android RecyclerView Drag and Drop feature seamlessly
date: 2020-07-07 12:41:05
author: Ashish Jajoria
categories:
- ["Android"]
tags: 
- android
- kotlin
- ItemTouchHelper
- Callback
- RecyclerView
- Drag and Drop
- Ashish Jajoria
---

We some times want to implement Drag and Drop feature OR Swip to dismiss feature on our recycler view. For implementing that we usually go for a library that already have this implemented, and at this point of time most of those libraries are using old APIs and complex logic to handle the things. But now we have simple and better ItemTouchHelper in the Android Support Library itself, so now we don't need those good old libraries. Lets start implementing.

![Drag n Drop and Swipe feature](/blog/Android/android-recycler-view-drag-and-drop/drag_n_drop.gif)

## 1. Lets create an `ItemTouchHelper.Callback`

We'll create an `ItemTouchHelper.Callback` to handle the events.

```kotlin
private val itemTouchHelperCallback = object: ItemTouchHelper.Callback() {
        override fun getMovementFlags(
            recyclerView: RecyclerView,
            viewHolder: RecyclerView.ViewHolder
        ): Int {
            // Specify the directions of movement
            return makeMovementFlags(null, 0)
        }

        override fun onMove(
            recyclerView: RecyclerView,
            viewHolder: RecyclerView.ViewHolder,
            target: RecyclerView.ViewHolder
        ): Boolean {
            // Notify your adapter that an item is moved from x position to y position
            return true
        }

        override fun isLongPressDragEnabled(): Boolean {
            // true: if you want to start dragging on long press
            // false: if you want to handle it yourself
            return true
        }

        override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {

        }

        override fun onSelectedChanged(viewHolder: RecyclerView.ViewHolder?, actionState: Int) {
            super.onSelectedChanged(viewHolder, actionState)
            // Hanlde action state changes
        }

        override fun clearView(recyclerView: RecyclerView, viewHolder: RecyclerView.ViewHolder) {
            super.clearView(recyclerView, viewHolder)
            // Called by the ItemTouchHelper when the user interaction with an element is over and it also completed its animation
            // This is a good place to send update to your backend about changes
        }
    }
```

## 2. Set directions

Add your movement flags for the directions you want to handle.

```kotlin
private val itemTouchHelperCallback = object: ItemTouchHelper.Callback() {
        override fun getMovementFlags(
            recyclerView: RecyclerView,
            viewHolder: RecyclerView.ViewHolder
        ): Int {
            // Specify the directions of movement
            val dragFlags = ItemTouchHelper.UP or ItemTouchHelper.DOWN
            return makeMovementFlags(dragFlags, 0)
        }

        ...
    }
```

## 3. Update Adapter

Tell your adapter about the positions updates of the items.

```kotlin
private val itemTouchHelperCallback = object: ItemTouchHelper.Callback() {
        ...

        override fun onMove(
            recyclerView: RecyclerView,
            viewHolder: RecyclerView.ViewHolder,
            target: RecyclerView.ViewHolder
        ): Boolean {
            // Notify your adapter that an item is moved from x position to y position
            yourAdapter.notifyItemMoved(viewHolder.adapterPosition, target.adapterPosition)
            return true
        }

        ...
    }
```

## 4. Create `ItemTouchHelper`

From `itemTouchHelperCallback` we'll create an `ItemTouchHelper` object to attach RecyclerView to it.

```kotlin
val itemTouchHelper = ItemTouchHelper(itemTouchHelperCallback)
```

## 5. Attach to RecyclerView

```kotlin
itemTouchHelper.attachToRecyclerView(yourRecyclerView)
```

By this time now you have Drag and Drop feature enabled in your RecyclerView you can just build the project and run the app.

## 6. Handle final state

Now when you want to send and update to you backend about the new order of the items, then just override the `clearView` method of `itemTouchHelperCallback` and you are good to go.

```kotlin
private val itemTouchHelperCallback = object: ItemTouchHelper.Callback() {
        ...

        override fun clearView(recyclerView: RecyclerView, viewHolder: RecyclerView.ViewHolder) {
            super.clearView(recyclerView, viewHolder)
            // Called by the ItemTouchHelper when the user interaction with an element is over and it also completed its animation
            // This is a good place to send update to your backend about changes
            // Your API call
        }
    }
```

## Make it working with `SwipeRefreshLayout`

Now comes a strange problem when you try to implement this feature in swipe refresh layout. The problem is, when you try to drag the item from bottom to top direction, then it will work, but when you try to drag from top to bottom, then it fails, as `SwipeRefreshLayout` intercepts the callback of dragging from top to bottom.

We just have to disable the `SwipeRefreshLayout`'s swipe to refresh feature for the drag time and enable it back when the user has dropped the item to its final position.

So, lets override `onSelectedChanged` to handle this.

```kotlin
private val itemTouchHelperCallback = object: ItemTouchHelper.Callback() {

        ...

        override fun onSelectedChanged(viewHolder: RecyclerView.ViewHolder?, actionState: Int) {
            super.onSelectedChanged(viewHolder, actionState)
            // Hanlde action state changes
            val swiping = actionState == ItemTouchHelper.ACTION_STATE_DRAG
            pullToRefresh.isEnabled = !swiping
        }

        ...
    }
```

That's all you have to do for implementing Drag and Drop feature for your RecyclerView

## References:-

1. [Android RecyclerView drag and drop](https://medium.com/@gopalawasthi383/android-recyclerview-drag-and-drop-a3f227cdb641)
2. [Drag and Swipe with RecyclerView](https://medium.com/@ipaulpro/drag-and-swipe-with-recyclerview-b9456d2b1aaf)
3. [ItemTouchHelper and SwipeRefreshLayout (RecyclerView)](https://stackoverflow.com/a/32075806/5752113)

## Some good reads you may like:-

1. [Paytm Gateway Integration](https://nayan.co/blog/Ruby-on-Rails/paytm-gateway-integration/)
2. [Android Testing Strategy](https://nayan.co/blog/Android/Android-Testing-Strategy/)
3. [How to draw custom paths/lines in Android usign PathEffect](https://nayan.co/blog/Android/drawing-custom-paths-in-android/)
