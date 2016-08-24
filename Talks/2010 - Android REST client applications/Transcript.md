--------------------------------------------------------------------------------
>>Virgil Dobjanschi: Good morning.

My name is Virgil Dobjanschi.

I am a software engineer at Google.

I work in the Android Application Group.

I am also the author, more recently, of the official Twitter app.

I want to welcome you to the session today.

We are going to talk about the development of REST client applications.

Before we begin, I would like to invite you to view live notes and ask
questions during this session by using Google Wave.

I am going to give you just a few seconds to write down this URL if you haven't
done that already.

While you are doing that, I just want to gauge your opinion about the Froyo
announcement.

Were you guys impressed by this announcement, by everything we announced today
in the keynote?

[ Applause ] >>Virgil Dobjanschi: Well, I work in that group, and I am
impressed.

I'm amazed by the amount of stuff that happens in that group.

And I invite you to go to all the other Android sessions this afternoon, learn
about Android cloud-to-device messaging and all the other great lessons you can
learn and you can use when you are developing Android applications.

So we have a lot of content to cover, so let's get started.

So what is REST?

REST stands for representational state transfer.

It is a broadly adopted architecture style.

This style is comprised of clients and servers.

Clients make request to the server to get or change the state of a particular
resource.

Servers respond to the client with representations of resource.

So what is a resource?

A resource is any meaningful concept that can be addressed.

For the purpose of this talk, I am going to reference back to the Google
Finance API as a simple example of an API.

So a resource in case of the Google Finance API is a stock portfolio, which is
characterized by its name and the currency in which it operates.

The representation of the resource is simply a document that fully describes
that particular resource.

In the case of Google Finance API, it's an XML document which simply describes
that particular resource in terms of its name and the currency, and maybe its
ID for the purpose of referring to that particular resource later on when you
need to update it or you need to delete it.

REST is in no way associated with HTTP.

You can use any transport protocol.

But certainly HTTP is the most common used transport protocol for the
REST-style architectures.

There are a large number of REST APIs available today.

Google code offers already a host of such APIs.

Social networking sites, such as Twitter, Facebook, MySpace, already have a
very rich REST API.

You can find all kinds of information-based REST services for calendars, for
flight schedules and so forth.

The first question you are going to ask yourself before you start the
development of a REST client application is why would I possibly develop an
application like that if the service that I am connecting to already has a
mobile-friendly Web site.

In other words, why don't I just use the browser to connect to that service and
I'm done?

Well, until the browser technology catches up, I have five reasons for you.

I hope I can remember them all.

The first one is that your application, your Android application, will be able
to integrate with the Android platform.

It will be able to use intents content providers.

It's going to be able to access all the private APIs available today, only to
Android applications, things that at least at this point you can't do from a
browser.

Your application will be able to invoke intents, not only to pick photos but
all the other intents that are today available for all the applications to use.

The second reason is, your application can offer intents to other applications.

In other words, you are going to be able to enrich the behavior of the platform
by offering new functionality to other applications.

Content providers are there for you to access.

The context content provider is a very popular content provider.

Lots of applications expose it.

The second reason is your applications can run in the background.

What does that mean?

It means that if you would like to have your application refresh the data from
the server and new data is actually retrieved from the server, your application
has the option to present a notification to the user to let them know that that
particular data is available.

A browser can't do that.

Your application runs on devices with limited connectivity.

What does that mean?

You are going to try to initiate some of the REST methods, and they will
sometimes fail because you operate in environments just like here at the
conference where the network comes and goes.

An Android application has the option to run in the background and retry
operations in the background by using an alarm, for example, for the purpose of
relieving the user for trying to hit that refresh button or post button in the
browser.

Your application can actually be faster than a Web browser.

Your application has the option of retrieving all that network content in JSON
or some binary or some XML format, parse it and store it in a database.

From there on, when you want to retrieve new content, you may have the option
of retrieving only content that's newer than the one that you already have or
older than the one you already have, but not the same data.

You also are not retrieving all the HTML that comes along or any kind of
JavaScript that might be too big and might take too long to download.

Finally, your application has the option of being consistent in terms of user
interface with the REST of the operating system.

You can innovate in many ways to deliver a fantastic user experience to the
user.

If you have participated yesterday in the user experience talk, please take
advantage of those UI patterns.

Your application is going to be so much faster and easier in some cases to use
than the browser.

And finally, when users are presented with the option of using a Web browser
versus a REST application, I can guarantee you that most users will choose the
REST application.

Before we begin, I would like to describe the way a novice programmer would
approach the development of a REST application.

That's not you.

You guys know it all, but just for the purpose of this discussion, let's look
at these things.

Hopefully, it won't take you 10,000 tries to get there.

But I can tell you that I make these mistakes myself.

It's actually human nature to try to learn as little as you can about a
particular thing and move quickly forward and say, okay, I am going to have
this app ready in ten minutes.

And I am going to say slow down.

Get all the facts first.

Learn what's correct.

