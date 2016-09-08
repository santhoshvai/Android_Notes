## View

* In order to build a UI in Android, we use Views.
* Views are rectangles on the screen. And we may or may not see the borders of that rectangle.
* Essentially, a view handles drawing and event handling, and all the basic widgets in Android extend from this base class.
* The [Android design guide](http://developer.android.com/design/building-blocks/index.html) has visual examples of these basic building blocks of apps.

## ViewGroups

![ViewGroups](http://i.imgur.com/ph9aK6S.png)

* If you want to display multiple views together, you'll need a view group. A view group is a container for children views.
* Here are a couple of common ViewGroups,
  * **FrameLayout** - A child that gets added will be default positioned in the top left corner of the view group.
    * If you add a second view, it will overlap the first one. So typically we have *one child* per frame layout.
  * **LinearLayout** - Its composed of children either in a horizontal row or in a vertical column.
    * We can also specify layout weight. It allows us to distribute the width or height of a parent across the children. For example, if there are 2 childs and both have weight 1. Then the width can be split evenly.
  * **RelativeLayout** - We can specify that a view should be aligned to the parent's left, top, right or bottom edge. We can also specify that one view should be relative to another view.
    * It doesn't fill up the spaces nicely compared to a linear layout, but there are pros and cons to each.
  * **GridLayout** - The views fill up cells in a grid. you can also have views that span multiple cells.

**Note**: A ViewGroup is a view. So we can nest ViewGroups within ViewGroups. The reason why we care so much about parent child view relationships, is because *the way a child view gets laid out depends on its parent*.

### View width/height

![view width and height](http://i.imgur.com/cU5fwFy.png)

### Gravity

![View gravity](http://i.imgur.com/gZLVX2w.png)

### Padding vs margin

![View padding](http://i.imgur.com/GXUOukZ.png)

### View visibility

![view visibility](http://i.imgur.com/dGGudCt.png)

**It can also be toggled dynamically**
