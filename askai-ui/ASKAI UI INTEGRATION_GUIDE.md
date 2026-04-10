# AskAI UI Components Integration Guide


The AskAI UI Components library provides ready-to-use Compose screens and UI building blocks for the Rivi Ask AI experience on Android.


It is intended for apps that want to integrate:


- an Ask AI dashboard experience
- flight search UI powered by Ask AI
- hotel search UI powered by Ask AI
- Ask AI result rendering and supporting UI states


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


### Add Dependencies


Add the UI library:


```kotlin
dependencies {
   implementation("co.rivi:askai-ui-components:1.0.8") // Use latest available version
}
```


The library depends on other Rivi libraries such as `askai` and `rivi-ui-core`. If your Maven setup resolves transitive dependencies correctly, you should not need to add them manually.


If your environment requires explicit declarations, add:


```kotlin
dependencies {
   implementation("co.rivi:askai-ui-components:1.0.8")
   implementation("co.rivi:rivi-ui-core:1.0.8")
   implementation("co.rivi:askai:1.0.8") // Use latest compatible version
}
```


## 2. Host App Requirements


Your app should provide:


- Jetpack Compose enabled
- Hilt enabled
- access to the Rivi backend/authentication setup expected by the underlying Rivi libraries
- a logged-in/authenticated app flow before running AI-backed user flows


### Typical Gradle Plugins


```kotlin
plugins {
   id("com.android.application")
   kotlin("android")
   id("com.google.dagger.hilt.android")
   id("com.google.devtools.ksp")
}
```


### Application Setup


```kotlin
@HiltAndroidApp
class MyApp : Application()
```


### Activity Setup


```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity()
```


## 3. Quick Start


The main entry points are:


- `DashboardScreen`
- `FlightSearchInputScreen`
- `HotelSearchInputScreen`


### Dashboard Example


```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContent {
           val dashboardViewModel: DashboardViewModel = hiltViewModel()


           DashboardScreen(
               viewModel = dashboardViewModel,
               onNewChat = { message, attachmentUri ->
                   // Open your chat screen here
               },
               onSearchClick = {
                   // Navigate to your search experience
               },
               onSeeAllChatClick = {
                   // Open chat history
               },
               onChatClick = { chatId ->
                   // Open an existing chat
               },
               onLogout = {
                   // Handle logout
               }
           )
       }
   }
}
```


### Flight Search Example


```kotlin
@Composable
fun FlightRoute() {
   FlightSearchInputScreen(
       onBackClick = { /* navigate back */ }
   )
}
```


### Hotel Search Example


```kotlin
@Composable
fun HotelRoute() {
   HotelSearchInputScreen(
       onBackClick = { /* navigate back */ }
   )
}
```


## 4. Available Screens


### `DashboardScreen`


Top-level Ask AI entry screen with:


- AI tab
- Flight tab
- Hotel tab
- prompt input
- recent chat drawer
- suggestion UI


Current API:


```kotlin
@Composable
fun DashboardScreen(
   onNewChat: (String, String?) -> Unit,
   onSearchClick: () -> Unit,
   onSeeAllChatClick: () -> Unit,
   onChatClick: (chatId: String) -> Unit,
   onLogout: () -> Unit,
   onTalkToAiClick: () -> Unit = {},
   onProfileClick: () -> Unit = {},
   onMediaPicked: (((Uri) -> Unit) -> Unit) = {},
   isVoiceEnabled: Boolean = true,
   isAttachmentEnabled: Boolean = true,
   viewModel: DashboardViewModel
)
```


You provide navigation/actions. The library provides the UI and state wiring.


### `FlightSearchInputScreen`


Full flight experience with:


- search form
- autocomplete
- passenger and cabin controls
- Ask AI result states
- sorting/filtering UI
- result cards


Current API:


```kotlin
@Composable
fun FlightSearchInputScreen(
   onBackClick: () -> Unit,
   viewModel: FlightSearchInputViewModel = hiltViewModel(),
   askAIViewModel: AskAIViewModel = hiltViewModel()
)
```


### `HotelSearchInputScreen`


Full hotel experience with:


- destination input
- date range selection
- guest/room selection
- Ask AI hotel result states
- sorting/filtering UI
- result cards


Current API:


```kotlin
@Composable
fun HotelSearchInputScreen(
   onBackClick: () -> Unit,
   viewModel: HotelSearchInputViewModel = hiltViewModel(),
   askAIViewModel: AskAIViewModel = hiltViewModel()
)
```