Implement it correctly so you have a good base for your application to get
going.

So you are a Java programmer.

You have looked around the Android SDK a little bit, and you say to yourself, I
know what an activity is.

How hard would it be?

I am just going to create this activity.

I understand a REST method takes a long time to execute because I need to
connect to a server, so I am going to run it in the context of a thread.

You decide that this thread is going to be an inner class of your activity.

And you decide that you are going to store all this data that you retrieve from
the server in memory.

Maybe the application I am describing here is your attempt at displaying the
list of stock portfolios that I already have set up for my own account in
Google Finance.

So you get your application to run.

It runs fast, it doesn't crash.

So why is this not the correct approach?

You have to understand how the Android operating system works.

The Android operating system was designed to work on devices with limited
memory.

What does that mean?

It means that, for example, when the operating system is attempting to start a
new application and it ran out of memory, it needs to make a decision about
forcefully shutting down an existing application.

So how does it make that decision?

Well, the answer is you have to help the operating system make that decision.

If your application does not have a foreground activity running -- in other
words, a visible activity running -- or a service running, the operating system
can say, okay, well, this application is not currently displayed to the user,
it's not currently performing any operation, so it's safe to shut it down.

So let's now go back to our diagram and try to understand what's wrong with
this picture here.

Well, here's what happens here.

You launch the thread that initiates the REST method, and guess what?

You pause your activity because maybe an incoming call needs to be answered.

What happens is the operating system says, oh, well, this particular process
doesn't have any foreground activity, so you know what?

I am going to kill it.

I am going to forcefully shut it down.

Well, guess what?

Your post, put, delete method maybe has executed on the server and you will
never learn the result of that particular method in your app.

The get method, you get all this data in, once you get it in, you parse it, you
are happy, the operating system will shut it down for you.

Wasted bandwidth.

Another thing that's wrong with your approach is that you decided to store the
data in memory.

So you say, well, hang on.

My application is fast.

Well, the reason why that's not a good idea, it's because by having to
repeatedly retrieve data from the server, either because the user has simply
restarted the device or simply because at one point your process had to be
forcefully shut down, you are wasting CPU, you are wasting battery, you are
wasting network bandwidth.

That's not a great way to write an Android application.

And for those of you who are going to say, well, it's much faster to get this
data from memory, I am going to say by using a content provider, you have the
option, should you consider that necessary, to cache this data on memory.

So that's really not a good reason to say I'm not going to store my data.

As I said, none of you will probably write the application this way.

So let's describe today three design patterns that you can use in your
applications that would give the best user experience and the best performance.

These are not the only three patterns you can use.

Once you look at them, once you understand what they are all about, you can
possibly create your own.

That's great.

As long as your application complies with the basic principles that we're
laying here for you, it's okay to continue to innovate from here.

We're not going to talk about implementation details.

I am not going to show you code.

Hopefully you like that.

So let's introduce these three patterns.

One is based on the service API, one is based on the content provider API, and
the third one is simply a variant of the second one.

We are going to use a sync adapter.

This is an advanced talk.

Last year we got a lot of feedback from you guys saying some of the sessions
were too light.

We decided to compensate this year.

Even though it's an advanced slide, I am going to try to introduce these
concepts which some of you may not be familiar with so you understand the basic
idea behind it.

So let's get started with the first pattern.

The first pattern makes use of a service.

Again, for those of how are not familiar with a service, it's an Android
component that does not interact with a user, and it's specifically designed to
execute operations in the background.

It runs in the context of the main thread.

We can call that the UI thread.

And in order for you to execute these operations, you need to still create a
worker thread in the service.

Some people think that by simply starting the service, I can simply execute in
the context of the thread of the service that particular long-running
operations.

That's false.

So now that we understand that the service exists in the platform and has, by
the way, a very simple API.

You can either implement the services API by using an intent API or by using a
binder interface.

By the way, for the purpose of this talk, we are going to use the intent API
which is, by the way, a little bit easier to understand.

We can now get started with this first pattern.

This diagram is not exactly simple.

To best understand it, we are going to take a bottom-to-top approach by
explaining the role of each component.

And once we are done, we are going to do one example starting from the top and
running through an entire code flow to see how this works.

So let's get started with the REST method.

What is the REST method?

Really simple.

It's an entity which has the ability to prepare an HTTP URL, to prepare the
HTTP body in some cases, execute the HTTP transaction with the server and
process the response, which typically means parse the response from the server.

There's not much to say about this.

Let's make some performance remarks.

Many REST APIs offer today the ability that you select the content type of
responses.

Prefer them in this order.

Some binary format, AMF3, whatever is out there, JSON and XML.

We are happy to announce that if you choose JSON, we have a brand-new
implementation of org.json API.

It has exactly the same API as before but it performs much faster.

There is very little garbage collection going on.

For those of you who tried the older version, you are going to be very happy
with this particular implementation.

Enable gzip.

If the REST API that you are using allows it.

Gzip is a very fast library.

It's a native implementation.

