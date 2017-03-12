# TODO

- (ok) Live on command line (except initial scaffolding)
  - build application (gradlew)
    - know gradle ecosystem more (e.g. android plugin)
    - gradle installation itself
  - create avd (avdmanager)
  - launch emulator (emulator)
  - install application (adb install)
  - launch application (adb shell, am)

- (ok) Testing framework
  - https://developer.android.com/studio/test/index.html
  - unit test: non-android java unit test
  - instrumented test: some test on offscreen android app runtime (or maybe onscreen?)

- (later) Debugging framework
  - Android monitor (https://developer.android.com/studio/profile/am-basics.html)
  - Android device monitor (https://developer.android.com/studio/profile/monitor.html)
  - looks like we should use android studio: https://developer.android.com/studio/debug/index.html
    - Find internal of java source level debugger
    -

- (ok) Basic android gui layer
  - xml file
  - layout part and paint (called style?) part
  - navigation state management
    - how is it different from web platform (e.g. url)
    - is it context, isn't it ?

- Include third party library

- Know emulator more
  - architecture
  - qemu
  - gui frontend (qt)

- Find main function
  - see the callstack with step execution debugging


# Android tasks

- follow tasks from `./gradlew -p ./app tasks`
  - androidDependencies: show java package dependencies for each build types
  - sourceSets: list relevant source code for each build types
  - compile: run compiler or whatever necessary process to source code
  - assemble: compile + packaging
  - build: assemble + test
  - installDebug: install package
  - installDebugAndroidTest: install instrumented test version of package
  - test: run unit test
  - connectedAndroidTest: installDebugAndroidTest + run instrumented test


# Example

```
## build and run normal application ##

$ ./gradlew -p ./app assembleDebug
...

$ ls -l app/build/outputs/apk
total 1412
-rw-rw-r-- 1 hiogawa hiogawa 1445477 Mar 12 11:08 app-debug.apk

$ avdmanager create avd --name x --tag google_apis --package 'system-images;android-24;google_apis;x86_64'
Auto-selecting single ABI x86_64
Do you wish to create a custom hardware profile? [no] no

$ avdmanager list avd
Available Android Virtual Devices:
...
    Name: x
    Path: /home/hiogawa/.android/avd/x.avd
  Target: Google APIs (Google Inc.)
          Based on: Android 7.0 (Nougat) Tag/ABI: google_apis/x86_64

$ emulator -avd x
Creating filesystem with parameters:
...

(from different shell)
$ adb install app/build/outputs/apk/app-debug.apk
Success


## build and run instrumented test ##

$ ./gradlew -p ./app assembleAndroidTest
...

$ ls -l app/build/outputs/apk
total 2512
-rw-rw-r-- 1 hiogawa hiogawa 1124178 Mar 12 11:30 app-debug-androidTest.apk

$ adb root

$ adb shell

$# pm list packages
package:net.hiogawa.androidstarter
package:net.hiogawa.androidstarter.test

$# pm path net.hiogawa.androidstarter
package:/data/app/net.hiogawa.androidstarter-1/base.apk

$# ls -l data/app
ls -l data/app
total 16
drwxr-xr-x 4 system system 4096 2017-03-12 11:27 net.hiogawa.androidstarter-1
drwxr-xr-x 4 system system 4096 2017-03-12 11:34 net.hiogawa.androidstarter.test-1

$# am instrument -w net.hiogawa.androidstarter.test/android.support.test.runner.AndroidJUnitRunner

net.hiogawa.androidstarter.ExampleInstrumentedTest:.

Time: 0.008
OK (1 test)
```


# GUI architecture

- Navigation
  - https://developer.android.com/guide/components/activities/tasks-and-back-stack.html
  - Activity holds onscreen View and application's "session" is managed as a stack of activities.
  - Navigate to next by Activity.startActivity and to back by Activity.finishActivity
  - In the case of webkit view, this view itself could hold URL navigation as normal browser does.
    - https://developer.android.com/reference/android/webkit/WebView.html#goBack()
- Layout system
  - https://developer.android.com/guide/topics/ui/declaring-layout.html
  - Programatic alternative of layout file: View.measure and View.layout
    - like LayoutObject::layout in blink
  - layout file just defines document tree statically (possibly with some inlined layout and style). that's exactly like html file.
- Styling system
  - https://developer.android.com/guide/topics/ui/themes.html
  - like, you can separate css "paint" property in the separate file and reference it from layout file.
- (UI) Event
  - https://developer.android.com/guide/topics/ui/ui-events.html
- UI component
  - built-in: https://developer.android.com/guide/topics/ui/controls.html
  - non built-in: https://developer.android.com/guide/topics/ui/custom-components.html
    - intercept view lifecycle (like html's document lifecycle) by using
      View.onMeasure(Canvas) and View.onDraw(MeasureSpec)

- Main first citizens
  - Application
  - Activity (extends android.content.Context)
    - hold onscreen view by Activity.setContentView
  - View (any layoutable/stylable UI Component)
    - e.g.:
      - View (android.view)
      - TextView extends View (android.widget)
      - Button extends TextView (android.widget)
      - ViewGroup extends View (android.view)
      - LinearLayout extends ViewGroup (android.widget) (I hate this naming)
  - LayoutParams
  - android.R.attr (e.g. color, layout_width)