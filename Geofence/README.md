## Geofence Plugin for Xamarin

Simple cross platform plugin to handle geofence events such as entering, leaving and staying in a geofence region.

### Setup
* Available on NuGet: http://www.nuget.org/packages/Xam.Plugin.Geofence
* Install into your PCL project and Client projects.

**Supports**
* Xamarin.iOS
* Xamarin.iOS (x64 Unified)
* Xamarin.Android

### TODO
* Include Windows Phone 8 (Silverlight), Windows Phone 8.1 RT, Windows Store 8.1 Support. <b>(Currenty working - Coming Soon!)</b>
* Region expiration time support
* Refactor error handling (Error codes)
* Implement an Android Location Service for location updates
* Geofence general settings configuration support
* Android support for more than 100 geofence regions

### API Usage

Call **CrossGeofence.Current** from any project or PCL to gain access to APIs. Must initialize plugin on each platform before use. Should only be used after initialization, if not will get <b>GeofenceNotInitializedException</b>.

**CrossGeofence.Initialize<'T'>**
This methods initializes geofence plugin. The generic <b>T</b> should be a class that implements IGeofenceListener. This will be the class were you would listen to all geofence events.

####iOS
 On the AppDelegate:
```
public override bool FinishedLaunching (UIApplication app, NSDictionary options)
{
    //Initialization...
    CrossGeofence.Initialize<CrossGeofenceListener> ();

    return base.FinishedLaunching (app, options);
}
```

####Android
 On the Android Application class, is better to use the OnCreate of the Android Application class so you can handle geofence events even when activities are closed or app not running by starting a sticky android service to keep listening to geofence events. <b>(See Helpers/GeofenceService.txt and GeofenceAppStarter.txt)</b>

Initialization on your MainActivity/Application class.
```
public override void OnCreate()
{
    base.OnCreate();

     //Initialization
                        
     CrossGeofence.Initialize<CrossGeofenceListener>();            
}
```

**IGeofenceListener implementation**

Must implement IGeofenceListener. This would be commonly implemented in the Core project if sharing code between Android or iOS. In the case you are using the plugin only for a specific platform this would be implemented in that platform.

```
public class  CrossGeofenceListener : IGeofenceListener
{
         public void OnMonitoringStarted(string region)
        {
            Debug.WriteLine(string.Format("{0} - Monitoring started in region: {1}", CrossGeofence.Tag, region));
        }

        public void OnMonitoringStopped()
        {
            Debug.WriteLine(string.Format("{0} - {1}", CrossGeofence.Tag, "Monitoring stopped for all regions"));
        }

        public void OnMonitoringStopped(string identifier)
        {
            Debug.WriteLine(string.Format("{0} - {1}: {2}", CrossGeofence.Tag, "Monitoring stopped in region", identifier));
        }

        public void OnError(string error)
        {
            Debug.WriteLine(string.Format("{0} - {1}: {2}", CrossGeofence.Tag, "Error", error));
        }


        public void OnRegionStateChanged(GeofenceResult result)
        {
            Debug.WriteLine(string.Format("{0} - {1}", CrossGeofence.Tag, result.ToString()));
        }
```


Enum of Geofence Transaction Types:

```
    /// <summary>
    /// GeofenceTransition enum
    /// </summary>
    public enum GeofenceTransition
    {
        /// <summary>
        /// Entry transition
        /// </summary>
        Entered,
        /// <summary>
        /// Exit transition
        /// </summary>
        Exited,
        /// <summary>
        /// Stayed in transition
        /// </summary>
        Stayed,
        /// <summary>
        /// Unknown transition
        /// </summary>
        Unknown
    }
```
####Geofence Circular Region Class  - Properties

Class to define and configure geofence region

