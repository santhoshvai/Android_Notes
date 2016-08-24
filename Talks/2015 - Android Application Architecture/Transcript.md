ADAM POWELL: Hi there.

So as Dave said, we're always a little bit more personable right after lunch.

We're also a little bit more excitable right after coffee.

So you're catching us at a good time.

YIGIT BOYAR: Yeah, just had coffee.

ADAM POWELL: So this talk is about Android application architecture.

I'm Adam Powell.

I'm on the Android framework team.

YIGIT BOYAR: I'm Yigit Boyar.

I'm also in the Android framework team.

ADAM POWELL: So this is actually going to be a little bit of a special extended edition version of a talk that we gave earlier this year at Google I/O.

And just like every good special edition, the original version isn't available
on video.

So here you are.

So the important thing about application architecture is, really, you need to act early in your applications.

As soon as you start writing code, you've made these decisions that are going to affect how your application is going to be developed from there forward.

So those decisions stick with you. These really, really deeply influence your application. And we're going to show you a few of the ways that this really affects how you might write your app as you move forward.

It really makes it easier or harder to think about the problems that you're really setting to solve in your application.

So you might be asking, OK, well, which patterns do I use?

There's a lot of stuff out there.

And you can really get into a lot of alphabet soup as you start looking over all of the different prescriptive patterns that you can look into--

YIGIT BOYAR: I'm super confused.

ADAM POWELL: Yeah, I mean, I've got all these things here.

YIGIT BOYAR: That's me, by the way.

That's me, this cartoon.

ADAM POWELL: So this isn't a survey of libraries.

That's not what we're here to talk about today.

There really are a lot of great libraries out there.

And you should really look into many of them and see what they can do for you.

And the trends change quickly.

The challenges you're going to face are really timeless.

The properties of some of these devices that we're writing for have been the same properties from the first time any one of us held a smartphone.

And really, lava flow happens over time.

This is kind of this phenomenon where, after your code goes through a number of refactorings, there isn't really kind of an overarching principle that ties it all together anymore.

Someone new who comes to your code base may or may not understand what's going on as you start piling up all of these different pieces that really no longer fit together.

**The important takeaway from this is to architect for user experience.**

Really, nobody in your user base is going to care exactly what patterns you used to produce your application.

They're going to care about the experience that they have while they've got it in their hand.

YIGIT BOYAR: All right, so let's look at some examples.

There's some videos.

So here's an application.

It's very simple.

I'm posting a comment on a photo, trying to type.

All right, there you go, Send.

And I'm waiting.

Why am I waiting?

I have no idea.

I'm just waiting, waiting.

All right, it sent.

So what was the problem here?

So what's happening was, the user clicks on the Send button.

It goes to your controller, goes to your network, network comes back whenever it wants to come back.

And then you update your UI.

Meanwhile, you blocked the user because you don't know what happened.

Maybe the comment will not go.

What if we get rid of this, introduce something you can call a model, which is
your storage, which is where you keep your comments and everything?

So then when the user hits the Send button, you keep it in your model, update
your UI, then tell the network that this has been added.

And when the network comes back, you can update your UI.

So let's look at the same example, this time with the fix, how it looks.

So I come here.

I type my comment-- the same comment so you make sure it's the same use case.

I send it.

I still see the comment.

It's a little bit gray, but I see it there.

When the network finally comes back, it turns black.

User has a clue.

You can design it better.

I couldn't.

But that's the idea.

You give the feedback instantly.

ADAM POWELL: Your designers are wonderful.

Make sure that you appreciate them.

YIGIT BOYAR: I told you, why did you give me a design job?

OK, let's look at one more demo.

It's the same puppy.

So I keep commenting that I really like it.

I keep commenting.

And I hit Send.

OK, you're going to see that it's greyed out.

I see the responses only.

That's great.

It turns black.

That's great.

I go back to that page again.

It's empty.

Like, I was here literally two seconds ago.

