# AskAI Android SDK Integration Guide

The AskAI SDK provides AI-powered flight and hotel search optimization and filtering for Android applications.

## 1. Installation

### Toolchain (Rivi Android repo)

This SDK is built and published with **JDK 17**, **Gradle 8.13** (see `gradle/wrapper/gradle-wrapper.properties`), and **Android Gradle Plugin 8.13.x** (`agp` in `gradle/libs.versions.toml`). Consumer apps should use a compatible JDK and Gradle/Android Gradle Plugin combination; see [Gradle–Java compatibility](https://docs.gradle.org/8.13/userguide/compatibility.html) and [AGP release notes](https://developer.android.com/build/releases/gradle-plugin).

### Add repository

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

### Add dependency

```kotlin
dependencies {
    implementation("co.rivi:askai:1.0.8")
}
```

Use the version from the `askai/build.gradle.kts` `libraryVersion` value when pinning releases.

## 2. Quick Start

```kotlin
// 1. Create instance (Singleton/DI)
val askAI = AskAI.create(tenantToken = "token", baseUrl = "https://api.rivi.co")

// 2. Trigger AI Ranking
askAI.flights.sortBest(
    AskAIFlightSortBestRequest(
        searchId = "search-123", /* ... params ... */
    )
)

// 3. Subscribe to Results
val parser = JsonParser<MyResult> { gson.fromJson(it, MyResult::class.java) }

askAI.subscriber.subscribe("search-123", parser).collect { event ->
    when (event) {
        is SSEEvent.Data -> showResults(event.value)
        is SSEEvent.ParameterChange -> showNotice(event.event.message)
        is SSEEvent.StateChange -> updateLoader(event.state)
        is SSEEvent.Error -> if (!event.willRetry) showError(event.throwable)
    }
}

// 4. Cleanup
askAI.close()
```

## 3. Initialization

Initialize the SDK once, typically in your Dependency Injection module (e.g., Hilt/Koin) or Application class.

```kotlin
val askAI = AskAI.create(
    tenantToken = "your_tenant_token",
    // Optional: Defaults to Production (https://api.rivi.co)
    // Use Staging for testing: "https://api.staging.rivi.co"
    baseUrl = "https://api.rivi.co",
    logLevel = AskAILogLevel.INFO
)

// Access clients
askAI.subscriber  // For SSE subscription
askAI.flights     // For flight operations
askAI.hotels      // For hotel operations

// Clean up when done
askAI.close()
```

## 4. API Overview

The SDK exposes three main clients to handle specific domains:

* `askAI.flights`: Handles flight-specific operations like "Sort Best" (AI ranking) and "Filter" (Natural Language).

* `askAI.hotels`: Handles hotel-specific operations, similar to flights.

* `askAI.subscriber`: Manages the server-sent events (SSE) connection. This is the pipeline that delivers results asynchronously and handles reconnection logic.

## 5. Flight Operations

### Sort Best

Returns AI-optimized flight recommendations.

**Note on Parameters:**

* `departureDate`: Uses `java.time.LocalDateTime`.

* `searchId`: Must be a unique string (e.g., UUID) for each search session.

```kotlin
import java.time.LocalDateTime

try {
    val request = AskAIFlightSortBestRequest(
        searchId = "unique-search-id",
        origin = "DXB",
        destination = "DEL",
        departureDate = LocalDateTime.now().plusDays(1), // Accepts java.time.LocalDateTime
        adults = 1,
        children = 0,
        infant = 0,
        isRound = false,
        cabinType = "economy"
    )

    // Triggers the AI processing. Data is received via the subscriber (SSE).
    val result = askAI.flights.sortBest(request)

} catch (e: IllegalArgumentException) {
    // This is the ONLY exception thrown by the SDK during request validation.
    // It indicates invalid parameters (e.g., negative adults, check-out before check-in).
    Log.e("AskAI", "Invalid request: ${e.message}")
}
```

### Filter (Natural Language)

Filters results using natural language queries (e.g., "non-stop flights under $500").

```kotlin
val result = askAI.flights.filter(
    AskAIFlightFilterRequest(
        filterQuery = "non-stop flights under $500",
        searchId = "unique-search-id",
        // ... context params same as SortBest
        origin = "JFK",
        destination = "LAX",
        departureDate = LocalDateTime.of(2024, 6, 15, 10, 0),
        returnDate = LocalDateTime.of(2024, 6, 22, 18, 0),
        adults = 2,
        isRound = true,
        cabinType = "economy"
    )
)

// 1. Chips (UI Suggestions)
// NOTE: Chips are generated in ENGLISH.
// For other languages (e.g., Arabic), rely on the `entities` to build your own UI.
val chips: List<String> = result.chips

// 2. Entities (Logic/Data)
// Use these structured objects to filter your local list or send to your backend.
result.entities.forEach { entity ->
    // e.g. entity.flightBudget -> 500.0
    // e.g. entity.stopsPreference -> \"no_stops\"
}
```

## 6. Hotel Operations

### Sort Best

Similar to flights, requires valid `LocalDateTime` for check-in/out.

```kotlin
try {
    val request = AskAIHotelSortBestRequest(
        searchId = "unique-search-id",
        destination = "Singapore",
        checkIn = LocalDateTime.now().plusDays(1),
        checkOut = LocalDateTime.now().plusDays(4),
        rooms = 1,
        adults = 2
    )
    askAI.hotels.sortBest(request)
} catch (e: IllegalArgumentException) {
    // Handle validation errors
}
```

### Filter

```kotlin
val result = askAI.hotels.filter(
    AskAIHotelFilterRequest(
        filterQuery = "hotels near bugis with pool",
        searchId = "unique-search-id",
        destination = "Singapore",
        checkIn = LocalDateTime.of(2026, 2, 12, 0, 0),
        checkOut = LocalDateTime.of(2026, 2, 16, 0, 0),
        adults = 2
    )
)

result.chips      // [\"Near Bugis\", \"Pool\"]
result.entities   // Parsed: hotel names, amenities, star rating, etc.
```

## 7. SSE Subscription (Real-time Updates)

The `subscriber` is the core of the AskAI experience. It maintains a persistent connection to receive AI results.

> [!IMPORTANT]\
> **Critical: Coroutine Isolation Required**
>
> `askAI.subscriber.subscribe()` is a **long-running, blocking operation** that maintains an active connection for the entire duration of\
> the SSE stream. You **must** launch it in a **dedicated coroutine** without any other logic in the same scope.
>
> **Incorrect Usage:**
>
> ```kotlin
> // WRONG: Mixing subscription with other logic
> viewModelScope.launch {
>     val result = askAI.flights.sortBest(request)  // Executes first
>     askAI.subscriber.subscribe(searchId, parser).collect { ... }  // Blocks indefinitely
>     doSomethingElse()  // This line will NEVER execute
> }
> ```
>
> **Correct Usage:**
>
> ```kotlin
> // CORRECT: Separate coroutines for independent operations
> viewModelScope.launch {
>     askAI.flights.sortBest(request)  // Triggers AI processing
> }
>
> viewModelScope.launch {
>     askAI.subscriber.subscribe(searchId, parser).collect { event ->
>         // Handle streaming events
>     }
> }
> ```

### JSON Parsing

You must provide a `JsonParser` to convert the raw JSON payload into your domain object (or the SDK's DTOs).

```kotlin
//This is the JSON structure your backend sends. Use your own model:
data class FlightResult(
    val id: String,
    val price: Double,
    val airline: String,
    val rank: Int
)

val parser = JsonParser<MyFlightResult> { jsonString ->
    gson.fromJson(jsonString, MyFlightResult::class.java)
}
```

### Connection States

Map states directly to UI:

| State          | Action                                   |
| -------------- | ---------------------------------------- |
| `Connecting`   | Show loader                              |
| `Connected`    | Hide loader                              |
| `Reconnecting` | Show "Reconnecting..." banner (optional) |
| `Disconnected` | See reason below                         |

### Disconnected Reasons

| Reason               | Auto-Retry? | What to do                                                             |
| -------------------- | ----------- | ---------------------------------------------------------------------- |
| `ServerClosed`       | —           | ✅ Stream complete. Do nothing.                                         |
| `Cancelled`          | —           | ✅ You cancelled it. Do nothing.                                        |
| `NetworkError`       | ✅ Yes       | SDK retries automatically. You'll see `Reconnecting` state. Just wait. |
| `Timeout`            | ❌ No        | All retries failed. Show retry button.                                 |
| `MaxRetriesExceeded` | ❌ No        | All retries failed. Show retry button.                                 |
| `Unknown`            | ❌ No        | Unexpected error. Show error message.                                  |

### Simple ViewModel Example

```kotlin
private fun updateConnectionUI(state: SSEConnectionState) {
    when (state) {
        is SSEConnectionState.Connecting -> showLoading()
        is SSEConnectionState.Connected -> { hideLoading(); hideBanner() }
        is SSEConnectionState.Reconnecting -> showBanner(\"Reconnecting... (${state.attempt}/${state.maxAttempts})\")
        is SSEConnectionState.Disconnected -> {
            // Only show manual retry if it wasn't a normal closure
            if (state.reason !is DisconnectReason.ServerClosed) showRetryButton(state.reason)
        }
    }
}
```

### Error Handling Strategy

The `SSEEvent.Error` event is critical for deciding whether to intervene or let the SDK handle the issue.

* `willRetry = true`:

  * **Behavior**: The SDK encountered a transient error (e.g., Timeout, SocketException) and is scheduling a retry.

  * **Action**: **Do not** show a blocking error dialog. You may show a non-intrusive toast or indicator (e.g., "Retrying connection...").

* `willRetry = false`:

  * **Behavior**: A fatal error occurred (e.g., server returned 400 Bad Request, max retries exceeded) or the error is not recoverable.

  * **Action**: Show a manual "Retry" button or an error dialog explaining the issue.

* **Malformed Data**:

  * **Behavior**: If the JSON payload cannot be parsed by your `JsonParser`, the SDK catches the exception, logs it, and emits an `SSEEvent.Error` with `willRetry = false`.

  * **Details**: The `message` field of the error event will contain a **preview** of the malformed data for debugging.

  * **Stream Continuity**: Crucially, **the stream does not crash**. It continues listening for subsequent valid events.

### Custom Configuration Strategies

You can tune `SSEConfig` to match your app's needs.

#### 1. Aggressive Strategy (Real-Time Focus)

* **Theory**: When the user is actively waiting (e.g., Chat, Immediate Search), you want to fail fast and retry quickly to recover the connection before they lose patience.

* **Settings**: Short `connectTimeout` (e.g., 10s), short `initialReconnectDelay` (e.g., 500ms), and higher `maxReconnectAttempts`.

#### 2. Conservative Strategy (Battery/Background Focus)

* **Theory**: For non-critical updates or background syncing, prioritize battery life and reduce network congestion. If it fails, wait longer before retrying.

* **Settings**: Standard/Long `connectTimeout` (e.g., 30-60s), long `initialReconnectDelay` (e.g., 2-5s), and standard `reconnectBackoffMultiplier`.

```kotlin
// Example: Aggressive Config for active search screens
val aggressiveConfig = SSEConfig(
    connectTimeout = 10.seconds,
    initialReconnectDelay = 500.milliseconds,
    maxReconnectAttempts = 10
)

askAI.subscriber.subscribe(searchId, parser, aggressiveConfig).collect { /*...*/ }
```

## 8. Advanced Integration

### Concurrent Searches & Job Management

**Scenario**: A user triggers a search, then quickly triggers it again (e.g., retry or tab switch).

**Question**: Doesn't the SDK handle cancellation?\\
**Answer**: The SDK **deduplicates** connections. If you call `subscribe(\"id-1\")` twice, you get the _same_ shared stream (Multicast). It\
does _not_ cancel the previous stream.

**Why you need concise Job management**:\\
You must cancel your local **collection Job** to ensure your UI doesn't process events from a stale or redundant observer.

```kotlin
private val searchJobs = mutableMapOf<String, Job>()

fun startSearch(searchId: String) {
    // 1. Client-side Check: Cancel any existing listener for this ID
    // so we don't have two coroutines fighting to update the UI.
    searchJobs[searchId]?.cancel()

    // 2. Start new collection
    searchJobs[searchId] = viewModelScope.launch {
        // SDK returns the shared stream (existing or new)
        askAI.subscriber.subscribe(searchId, parser).collect { event ->
            // Update UI
        }
    }
}
```

### RxJava Integration

If your app uses RxJava, convert the Flow:

```kotlin
fun searchRx(searchId: String): Observable<SSEEvent<Result>> {
    return askAI.subscriber.subscribe(searchId, parser)
        .asObservable() // using kotlinx-coroutines-rx2
}
```

### LiveData Integration

```kotlin
fun searchLiveData(searchId: String): LiveData<Result> = liveData {
    askAI.subscriber.subscribe(searchId, parser).collect { event ->
        if (event is SSEEvent.Data) emit(event.value)
    }
}
```

## 9. UI Integration Logic

### Chips Logic

When `filter()` returns `chips`, these are ready-to-render string suggestions.

* **Display**: Show as a horizontal scrollable list.

* **Interaction**: When a user clicks the "X" on a chip, call your local remove logic. The SDK doesn't manage valid chip state; it only\
  suggests them based on the query.

### Parameter Change Notices (SDK Events)

When the AI implies a change to parameters (e.g., "Flights for **tomorrow**" when the original search was today), the response includes a\
`parameterChangeNotice`.

* **Action**: Display this string in a dialog or toast.

* **Parameters**: Check `result.entities` to see specific changes (e.g. new date or location) and update your search request object accordingly for the next call.

## 10. Resource Cleanup & Lifecycle Management

Proper resource management is critical to prevent memory leaks and redundant network connections.

| Scope         | Recommendation                                       |
| ------------- | ---------------------------------------------------- |
| **ViewModel** | Call `askAI.close()` in `onCleared()`. (Recommended) |
| **Singleton** | Only close on app termination or user logout.        |

### Implementation

```kotlin
@HiltViewModel
class SearchViewModel @Inject constructor(private val askAI: AskAI) : ViewModel() {
    override fun onCleared() {
        super.onCleared()
        askAI.close() // Releases SSE connection and resources
    }
}
```

**Key Safety Note:** Always cancel any active coroutine `Job`s collecting the SSE flow before closing the SDK to avoid processing events on a dead connection.

## 11. Testing

The SDK interfaces (`AskAI`, `AskAIFlights`, `AskAISubscriber`) are mockable. Use `MockK` to test your ViewModel logic without network calls.

```kotlin
@Test
fun `search connects to SSE`() = runTest {
    // 1. Mock the SDK
    val mockAskAI = mockk<AskAI>()
    val mockSubscriber = mockk<AskAISubscriber>()
    every { mockAskAI.subscriber } returns mockSubscriber

    // 2. Mock the Stream
    val events = flowOf(SSEEvent.StateChange(SSEConnectionState.Connected))
    every { mockSubscriber.subscribe(any(), any(), any()) } returns events

    // 3. Test your ViewModel
    viewModel = SearchViewModel(mockAskAI)
    viewModel.search(\"test-id\")

    // 4. Verify
    assertEquals(SSEConnectionState.Connected, viewModel.connectionState.value)
    verify { mockSubscriber.subscribe(\"test-id\", any(), any()) }
}
```

## 12. Key Points

* **SSE Data Model**: Pass YOUR model that matches your backend's JSON structure.

* **Auto-retry**: `NetworkError` triggers automatic retry → `Reconnecting` state.

* **Manual retry needed**: `MaxRetriesExceeded` or `Timeout` → show retry button.

* **Ignore**: `SSEEvent.Error` with `willRetry = true` (SDK handles it).

## 13. What to Ignore

| Thing                             | Why                                                        |
| --------------------------------- | ---------------------------------------------------------- |
| `SSEConfig` presets               | `DEFAULT` is fine. Only change if you have specific needs. |
| `DisconnectReason.NetworkError`   | SDK auto-retries.                                          |
| `SSEEvent.Error` (willRetry=true) | SDK is retrying. Don't show error UI.                      |

## 14. Quick Reference

### Request Models

| Model                        | Purpose                      |
| ---------------------------- | ---------------------------- |
| `AskAIFlightSortBestRequest` | Trigger AI flight ranking    |
| `AskAIFlightFilterRequest`   | Filter flights with NL query |
| `AskAIHotelSortBestRequest`  | Trigger AI hotel ranking     |
| `AskAIHotelFilterRequest`    | Filter hotels with NL query  |

### Request Fields (Common)

| Field                       | Required | Notes                                |
| --------------------------- | -------- | ------------------------------------ |
| `searchId`                  | ✅        | Unique ID for this search session    |
| `filterQuery`               | ✅        | Natural language query (filter only) |
| `origin` / `destination`    | ✅        | Airport codes or city names          |
| `departureDate` / `checkIn` | ✅        | `LocalDateTime`                      |
| `adults`                    | ✅        | Number of adults                     |

### Response Fields

| Field      | Use for            |
| ---------- | ------------------ |
| `status`   | Check if "success" |
| `message`  | AI summary text    |
| `chips`    | Filter tags in UI  |
| `entities` | Parsed preferences |

## 15. Custom Logging

```kotlin
// 1. Create a simple wrapper
class TimberAskAILogger : AskAILogger {
    override fun debug(message: String) = Timber.d(message)
    override fun info(message: String) = Timber.i(message)
    override fun warn(message: String) = Timber.w(message)
    override fun error(message: String) = Timber.e(message)
    override fun error(throwable: Throwable) = Timber.e(throwable)
}

// 2. Inject during initialization
val askAI = AskAI.create(
    tenantToken = "token",
    logger = TimberAskAILogger(), // Custom logger
    logLevel = AskAILogLevel.DEBUG // Set interaction verbosity
)
```

## 16. TL;DR

1. **Create** `AskAI.create(tenantToken)`

2. **Call** `sortBest()` to start AI ranking

3. **Subscribe** to SSE with your Gson parser

4. **Handle** `Data` (add to list), `StateChange` (loader), `Error` (if `!willRetry`, show error)

5. **Close** in `onCleared()`

That's it.