Sometimes you get compression ratio five to one, ten to one, depending how much
data you are retrieving.

And by doing so, you are simply speeding up the download of data that you want,
plus the battery life will be preserved better in this case because the radio
will be in use for less amount of time.

Always think about the radio, always think about the network and the
implication it has on the battery life of that particular device.

Always run this REST method in a worker thread.

Understand that an HTTP transaction takes time to implement, sometimes a long
time if a timeout occurs.

And you should never run such operations in the context of the main thread.

And finally, use the Apache HTTP client, not the Java URL connection.

All right.

So what do we have so far?

We have the REST method, a simple API.

It fires back a Java listener callback.

Really simple.

Let's take a look at the processor.

The role of the processor is to mirror the state of server resources in your
local database.

We're going to use the database.

The role of the processor is to say, okay, this is the state of this particular
resource.

So to better understand how the processor works, we need to create this concept
of a resource in the database.

Well, it's really simple.

It's exactly one row in the database.

In the case of Google Finance, it's a row that maybe has the idea of this
particular resource, the name of this particular portfolio, and the currency.

And we are going to add two more columns to that.

One, a status column, and, two, a result column.

The status column is the one that simply indicates the transitional slash
at-REST or steady state of that particular resource.

What that means is that every resource, when it executes a REST method, has a
transitional state; right?

While you are executing this particular transaction, the resource is not yet in
sync with the server so we want to keep track of this particular state.

Your question is why.

Why would I possibly need this?

Two reasons.

One, your user interface may want to display maybe a little icon next to the
resource that says, oh, well, you know, I don't have to stay in the activity
that actually started this post, the creation of the portfolio.

I can just leave that activity.

And my list of items is just going to display a little icon next to that
particular resource saying I'm in the process of synchronizing the resource,
but go ahead and use the app.

I will let you know when that's done.

When that disappears, that's a good indication that your resource has now been
synchronized.

So in the standard set of these, we are going to hold some flags.

These flags are going to say I am in the process of executing a post method for
this particular resource.

I am in the process of updating, deleting this particular resource.

And we are going to use one more flag to say we are in the process of executing
an HTTP transaction for this REST method.

Now you have a full description of anything what's going on.

At any time you can look at the database and say, okay, I know exactly what's
going on with this particular resource.

The result method can simply hold the HTTP result of the last REST method you
executed in regards to that particular resource.

So now let's come back to the processor.

The processor executes before and after the REST method executes.

Before the REST method executes, in case of the post method, it's simply going
to insert the content -- it's going to create a row in the database, it's going
to call an insert method to create the row that corresponds to the resource we
need to update.

We are going to set the flags to posting, meaning we're in the process of
executing a post method for this particular resource.

We're going to say yep.

We are also transacting, by the way, so we are going to set that flag, too, in
the column, and we initiative the REST method.

We are going to update the row in the database with the help of the content
provider.

We are going to clear the state posting fire because that particular method is
no longer pending, and we are also going to clear the transacting flag so we
know we are no longer transacting the server.

Put works very similarly.

It's hardly worth going through this, but we're going to update before the
(inaudible) execute, we are going to update this particular row in the
database.

We are going to set the updating flag, meaning we're in the process of updating
this resource on the server.

Set the transaction flag, execute the REST method.

Once it's done, update it again, and clear the two flags that we just set, if
everything goes right.

By the way, as you well know, most correction REST API implementations are item
potent.

It means that you want to send a document to the server saying, okay, I want to
change the state of this particular resource on the server, and you send this
document representing the change you want to make, and the response that comes
back, it's the full description of that particular resource as the server sees
it.

In other words, the state on the server side.

That's why it's recommended that you go ahead and update again, although many
times that's not necessary.

It's up to you to decide.

But either way, in this case you are going to have to clear the flags.

Delete.

Very simple.

You are just going to delete -- tag that particular method for deletion.

You are not actually going to delete the resource.

Why?

Because we have not yet successfully executed the REST method.

You go ahead, execute the REST method.

It succeeds finally, you delete that row.

Finally, you are done.

Your user interface, by the way, in this case, has the option of saying, well,
you know what?

If the state deleting flag is set in the status column, I am not even going to
bother displaying that particular resource.

Or you have the option to display a little synchronizing resource right next to
that particular resource to know that we have not yet deleted it but we are
about to.

By the way, these flags can serve a different purpose.

What happens when you need to retry these operations?

Well, wouldn't it be great to know what exactly do you need do.

The posting flag would mean, oh, I need to execute a post method.

I am going to look at the transacting flag saying is there an HTTP transaction
already posting?

So if that's the case, I am not going to start another one.

If it's not, I have the option of saying, okay, well, let me retry this.

Maybe the network didn't work last time.

But that's a great way to mirror the state of the resource in the database and
retry these methods.

Get, simplest one of all.

There's nothing to do.

Processor has nothing to do before the REST method inserts the new resources it
wants to get from the server.

This could be the list of stock portfolios, maybe you have many of them on
Google Finance.