I came back.

I come back to the page, and it's not there.

This is inconsistent.

I feel like something got lost.

So what was happening here?

So the view controller, as we said before, told the model, model told the
network, it came back, we updated the UI-- great.

But the problem is the model didn't have anything.

It only is still fed by the network.

What if, instead, you kept a persistent model?

So the idea is that your data stays on your client.

There's something that you can always access unless the phone crashes, which--you can't do anything in that case.

Instead, we have some application logic that is responsible to synchronize network and your persistent model.

It's really important to look at this problem as a synchronization problem
rather than making API calls.

That simplifies a lot.

So this application logic syncs with the network, fetches whatever you want to fetch, updates the model, then just notifies in the event-- OK, I'm done.

I made some changes.

Hey guys, if anybody wants to know about it, here they are.

And then your view controller goes and fetches the data and updates the view.

Let's say the user clicked on the Send button.

The view control tells the application logic, OK, I have this comment.

Please send it.

The logic instantly updates the disk.

That's the first thing you have to do.

You update the local storage and tell, hey, I updated this comment.

I added this comment.

And it also calls the network.

Now, there's two things going on right now.

While you're making the network query, the view controller gets notified.

OK, I have a new comment to fetch, goes to model, update the UI.

When the network finally returns, the application logic updates the model again
and says, OK, I seek to comment to server.

If anybody's inserted, do something.

And then the view controller goes, updates the view again.

So everybody has very simple duties.

And they are decoupled.

So see how it looks when we implement it properly.

I come here.

It's instant.

Because it's coming from the local storage.

It cannot be slow unless something is going really bad on your device, which
sometimes happens.

All right, so one more example-- OK, it's running.

So I'm so excited that I keep sending comments.

But they're only seeing the first comment I sent.

Something is-- oh, OK, they just showed up.

ADAM POWELL: What happened there?

YIGIT BOYAR: I don't know.

What happened there?

Now I figure out what happened there.

So this was my background.

Because I had this great pool on the background that processes all these things
one by one so that I don't create too many threads.

So And I loaded the activity to-- a fetch bitmap job came for the photo, and
the other one to load the comments from the disk.

So I had two executors.

They started running.

Meanwhile, this user started spamming my application with all these new
comments.

I finished it in the comments.

The other one came.

We put the comment in the disk.

And now we are trying to send it to servers.

Meanwhile, the network is really bad.

So the fetch bitmap job is still running.

The view controller got the notification about the new comment and wanted to
refresh itself.

But the queue already is full.

I'm using.

And then the job cannot run.

So the UI is not updating because I'm trying to fetch a bitmap.

This is bad.

This is bad.

That means the priorities are not good.

But it's very hard to know.

You don't know how long the job will take.

You cannot estimate it all the time.

But what you can do is separate these things.

So if I have a different queue for my network tasks, and a different queue for
my local tasks, when my network is flaky, my application is still responsive.

Because they are never affected.

So if we're on the same jobs that were on the previous center, as you can see
now, they're going into different threads.

And the local test queue just keeps running because disk fine.

Because we couldn't fetch the bitmap, we couldn't update.

We couldn't send the comment to server.

But who cares-- user is still seeing it.

And if we make the changes, well, you see everything is sending.

And as it gets time to run the server jobs, we're seeing there they're starting
to turn black, from top, one by one.

They will eventually be synced.

  So one of the feedbacks we get after [INAUDIBLE]
is, OK, but how does the activity get notified?

So activity is a very, very simple state machine in this scenario.

When it is created, you set up your UI.

That's all you do.

When you start, you register for the events you want to know about.

And you load your data.

This is how it ensures that between restarts or whatever, if there are some
events you miss, you'll never miss them.

Because the onStart, you register and you refresh your data.

And when an event comes, just refresh your data.

It's like the same thing over and over.

OnStop-- don't refresh your data.

You just unregister from events.

So your life cycle is very simple.

