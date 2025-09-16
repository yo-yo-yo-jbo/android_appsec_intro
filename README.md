# Introduction to Android Application security
This blogpost is something I've been wanting to mention for a while - an introduction to Android App security!  
In this first blogpost, I will mention how Android Apps work, their connection to the Linux ecosystem and what is unique in Android.

## Apps and security
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