```
 /// <summary>
 /// Region identifier
 /// </summary>
 public string Id { get; set; }
 /// <summary>
 /// Region center Latitude
 /// </summary>
 public double Latitude  { get; set; }
 /// <summary>
 /// Region center Longitude
 /// </summary>
 public double Longitude { get; set; }
 /// <summary>
 /// Radius covered by the region in meters
 /// </summary>
 public double Radius { get; set; }
 /// <summary>
 /// Notify when enters region
 /// </summary>
 public bool NotifyOnEntry { get; set; }
 /// <summary>
 /// Notify when stays in region based on the time span specified in StayedInThresholdDuration
 /// Note: Stayed in transition will not be fired if device has exit region before the StayedInThresholdDuration
 /// </summary>
 public bool NotifyOnStay { get; set; }
 /// <summary>
 /// Notify when exits region
 /// </summary>
 public bool NotifyOnExit { get; set; }
 /// <summary>
 /// Notitication message when enters region
 /// </summary>
 public string NotificationEntryMessage { get; set; }
 /// <summary>
 /// Notification message when exits region
 /// </summary>
 public string NotificationExitMessage { get; set; }
 /// <summary>
 /// Notification message when stays in region
 /// </summary>
 public string NotificationStayMessage { get; set; }
 /// <summary>
 /// Persist region so that is available after application closes
 /// </summary>
 public bool Persistent { get; set; }
 /// <summary>
 /// Enables showing local notifications 
 /// Messages could be configured using properties: NotificationEntryMessage, NotificationExitMessage, NotificationStayMessage
 /// </summary>
 public bool ShowNotification { get; set; }
 /// <summary>
 /// Sets minimum duration time span before passing to stayed in transition after an entry 
 /// </summary>
 public TimeSpan StayedInThresholdDuration;
```

####Geofence Result Class - Properties

When there is a geofence event update you will get an instance of this class.

```
 /// <summary>
 /// Last time entered the geofence region
 /// </summary>
 public DateTime? LastEnterTime { get; set; }
 /// <summary>
 /// Last time exited the geofence region
 /// </summary>
 public DateTime? LastExitTime { get; set; }
 /// <summary>
 /// Result transition type
 /// </summary>
 public GeofenceTransition Transition { get; set; }
 /// <summary>
 /// Region identifier
 /// </summary>
 public string RegionId { get; set; }
 /// <summary>
 /// Duration span between last exited and entred time
 /// </summary>
 public TimeSpan? Duration { get { return LastExitTime - LastEnterTime; } }
 /// <summary>
 /// Time span between the last entry and current time.
 /// </summary>
 public TimeSpan? SinceLastEntry { get { return DateTime.Now - LastEnterTime; } }
 /// <summary>
 /// Result latitude
 /// </summary>
 public double Latitude { get; set; }
 /// <summary>
 /// Result longitude
 /// </summary>
 public double Longitude { get; set; }
 /// <summary>
 /// Result accuracy
 /// </summary>
 public double Accuracy { get; set; }
```

####**CrossGeofence.Current**
Methods and properties

**StartMonitoring**

Start monitoring in specified region 

```
void StartMonitoring(GeofenceCircularRegion region);
```

Starts monitoring multiple regions

```
void StartMonitoring(IList<GeofenceCircularRegion> regions);
```

**StopMonitoring**

Stop monitoring specified geofence region. 

```
void StopMonitoring(GeofenceCircularRegion region);
```

Stop monitoring multiple regions. 

```
void StopMonitoring(IList<GeofenceCircularRegion> regions);
```

**StopMonitoringAllRegions**

Stop monitoring all geofence regions. 

```
void StopMonitoringAllRegions();
```

**LastKnownLocation**

Last known geofence location. This location will be null if isn't monitoring any regions yet.

```
GeofenceLocation LastKnownLocation { get; }
```

**IsMonitoring**

Indicator that is true if at least one region is been monitored

```
bool IsMonitoring { get; }
```

**Regions**

Dictionary that contains all regions been monitored

```
IReadOnlyDictionary<string, GeofenceCircularRegion> Regions { get; }
```

**GeofenceResults**

Dicitonary that contains all geofence results received

```
IReadOnlyDictionary<string, GeofenceResult> GeofenceResults { get; }
```

#### Notes

##### CrossGeofence Features

This are special features you can enable or change values

```
    //Set the Priority for the Geofence Tracking Location Accuracy
    public static GeofencePriority GeofencePriority { get; set; }

    //Set the smallest displacement should be done from current location before a location update
    public static float SmallestDisplacement { get; set; }

```
##### Android Specifics