Like, this happens, I do this.

This happens, I do this.

This makes it easier to test, much more stable, much more easier to understand.

OK, back to you, Adam.

ADAM POWELL: All right, so the key takeaway from all the demo that we just saw
there was, really, you need to design for offline.

Always assume that the network is not your friend.

It's going to be slow.

And if you want to deliver a good user experience, you really can't just blame
the network for why something isn't popping in quickly.

The table stakes have really been raised on this when it comes to mobile.

So to solve this, back your UI with a local model and use your other
application logic to sync the model and server rather than doing this as live
API calls.

It can be really tempting to try to optimize for the case where you want data
freshness above all else, where you really don't want to show anything to the
user unless you're really, really sure about it.

But this is one of those things where, if you assume success versus assuming
failure, then what types of claims are you really making about your software
when you assume failure in the common case?

I mean, your software is going to work, right?

Your server is going to come back with the answer that you expected, right?

So optimize for those cases.

Predict it.

Show it to the user early.

YIGIT BOYAR: By the way, I just learned yesterday there's is a fancy new name
called optimistic rendering.

I feel like now it is called optimistic rendering.

ADAM POWELL: Yeah, you see this principle at work in a lot of other areas of
software development as well.

Games is one really prominent example where a lot of network latency
compensation systems in multiplayer games are based on this principle.

They predict based on what happened in the previous known state from the
network.

So you can use these same ideas in your own apps.

So make sure that these things don't necessarily depend on each other unless
it's in a very loose fashion.

Use events and callbacks and such when needed to notify other parts of your
system about a state that's changed.

So everybody always asks us about dependency injection as well.

Is it good?

Is it bad?

What should I use?

What library should I use?

Well, I mean, use it if it helps.

Everything has a cost.

So know the side effects of what it is that you're doing and what it is that you're including.

It's very, very easy to include a new dependency in your application.

And it's not always easy to see what the costs of that are going to be at run time, especially when you're just bench testing at your desk.

Everything is in the optimal case when you're sitting there at your desk.

So avoid reflection for better performance as well.

Many dependency injection frameworks have a heavy runtime component versus a compile time component.

Some systems like Dagger 2 do this at compile time, and it ends up being much more optimal for your application.

So take a look into these and see what the tradeoffs are.

# Networking

## API Design

When it comes to networking, so we talked about
other things that we can do.

Always assume the network is going to be slow, janky.

Always assume that you've got your phone in the middle of a conference with a crowded Wi-Fi and completely slammed cell towers.

That's basically your case that you want to optimize for here.

Because this is when people are going to notice that your application is broken when it falls over.

So your API design at the network layer can actually affect this as well.

So design your back end for your client, not the other way around.

And this is one of those things that sounds really obvious when you get up here and say it.

But as you start building server back ends for these things, you start exposing pieces that you may need, so on and so forth.

And really you start building it up as a series of layers.

So you have the very low level components that provide your data, such as the network.

And then you have the higher level components that provide the presentation.

And it's very tempting to not have the higher levels inform the design of the lower levels.

But you can get a lot of wins by doing this.

Also, process as much as you can on the server side to begin with.

You have these big, beefy servers in the cloud that can do a lot of work for you.

Don't offload stuff to the client that you don't have to.

More specifically, pass metadata to the client.

This is something that can really, really improve your user experience.

If you have a large blob of data, such as a big photo, that you're going to pull down as part of syncing some sort of social app, for example, what do you know about this image at the time that you've made this request and gotten this
response back?

Well, not a whole lot.

So what are you going to show while that loads, while you actually go fetch that image?

A better case would be to pass along a little bit more metadata. Give your application some hints that it needs to succeed.

In this case, we're passing the width and height of this image in advance before we go fetch the image itself.

This allows us to perfectly size a placeholder in our UI so that you don't have your UI jumping back and forth as the user scrolls through or attempts to move back and forth.

