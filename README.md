# Signals
- [Signals](#signals)
  - [Getting Started](#getting-started)
  - [Configuration Options](#configuration-options)
  - [Debug Mode](#debug-mode)
  - [Breaking Changes](#breaking-changes)
    - [0.8.0 -\> 0.9.0](#080---090)

## Getting Started

1. Add the dependency to your module’s Gradle build file.
```groovy
// analytics kotlin 
implementation ("com.segment.analytics.kotlin:android:1.21.0")
// live plugin 
implementation("com.segment.analytics.kotlin:analytics-kotlin-live:1.3.0")
// signal core   
implementation ("com.segment.analytics.kotlin.signals:core:0.9.0")
// signal compose plugin if need to track compose     
implementation ("com.segment.analytics.kotlin.signals:compose:0.9.0")
// signal okttp3 plugin if need to track okhttp3 network activity  
implementation ("com.segment.analytics.kotlin.signals:okhttp3:0.9.0")
// signal navigation plugin if need to track screen/route  
implementation ("com.segment.analytics.kotlin.signals:navigation:0.9.0")
```

2. Setup the initialization code
```kotlin
 //... <analytics config>....
analytics.add(LivePlugins())
analytics.add(Signals)  // Add the signals plugin

Signals.configuration = Configuration(
  // size of the signals buffer queue, also serve as the counter to relay cached signals to broadcasters
  maximumBufferSize = 1000,
  // the counter to relay cached signals to broadcasters
  relayCount = 10,
  // the interval (in seconds) to relay cached signals to broadcasters
  relayInterval = 60,
  // a list of broadcasters to relay signals. SegmentBroadcaster is always added by the SDK
  broadcasters = listOf(WebhookBroadcaster("YOUR WEBHOOK")),
  // sendDebugSignalsToSegment will relay events to Segment server. should only be true for development purpose.
  sendDebugSignalsToSegment = true
)

// Add the compose plugin for UI events and screen tracking
// or add `SignalsUIToolkitTrackingPlugin` for legacy UI toolkit
analytics.add(SignalsComposeTrackingPlugin())

// Add the screen plugin to track Activity based screens
analytics.add(SignalsActivityTrackingPlugin())
// If use Navigation with Compose, turn on screen tracking on NavController
navController.turnOnScreenTracking()
```

3. Setup network tracking accordingly.

* For OkHttp:
  
  add dependency:
  ```groovy
  implementation ("com.segment.analytics.kotlin.signals:okhttp3:0.9.0")
  ```

  install plugin:
  ```kotlin
      private val okHttpClient = OkHttpClient.Builder()
          .addInterceptor(SignalsOkHttp3TrackingPlugin())
          .build()
  ```

* For Retrofit:

  add dependency:
  ```groovy
  implementation ("com.segment.analytics.kotlin.signals:okhttp3:0.9.0")
  ```
  
  install plugin:
  ```kotlin
      private val okHttpClient = OkHttpClient.Builder()
          .addInterceptor(SignalsOkHttp3TrackingPlugin())
          .build()
      
      val retrofit = Retrofit.Builder()
          .client(okHttpClient)
          .build()
  ```

* For java.net.HttpURLConnection:
    add dependency:
    ```groovy
    implementation ("com.segment.analytics.kotlin.signals:java-net:0.9.0")
    ```
    
    install plugin:
    ```kotlin
        JavaNetTrackingPlugin.install()
    ```

4. Build and run your app

## Configuration Options

Using the Signals Configuration object, you can control the destination, frequency, and types of signals that Segment automatically tracks within your application. The following table details the configuration options for Signals-Kotlin.

| OPTION            | REQUIRED | VALUE                     | DESCRIPTION |
|------------------|----------|---------------------------|-------------|
| **maximumBufferSize** | No  | Integer                   | The number of signals to be kept for JavaScript inspection. This buffer is first-in, first-out. Default is **1000**. |
| **relayCount** | No  | Integer                   | Relays every X signals to Segment. Default is **20**. |
| **relayInterval** | No  | Integer                   | Relays signals to Segment every X seconds. Default is **60**. |
| **broadcasters**  | No      | List<SignalBroadcaster>    | An array of broadcasters. These objects forward signal data to their destinations, like **WebhookBroadcaster**, or you could write your own **DebugBroadcaster** that writes logs to the developer console. **SegmentBroadcaster** is always added by the SDK. |
| **sendDebugSignalsToSegment**      | No      | Boolean                    | Turns on debug mode and allows the SDK relaying Signals to Segment server. Default is **false**. It should only be set to true for development purpose. |
| **obfuscateDebugSignals**      | No      | Boolean                    | Obfuscates signals being relayed to Segment. Default is **true**. |

## Debug Mode


The SDK automatically captures various types of signals, such as interactions, navigation, and network activity. However, relaying all these signals to a destination could consume a significant portion of the end user's bandwidth. Additionally, storing all user signal data on a remote server might violate privacy compliance regulations. Therefore, by default, the SDK disables this capability, ensuring that captured signals remain on the end user's device.  

However, being able to view these signals is crucial for creating event generation rules on the Segment Auto-Instrumentation dashboard. To facilitate this, the SDK provides a `sendDebugSignalsToSegment` configuration option that enables signal relaying to a destination.  

> **⚠️ Warning:** `sendDebugSignalsToSegment` should only be used in a development setting to avoid storing sensitive end-user data.

Although `sendDebugSignalsToSegment` offers convenience for logging events remotely, having to redistribute the app each time it is toggled on or off can be cumbersome. Below are some suggested workarounds:

* Use Build Flavors to Configure `sendDebugSignalsToSegment`:
  
  1. Define different flavors in `build.gradle`
      ```groovy
      productFlavors {
        prod {
          buildConfigField "boolean", "SEND_DEBUG_SIGNALS_TO_SEGMENT", "false"
        }
        dev {
          buildConfigField "boolean", "SEND_DEBUG_SIGNALS_TO_SEGMENT", "true"
        }
      }
      ``` 
  2. Initialize Signals with the appropriate configuration:
      ```kotlin
      Signals.configuration = Configuration(
        // ... other config options
        sendDebugSignalsToSegment = BuildConfig.SEND_DEBUG_SIGNALS_TO_SEGMENT
      )
      ```
* Use a Feature Flag to Configure `sendDebugSignalsToSegment`. If you're using Firebase Remote Config or a similar feature flag system, you can dynamically control `sendDebugSignalsToSegment` without requiring a new app build:
  ```kotlin
      Signals.configuration = Configuration(
        // ... other config options
        sendDebugSignalsToSegment = remoteConfig.getBoolean("sendDebugSignalsToSegment")
      )
  ```

## Fragment Tracking

Fragment tracking is now supported with some limitations:

- Fragment tracking is enabled by default via `SignalsActivityTrackingPlugin`.
- Transitions between **activities and fragments** are supported.
- **VERY LIMITED** support for `ViewPager` / `ViewPager2`:  
  Due to caching behavior, the data reported by `FragmentManager.FragmentLifecycleCallbacks` can be unreliable. This may result in inaccurate tracking—such as repeated navigation events for the same screen followed by an unexpected jump to a new screen.
- It is **strongly recommended** to migrate to **Jetpack Compose**, which provides significantly more reliable tracking.


## Breaking Changes

### 0.8.0 -> 0.9.0

* Configuration changes:
  * renamed:
    * `broadcastInterval` -> `relayInterval`
    * `debugMode` -> `sendDebugSignalsToSegment`
  * added:
    * `relayCount` to control when to relay based on signals count
    * `obfuscateDebugSignals` to enable/disable obfuscation on signals