One more thing to say about processors.

They execute database operations.

Never execute database operations or content provider operations in the context
of the main thread.

Sooner or later you are going to run into application not responding errors.

The so-called ANRs.

It's not a good practice.

Learn to stay away from that.

Also, use transactions when you SQL light.

Not only will they preserve the data integrity but they will increase the
performance of your database operations.

We have the processor, we have the REST method.

The processor has almost the same API as the REST method, it goes through.

But this time we are able to mirror the state of that particular resource in
the database.

We said at the beginning that it's not a good idea to start long-running
operations in that activity.

That's where the service comes in.

The role of the service is to simply allow your application to execute
operations in the background while an activity is long gone.

I am going to say it again.

An activity comes and goes.

It's just the user interface of your application.

The user has the option to hit the home button, hit the destroy button -- in
other words, go back home.

Your long running operation, it's still executing.

And the nice thing is that you have the database.

Simply store the state of that particular resource in the database and when you
come back to your application, regardless of what happened, you can simply
learn what happened.

Did it work?

Is it still pending?

What's going on?

On the forward path, the service simply receives an intent.

We said we were going to use the intent-based API.

It unpacks the content of that intent, which think of it as a map.

Oh, yeah, I passed in, for example, the name of this particular stock portfolio
and the currency.

I am going to extract it from there.

Make a Java call to the processor.

Done.

On the return path, I am simply going to handle the processor callback, and I
am going to invoke what is called a binder callback.

What is a binder callback?

The binder callback, think of it as an interface that was passed in the request
intent.

So the upper layer says when you are done with this particular operation, I
want you to invoke this binder callback and let me know that everything worked
or failed.

Here is another thing you can do in the service.

You write a social application.

And all of a sudden your UI needs to download ten, 20 images just because you
are scrolling really fast through that list.

Is it a good idea to download 10, 20 little images at the same time?

The answer is no.

Be nice to all the other applications.

They might need the bandwidth at that time.

Implement a little queue in this particular service.

Allow maybe one, two, or three, as it is the case in your application, for
parallel downloads of these particular images, especially if these particular
images are larger.

Always think about the other apps when you are writing one.

And finally, the service is a stateless component.

Everything it needs, it receives the intent.

Fires the callback.

Unless you implement the queue we were just talking about for downloads,
there's no reason to keep any state.

And most importantly, while all the pending operations have completed, shut
down the service.

If you don't, the operating system is going to say, well, this guy is busy.

You know, let him keep the memory that it wants.

Let him do whatever he needs to do.

Be nice, again, to all the other applications.

Shut down the service once you are done.

It's very easy.

We are going to say for the purpose of the service helper to stop the service
or the service can stop itself.

There are many ways to do it.

You have no excuse to leave that service running.

Almost there.

We have a service with an intent API, and asynchronously fires a binder
callback interface.

Sometimes preparing these intents is not that easy.

Or at least not elegant.

I know that.

And handling the binder callback is not something we should leave as an
exercise for the reader.

What we should do is have a very thin layer, we call this the service helper,
that prepares these intents for us on the forward path and handles the binder
callback on the return path.

So the role of the service helper is to simply expose a very simple
asynchronous API to the caller.

Let's say we want to create a portfolio.

The name of the method is create portfolio, here is my name, here is the
currency.

It returns the request ID, which uniquely identifies that particular pending
REST method.

It's a Singleton.

There's nothing to it.

You make this call and here is what the service does.

It's much simpler than it looks.

The first thing you may want to do is have the ability to ask the question is
this particular REST method already pending?

That's certainly not the case with create.

But if you do maybe a get my stock portfolio list, what if you issue this
request and some other activity which quickly you switch to issues yet another
request.

Well, you certainly don't want this particular situation where, let's say, you
are downloading and parsing the stock portfolios in parallel in two different
threads.

It makes no sense.

So you can protect yourself in this particular pattern here in the service
helper.

You can implement the map from the request ID to the this intent.

And you can always, first of all, ask the service is this particular request ID
still pending?

You can do something else.

You can look at the values, the intents in the values method and say by looking
at the intents you will be able to answer the question is this particular type
of operation, maybe even with these parameters, already pending?

And if yes, you simply return that request ID to the user and it's a nice way
to get around some of these problems.

And finally, you put the operation type, you put any method-specific parameters
such as the name and the currency.

You set the binder callback and you finally call start service which simply
sends that service to the service that we discussed earlier.

On the return path, really easy.

You get the binder callback, and this time you say, okay, this particular
operation is no longer pending.

And you can dispatch a callback to any listeners.

Listeners are typically going to be activities that would like to know the
result of this particular REST method.

One interesting discussion here is what kind of data should I pass from the
service to the service helper?

And you are going to say, okay, you just retrieved that list of stock
portfolios.

Should I just pass it on?

Should I just move all that data through this particular binder callback?

Well, think of it as a marshaling interface.

In other words, something that has to cross process boundaries.