We've also passed along some palette information here just to be able to give our UI a little bit more of a splash of color that keeps the flavor of the image before we actually have the real bits.

## Battery and data

this is another concern that you've got whenever you're talking to the network as well, especially when you're doing any sort of background syncing.

Batch your requests as much as possible.

Now, we've provided a lot of other tools in recent versions of Android to help
you do this effectively.

Things like Doze and Marshmallow are great examples.

Doze will do a lot of this for you.

But really, don't just rely on that as your sole way of dealing with this.

Doze is essentially the last line of defense for the user at this point.

Use the job scheduler when it makes sense.

Give Android as much information about what it is that your app is actually
trying to do.

And that way, Android can make smarter decisions about how best to optimize
that.

The key takeaway here is act locally, sync globally.

You really want to give the user a very fast, very responsive local experience
whenever they're using your application.

And then make sure that you keep that in sync with the cloud almost as an
afterthought compared with the presentation that you're trying to give.

So no talk of application architecture would be complete without some sort of addressing of the question of activities and fragments.

This is something that we constantly get questions about, even within Google.

We'll get questions, like, OK, well, should I use activities or should I use fragments to build my app?

It's like, well, that doesn't entirely make a whole lot of sense to question.

Fragments are really just encapsulated parts of an activity.

So when you build up your application, when your activity starts getting too big, and you need to start breaking it up so you can still think about it effectively, it's a great time to break it down into some fragments.

Or if you start by building single fragments that sort of compose together a little bit more than that so that you keep your concerns separated, that can help keep things a little bit more straight in your head so that you don't end
up with these giant activity god classes.

Fragments and views is another one that we get a lot.

Now, this is something that we talk about quite a bit on the framework team in terms of, one of the almost regrets of the fragment API at this point is that you've got this handy little tag that you can just stick in your layout that says, fragment, and says, go ahead and instantiate a fragment and stick it right here in my layout when I inflate it.

And this thing is really, really convenient.

It makes for great code examples and demos, because you can do things like have a layout that sticks two fragments side by side.

And then you rotate the device, and it sticks them stacked on top of one another.

And you say, hey, look.

Look at this great decoupled system that I've got.

It makes for fantastic demo.

But realistically, this makes people think of fragments as being view constructs themselves.

And really, they kind of aren't.

So they live in these very different worlds.

So views are really the nuts and bolts of your UIs that you're building, whereas activities and fragments are the lifecycle constructs that basically provide the plug-in points of contact with the rest of the system that tells you what's going on.

So many times, you're going to want to use both to keep these responsibilities clear.

Just as Yigit mentioned earlier, you're going to want to use these signals that you get from your activity and, in conjunction with that, through your fragments as well, if you're using those, to basically inform when you should register, unregister for events, so on and so forth.

But the views should really be their own encapsulated pieces separate from that that are controlled by these other constructs.

OK, so moving along to another topic of contention often-- memory.

So this is something that we talk about a lot.

If you've watched any of Colt's talks on the Android performance practices, you've seen some of these.

Avoid allocating objects in hot code paths.

Putting pressure on the GC is not so great.

Art is a lot better at this these days than Android used to be at all of this.

But there's still a lot of those devices out in the wild.

And if you watch this in a few sensitive code paths, then you're going to end up with a little bit of a smoother experience.

So you can pool and reuse some objects when you measure a problem.

That last part is really the key-- when you measure a problem.

Make sure that you're measuring these things before you go and contort the internals of your system.

And these are the sorts of things where, some things, you can make some decisions early that will help you move into these optimizations should you need them.

And some things, it really doesn't matter.

So write the code the way that it's clearer first and then optimize later.

So GC still kind of remains your enemy, especially on some of these older devices, even as the garbage collector in Android gets much better.

So a perfect example of how this may affect your API design internally-- just with smaller components.

Now, what's the difference between these two approaches?

Well, the top one is arguably more idiomatic, much cleaner.

