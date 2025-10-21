# Signals
- [Signals](#signals)
  - [Prerequisites](#prerequisites)
  - [Getting Started](#getting-started)
  - [Additional Setup](#additional-setup)
    - [Capture Interactions](#capture-interactions)
      - [Kotlin Compose](#kotlin-compose)
      - [Legacy XML UI](#legacy-xml-ui)
    - [Capture Navigation](#capture-navigation)
    - [Capture Network](#capture-network)
      - [OkHttp:](#okhttp)
      - [Retrofit:](#retrofit)
      - [java.net.HttpURLConnection:](#javanethttpurlconnection)
  - [Configuration Options](#configuration-options)
  - [Debug Mode](#debug-mode)
  - [Breaking Changes](#breaking-changes)
    - [0.8.0 -\> 0.9.0](#080---090)

## Prerequisites

Auto-Instrumentation (aka Signals) works on top of Analytics and Live Plugins. Make sure to add the following dependencies to your module's Gradle file if you don't have them already.

```groovy
// analytics kotlin 
implementation ("com.segment.analytics.kotlin:android:1.22.0")
// live plugin 
implementation("com.segment.analytics.kotlin:analytics-kotlin-live:1.3.0")
```

## Getting Started

Instrumentation becomes as simple as adding dependencies to your module's Gradle file:
1. Add Signals Core
    ```groovy
    // signal core   
    implementation ("com.segment.analytics.kotlin.signals:core:1.0.0")
    ```
2. Initialize Signals
    ```kotlin
    //... <analytics config>....
    analytics.add(LivePlugins()) // Make sure LivePlugins is added
    analytics.add(Signals)  // Add the signals plugin
  
    Signals.configuration = Configuration(
      // sendDebugSignalsToSegment will relay events to Segment server. Should only be true for development purposes.
      sendDebugSignalsToSegment = true
      // obfuscateDebugSignals will obfuscate sensitive data
      obfuscateDebugSignals = true
      // .. other options
    )
    ```
3. Add proper dependency and plugin as needed to: 
     * [Capture Interactions](#capture-interactions)
     * [Capture Navigation](#capture-navigation)
     * [Capture Network](#capture-network)

## Additional Setup

### Capture Interactions

#### Kotlin Compose

1. Add the dependency to your module’s Gradle build file.
    ```groovy
    implementation ("com.segment.analytics.kotlin.signals:compose:1.0.0")
    ```

2. Add `SignalsComposeTrackingPlugin` to analytics
    ```kotlin
    analytics.add(SignalsComposeTrackingPlugin())
    ```

#### Legacy XML UI

1. Add uitoolkit Gradle Plugin dependency to project level `build.gradle`
    ```groovy
    buildscript {
        dependencies {
            classpath 'com.segment.analytics.kotlin.signals:uitoolkit-gradle-plugin:1.0.0'
        }
    }
    ```
2. Apply the plugin in your app level `build.gradle` and add the dependency
    ```groovy
    plugins {
        // ...other plugins
        id 'com.segment.analytics.kotlin.signals.uitoolkit-tracking'
    }
    
    dependencies {
      // ..other dependencies
      implementation ("com.segment.analytics.kotlin.signals:uitoolkit:1.0.0")
    }
    ```


### Capture Navigation

1. Add navigation Gradle Plugin dependency to project level `build.gradle`
    ```groovy
    buildscript {
        dependencies {
            classpath 'com.segment.analytics.kotlin.signals:navigation-gradle-plugin:1.0.0'
        }
    }
    ```
2. Apply the plugin in your app level `build.gradle` and add the dependency
    ```groovy
    plugins {
        // ...other plugins
        id 'com.segment.analytics.kotlin.signals.navigation-tracking'
    }
    
    dependencies {
      // ..other dependencies
      implementation ("com.segment.analytics.kotlin.signals:navigation:1.0.0")
    }
    ```
3. (Optional) Add `SignalsActivityTrackingPlugin` to analytics to track Activity/Fragment navigation. **Not required for Compose Navigation**  
    ```kotlin
    analytics.add(SignalsActivityTrackingPlugin())
    ```

### Capture Network

#### OkHttp
  
1. add dependency:
    ```groovy
    implementation ("com.segment.analytics.kotlin.signals:okhttp3:1.0.0")
    ```

2. add `SignalsOkHttp3TrackingPlugin` as an interceptor to your OkHttpClient:
    ```kotlin
       private val okHttpClient = OkHttpClient.Builder()
           .addInterceptor(SignalsOkHttp3TrackingPlugin())
           .build()
    ```

#### Retrofit

1. add dependency:
    ```groovy
    implementation ("com.segment.analytics.kotlin.signals:okhttp3:1.0.0")
    ```

2. add `SignalsOkHttp3TrackingPlugin` as an interceptor to your Retrofit client:
    ```kotlin
       private val okHttpClient = OkHttpClient.Builder()
           .addInterceptor(SignalsOkHttp3TrackingPlugin())
           .build()
       
       val retrofit = Retrofit.Builder()
           .client(okHttpClient)
           .build()
    ```

#### java.net.HttpURLConnection
 1. add dependency:
     ```groovy
     implementation ("com.segment.analytics.kotlin.signals:java-net:1.0.0")
     ```
 
 2. install the `JavaNetTrackingPlugin` on where you initialize analytics:
     ```kotlin
         JavaNetTrackingPlugin.install()
     ```


## Configuration Options

Using the Signals Configuration object, you can control the destination, frequency, and types of signals that Segment automatically tracks within your application. The following table details the configuration options for Signals-Kotlin.

| OPTION            | REQUIRED | VALUE                     | DESCRIPTION |
|------------------|----------|---------------------------|-------------|
| **maximumBufferSize** | No  | Integer                   | The number of signals to be kept for JavaScript inspection. This buffer is first-in, first-out. Default is **1000**. |
| **relayCount** | No  | Integer                   | Relays every X signals to Segment. Default is **20**. |
| **relayInterval** | No  | Integer                   | Relays signals to Segment every X seconds. Default is **60**. |
| **broadcasters**  | No      | List<SignalBroadcaster>    | An array of broadcasters. These objects forward signal data to their destinations, like **WebhookBroadcaster**, or you could write your own **DebugBroadcaster** that writes logs to the developer console. **SegmentBroadcaster** is always added by the SDK. |
| **sendDebugSignalsToSegment**      | No      | Boolean                    | Turns on debug mode and allows the SDK to relay Signals to Segment server. Default is **false**. It should only be set to true for development purposes. |
| **obfuscateDebugSignals**      | No      | Boolean                    | Obfuscates signals being relayed to Segment. Default is **true**. |

## Debug Mode


The SDK automatically captures various types of signals, such as interactions, navigation, and network activity. However, relaying all these signals to a destination could consume a significant portion of the end user's bandwidth. Additionally, storing all user signal data on a remote server might violate privacy compliance regulations. Therefore, by default, the SDK disables this capability, ensuring that captured signals remain on the end user's device.  

However, being able to view these signals is crucial for creating event generation rules on the Segment Auto-Instrumentation dashboard. To facilitate this, the SDK provides a `sendDebugSignalsToSegment` configuration option that enables signal relaying to a destination and an `obfuscateDebugSignals` configuration option to obfuscate signals data.

> **⚠️ Warning:** `sendDebugSignalsToSegment` should only be used in a development setting to avoid storing sensitive end-user data.

Although `sendDebugSignalsToSegment` offers convenience for logging events remotely, having to redistribute the app each time it is toggled on or off can be cumbersome. Below are some suggested workarounds:

* Use Build Flavors to Configure `sendDebugSignalsToSegment` and `obfuscateDebugSignals`:
  
  1. Define different flavors in `build.gradle`
      ```groovy
      productFlavors {
        prod {
          buildConfigField "boolean", "SEND_DEBUG_SIGNALS_TO_SEGMENT", "false"
          buildConfigField "boolean", "OBFUSCATE_DEBUG_SIGNALS", "true"
        }
        dev {
          buildConfigField "boolean", "SEND_DEBUG_SIGNALS_TO_SEGMENT", "true"
          buildConfigField "boolean", "OBFUSCATE_DEBUG_SIGNALS", "false"
        }
      }
      ``` 
  2. Initialize Signals with the appropriate configuration:
      ```kotlin
      Signals.configuration = Configuration(
        // ... other config options
        sendDebugSignalsToSegment = BuildConfig.SEND_DEBUG_SIGNALS_TO_SEGMENT
        obfuscateDebugSignals = BuildConfig.OBFUSCATE_DEBUG_SIGNALS
      )
      ```
* Use a Feature Flag to Configure `sendDebugSignalsToSegment` and `obfuscateDebugSignals`. If you're using Firebase Remote Config or a similar feature flag system, you can dynamically control `sendDebugSignalsToSegment` and `obfuscateDebugSignals` without requiring a new app build:
  ```kotlin
      Signals.configuration = Configuration(
        // ... other config options
        sendDebugSignalsToSegment = remoteConfig.getBoolean("sendDebugSignalsToSegment")
        obfuscateDebugSignals = remoteConfig.getBoolean("obfuscateDebugSignals")
      )
  ```