Well, you don't want to do that.

Keep as little data as possible in that particular callback.

For example, you can say the result of this particular REST method was 200.

Okay.

You can say here is, by the way, the original intent which I used to start this
operation.

Maybe the caller would like to analyze the content of that particular intent to
remind itself what it was executing at that time.

All the day that you already retrieved, it's already in the database.

The activity has the option of simply saying, okay, thanks for letting me know.

I am going to go and retrieve that content.

There's absolutely no harm done if you choose once in a while to pass some data
through this binder interface.

You can use any parcel-able, think of it as serialization technique, to send
data over this interface.

The less data you parcel, the better it is.

So we are almost done.

The service helper has a simple API.

If there are any listeners that register, these calls go back to the activity.

Let's look at the activity.

Always remember the activities are pause, resume, or destroy.

They have their own mind.

They have, better said, their own life cycle.

Think of them again as just user interface that is either controlled by the
user.

In some cases activities pop in front of your application just because maybe a
phone call comes in, in which case the application will be paused, the activity
will be paused.

And on resume, we have the option of adding a callback to the service, say,
okay, I'm back.

I'm interested in what you have to say.

On pause, remove that, because if a callback will occur while your activity is
paused, sooner or later, you'll get a crash.

You're trying to access this activity while it's paused, and that's certainly
an incorrect technique.

Always consider these three cases.

A request method is started by the activity.

The activity just sits there in the foreground.

Everything is nice, it gets the callback from the service helper.

Easy.

You, in case of an error, you can say, I'm sorry, I couldn't perform your
operation.

Or, hey, cool, we created the stock portfolio.

What if the activity starts the REST method, pauses, it comes back, it resumes,
and now the callback occurs.

Well, that's not much more difficult.

However, you can still handle the callback.

Everything is great.

You -- in this case, you may need to store the request ID so you know to ask
the service helper, is this particular request still pending?

And so -- and on resume, the answer this time is going to say, yep, that's
still pending.

Okay.

So I'll just wait for the call back.

Nothing special.

Third one is the most complex.

REST method is started.

Activity's paused.

REST method completes while the activity is paused.

Activity comes back to life and resumes.

What's happening?

Well, in unresume this time you're going to have again the request ID that you
started, and you'll say is this request still running, and the answer is going
to be no, it's not.

So now you understand why we chose to store the result in the database.

The application has the option to go back to the database and figure out
exactly what happened with this particular pending request while it was maybe
destroyed, maybe paused, and so on.

Remember, that operation -- that long-running operation, the REST method, was
running, because it executed in the content of a service, and the operating
system said, "Okay, I'm good, just going to leave this particular process
alone, because it's executing what it thinks it's important." So that's great.

On the return path, here we have the option to receive callbacks from the
ContentProvider itself.

The ContentProvider supports what's called content observers.

Content observers are simple ways to receive notifications from the
ContentProvider when data changes for a particular set of entries in a
particular table.

That's a simple explanation, but that should do.

In other words, you can monitor the status of a particular row or maybe all the
rows, all the resources in that database.

We reached the top.

Let's go -- take a full example and run with it.

The activity decides to create a portfolio.

It says, okay, I'm going to call the simple method in the service helper called
"create portfolio." The service helper says, okay, I'm going to  -- new intent,
packet everything I need, the name of the service, the currency, the binder
callback, the type of operation, I'm creating a portfolio, the unique request
ID.

It calls start service.

It says this particular operation is pending and returns the request ID to the
caller.

The caller has the option to use this particular request idea to find out if
this particular REST method is still pending.

Finally, the intent gets the service, the service unpacks it, makes a Java call
to the processor.

The processor said, what kind of method corresponds to this particular Java
method?

Oh, this is a POST, I need to create a new stock portfolio.

So I'm going to go and insert that content into the database, in other words,
I'm going to create a row in the database.

I'm going to set the status posting, I'm going to set the transaction flag in
the status column and I'm going to say, okay, I'm ready to execute my REST
method.

The REST method prepares the URI parameters, prepares the request body, if
that's required  -- and for Google Finance, that's certainly the case -- gets
the callback once the REST method transacts with the server, it obviously gets
the parts resolved back from the REST method.

The processor says, okay, I'm going to update the state of this particular
resource in the database.

So it updates the content, it updates the flag by saying, okay, this particular
method is no longer pending.

We assume, obviously, that it worked.

And the ContentProvider, with fire notifications as needed, if there are any
kind of content observers.

On the return path, processor says to the service, thank you.

I'm done.

Service says, okay, I'm going to invoke the binder callback that was already in
the intent.

And, finally, if the service -- service helper sales, you know what?

This REST method is no longer pending.

I'm going to let any activities know about the result.

And the activities have the option to handle that callback and display to the
users and so forth.

So that's a quick run through this particular design pattern.

Next one.

Based on a ContentProvider API.

What is a ContentProvider?

For those of you who don't know, it's a simple way in Android to store and
retrieve data across process boundaries.