There are a few things you can configure in Android project using the following properties from CrossGeofence class:
```

    //The sets the resource id for the icon will be used for the notification
    public static int IconResource { get; set; }

    //The sets the sound  uri will be used for the notification
    public static Android.Net.Uri SoundUri { get; set; }

    //Sets update interval for the location updates
    public static int LocationUpdatesInterval { get; set; }

    //Sets fastest interval for the location updates
    public static int FastestLocationUpdatesInterval { get; set; }


```
* Requires the following permissions:
   * <b>android.permission.ACCESS_FINE_LOCATION</b>
   * <b>android.permission.ACCESS_COARSE_LOCATION</b>
   * <b>com.google.android.providers.gsf.permission.READ_GSERVICES* permissions</b>
   *  <b>android.permission.RECEIVE_BOOT_COMPLETED</b>. This permission allows the plugin to restore any geofence region previously monitored marked as persistent when rebooting.

* The <b>package name</b> of your Android aplication must <b>start with lower case</b> or you will get the build error: <code>Installation error: INSTALL_PARSE_FAILED_MANIFEST_MALFORMED</code> 
* Make sure you have updated your Android SDK Manager libraries:

![image](https://cloud.githubusercontent.com/assets/2547751/6440604/1b0afb64-c0b5-11e4-93b8-c496e2bfa588.png)


##### iOS Specifics

* You need to do is to add the following key to your Info.plist file:

    <b>NSLocationAlwaysUsageDescription</b>

    You can enter a string like “Location is required to find out where you are” which, as in iOS 7, can be localized in the Info.plist file.

##### Important Notes

* The region monitoring feature of iOS only provides the possibility to monitor up to 20 regions at a time. Trying to monitor more than 20 regions will result in an error, indicating that the limit is crossed. See more at: [Reference](http://www.rapidvaluesolutions.com/tech_blog/geofencing-using-core-location-for-regional-monitoring-in-ios-applications/#sthash.AYIWNg19.dpuf). Actually this plugin handle this by only monitoring the 20 nearest regions when there is more than 20 regions been monitored by using significant location updates.
* The region monitoring feature of Android has a limit of 100 regions per device user. Currently the plugin is not handling more than 100 regions on Android.

#### Contributors
* [rdelrosario](https://github.com/rdelrosario)
* [aflorenzan](https://github.com/aflorenzan)
* [frankelydiaz](https://github.com/frankelydiaz)

This wouldn't be possible without all these great developer posts, articles and references:

#####iOS References:
* http://www.rapidvaluesolutions.com/tech_blog/geofencing-using-core-location-for-regional-monitoring-in-ios-applications/
* http://hayageek.com/ios-geofencing-api/
* http://developer.xamarin.com/recipes/ios/multitasking/geofencing/
* http://nevan.net/2014/09/core-location-manager-changes-in-ios-8/
* http://www.rapidvaluesolutions.com/tech_blog/geofencing-using-core-location-for-regional-monitoring-in-ios-applications/

#####Android References:

* http://paulusworld.com/technical/android-geofences-update
* http://sysmagazine.com/posts/210162/
* http://www.zionsoft.net/2014/11/google-play-services-locations-2/
* https://github.com/xamarin/monodroid-samples/blob/master/wear/Geofencing/Geofencing/GeofenceTransitionsIntentService.cs
* http://stackoverflow.com/questions/19505614/android-geofence-eventually-stop-getting-transition-intents/19521823#19521823
* https://github.com/googlesamples/android-play-location/blob/master/Geofencing/app/src/main/java/com/google/android/gms/location/sample/geofencing/MainActivity.java
* http://aboutyusata.blogspot.com/2013/08/getting-gps-reading-in-background-in.html?m=1
* http://developer.android.com/guide/components/bound-services.html
* https://software.intel.com/en-us/android/articles/implementing-map-and-geofence-features-in-android-business-apps
* https://github.com/CesarValiente/GeofencesDemo
* https://github.com/googlesamples/android-play-location
* http://www.toptal.com/android/android-developers-guide-to-google-location-services-api/#remote-developer-jo
* http://stackoverflow.com/questions/19434999/android-geofence-only-works-with-opened-app

#####General References:
* http://whatis.techtarget.com/definition/geofencing
* http://welltechnically.com/?p=4691
* http://jwegan.com/growth-hacking/effective-geofencing/
* https://github.com/cowbell/cordova-plugin-geofence
* https://github.com/cowbell/cordova-plugin-geofence-wp8.1/tree/master/GeofenceComponent

Thanks!

#### License
Licensed under main repo license