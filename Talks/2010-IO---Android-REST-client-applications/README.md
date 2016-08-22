* [Video](https://www.youtube.com/watch?v=xHXn3Kg2IQE)

## Conclusion

![Google I/O 2010 - Android REST client applications](http://i.imgur.com/R7nwYWS.png)

* **please do not implement REST methods inside activities.**
  
  * It's poor practice. An activity is a piece of user interface. It's no different than any other Java framework you work with. It's -- there's a clear separation there between user -- user interface and functionality.

* **Always start a long-running operations from a service.**

  * Also, always stop the service while -- if all the pending transactions have completed. Always remember that the service executes in the context of the main thread. If you need to execute these long-running operations, start a worker thread in order to execute these operations.

* **SQLite is your friend. Persist early and persist often.**

  * What does that mean?
    * POST PUT INSERT as soon as you can. UPDATE it as soon as you can. Once you get the result, do it again, if you think that's a smart thing to do. Please don't let your database grow infinitely. Think about this: You keep getting new items from the server. And all of a sudden your table holds thousands, tens of thousands, of items. Well, your database grows. It is the responsibility of your application to possibly purge old data.

  * Why is that a good idea? 
    * Not only because the user will be mad at you because you made their phone useless, but because the cursor has the ability to hold only about one megabyte of data. Beyond one megabyte, it will have to do windowing, which is a very slow operation. Don't hold a lot of data in these cursors.
    * For example, images. If you use images in your cursor, which absolutely you could, what you're doing is, you're taking away from your ability to store more items in a cursor.

* **Minimize network usage.**

  * Only request data that's newer than the one you have. Older than the one you have, but don't go back to the server and get that one again.
  * Page data. If you don't page data, you are retrieving this huge amount of data from the server. It takes time. You parse it, takes a lot of time to parse it. You insert it in a database. That application will never perform fast enough. Always use paging.

* **use a sync adapter to synchronize the content of your local database with the state of your server.**

  * We're very happy to announce here at Google I/O that the Android cloud-to-device message is now available. 
  * You won't have to set up an alarm in order to continuously start the sync adapter and ask the server, are you done? Do you have new data? Do you have new data?
  * Use the push notification.
  * I would save a lot of battery life for a particular phone when using this technology.
