# Introduction to Android Application security
This blogpost is something I've been wanting to mention for a while - an introduction to Android App security!  
In this first blogpost, I will mention how Android Apps work, their connection to the Linux ecosystem and what is unique in Android.

## Apps and the platform
Apps are bundles that contain code, resources and signatures, and are shipped in an archive that commonly has the `*.apk` extension.  
The majority of an App's code in written in Java (or other managed languages) and ultimately run in a Java Virtual Machine (JVM).  
In the past, the JVM was used to be known as `Dalvik` and ran `Dalvik Executable (DEX)` files, but these days the JVM is known as `Android Runtime (ART)`, which unlike Dalvik (which had `Just-In-Time (JIT)` compilation), has `Ahead-of-time (AOT)` compilation.

The capabilities of each App are mentioned in a special file called `AndroidManifest.xml`, which contains App metadata, components, and most importantly - permissions.  
App permissions are derived from that manifest - for example the ability to use the camera, access location services, make calls and even use the internet.

Android is built on top of Linux and thus - a lot of different Android App capabilities are implemented via the usual Linux permissions model.  
In particular:
- Each Android App gets a special ID called `App ID`, which is really a Linux user (the App ID is a Linux UID)!
- Filesystem isolation comes "for free" - each App data is saved under `/data/data/<package>` and will only be readable and writable to the App ID that owns it - in other words - `rw-------`.
- Certain permissions translate into Linux group memberships - for instance, `sdcard_rw` group is a Linux group that has permissions to read or write to external storage, and having that capability in `AndroidfManifest.xml` translates into the Linux user belonging to that group.

One more thing to note about Android Apps is that they are signed - Android Apps must be digitally signed with a certificate.  
Signatures are typically verified at App install time and upgrade time, but there are mechanism to check an App's signature at arbitrary times too.

## App components
As I mentoned, the `AndroidManifest.xml` contains metadata about the App. In addition to permissions, it contains a set of App components:

### Activities
An `Activity` is kind of a UI screen, and an entry point to user interactions. Here is an example of an Activity declaration:

```xml
<activity android:name="LoginActivity"
          android:exported="true"
          android:permission="com.example.MY_PERMISSION">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

Note the activity name `LoginActivity`, as well as:
1. The fact the activity is `exported`. Exported activities can be referred to from different Apps (more on that when we discuss IPC). Note Apps attacking other Apps in Android is a huge concern, and so, `exported` Activities are a significant threat in the Android ecosystem.
2. There is an `Intent filter`, which specifies how the Activity can be launched. We haven't explained what `Intents` are yet, but in essence - they are means of communications between different Apps. Intenrs have `actions` and `categories`, and the `intent filter` is enforced by matching those Intent members against the filter.

### Services
A `Service` is a component meant to run for a long time in the background without UI (e.g. playing music). Here is an example of a Service being declared:

```xml
<service android:name="SyncService"
         android:exported="false" />
```

As before, Services have names and might be exported, which has threat landscape implications.

### Brooadcast Receivers
Broadcast Receivers are components that listen for system-wide messages (known as `broadcast Intents`), for example: network state changes, incoming SMS, boot completion.  
Here is an example of a Broadcast Receiver being declared:

```
<receiver android:name="BootReceiver"
          android:enabled="true"
          android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

In this case, the Broadcast Receiver starts when it receives a `BOOT_COMPLETED` Intent - which triggers when the phone boots up.  
Interestingly, that could be used for persisting malware (and in fact, the main persistence mechanism in Android).

## IPC - the Binder
One critical aspect of Android is known as the `Binder`.
