+++
date = 2020-02-25T06:30:00Z
disable_comments = false
tags = ["privacy", "crashlytics", "firebase", "android"]
title = "Enable/Disabling Firebase Crashlytics on runtime in Android"
type = ""

+++
Recently I integrated Firebase's Crashlytics solution into one of our main Android products. It's fairly easy to integrate, just 4 steps as per Crashlytics's documentation [here](https://firebase.google.com/docs/crashlytics/get-started?platform=android). But as with any reporting system there involves the issue of privacy and data collection problem. Always important to inform the user if you are collecting any information and also request for their consent to do so. By default Crashlytics's behavior is to start reporting on every crash. So instead if you want to setup an opt-in process then all you have to do is

#### Step 1

In your AndroidManifest.xml, add this <meta-data> under <application>

```xml
<meta-data
    android:name="firebase_crashlytics_collection_enabled"
    android:value="false" />
```

#### Step 2

In your app's main Activity file, in the onCreate method add this

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
  
    // toggle Crashlytics on/off based on user consent
    SharedPreferences preferences = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());
    boolean enableCrashReporting = preferences.getBoolean("enable_crash_report", false);
    if(enableCrashReporting) {
       Fabric.with(this, new Crashlytics());
    }
}
```

### Points to remember

* If you use the the steps above crash reporting is disabled by default and its dependent on the default value of your shared preference.
* If you enable the crash reporting in your shared preference you will have to manually have to restart the Main activity again to start reporting the crash.
* If you disable the crash reporting from your shared preference you will have ensure that the app is completely closed and destroyed. Otherwise it will continue to report until the application is completely restarted.

##### References

* [https://stackoverflow.com/posts/49836972/revisions](https://stackoverflow.com/posts/49836972/revisions "https://stackoverflow.com/posts/49836972/revisions")
* [https://firebase.google.com/docs/crashlytics/customize-crash-reports?authuser=0&platform=android#enable_opt-in_reporting](https://firebase.google.com/docs/crashlytics/customize-crash-reports?authuser=0&platform=android#enable_opt-in_reporting "https://firebase.google.com/docs/crashlytics/customize-crash-reports?authuser=0&platform=android#enable_opt-in_reporting")