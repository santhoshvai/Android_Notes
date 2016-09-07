# Create custom views

## Why create a custom view?

### Sometimes you can just use a viewGroup

Why not just use existing view groups?. For example, instead of a staggered grid-view we can just put a bunch of linear layouts. perfectly possible!

BUT -

* **Beware of using lots of viewGroups:** viewGroups are expensive. Expensive in the sense that each time a layout
pass comes over it, it has to measure and then it has to layout. It is an expensive operation.

* **Beware of unncessary layouting:** With any view that has lot of components in it and when you are not recycling it well, the items that are out of view are going to be loaded into memory.

### Sometimes you cannot use a viewGroup

* Like a fontTextView .. because Android doesnt support loading a typeface just through setting some attributes.

### Why not just write it out in code?

* Keep your views(```res/layout```) and models(```Activity```) and separated. it leads to messy code. You can in theory, for example add a new textview in code instead of xml. But think about how messy the code can be.

### Why not just write a clumsy combination of layout and code?

* That is what we are going to do. Except it **will not be clumsy!**.

# HOW

## Happens in 3 phases

1. Measures the view - ```onMeasure```
2. Layouts the view - ```onLayout```
3. Draws the view  - ```onDraw```

## ```onMeasure```

[Docs link](http://goo.gl/QmrqCx)

**CONTRACT:** When overriding this method, you must call setMeasuredDimension(int, int) to store the measured width and height of this view.

```(int widthMeasureSpec, int heightMeasureSpec)```

Information about how big the ViewParent wants this view to be. Essentially, the ViewParent will give your view a hint about how big the ViewParent wants this view to be.  A MeasureSpec is a 32 bit int encoded with info comprised of a size and a mode. [More info on MeasureSpec](https://developer.android.com/reference/android/view/View.MeasureSpec.html). Use ```resolveSizeAndState``` to determine the MeasureSpec by giving in the width.height you want and the suggest min width/height specified by parent.

## ``onLayout``

```(boolean changed,
                int left,
                int top,
                int right,
                int bottom)```

the l,t,r,b are the information about where the parent wants the view to be. This method is overided only when we want to make viewGroups, suppose we have a viewGroup all its children will be nested inside. You would then layout your children based on their l,t,r,b within your l,t,r,b.

## ``onSizeChanged``

You get the width and height here. Set the bounds of your rectangle here.

## ``onDraw``

Most important part where the logic for your view happens. Draw canvas here..

[How to draw is given in detail in docs](https://developer.android.com/guide/topics/graphics/2d-graphics.html)

* **NOTE:** Call ``view.invalidate()`` to force a ``Draw()``

# References

* [Make It Fit, how Android measures and draws views  - Droicon Paris 2013](https://www.youtube.com/watch?v=QVcXUi37AQY)

# More on this

* [2016 - Measure, Layout, Draw, Repeat: Custom Views and ViewGroups - Huyen Tue Dao](https://realm.io/news/360andev-huyen-tue-dao-measure-layout-draw-repeat-custom-views-and-viewgroups-android/)

* [Google I/O 2013 - Writing Custom Views for Android
](https://www.youtube.com/watch?v=NYtB6mlu7vA)

* [Crafting Custom Android Views - Droicon Paris 2013](https://www.youtube.com/watch?v=YIArVywQe8k)