Typically, a ContentProvider sits in front of the database.

It offers the insert, update, delete, and query methods that, in the end,
access the database.

Really simple stuff.

I can go into details.

Hopefully, that explanation will be sufficient for the purpose of this
discussion.

So the first question you're going to have from me, okay, how in the world did
you get to the conclusion that you can use a ContentProvider API to initiate
REST methods?

Well, the answer is simple.

There's a one-to-one mapping between REST method types and method -- some of
the methods in the ContentProvider.

So the get REST method corresponds to query in the database.

The POST REST method corresponds to insert.

The PUT REST method corresponds to update.

And, finally, the DELETE REST method corresponds to delete.

Really simple.

It's not hard to imagine how this works.

So here's the pattern that we can use.

Let's say -- By the way, in this particular diagram, the service helper, the
service, the REST method work just as before.

The ContentProvider and the processor have a slightly different functionality,
and we can cover that once we go through an entire flow to this particular
diagram.

So let's assume as before that we would like to create a stock portfolio.

The activity says, okay, this is going to be a creation, so that corresponds to
a POST.

Obviously, I'm going to do an insert.

It calls the insert -- prepares the content values.

For those of you who are not familiar, it's just a way to specify the values
that you would like to insert in specific columns in the ContentProvider.

Calls insert.

ContentProvider says, okay, I'm going to create this row for you.

And before it returns, however, it does a trick and says, "I understand that
this particular operation needs to execute a POST REST method." So it
initiates, with the help of the service helper, the POST method.

It returns the URI for the new list -- the caller, the activity in this case.

It says, okay, I inserted for you, and by the way, I'm going to keep the
transactional state for this particular resource.

The state posting will be on, the transactional flag -- transactioning flag
will be on.

And, finally, the activity has the option to use these flags, should it need,
to display the status of this particular resource while it's transitioning in
the UI.

That's up to the application.

Or choose not to so this it at all, or, you know, whatever policy would like to
institute.

Finally, the service helper calls the service, executes the REST method.

Once the REST method is successful, executes.

The processor says, okay.

Because the REST method succeed, I'm going to clear these flags, update the
row.

And the activity, maybe a cursor adapter, any content observer, will have the
ability to understand that this particular -- the status of this particular
resource has been updated.

By the way, in this case, if the REST method fails, it's up -- it's the
responsibility of the ContentProvider to say, okay, I'm going to set up an
alarm here, and maybe after five minutes, I'm going to retry it.

And maybe I'll implement some nice exponential backoff so I don't keep
retrying, you know, and kill the battery in the process of doing so.

Please note that in this particular pattern, we broke the ContentProvider
contract a little bit.

For insert and update, everything works as expected.

You're calling insert.

We insert the resource right away into the database as a row.

For update, we do the same thing.

We update the resource as the database --  forget, we do something funny.

In other words, for query, we do something funny.

We go to the database first.

We'll get the rows that already exist.

Now we have a cursor, and this is the cursor that a query typically returns to
the caller.

Now, if the activity would like to retrieve brand-new, fresh content from the
server, it has the option in the query URI to set a flag and say, I would like
to also get the latest data from the server.

So the ContentProvider says, okay, not only am I going to give you the results
in the cursor, but also I'm going to let the service helper know that it's time
to execute a get method maybe in the refresh mode to say, just give me items
that are newer than the one I have.

Delete breaks the contract.

For delete, you say, I'm going to just tag this row for deletion.

I'm not going to delete it right away.

And, finally, the row gets deleted once the particular delete REST method has
completed.

It's not a big deal.

You just need to think about it.

And maybe once you display your results in a list activity with a cursor, again
look at the flag to see what is the state of that particular resource in the
database there.

There are other ways to do this.

You can remove the row from the database, possibly move it to another table
where you're going to be able to retry delete requests.

Again, we're not forcing you to adopt these particular design patterns.

The last pattern is simply a variant of the previous one.

We're still going to use the ContentProvider API.

But we're going to use the help of a sync adapter.

How many of you know what a sync adapter is?

I was afraid of that.

The sync adapter is a concept that you should learn as soon as you get home and
you start developing for applications.

What it is is the ability of the system to help you synchronize remote and
local content.

The system employs the help of a service manager  -- I'm sorry, a sync manager.

The sync manager manages all the sync adapters across all apps.

The sync adapters are simply services started by the sync manager.

The sync manager implements a queue of sync adapters.

In other words, if you say I would like to sync my data, there's a request sync
method in the sync manager.

It does not mean that the sync manager will say, sure, right away.

What it does instead, it queues the execution of this particular adapter behind
any other sync adapters that need to execute.

By the way, even if the queue is empty, the sync manager has the option of
saying, okay.

I'll get to it.

Why does it do that?

Why doesn't the sync manager just run out there and start your sync operation
right away?

Well, the reason is, is because it's trying to preserve the integrity of the
entire system.

What if you start your GMail sync with an e-mail sync, you develop your own
app.

