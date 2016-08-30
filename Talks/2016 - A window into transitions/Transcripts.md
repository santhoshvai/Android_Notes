Hello. Hello, everyone. Thank you very much for coming to this talk, "A Window into Transitions." It's one of the last talks of I/O, or the ultimate talk, if you will. My name is Nick Butcher. I'm joined by my colleagues Ben Vice and George Mount. We're engineers on the developer relations and Android engineering team. OK. So we're here today to talk about the Android transitions API. We've got a lot to cover, so we're just going to jump straight into the topic and get going.

## Transitions

So first up, transitions.

What are they?

The basic idea of the transitions API is to help you whenever there's a change in the scene, where a scene is basically a view hierarchy on screen.

So the idea is that when a scene changes, my animation doesn't work properly, ironically. And we perform a transition.

So a transition being look at what has changed, and then animate that difference.

So the API for the transition is pretty simple.

It's just exactly the same.

What has changed?

Firstly, there will be two callbacks into your transition, which is ```captureStartValues``` and ```captureEndValues```. Passing in this object, this transition values object, so this object has the view. And you can look at the view and capture properties about it, and save it into some kind of map in the transitions values. And so you do that in a start state and the end state.

And then we'll call the createAnimator method, given those two states. And you create animators to animate the change between those two states. Cool. So that's the very basic API covered off.

## Built-in Transitions

There's a bunch of transitions which come in the Android framework, which you can and absolutely should use. So let's have a quick survey of what they are, so you know what these, then, primitives that you can build with are.

### API 19+

#### ChangeBounds

The first, and a super handy one to know about, is the ```ChangeBounds``` transition. So this will help you out whenever you are moving or resizing a view. It will animate the top left and bottom right properties to help you move it around. Crucially, this ChangeBounds also helps suppress layouts.

So you don't want to be calling RequestLayout repeatedly, because that's going to be bad for performance, whereas ChangeBounds will do this in a performant way for you.

#### Fade

Secondly, there's a fade transition, which, as the name suggests, helps fade things in and out. So anytime an object's visibility changes or gets added or removed from the hierarchy, you can run a fade to ease that change.

#### AutoTransition

And lastly, there's this awesome little guy called the AutoTransition, which will put some of those things together.

So when a hierarchy changes, it will fade newly-arriving things in, it'll move and resize items which are remaining or changing position, and then fade out anything which is disappearing. Very handy.

### API 21+

In API21, or Lollipop, transitions API got a bunch of new and exciting additions.

#### Slide

First up was the slides. So you can-- and anytime anything is entering or exiting the scene, you can, like, do so from an edge, in a kind of, like, nice, staggered manner.

#### ChangeClipBounds ChangeTransform ChangeImageTransform

And then there's a bunch of transforms for whenever you're changing the bounds. So you can actually just clip the bounds of something changing size, or if you're doing a ChangeTransform, which will affect the rotation and scale of a
view, or if you're doing an image-- say you're changing from one scale type to another scale type-- this will help animate the changes to the matrix of the image view to change between those scales.

### API 21+ - Transition object upgrades

The transitions object itself got some upgrades in 21.

#### PathMotion

So firstly, it got this thing called PathMotion, which basically allows you to control the movement along a path. So here, for example, we're looking at a transition which is a shared element.

And then we can control the path that it follows when it moves from one state to the other. Don't actually do this kind of horrible loopy thing. This is exaggerated for effect. Please don't do that.

#### ArcMotion and PatternPathMotion

And the classes you'll probably want to look at when doing this kind of control on the path is called ArcMotion and PatternPathMotion. And we'll return to those later.

#### Propagation + Epicenter

And lastly, the transitions API got this new ability to control the propagation and epicenter. So what these two properties let you do is to choreograph the way a group of views are going to move.

Easier shown with an example. So in this example here, when I click on an item, see how the views around the grid all move out together? This is called an ```explode transition```.

