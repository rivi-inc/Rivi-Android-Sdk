# Rivi UI Core Integration Guide


Rivi UI Core is the shared Android Compose design-system library for Rivi UI surfaces.


It provides:


- `RiviTheme`
- shared colors, typography, and spacing
- reusable UI primitives such as `AsyncImage`, `Chip`, and `LikeButton`
- shared cards and calendar UI
- shimmer loaders
- utility extensions such as `Modifier.onClick`, `PaddingValues.scale`, and `SystemBars`


This guide is for third-party Android apps consuming the library as a published dependency.


## 1. Installation


### Add Repository

Packages are published to [GitHub Packages](https://github.com/orgs/rivi-inc/packages?repo_name=Rivi-Android-Sdk) for repository **rivi-inc/Rivi-Android-Sdk**:

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        maven {
            name = "GitHubPackagesRiviAndroid"
            url = uri("https://maven.pkg.github.com/rivi-inc/Rivi-Android-Sdk")
            credentials(PasswordCredentials::class) {
                username = providers.gradleProperty("github.packages.user").get()
                password = providers.gradleProperty("github.packages.token").get()
            }
        }
    }
}
```

In `~/.gradle/gradle.properties` set `github.packages.user` and `github.packages.token` (a GitHub PAT with `read:packages`; do not commit tokens). See [Working with the Gradle registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry).


### Add Dependency


```kotlin
dependencies {
   implementation("co.rivi:rivi-ui-core:1.0.8") // Use latest available version
}
```


## 2. Quick Start


Wrap your UI in `RiviTheme`:


```kotlin
setContent {
   RiviTheme {
       MyScreen()
   }
}
```


Simple example:


```kotlin
@Composable
fun MyScreen() {
   RiviTheme {
       Column(
           modifier = Modifier
               .fillMaxSize()
               .background(RiviTheme.colors.background)
               .padding(RiviTheme.spacing.medium)
       ) {
           Text(
               text = "Hello Rivi",
               style = RiviTheme.typography.titleMedium,
               color = RiviTheme.colors.onSurface
           )
       }
   }
}
```


## 3. Main APIs


### Theme


Core theme APIs:


- `RiviTheme`
- `RiviColors`
- `RiviTypography`
- `RiviSpacing`


These are the main design-system entry points used by Rivi UI components.


### Core Components


Common reusable components include:


- `AsyncImage`
- `Chip`
- `LikeButton`
- `FlightStopCircles`
- `RiviCalendar`
- `ShimmerFlightCard`
- `ShimmerHotelCard`


### Shared Cards


This library also contains reusable card UIs such as:


- `HotelCard`
- `OneWayFlightCard`
- `RoundMatchFlightCard`
- `TwoOneWayFlightCard`


## 4. End-to-End Integration Pattern


Typical third-party usage:


1. Add the Maven repository.
2. Add `co.rivi:rivi-ui-core` as a dependency.
3. Wrap your Compose UI with `RiviTheme`.
4. Use the shared components directly in your feature screens.


Example:


```kotlin
@Composable
fun ExampleScreen() {
   RiviTheme {
       SystemBars(
           statusBarColor = RiviTheme.colors.background,
           handleInsets = false
       )


       Chip(
           chip = co.rivi.core.model.data.Chip(text = "Recommended")
       )
   }
}
```


## 5. Theme Usage


`RiviTheme` exposes:


- `RiviTheme.colors`
- `RiviTheme.typography`
- `RiviTheme.spacing`


Example:


```kotlin
val background = RiviTheme.colors.background
val titleStyle = RiviTheme.typography.titleMedium
val spacing = RiviTheme.spacing.medium
```


This library is designed so higher-level UI libraries can build on top of these values consistently.


## 6. Useful Components


### `AsyncImage`


Use for shared image-loading behavior:


```kotlin
AsyncImage(
   imageUrl = imageUrl,
   modifier = Modifier.size(72.dp),
   fallbackModifier = Modifier.size(72.dp)
)
```


### `Chip`


Use for labels, tags, or compact pill UI:


```kotlin
Chip(
   chip = co.rivi.core.model.data.Chip(
       text = "Non-stop"
   )
)
```


### `RiviCalendar`


Use for single-date or range-date selection:


```kotlin
RiviCalendar(
   mode = CalendarMode.Range,
   selectedDate = null,
   selectedRange = CalendarDateRange(startDate, endDate),
   onDateSelected = { },
   onRangeSelected = { range -> /* update state */ },
   onDismiss = { },
   onApply = { }
)
```


### Shimmer Loaders


Use while data is loading:


```kotlin
ShimmerFlightCard()
ShimmerHotelCard()
```


## 7. Utility APIs


### `Modifier.onClick`


Shared press-scale click behavior:


```kotlin
Box(
   modifier = Modifier.onClick {
       onItemClick()
   }
)
```


### `PaddingValues.scale`


Useful for dynamically scaling component padding:


```kotlin
val scaledPadding = PaddingValues(horizontal = 8.dp, vertical = 4.dp).scale(0.9f)
```


### `SystemBars`


Use when you want screen-level control over status/navigation bar styling:


```kotlin
SystemBars(
   statusBarColor = RiviTheme.colors.background,
   handleInsets = false
)
```


## 8. Library Structure


Main package areas:


```text
co.rivi.ui.core
- component
- extension
- model
- theme
- util
```


This is useful mainly for discoverability when browsing the public API.


## 9. Host App Requirements


Your app should provide:


- Jetpack Compose enabled
- Android min/compile SDK compatible with the published artifact
- standard Android/Kotlin toolchain support for the library version you consume


Unlike `askai-ui-components`, this library does not require Hilt just to render its core UI APIs.


## 10. Recommended Verification


After integrating in a third-party app, verify:


1. `RiviTheme` applies correctly.
2. Typography, colors, and spacing render as expected.
3. Shared components such as `Chip` and `AsyncImage` render correctly.
4. Calendar and shimmer components behave correctly in your screen flows.
5. System bar styling works correctly on your target devices.


## 11. Key Points


- `rivi-ui-core` is the shared Compose design-system dependency.
- It is suitable for third-party apps consuming Rivi UI building blocks.
- It can be used directly in feature screens or indirectly through higher-level Rivi UI libraries.
- You typically start by wrapping your UI in `RiviTheme`.


## 12. TL;DR


1. Add the GitHub Packages Maven repository.
2. Add `implementation("co.rivi:rivi-ui-core:1.0.8")`.
3. Wrap your UI in `RiviTheme`.
4. Use the provided shared components and utilities in your Compose screens.