You can make some assumptions about it in terms of, OK, well, the object it returns is hopefully safe if that's a mutable object to begin with.

But in order for that to be true, then this getter has to be allocating a new rect for me that it returns.

Well, now we've got allocation.

So where do you end up using that?

Versus the second form here, which is basically just taking an input parameter that the method is going to fill out.

You know that the second one isn't going to perform any additional allocation in this case.

This is the sort of thing that you really only want to worry about in extremely hot code paths.

We're talking layout that gets run many, many times.

We're talking drawing that gets run 60 times per second.

If you're talking about things that happen in terms of your event handlers, like clicks, and so on, and so forth, you really don't need to be contorting your system this much internally.

But for that very small percentage of your code, it is OK to write ugly code if it helps your users.

Your users are not going to see your code.

They're going to see your UI.

If your UI is ugly because it's behaving badly, then that's something that they
do see.

That's something that really does affect their experience.

The performance critical code really isn't the majority.

And, well, all compiled code looks ugly eventually anyways.

So with that, we're going to move over to another demo that kind of brings a bunch of these ideas together.

YIGIT BOYAR: OK, so after the I/O talk, one of the feedbacks we received was, OK, talk these things, but show me some code.

So this time, we wrote this sample application.

Keep in mind, this is a sample implication.

And the main focus here is, how do you look at this problem more as a synchronization problem?

How do you design an application to work offline?

And we are going to release the source code.

I'm just waiting for some approvals.

So here's my application.

I just lost my Chrome.

All right.

Let's come here.

So it's a real app.

There is a real server running on my computer just for the purpose of the demo.

And the application is just a list of posts.

Hello, everybody, I can say.

I can say, everybody.

So how does this work?

So let's do something dangerous.

And it's not that dangerous.

I'll do more dangerous things later.

So I just disabled the network.

And I send one more.

So right now, please forgive my UI.

So there is an upload club on the right bottom of text that tells the user,
this is not the same as other ones.

It's almost going to send.

It's not there yet, so if I actually look at my real server.

And my application, when the network recovers, it will eventually post it.

And it's going to update the UI.

Now, with some other examples we saw in the previous time-- actually, we can
crash the server, too.

So now the server is not running anymore.

So for this demo, I have a very simple back off time.

But it'll try back off.

ADAM POWELL: I mean, it does cry, too.

YIGIT BOYAR: These are my application-- yeah, it does cry too.

These are my application logs here.

It actually keeps trying to post it, because there is network.

It's just the server is crashing.

And now I finally bring the server up-- oops, not like that, not like that.

OK, so it started backing off.

It will eventually send the comment to the server.

It Dies there.

It keeps retrying.

And yes, it did send.

So let's do one more case where we do crash the server again.

OK, one more comment-- it's the same.

Oh wait, it didn't crash.

Sorry, I crashed the logger.

So we crashed the server this time.

See, the application works.

ADAM POWELL: You wrote a server that doesn't even crash right.

Come on, what do we have here?

YIGIT BOYAR: It should crash for the purpose of
the demo.

I hope your servers won't crash.

So I sent one more comment.

And then I simply go out.

So my emulator may crash, though.

It's been having some problems.

I did go out.

And I am going to kill that application.

I come back.

OK, so the comment is still there.

It cannot update the feed, but the comment is waiting for me there.

And I will just run the server.

The screen does eventually synchronise.

So the idea is the application doesn't care about when the comment goes.

It's staying in the disk waiting to be uploaded.

There is some other job that's taking care of the upload.

But it will eventually go to server, unless I have a bug, of course.

So how do we take care of these things?

The way it works is we have this feed activity.

By the way, again, this is a demo application.

I'm trying to focus on certain things.

There might be toerh problems with this demo application.

The important take away is the thing we're focusing on here.

So this application, when you call Send Post, it simply tells the feed
controller to send the post.