But basically, by passing in the epicenter, which would be the view you touched on, you can then set the start delays on the other views which are exiting, as well, in order to have this coordinated movement.

So these are the kind of APIs to look for if you're trying to build this kind of transition yourself.

So that's transitions.

## Window Transitions

It turns out that, quite frequently, when you're changing scenes and want to perform an animation on those changes, it is activity boundaries, so when you're going from one activity to another.

So API21 added a whole bunch of new APIs to help do this kind of thing, which we call window transitions.

There's two main types.

#### Content Transition

The first type is a content transition. So that's when a window exits or a new window enters, you have an opportunity
to run an animation on the content coming in.

So in this example here, if we go from the grid and we launch into like a details-type screen, where we're going to enter animation on the views coming into the details, where they're sliding in from the bottom.

#### Shared element transition

The second type is a shared element transition. A shared element is where, when you tap on one view, the element transitions smoothly from one activity into the second activity.

And as you just had a sneak peek of there, those two things aren't mutually exclusive. You can, of course, run a shared element and the content transition, as well. So here we're sharing a view and also sliding in the unshared content, as it were.

Now, a note on shared element transitions. You can't actually share a view between two activities. You can't pick up an image view from one activity and supplant it into the
other one.

So what we do, as many animation things-- APIs-- do is kind of like fake it. It will allow you to give the appearance of that's what's happening, while still maintaining your either one activity or the other.

So the way we do it is, when you click on an image in the source activity, we record some information about that view. Just small amount of information like the position and the bounds, and a few other things. And then we pass that information over to the destination activity. So we then launch the destination activity, but don't show anything just yet. And then we lay out and measure all the views in the destination activity, and work out where that shared view wants to end up.

Then we do something a bit sneaky. We actually-- we apply transforms to the shared element in the destination view to kind of put it back into the position it was in in the source view, and then fade out any unshared content. So at this point, destination activity is launched. It's the activity on top. But it, you know, appears as if you're still in the source activity, because it's been transformed to appear so. Then we simply run animations to move the shared element back into place, and fade back up any unshared content. Voila. Looks like you've shared a view from one activity to the other, but it hasn't. ***So this is important to remember, that everything happens in the destination activity.*** You're never actually changing the view in the source activity, other than hiding it.

So shared element and content transitions can run at a number of points. So there's a kind of a life cycle to think about here. So when you go from one screen to the other, so when you call startActivity, you have the opportunity to run transitions on the exiting source activity and on the entering destination activity. And then on the way back, you can also then animate on the return and re-enter them. So these are the four points you can hook into.

And it's worth noting that these kind of come in pairs, such that if you were to set an exit transition from the source activity-- so, you know, do something with the views on the way out-- well, by default, ***if you don't specify a different re-enter one, it will just run the same transition in reverse.*** So worth bearing in mind that's what's going on there. So that's, in a nutshell, what the transitions API is trying to do.

To give you an example of how to do it, Ben is going to walk you through an example application that implements these APIs.

## Example