All of a sudden you have four apps executing these sync operations in parallel.

They all do network transactions.

They all do parsing.

They all do database operations.

It's a mess.

Don't do that.

Also, when you -- obviously, the sync adapter is going to help you, it's going
to prevent you from getting into that trouble.

But the characteristic of a sync adapter is that it does not execute maybe as
fast as you would like to.

So with that in mind, let's look at this pattern.

Activity works the same way.

You need to create a stock portfolio.

You call insert into the ContentProvider.

The ContentProvider creates that row in the database that sets its -- a
transitional state to whatever it needs to be.

That's it.

It returns to say thank you, I'm done.

So how does the REST method execute?

That's where the sync adapter comes in.

Well, I lied.

The ContentProvider, before returning, calls request sync.

It says, when you have a chance, come back to me, start my sync adapter.

The sync adapter starts, looks in the database, and asks, are there any
resources that are in the transitional state at this point?

Yes.

Here's our resource.

It needs to be posted.

It goes, reads the row from the database, starts the REST method, executes it,
gets the result, updates the database, and says done.

What if it fails?

Well, awesome.

The sync adapters have the ability to handle errors nicely, in other words, I
should say the sync manager.

The sync adapter is responsible for simply throwing errors back to the sync
manager.

The sync manager says, hmm, there was an error while I tried to sync this, you
know what, by implementing exponential backoff, I'll queue this particular sync
adapter behind any others, if any, and I'm going to restart this sync when I
have a chance.

All of that is already there for you.

You don't have to write your code with alarms.

You don't have to do all this stuff.

All you have to do is learn how to report errors from the sync adapter to the
sync manager.

All our applications use the content of the sync adapter to refresh content.

GMail, e-mail, all these apps use that particular concept.

Please use it in your apps.

In conclusion, please do not implement REST methods inside activities.

It's poor practice.

An activity is a piece of user interface.

It's no different than any other Java framework you work with.

It's -- there's a clear separation there between user -- user interface and
functionality.

Always start a long-running operations from a service.

Also, always stop the service while -- if all the pending transactions have
completed.

Always remember that the service executes in the context of the main thread.

If you need to execute these long-running operations, start a worker thread in
order to execute these operations.

SQLite is your friend.

Persist early and persist often.

What does that mean?

POST PUT insert as soon as you can.

Update is as soon as you can.

Once you get the result, do it again, if you think that's a smart thing to do.

Please don't let your database grow infinitely.

Think about this: You keep getting new items from the server.

And all of a sudden your table holds thousands, tens of thousands, of items.

Well, your database grows.

It is the responsibility of your application to possibly purge old data.

Why is that a good idea?

Not only because the user will be mad at you because you made their phone
useless, but because the cursor has the ability to hold only about one megabyte
of data.

Beyond one megabyte, it will have to do windowing, which is a very slow
operation.

Don't hold a lot of data in these cursors.

For example, images.

If you use images in your cursor, which absolutely you could, what you're doing
is, you're taking away from your ability to store more items in a cursor.

Minimize network usage.

Only request data that's newer than the one you have.

Older than the one you have, but don't go back to the server and get that one
again.

Page data.

If you don't page data, you are retrieving this huge amount of data from the
server.

It takes time.

You parse it, takes a lot of time to parse it.

You insert it in a database.

That application will never perform fast enough.

Always use paging.

Hosting, the REST API that you're using sports paging.

Finally, use a sync adapter to synchronize the content of your local database
with the state of your server.

We're very happy to announce here at Google I/O that the Android
cloud-to-device message is now available.

You won't have to set up an alarm in order to continuously start the sync
adapter and ask the server, are you done?

Do you have new data?

Do you have new data?

Use the push notification.

Of course it takes an effort on your behalf.

I warmly recommend this afternoon's session on the Android cloud-to-device
messaging.

I would save a lot of battery life for a particular phone when using this
technology.

If you use all these techniques in your REST method and you create your own, --
let us know, by the way, we're happy to hear from you -- you will create
awesome REST client applications, very performant, and the user will simply
enjoy using it maybe over a Web browser that accesses that same service.

Thank you.

[ Applause ] >>Virgil Dobjanschi: So I'll take some questions now.

>>> Excuse me?

>>> Ladies and gentlemen, if you have any questions, please go to the
microphone, and we'll be with you in a moment.

>>Virgil Dobjanschi: So any questions?

I'm sorry.

Go ahead.

>>> Yes, I have actually two questions.

One is, what are the advantages of intent-based service invocations versus
AIDL?

And second is, would the person need to approve the sync adapter in the
settings account and sync UI for your sync adapter to work?

>>Virgil Dobjanschi: I need you to repeat the second question.

>>> The second part is, the -- would user have to explicitly approve your sync
adapter in the settings UI for your sync adapter to work?

>>Virgil Dobjanschi: So the answer to the second question is yes.

One of the nice things about the sync adapter, it obeys the user settings on a
per-account basis, which is fantastic.

Because sync adapters are linked to accounts.