Which, as the job to the disk-- there's something persistent that will
eventually run, which is responsible to update the disk, as well as sending the
post.

So it updates the disk.

And then this [?

pitches ?] an event.

When this event is this page, my activity knows about this event, refreshes
itself.

When it runs, it does the same thing.

Or when it's cancelled, it does the same thing.

This is a persistent job.

That means it's going to be saved in the disk until it succeeds or reaches the
retry limit.

They say, OK, all right.

The comment is posted-- nice.

And so another problem about synchronization-- what happens if I'm offline?

Send comments.

So it's not working, eventually.

And I go to this website.

And I create a comment here.

I'm from the web.

So now these two states are inconsistent.

My client doesn't know about the new one.

And I have a comment locally.

So if I enable my network-- by the way, normally, when the network comes back,
you would refresh your feed.

But I'm not doing it for the purpose of the demo.

So when the network comes back, it's going to send the post instantly.

So it's out of sync.

But when I eventually refresh, it's going to add anyone to the correct order.

This is because everything is time stamped by the server.

So until a post is sent to the server, you make a best guess time stamp on it.

So for this example, the way I do it is-- again, it may change per your
application.

But the way this one does is-- wait, let me open the [?

safe ?].

OK, so when it creates, it actually creates the
model to get the best time stamp I have.

This is a way to synchronize if my client's time is, really, really off.

And if the new comment is not on top, that may look really bad.

Because it changed your server ordering, your UI ordering.

But that may cause problems.

So what I do here is I assign a best case time stamp locally.

And then we do it on later on server.

So we get the network again.

And again, like another case, because of the persistence, so I send the post
here, click on the usernames, moves it to the user's post.

And it's done.

I never fetched this user feed.

But the post is there.

It's everywhere in my application.

Because it's on my disk.

I don't care if it is not synced or not.

I have a proper model for that object.

It's a simple value object.

But it's always working.

And let's say, I'm in the user feed, but then recover the network.

It will eventually send.

And the feed will update, too.

If I go back, it's already updated here instantly.

This is because if a UI is interested in a certain event, it listens for it.

Now, I'm using a global event bus here.

You can probably implement the same thing with RX or Broadcase.

This is a sample application that does it this way, and it works.

There might be multiple ways to do the same thing, the right thing, or maybe a
better way.

So we'll get more fancy.

OK, what if the server is crashing?

So this is my real server, by the way.

It's a lot of fun.

It's a magical language.

So I'm going to turn on a flag here.

What this will do is the server is going to save the post, but it's going to
crash afterwards.

So I won't be able to realize that I could synchronize that post.

So the server is crashing now.

I go back here.

I can say, OK, [INAUDIBLE].

So the job tries 20 times.

And it backs off 250 milliseconds exponentially.

So if will keep trying, trying, trying, try me again.

  So now the server is crashing, as you can see the
crash logs.

It saved the post to the disk.

But the client doesn't know.

So the client keeps retrying.

But what happens is that your fetch feeds post endpoint is working fine.

So you refresh.

So the post came back.

I know it's the same post.

So how do I know it's the same post?

This comes into a little bit API design.

But we want to do a full demo.

So I did figure out it's the same post and updated it.

Although my sent posts have never succeeded.

So the way it works in this application is, by design, each element is assigned
a client ID.

So if you look at these examples here-- are they visible?

Yeah, all right, so there's a user ID.

And there's also a client ID.

This client ID is randomly generated a unique per user per post.

So there is a unique key in my [INAUDIBLE].

It says user ID and client ID.

It's a unique tuple.

You rely on the fact that it's not going to conflict.

I mean, technically it can.

Probably it won't.

So by doing this, when I receive the post from the server, I know the post
already exists in my disk.

And I update that one, saying that, OK, I sync this one.

I don't know what happened to the job.

I don't care.

And then the job knows while running.

Before trying to send the post, it checks, hey, is this already synced?

So here, if you can see the code here, it just loads it.

