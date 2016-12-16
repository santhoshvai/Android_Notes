[![Beyond the Blue Dot: New Features in Android Location](http://i.imgur.com/J8XJFsG.png)](http://www.youtube.com/watch?v=Bte_GHuxUGc)

## Notes

![Complexity](http://i.imgur.com/DdhJEn0.jpg)

#### Pending intent

For contextual applications, they need to run in the background. The app process dont want to be bound to the location process.

**pending intent** is a token that your app process will give to the location process and the location process will use it to *wake up your app* when anything of interest happens.

![Priority options](http://i.imgur.com/348ofkk.jpg)

* **HIGH_ACCURACY**: Corresponds to GPS when outside and cell and WiFi when I am inside. Used for maps and navigation applications which want really good accuracy. Because they are running in the foreground, the battery drain is fine and the typical polling interval is around 5 seconds.

* **BALANCED_POWER**: Doesn't use GPS, it uses cell and WiFi. It is fine with lower accuracy and it doesn't want to drain the battery that much. So if you have a weather app this is when you use the balance power. The typical polling interval is around 20 seconds.

* **NO_POWER**: Useful when you just want to get a location when someother app has requested for a location. You don't want to actively poll, but if system knows the location, very well I can enhance my app. The best part, your app will not get any battery blame.

### Geofence

Add fences around a location to detect enter or exit. Geofence monitioring is done on the hardware and continous polling is not done to detect enter or exit. The hardware knows about the fence and only triggers when we enter that location. Monitoring User state is the biggest part to determine polling.

![Activity Recognition](http://i.imgur.com/pjuhvgM.jpg)

Machine learning based classifiers are used to determine user activity.

```ActivityRecognitionApi``` can detect if walking, running, bicycling etc in one api with a polling interval given.