## 5. Theme and UI Requirements


The library renders inside its own Ask AI theme layer, which is built on top of `rivi-ui-core`.


In practice:


- `DashboardScreen` wraps itself with `AskAIMaterialTheme`
- `FlightSearchInputScreen` wraps itself with `AskAIMaterialTheme`
- `HotelSearchInputScreen` wraps itself with `AskAIMaterialTheme`


You do not need to wrap these screens manually just to make them work.


## 6. End-to-End Integration Pattern


### Dashboard Flow


Typical host-app flow:


1. Render `DashboardScreen`.
2. Let the user enter a prompt or switch tabs.
3. Handle `onNewChat(message, attachmentUri)` by navigating to your chat experience.
4. Handle `onChatClick(chatId)` by reopening an existing chat.
5. Handle profile, voice, and attachment actions in the host app.


### Flight Flow


Typical host-app flow:


1. Render `FlightSearchInputScreen`.
2. User fills search criteria.
3. The screen ViewModel manages form state.
4. `AskAIViewModel` handles Ask AI requests and result state.
5. The library renders loading, errors, notices, result cards, and sorting/filter UI.


### Hotel Flow


Typical host-app flow:


1. Render `HotelSearchInputScreen`.
2. User selects destination, dates, and guests.
3. The screen ViewModel manages input state.
4. `AskAIViewModel` drives hotel AI result state.
5. The library renders result cards and related UI.


## 7. Architecture Overview


This library includes:


- screen composables
- Ask AI-specific UI components
- ViewModels for dashboard and search flows
- integration logic between UI state and the underlying Rivi/AskAI layers


Main public concepts:


- `DashboardViewModel`
- `FlightSearchInputViewModel`
- `HotelSearchInputViewModel`
- `AskAIViewModel`


Main screen packages:


```text
co.rivi.askai.ui
- component
- screen
- theme
- util
- viewmodel
```


## 8. Key Dependencies Used by the Library


The library internally uses:


- Jetpack Compose
- Material 3
- Hilt
- Navigation Compose
- Paging Compose
- Kotlinx datetime
- Kotlinx serialization
- Coil Compose
- Compose shimmer
- Rivi AskAI SDK
- Rivi UI Core


For most consumers, these are implementation details unless you are debugging dependency resolution.


## 9. Integration Notes


### Hilt


The default screen APIs use `hiltViewModel()`.


That means:


- your host activity/fragment/composable entry point should be under Hilt
- your app should have working Hilt setup


### Navigation


The library does not force a navigation framework on your app.


Instead, the host app supplies navigation behavior through callbacks such as:


- `onBackClick`
- `onNewChat`
- `onChatClick`
- `onSearchClick`


### Chat / Voice / Media Actions


The dashboard screen delegates external actions to the host app.


You should connect:


- chat opening
- chat-history navigation
- voice route opening
- media picker launch
- logout/profile flows


## 10. Error Handling Expectations


This library handles UI state for loading, empty, and result states, but host apps should still be prepared to handle:


- missing authentication/session state
- dependency injection misconfiguration
- backend/network issues surfaced through the underlying Rivi and AskAI layers


If you are integrating the full AI result experience, refer to the AskAI SDK guide as well:


- [askai/INTEGRATION_GUIDE.md](../askai/INTEGRATION_GUIDE.md)


That guide explains:


- SDK initialization
- SSE subscription model
- request/response flow
- lifecycle cleanup


## 11. Recommended Verification


After integrating in a third-party app, verify:


1. `DashboardScreen` renders correctly.
2. Flight search input works end to end.
3. Hotel search input works end to end.
4. Hilt-backed ViewModels resolve correctly.
5. The app can navigate out through all callback hooks.
6. Ask AI results, loading states, and error states render correctly.


## 12. Key Points


- This library is for external app consumption as a dependency.
- It already includes Ask AI-specific UI and screen orchestration.
- It does not require `rivi-ui-components`.
- It depends on `rivi-ui-core` and the AskAI stack under the hood.
- Your app provides host-level navigation, auth/session readiness, and external actions.


## 13. TL;DR


1. Add the GitHub Packages Maven repository.
2. Add `implementation("co.rivi:askai-ui-components:1.0.8")`.
3. Enable Compose and Hilt in your app.
4. Render `DashboardScreen`, `FlightSearchInputScreen`, or `HotelSearchInputScreen`.
5. Wire the provided callbacks into your app's navigation and media/chat flows.