So you can even say I don't want -- the user can say, I don't want this sync
adapter to run all the time.

You can -- it can select which ones it will actually run.

Also, it will base the contract of out of sync and data, let's say you're
traveling.

What's going to happen?

Well, you probably didn't subscribe for roaming, and you don't want your sync
to go in the background.

There's a framework that takes care of that and it obeys that.

That's one of the great reasons to use a sync adapter.

>>> So in this question, if ContentProvider kicked off the sync, but the user
have never yet approved the sync adapter, it will not sync?

>>Virgil Dobjanschi: It's not a manual --  it's not one of those permission
things.

It's a setting for the platform.

>>> Okay.

>>Virgil Dobjanschi: So you have to go to settings and not even in your
application rather than the phone settings and approve it.

I know you had a first question.

I apologize.

>>> Sure.

>>Virgil Dobjanschi: What was the first question?

>>> The first question was, was the advantages of the intent-based service
notification versus AIDL.

>>Virgil Dobjanschi: So there's a difference there.

That's a very complex subject.

Please look at the API of a service.

In order to use the interface, the binding, you have to actually bind to that
service.

You have to connect to it.

That's an asynchronous call.

And you may not want to deal with the complexity of it.

The intent, on the other hand, it's very simple.

You can say, here's this intent, but note that that particular operation always
executes asynchronously.

Whereas in the binder call, you have the option to execute it synchronously.

Poor design by the way, but you have no option to do so.

>>> Hi.

Thank you.

I recently came across a very useful class called intent service.

And I was wondering if -- I mean, it seems like this would be a -- like,
exactly the kind of place that you would want to use that service it.

Handles, like, the threading and everything automatically.

Do you think that's a good idea or, like, do you have any recommendations about
using or not using that class?

>>Virgil Dobjanschi: I wanted to simplify the content of the presentation.

Intent services are great.

If what you want is to execute a long-running operation in the content of the
worker thread, the intent service can certainly help you with that.

In our case, however, I don't know if you picked up on this, everything happens
asynchronously.

So I didn't have a huge need for it.

Can you do it?

Absolutely.

And, actually, it's a very good way of not having to deal yourself to create
that particular worker thread.

It will be created for you.

So, yes, absolutely yes.

>>> Okay.

And I had a second quick question.

You said that you don't need to set a flag for the get method in your database.

And I'm wondering if I'm missing something or, like, if there is something that
I might be fetching an update for from the database, so it's something that
already exists in the  -- on the server and I've retrieve it before and now I
want to see if, like, there's some value in one of the fields and the rows have
changed and I want to see if I'm getting updates for this row, I would have to
set a state for -- >>Virgil Dobjanschi: Is it too late for me to change that?

No.

Yes, it's a good -- The problem is that you didn't get anything yet.

What are you setting these flags into?

>>> No, I mean, it's a resource that you have retrieved, but you want to check
to see if, like, a value in that row has been updated.

>>Virgil Dobjanschi: However, the flags I was describing were specific to a
particular resource, not for items that you haven't yet retrieved.

That type of status is okay if you store in memory.

We can -- if you want more, I'm happy to discuss this with you after.

>>> Okay.

Great.

>>Virgil Dobjanschi: Thank you.

Let me take a question, actually, from Wave users.

Unfortunately, we're not going to be able to show any calls.

Wow.

The question regarding the Apache HTTP client versus Java.

That's a very good question.

I would simply advise that you use the HTTP Apache client, because it has a
more robust implementation.

The URL connection type of HTTP transaction is not the most efficient
implementation.

And the way it terminates connections sometimes can have an adverse effect on
the network.

Let's leave it at that.

Sorry.

Let's go.

>>> Yeah, I do have a question on, say, if you have a search use case in the
application, do you still recommend to persist that in the database, because
user is very unlikely that he's going to come back and use that data again at a
later stage.

>>Virgil Dobjanschi: So you're not referring to search across resources you
already have in the database?

>>> Right, yeah.

>>Virgil Dobjanschi: So you're going across the network.

I'm sorry to say, it's best left to the implementation of the app.

>>> Okay.

>>Virgil Dobjanschi: Apps that I worked on had different requirements.

Because this is a requirement of the user interface.

For example, should you initiate requests.

>>> As you're typing?

>>Virgil Dobjanschi: Should you initiate these searches at every single key
that's being typed?

Or should you wait until the user has said, okay, let me search now?

Up to you.

Of course, you're overwhelming the network with requests once you're doing a
search, especially if you don't have some good policy in the lower layer to
say, "Ignore this response.

Ignore this response," because the user is still typing.

Otherwise, you keep parsing, you keep inserting possibly results in the
database while the user is still typing.

So it's probably not the best way to do it.

But there are absolutely -- I have nothing against that.

>>> Thanks.

>>Virgil Dobjanschi: Sure.

I'm sorry.

We have time for one more question.

Not even one.

I'm sorry.

If you have any more questions, please talk to me right now after the session.

[ Applause ]