And if it's already synchronized, it doesn't do anything.

Oh, hey, this has been updated.

Good news.

Everything is working fine.

  So one more thing-- OK, so the last idea here is
that this works because we look at the problem as two separate problems.

One of them is only responsible-- take the user interaction, process the model
locally.

And the other one is responsible to synchronize with the server however it
does.

It's not trivial.

You're writing a bunch of additional code to say, local client IDs have to
handle conflicts.

But it works.

So let's look at one more demo here.

This time, I will shut down the server.

So I'm sending-- I'm again that excited kid.

I keep sending all these messages.

Of course, they're not going to go.

If I look at here, they're not going.

When I recover the server, now the first job is backing off.

It will eventually finish.

Meanwhile, I can send more.

  As you can see, they are going out one by one,
one by one, in the correct order-- my scroll position lost.

This works because these jobs are running through the same queue per pause.

So I know that whoever came first will be handled first.

So we guarantee that they are in order.

We cannot guarantee they are time stamped.

Because that's technically not possible unless you want to rely on what your
client timestamp is.

But we granted their order.

Again, this is all independent of the UI.

It doesn't care at all.

The only thing it knows is, how do I load this from the disk, and how do I send
these comments to my application logic?

One more example I want to go-- actually I want to go through this code.

So one of the problems with this event-driven architecture is since all these
events are happening, how do I synchronize?

What if I miss an event?

Or what if the timeline's timestamps don't match?

Again, for this demo application, the way this one works is every single event
that is important comes with a timestamp.

So for example, for fetching a feed event, timestamp is the timestamp of the
oldest post.

So if someone is going to load those once, it makes sure it includes that
timestamp in the query.

So it loads things afterwards, that.

So every time the feed activity receives an event-- let's say [INAUDIBLE]-- it
checks if this is for my user.

And if this is for my user, it just calls the refresh with the oldest one.

And then that refresh method, while querying the model, it uses that timestamp.

Or if there's no timestamp, it uses top one.

Because you don't want to keep refreshing things.

 :33:25,285 Yeah, that is, mostly-- let me see if I miss
anything.

 :33:33,879 OK, I think we are going to publish this code,
both the server side and the client side, to Github as soon as-- I needed some
clarification here and there.

But you can play with it.

We'll be happy to get [INAUDIBLE] or whatever if you think something could be
done better.

And it's nice to share and just move on from there.

All right, sure, that's it.

It's time for some questions.

[APPLAUSE]    ADAM POWELL: I think that we've got a
microphone floating around perhaps.

There we go.

AUDIENCE: Will it contain tests?

YIGIT BOYAR: Huh?

AUDIENCE: Will the project contain tests?

YIGIT BOYAR: Yes, it actually does.

So actually, I should have showed it.

So if you look at-- well, they may not be great tests.

I am not a testing expert.

But they do test.

They're mostly interrogation tests.

If you look at something more fancy, for example, fetching the feed, so I say
we do-- ADAM POWELL: Can we switch the slide back over to the computer real
quick?

YIGIT BOYAR: Oh, sorry.

Can we go back to the computer?

 :34:53,199 OK, thank you.

So the application uses Dagger.

This is how I marked some of the API calls.

But let's see-- fetch feed, no, send comment.

That will be nice.

 :35:11,35 ADAM POWELL: So short answer, yes.

YIGIT BOYAR: They are here.

I had some very nice tests.

I cannot find-- OK, I found it.

400.

So this is creating the job, mocking the API service so that if someone tries
to send a post, return 404.

That injects the test component, runs the job, checks the appropriate event to
send.

So one of the things is when some outputs happen as events, it's not very easy
to test them.

In this example I'm using, in the test application, I'm using a logging event
bus that basically logs every single event so that I can assert on those later
on.

Again, there might be different or better ways to do this.

But this is something that works.

And we have tests.

  AUDIENCE: What's your opinion on using custom