* [Source Code](https://github.com/googlesamples/android-unsplash)

### Using built-in transitions

So as we saw, we had a couple of transitions going on there. And let's see what we're going to aim at. We will have an application having a master activity and a detail activity. And once you tap on one of the items, it will move the image and perform a couple of other transitions. So let's get started.

We have our app. Before we actually got started with the transitions, this is what the system gives us. We tap on one of the views, and the detail activity slides and fades in. Pretty cool.

But in order to use the transitions API, what we want to do is we first enable the window activity transitions within our app theme, or simply inherit from theme that material, or, as many of you use it, AppCompat is also inheriting from theme.material on Lollipop and +.

So once we've got this, we can go to the next step and do the shared element transition for the image view. In order to do this, we ***set a transition name on both the starting and the target view. And remember, those have to be-- or should be, at least-- unique for the view hierarchies, in order to avoid a couple of issues that we will cover later on.***

After we have taken a look into this, we need to tell the system how to actually deal with the shared element. Because as we just learned, sharing the element is something that happens within the receiving-- sorry, the detail activity.

So we have to tell the system that, at the point where we actually started the new activity, that a couple of things have changed. We do this by calling the ```makeSceneTransitionAnimation```, pass it the image that we want to share, as well as the identifier for target view. So the name that we just said on that. Pretty cool. Pretty straightforward and awesome to use.

After that, we add the exiting transition for the grid, which this is the explode that we just saw, which is probably one of the things that you should think about if you actually want to do it, because it can confuse users. It can be very handy, as well, on the other hand. So make sure that you don't just throw explodes in everywhere, because it can be not helpful, on the other hand, as well.

So to do this, we create a transition in the Transitions folder called grid_exit. And we just say explode, and that's it from the declaration of the transition. Pretty straightforward again. We then set that in-- we then have to declare the window exit transition within our homes activity-- home activity's theme.

And as we just learned, this will be replayed, reversed, on the way back in when the activity does its reenter transition, when we press back from the detail activity. Let's go and take the next step.

For the detail activity, we also want the content that is not shared to be slide-- to slide in from the bottom.

You can't just create the transitions from XML, but you also
can do this directly within code. So let's take a look at how you can do that. We just created a new slide transition. We want to slide-- to have the content slide in from the bottom. And we want to have it enter at full speed and then ease in all the way up until it reaches its resting point. To do this, we inflate an interpolator to do exactly that. Then on the window of detail activity, we set the enter transition to be the transition we just created.

This can be a lot more complex than this if you want to, but to get started, just adding one transition, seeing if you actually got the right things in place, works pretty well.

#### Curved motion

Since we are already in the motion of getting things feel more natural by having them enter very quickly, let's take a look at another thing that Nick already mentioned. It's curved motion.

So usually, you have just one linear motion of the shared element view. But we want to have this motion in the end so that it fills while you grow--while the view is growing, it should also have a curved path that it follows to make it feel more natural.

Let's take a look at one of the transitions that we use there. We have a ```transition set``` which is a-- which is a collection of different transitions that will be run at a point in the animation. And we basically just have to do this on the ```SetBounds```. We just say we want to add an ```ArcMotion``` to that. Pretty straightforward again, and it helps to get exactly this done.

You can also set maximum, minimum angles on that to make sure that the user actually sees what's going on. You shouldn't exaggerate, like Nick earlier said. So but for development purposes, this can be pretty handy. So those are all built-in transitions.

### Custom transitions

Let's take a look at a thing that we can do with getting into custom transitions. We also want to share the detail text. So the title-- this is the author of the picture. And we did a couple of things there other than just going-- just sharing that. We also grow it.

And to do this, from the declaration point of view, we just go back to the transition that we already had, add another transition-- add a target to both of the transitions to have the initial transition set only target the photo that we already shared. Then, we go and add a second transition set targeting the author that we want to share, as well.

And add a TextResize transition to that, which is a custom transition that George will cover in a minute. And then, of course, we need to tell the system what's going on, that things have changed. So far, we changed-- we shared the image and the target directly.

If you only have one shared element, this is the right-- this is a good way to actually get started with that, to share this information with the detail activity. If you have multiple shared elements, though, you want to have pairs, which can be created in separate ways.

But this is one of the ways. We just create a pair which basically contains the same information. It contains the detail view-- the starting view, as well as the target transition name. And then, instead of having this on here, you just pass those two pairs in here and start your activity as is. And after all this is done, this is the nice transition set you get.

##### Deep dive custom transition

And with that, let's take a look behind the curtain.

GEORGE: All right. How's that firehose, everyone?Good?Let's drink some more.

All right, let's talk about some-- making your own transitions. We saw the built-in ones-- ChangeBounds, ChangeTransform, those ones. Those are great basic transitions, but if you want to change the text size, for example, it's not built in. We have to do that ourselves. So let's see how we go about making it.

Let's make this resize text transition. All right. So we need to capture some data, right? And we need, of course, the font size. That's obvious, right? We need to capture the font size at the beginning, capture the font size at the end.

And we also need some other data. But when we return the data set that we're going to track in our transition, we only want-- we only care about the font size. Because if the other stuff changes, like if the bounds changes only on your text view, we're not going to do any transition on that.

We're only transitioning when the text size changes. So we just ignore the rest of the data. We still want to capture it. So that's why we have those pieces of information, that tag that we're going to use in our transition values. But we just don't want to keep that for the transition API-- for the transition framework.

So when we capture our start values and end values-- you remember this from what Nick said-- we want to capture them, and we said this is the same thing in both the start and the end. We have a common function to do. And first thing we have to do is we can just throw away anything that's not a text view, right? We're only animating text views. So anything else, we just ignore. Of course, we want to capture the font size and the text size here. And we also want to capture some other data. And I just stuck this all in its own class and kind of hide that information. But it's basic things like the bounds of the text view, because we're going to need that for the animation.

All right. When you animate text, what do we do? Are we going to animate the font size? Well if we do that, we're going to thrash our font cache, right? Every font size between the start and the end is going to be used and created in our font cache, and we're going to have, like, font size 26.281. And you know, we're never going to use that again. It's just going to thrash our font cache. We don't want to do that.

So what we're going to do is we're going to capture a bitmap at the start and a bitmap at the end, and we're going to animate.

So here, if we animate just the start, it looks pretty terrible, right? You see that thing blown up from the beginning. And also, if you animate just from the large size down, you're going to end up with a little bit of kerning error in the final text view. And you might not see it here on this slide, but you're definitely going to see it when you're looking at the phone. And you really don't want to see your user-- users to see that effect. You want to see a perfect transition. So instead of this, what we're going to do is we're going to animate piecewise.

###### Piecewise Animation

So we animate a little bit from the start. And you don't really see that blown-up image, right? And then we're going to swap it out for the final image, just scaled down. And then we're going to animate the rest of the way. And because it's done during the motion, nobody will see that animation, that change, that swap there in the middle. OK? Let's see how we do that.

When we create our animator, first thing we have to do, of course, is to capture-- we use our start values and end values. But here, we are checking the start values if they're null. What does null mean? It means the view didn't exist. So if the view didn't exist at the beginning or at the end, we don't want to animate size. That's for fade, right?That's for the other transitions.OK?

Next thing we have to do, of course, is capture the start bitmap, right? So we just capture that bitmap. That's where that extra data is useful. And we also have to capture the end state bitmap.

And then, of course, we have to say, don't-- text view, don't draw yourself. Set yourself to transparent. All that text stuff, make it transparent. We don't want you to draw it.

Because what we're going to do is create a drawable with those bitmaps that we just captured that's going to do the swapping for us. All right. So we need to animate some things on that drawable. One of them is the font size. We're going to have this font size that we're going to animate on the drawable. And we also have to animate those other things, all that other data I captured. But we need to know that stuff. So we get property values holders for those, and then we create the animator, right? And then we're all set to return, right? Everyone? Not quite. Remember what we just did? We set it to be invisible? Like no, don't draw that text? When the animation is done, we want it to go back to the normal state. So we set the text color back to normal, and we remove that overlay we just added. All right. So we're all set. Our transition looks great. We're all set.

#### In-Activity transition

Let's put it into activity transition. All right. What does it look like? Holy moly. Did you do this? All right. No, no, OK. Remember what happened-- what Nick was saying about this. The values are captured in the destination activity.

And when the values are captured in the destination activity, text view doesn't have the text size of the source activity's text. So it doesn't even know that it was there. It doesn't know what that font size was.

So what we have to do is send that along as extra data. Now, the activity transitions system will send the basic stuff for you. It will send you position, size, and image scale, right, because those are very common things. And those are the things that are built into the system. Those transitions are built into the system.

But the other stuff, anything that you need to animate, you're going to send along, as well. So we put those in as intent extras. That's easy enough. And then we get that intent extra in the target activity, and we get the-- our font size.

And then we call this SharedElementCallback. The SharedElementCallback is where you put all the really complex part of your activity transition. This is a callback that has lots of extra stuff for you to hook into, to do the little tweaking that you need to do.

And during the start, what we have to do, of course, is capture the transition-- the text size of the current text view, and then set it to the one that we want it to be in the start.

And in the end, of course, we have to reset the text size back to what it was before. Otherwise, you just animate to the little tiny font from the start. So what happens now?

You end up with a beautiful transition from activity to activity. Nice and smooth. Now Nick's going to come back and show this really awesome transition that he made up. You guys have to see this.

## FAB to linearlayout circular motion - custom animation by Nick in opensource app plaid

* [Plaid source code on github](https://github.com/nickbutcher/plaid)

NICK: Thanks, [INAUDIBLE]. Awesome. So that's been some pretty cool transitions we've looked at so far. One of my favorite things in the [updated motion guidelines in the materials spec is this section on transforming things](https://material.google.com/motion/transforming-material.html#transforming-material-radial-transformation).

If you haven't seen this, I highly recommend you check out both this, as well as there was an awesome session at I/O on Wednesday by John [INAUDIBLE] about the updated motion guidelines. This talk's about material design, about how things are, you know, sheets of digital paper and so on, and that they can transform from one thing to another.

This transition in particular caught my eye, which is transitioning--transforming, sorry-- a Floating Action Button, this circular FAB, into a different sheet of material. I thought this was pretty cool, and I wanted to use this in an opensource app which I built called **Plaid**.

And I wanted to transform from this FAB here into this bottom sheet, which is essentially a linear layout. So unlike the transitions we've looked at so far, which are pretty much acting on the same kind of view-- so transitioning an image view in one window to an image view in another, or a text view to a text view, for example-- this is, like, pretty different, right?

***We're going to go from a floating action button-- basically an image button--to a linear layout, essentially.***

So how do we do that?

So it's really, really important to remember, as we've talked about before, ***that everything happens in the destination window, so in the window with this kind of bottom sheet here.***

#### Setup

##### Center on FAB

So as with most animation things, we do some tricks to make it look like the one thing is transforming into the other. So what we do is, in the destination window, we use ```setTranslationXandY``` to basically center the dialogue on the-- the coordinates of the FAB. So where it started from. Just TranslationXandY.

##### Add color overlay and icon

And then we use the view overlay again. So if you haven't used the view overlay, this is the ability to add, like, drawables on top of the view. So we add a solid color drawable, as well as an icon on top of the view, and locate that it sits in the center.

##### Animate the entire view

And then we run some animations on this entire view.

1. **Fade color overlay and icon out: ** So first, we fade out the color and the icon.
2. **Ciruclar Reveal:** We run a circular reveal animation. So that applies a circular clipping mask. And we start from the size of the FAB, and then we animate that clip mask out to show the whole dialogue.
3. **Move along path:** And at the same time, we move along a curved path using this motion thing.

So when we run all of those animations at the same time, you get this effect. It looks like you're transforming from the FAB to the dialogue. So do you want to see how we build this? All right.

Before we do, let's think about what information we need to build it. So in order to build this transition, we need to know the positions, like where the FAB was to where the dialogue is. We need to know the color it's going to, and the icon because, as we've talked about, what we keep hammering on about, it all happens in the destination window. So the sheet doesn't know anything about the FAB, right? So how do we get all this information?

##### FAB & sheet position

So the position is pretty easy. As George said, the position is one of the few things that actually does get passed through. So we can just-- in the captureStart and captureEnd bounds, we do the same thing. We just call CaptureValues, and we put those bounds into the value-- transition values map. And we'll get access to those a bit later on. So we have the bounds of the FAB and the dialogue.

##### FAB color and icon

So for the color and the icon, I basically created a couple constructors that you can use. So just like views, transitions can have-- can be inflated from XML. So when we looked at the XML transition sets that Ben was showing you before, you can define your own transitions, as we did with the text resize, but you can also add your own attributes.

So here's an example where we're doing, you know, ```context.obtainStyledAttributes```, just as you would in the custom view. And you can define your own declare a set of stylables and you can pull them out of that attribute. So we could pass in the color and the icon free attributes. Just a quick note. We're setting the path motion here. So this is what we talked about before, adding a path motion to control the path which it's going to move along later. And we'll come back to using that in a second.

But as an alternative, I also added a dynamic way of determining this, because there's actually different entry points that I have into this screen. So I couldn't statically declare in the details activity what color and icon it was going to come from, because it can come from multiple places into that screen. So I added a constructor which will pass in the color and the icon to use. And so we can do the technique which George talked about earlier of using intent extras to pass this information to the details activity.

But here's a pattern which I found really helpful for implementing this. This isn't part of the transitions API contract, but it's just a pattern I found useful. So I added a static method onto the transition object itself called addExtras. So you can call this from the source activity, from the activity which has the FAB and knows about the icon and the color.

So pass in those things in a static method, and it will add them to the extras for you. And then a corresponding setup method, I call it, which you would call from the destination activity, so from the activity with the bottom sheet, which will then pull those activity-- those parameters out to the intent extra fields. And then it will actually create an instance of this transition object which I called FabTransform, itself set the target, and then set it on the window as the SharedElementEnter. The reason I like this pattern is because it means it keeps the keys and so on private to the transition. You don't have to worry about exposing that. It kind of keeps it nicely encapsulated.

Notice here that we're using setSharedEnterElementTransition. But if you remember from the animation, it goes both from the FAB to the sheet, as well as animating from the sheet back down. So what that means is the element-- sorry, the shared enter transition was going to get used on the return transition, as well. So bear that in mind. It's going to become important in a second.

##### ```createAnimator```

Right. So we have all the information. We've created the transition object itself. How do we actually create the animators to run this transition, to make it all happen?

First off, we grab out the bounds that we saved into the transitions values map, and hold onto those. I then do something where I just check basically which of those is wider. It's maybe not the most sophisticated method, but that's how I work out if I'm going from the FAB to the dialogue or from the dialogue to the FAB. I'll basically use that Boolean later on to set up the appropriate animation.

So for example, if you're going from the FAB to the sheet, you do an expanding circular view, or if you're going from the sheet to the FAB, you do a contracting circular reveal, or circular hide.

So next up, the view is always the endValues.view. So even though you're giving the start values and the end values, you're always acting on the endValues.view.

And something that took me a little while to get my head around was that that view is always in the state that it ends up in, which is to say when we were going from the FAB to the dialogue, it was already laid out and positioned in the dialogue's position. And then we reversed it by setting those translations and so forth to put it back into the FAB's position, right. So it was already in its end state. Well, the same is true when I'm going back. So what that means is, when I'm about to do the animation from the dialogue back to the FAB, the FAB is-- the dialogue is actually laid out like this, just in the exact same bounds as the FAB. **And so what you need to do is to manually call Measure and Layout onto the dialogue in order to put it back into the position that you want to animate from.** So it may sound counterintuitive at first that you have to call Layout and Measure yourself, but it is just how the system works, that it's already in that same state. It was easy for us on the way in. We have to do a little bit more work on the way back.

And then I just want to highlight the translation, as well. So this is how we use that path motion. So we set a path motion in the constructor early on. And here, I'm using one of the overloads of ```ObjectAnimator```, which will actually animate two properties at once. So here, I'm animating the TranslationXandY of the dialogue. And we get hold of that path motion again, and call getPath, passing in the start x and y and the end x and y. And that will generate the curved path and animate the translation along the curved path for us.

We do some more animators for the circular reveal and the fade and so on. And we put them in set and return that. And that gives you that entire animation. So that's how it's done. And this is all open source, as well, so there'll be a link at the end.

You can go and have a look if you wanted to go into all the nitty-gritty details.

## Pro Tips

So let's talk about a few things that I think you guys will find really useful here.

#### Transition names.

Now, if you do things well with transition names, the system will work for you,
and that will help you a lot.

For example, we have an animation here, a transition, to go from this little
guy to a big-- [INAUDIBLE] big thing.

We want to do an activity transition like that.

And what's happening in our activity transition is, of course, we call
MakeSceneTransitionAnimation.

And we have this view.

And we-- my view doesn't have a transition name here.

Or-- please don't do this-- you call view.setTransitionName right here in this
call.

Don't do that, OK?

And I'll show you why.

We have our activity here.

We launch a new one.

And then we change the orientation of the phone.

What happens now?

That's right.

Our activity gets rebuilt.

But further, the thing underneath it gets rebuilt, too.

So even if you called view.setTransitionName, you just lost that.

It's gone.

So now when the view tries to come back, it has no idea which is the shared
element.

The transition name is used to identify which-- you know, going from the source
to the target, and so it can map those two.

And if it loses that transition name, it won't know what to do.

You, of course, can fix that up for it, if you want.

And we'll talk about that a little bit later.

But the transition name is your friend.

So in a system like this, where you have a RecyclerView, for example, when you
bind your view, that's when you set your transition name.

And it should be a unique name that you have for that item.

And it can be a URL or whatever you want.

It's fine.

As long as it's a string that's unique.

Here, I bound it to the position, in my ViewHolder, of position is unique.

Whatever.

Or-- and a little plug for data bindingg-- you can do it right in the layout,
too, if you're using data binding.

That's awesome, by the way.

I know some guy who works on that.

00:32:06,980 --> 00:32:09,150 Let's talk about changing the shared elements.

That means we have a different shared element than you originally thought.

Here, I have a view-- an activity.

And it launches another one.

And then we scroll in the detail view.

And when we go back, we want to go back to a different shared element, right?

Now, this transition system, of course, is going to try to go back to the same
one you had before.

We have to tell it something else.

Step one.

When you launch the activity, ask for which shared element is coming back.

So you startActivityForResult.

OK?

Step two.

Before you go back, set the result.

Tell it which shared element is going to come back to you.

Here, I have an ID that I'm going to send back as a result.

Step three.

In onActivityReenter, not onActivityResult-- I've gotten bitten by this just
yesterday, I think was, when I was writing a demo-- onActivityReenter, that's a
new thing just for activity transitions here.

It will be called before the shared element comes back.

And this is where you can do some tweaking.

Here, I'm going to get the data, get the shared element.

OK?

And that's going to come back to me.

All that data is coming back to me.

And then we can also call a shared element exit callback, set the
SharedElementCallback for the exit.

Now the exit is used for the exiting, and also for the reenter.

It's the same callback used for both.

And this is where we can switch up our activity-- our shared elements.

We can set our shared element to whatever we want in our view hierarchy.

So here, I've just-- I used that ID I just had-- got before from the
onActivityReenter.

And I used it right now for my share-- to change my shared element.

And now it works perfectly.

Let's talk about overlays.

Shared elements often go into the overlay.

If it'll just go right into the overlay, unless we specifically tell it not to.

Now, the great thing about this is it's drawn on top of everything, right,
because you don't want to have your shared element drawn underneath something
over there, right.

And it's also not clipped by the parent.

And that's really important.

But the problem is, it's drawn on top of everything.

And when it's drawn on top of everything, that's a little bit of a problem.

And it's also not clipped by the parent.

00:34:33,960 --> 00:34:36,420 Sorry.

[INAUDIBLE] All right.

NICK: Here's a real example I had of this issue, which is, in the screen, again
from Plaid, when you click on a list item, I wanted to have this expand and
grow out.

So I wanted to have a shared element where the background was actually one of
the shared elements.

So if you look at this transition, there's two shared elements going on here.

There's an image going from one position to the other, but also the background
transitions to give that growing effect.

So this is all well and good.

But if you were to use the overlay to do the transition, if you put the
background into the overlay, it's going to be on top of everything else, right.

So you get this horrible effect where it kind of grows up and you don't see any
content, and it snaps in, right.

That's not what we want.

We actually want the content to be able to kind of come in on top of the
background.

So hopefully we can control that, George.

GEORGE: Yeah, we can.

Well, all we have to do to make the shared element do something different is
tell it not to.

And so we just set this value in your styles.

But you might end up with a little bit of a problem.

00:35:35,339 --> 00:35:36,130 I think it was you.

You did it, right?

No.

Well, the problem is this.

Let's see what's going on here.

The shared element is, of course, popped into the new activity.

And when that happens, it's being clipped by its parent.

And it's only visible when the shared element slides into the parent view.

So what we can do, of course, is tell the parent, don't clip it.

So you'd say, tell the parent, say setClipChildren to false, and also
setClipToPadding false, as well.

Now this is-- might cause you a little bit of a problem with your drawing.

So be careful with this.

And also, it's a performance issue.

You might consider doing this only during your transition.

But you can't do it just on the parent of the view of the shared element.

You have to go all the way up, because the grandparent can also clip the child
and clip your shared element.

And that's no good.

The other option is to do, as Nick did in his-- is to put it right into the
root, into the root of his layout as a child of that.

And now it's not going to be clipped by anything, because the root, of course,
owns the whole window.

It's fine.

00:36:43,51 --> 00:36:43,549 All right.

Let's take a look at it now that it has no clipping.

It comes in.

Instead of like a flash, it comes in on top, just like we want.

All right.

Ben.

BEN: Cool.

Whoop.

00:36:59,630 --> 00:37:03,200 So as you may have seen, within some of these
transitions, and in general, if you work with transitions, you also have access
to the window decorations.

That was basically a status bar in the navigation bar backgrounds.

So let's take a look at what you can do is you just say define a slide
transition.

And what will happen with that is that it slides all the content in.

We highlighted the status bar here to show you what's going on.

So it slides the status bar in, as well.

But we actually don't want to do this.

And as you can see on the bottom, under navigation bar, there's also Flash.

And the last moment of that, it still-- it slides up, as well.

So in order to avoid this, there's a couple of ways you can do this.

But what you want to do is you want to exclude both the status bar background
and the navigation bar background.

So we provided IDs, as well as transition names, for those so you can work with
them.

That is one of the ways that you can do it.

Obviously, you can do this not just in XML, as we showed you before, but also
when you do this within your code, you basically just say, on this transition,
exclude the target, and set those values to true.

But you can also do something a little bit more nifty.

You can add this to the shared element transitions.

So you could have recolors in there, or just fade out stuff when you want, or
fade them out within a shared element transition.

This is-- in order to do this, you basically go all the way up in the view
hierarchy.

You get the decor view of your window.

Then, you find the views by ID.

You create the pairs.

This is the second way you can create pairs.

So another-- it's another way to calling that.

And then adding them to the shared element transition to actually have the
system being aware of exactly that.

And then you get the desired behavior of no flashes, no exploding or just
moving out of the way status bar or navigation bar background.

And with that, we actually have a little room for questions.

So you can start lining up if you have questions, about six minutes.

Thank you very much for being here.

It's been a pleasure to talk to you about transitions, and we have open source
all the code already for the samples that we gave here.

So the new one, the one with the images that we just showed, is on [INAUDIBLE].

You definitely have to take a look in Nick [?

Butcher's ?] Plaid app if you want to go into the advanced things on
transitions.

There's also topeka out there, and another sample called ourstreets.

You can take-- we looked through the training that we have, and yeah.

Thank you very much.

[MUSIC PLAYING]  