views instead of fragments so you don't have to deal with the fragment quirks,
if you will?

So things like child fragment manager or popping in the back stack after you
have a saved state.

ADAM POWELL: So again, I think that those are kind of unrelated.

Just because a fragment controls views.

It's not a view itself.

That was kind of what we were trying to get at a little bit earlier.

Overall, I think that a lot of people write fragments when they really meant to
write a custom view.

So I think that in many cases, see exactly what it is that your particular
chunk of encapsulated UI needs to express and respond to.

If those are events that can be coming from some alternate component that's
kind of composing that piece's place within the overall UI, then that speaks to
it being probably a little bit more like a view group.

If it's something that really needs to respond to other elements of the
application lifecycle and the activity lifecycle, if it's doing something like
registering for events, dealing with things like that, then it may be more
appropriate for a fragment in that case.

But overall, I think that people tend to lean a little bit too heavily on fragments when they think of one particular subsegment of their UI.

  AUDIENCE: Hi.

One question I had was, how do you handle the serialization of these jobs to disk and back out when the app dies?

Do you roll your own logic, or do you use something like a framework or something?

YIGIT BOYAR: For this demo application, I'm using Job Queue, which is some library I wrote before.

You can simply serialize them.

You can use Tape.

There was recently another talk to use Tape to serialize your stuff, to make it like a job.

There's multiple solutions for this out there.

It's not that complex either.

You can choose one of them which fits your needs.

AUDIENCE: Cool, thanks.

ADAM POWELL: We've got one up front.

AUDIENCE: In this kind of application, would you use a sync adapter, or would you use like async REST calls to contact the
server?

YIGIT BOYAR: So for the demo-- so one of the things we gave in examples in the slides was to separate your local tasks from the server tasks.

The way we do it is for UI-related stuff, I'm using a sync task.

I never use it for network, only for UI stuff.

ANd for network, I'm using the job queue.

So the jobs are the only ones that are calling the network.

ADAM POWELL: Yeah, the sync adapter is generally useful once you have a more sophisticated account infrastructure, and you have some of the other machinery that that was really kind of built around.

So if it seems like you're going through a lot of additional machinery to work with a sync adapter, then maybe start taking a look at some more things, just like a job scheduler, and so on, and so forth, and see if that's something that is simpler that can still meet your needs.

AUDIENCE: When you start building all these applications with these different versions of data models-- so you might have, like, fetch data off the network, updates that the user has made to the data that hasn't been actually pushed to the database, do you guys have any classes that can help us maintain all this application logic or any libraries that you
would suggest we use to maintain all these versions?

ADAM POWELL: Do you have any suggestions?

YIGIT BOYAR: I'm sorry, nothing about versioning.

But that actually reminds me of something I forgot.

One of the important things in your client is your client's consistency.

And the server may crash.

But your server may also start returning bad data.

So one of the things this client does is, if I open a user model, let's say--everything save.

So before we save anything, we will date it.

And if it is not valid for this client, we just throw it out.

Because I don't understand.

The server has a bug.

Maybe the server is a new version.

I don't know.

If I don't understand object, I throw it out.

This makes it easier for the rest of the application to, like, OK, if the object is there, it's properly filled.

ADAM POWELL: So to the point of dealing with specific data inconsistencies, especially across upgrades, different server versions, and so on and so forth, what we found with working with a lot of teams, both within Google and external to Google, is that many times, the actual pieces of business logic that make an app interesting tend to differ so much between applications that we've had a lot of difficulty finding a one size fits all solution to just offer as a prescription of, hey everyone, go use this for dealing with this particular problem space.

It is something that you can make a lot of assumptions based on the shape of your data and what it is that your data is actually representing.

So often times, you need something a lot less complex than some general solution that solves for the entire world.

But if you've got some ideas, please track some of us down in the halls here.

And we'd love to hear about them.

Anyone else?

All right, well, thank you very much.

YIGIT BOYAR: Thank you.
