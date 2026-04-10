# Ask AI UI Integration Guide

## 1. Overview

Ask AI fits into your existing flight or hotel search screen. It is a set of components you add to your existing results flow:

1.  **`SortFilterRow`**: 	Place this at the top of the screen. It has the "Ask AI" button and shows the active filters (chips).
2.  **`AskAiSheet`**: A bottom sheet where the user enters a natural language query (e.g., "cheap flights with meals").
3.  **Warnings and banners**: Messages shown when Ask AI changes dates or returns a partial match

---

## 2. Integration Notes

These two integration details matter most:
*   **Keep SortFilterRow visible**: Always place the `SortFilterRow` in a sticky header. Users need to see their active AI filters (chips) at all times so they understand *why* they are seeing specific results.
*   **Always show parameter changes**: Do not hide the warning banner. If the AI shifted a user's search date, show the change immediately so the user knows the search context changed.

---

## 3. Theming

These components inherit styling from `RiviTheme`. Wrap your screen in RiviTheme and the components use your app theme automatically.

*   **Colors**: Components use `RiviTheme.colorScheme` (Primary, Surface, and On-Surface).
*   **Typography**: All text uses `RiviTheme.typography` (Montserrat by default).
*   **Dark Mode**: Every component has a built-in dark mode variant that triggers automatically based on your `RiviTheme` state.

---

## 4. Setup

### Registry Authentication
The SDK is published through GitHub Packages.

> [!IMPORTANT]
> **Authentication is Required**: GitHub Packages requires authentication for Maven downloads, even when the repository is public You must provide a valid GitHub PAT with `read:packages` scope.

See [Working with the Gradle registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry).

**In `gradle.properties`:**
```properties
github.packages.user=YOUR_USERNAME
github.packages.token=YOUR_GITHUB_PAT
```

**In `build.gradle.kts`:**
```kotlin
implementation("co.rivi:askai-ui-components:1.0.8")
```

---

## 3. State Management

This is the basic request and response flow.

### a. Initialization
Initialize Ask AI with your tenant token:
```kotlin
val askAI = AskAI.create(tenantToken = "YOUR_TENANT_TOKEN")
```

### b. Request Flow
1.  **Start Search**: You send a query to the SDK via `askAI.flights.filter(query)`.
2.  **The Result**: The SDK returns an `AskAIResult` object.
3.  **The Loop**: To keep the search session alive, you must use the same `searchId` (UUID) for every request in that session.

### c. ViewModel Implementation
We recommend using a single `UiState` data class to keep your screen synchronized.

```kotlin
data class SearchUiState(
    val activeChips: List<String> = emptyList(),
    val aiNotice: String? = null,
    val isPartialMatch: Boolean = false,
    val isLoading: Boolean = false,
    val results: List<FlightCardData> = emptyList()
)

class SearchViewModel(private val askAI: AskAI) : ViewModel() {
    private val _uiState = MutableStateFlow(SearchUiState())
    val uiState = _uiState.asStateFlow()

    fun handleSearch(query: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            val result = askAI.flights.filter(AskAIFlightFilterRequest(filterQuery = query, ...))

            _uiState.update { it.copy(
                activeChips = result.chips,
                aiNotice = result.entities.firstOrNull()?.parameterChangeNotice,
                isPartialMatch = result.entities.firstOrNull()?.isPartialMatch ?: false,
                isLoading = false
            )}
        }
    }
}
```

---

## 4. Components

### A. `SortFilterRow` (Header)
The entry point for the AI experience. It handles both traditional sorting and the AI sheet trigger.

**Parameters:**
- `activeSortType: String` — The current sort (e.g., "BEST", "PRICE").
- `chips: List<String>` — The AI-suggested filters currently applied.
- `onAskAiClick: () -> Unit` — Trigger to open the `AskAiSheet`.
- `onRemoveChip: (String) -> Unit` — Callback when a user cancels a specific preference.

**Implementation:**
```kotlin
SortFilterRow(
    activeSortType = uiState.sortType,
    onSortClick = { viewModel.setSort(it) },
    onAskAiClick = { viewModel.showAiSheet() },
    chips = uiState.activeChips,
    onRemoveChip = { chip -> viewModel.removeChip(chip) }
)
```

---

### B. `AskAiSheet` (Input)
The primary interaction area for natural language queries.

**Parameters:**
- `value: String` — The user's input text.
- `onValueChange: (String) -> Unit` — Logic to update state as the user types.
- `onApply: () -> Unit` — Triggered by the "Improve Results" button.
- `parameterChangeNotice: String?` — Displays the orange warning banner inside the sheet if the AI shifted context.
- `isFlight: Boolean` — Toggles between flight and hotel-specific placeholders.

**Implementation:**
```kotlin
AskAiSheet(
    value = uiState.query,
    onValueChange = { viewModel.updateQuery(it) },
    onApply = { viewModel.submitQuery() },
    onDismiss = { viewModel.hideSheet() },
    parameterChangeNotice = uiState.aiNotice,
    isFlight = true
)
```

---

### C. `AskAIResultsBanner` (Partial match)
**Purpose**: Show at top of results when AI made trade-offs.

**When to use**: Render this at index `0` of your `LazyColumn` if the results returned do not match 100% of the user's specific preferences (due to price or availability tradeoffs).

**Implementation:**
```kotlin
item {
    if (uiState.isPartialMatch) {
       AskAIResultsBanner()
    }
}
```

---

### D. `ParameterChangeWarning` & `Dialog`
**Purpose**: Notifying users about Context Shifting (e.g., AI searching for different dates).

- **Warning (Inline)**: Use `ParameterChangeWarning` inside the sheet for minor shifts.
    - *Parameter*: `message: String`
    - *Parameter*: `onClick: () -> Unit` (Optional: Use to re-open the search sheet).
- **Dialog (Modal)**: Use `ParameterChangeDialog` for major parameter shifts that require high visibility.

**Implementation (Dialog):**
```kotlin
if (uiState.showChangeDialog) {
    ParameterChangeDialog(
        message = uiState.aiNotice ?: "",
        onDismiss = { viewModel.dismissDialog() }
    )
}
```

---

### E. `ClearAskAIQueryDialog` (Sort conflict)
**Purpose**: Prevents logical search conflicts. If a user tries to change the "Sort By" setting while AI chips are active, show this dialog to confirm they want to clear the AI session.

**Implementation:**
```kotlin
if (uiState.showClearConfirm) {
    ClearAskAIQueryDialog(
        onDismiss = { /* Cancel */ },
        onProceed = { viewModel.clearAiAndSort(newSort) }
    )
}
```

---

## 5. Required Behaviors

*   **The Reset**: When a user removes the last chip, you **must** call a standard search (e.g., `sortBest()`) and clear the AI warnings. If you don't, the user will be stuck in a "stale" search session.
*   **Sorting Conflict**: If the user has AI chips active and manually clicks a "Sort By" button (like "Price"), you should show the `ClearAskAIQueryDialog` to confirm they want to clear their AI search.

---

## 6. Complete Work-through

### ViewModel Refinement Loop
```kotlin
fun removeChip(chip: String) {
    val newList = uiState.value.activeChips.filter { it != chip }

    if (newList.isEmpty()) {
        // RESET to Best Sort (e.g. askAI.flights.sortBest())
        _uiState.update { it.copy(activeChips = emptyList(), aiNotice = null) }
        triggerStandardSearch("BEST")
    } else {
        // RE-FILTER with remaining chips
        _uiState.update { it.copy(activeChips = newList) }
        handleSearch(newList.joinToString(", "))
    }
}
```

### Screen Assembly
```kotlin
Scaffold(
    topBar = {
        SortFilterRow(
            chips = uiState.activeChips,
            onAskAiClick = { /* show sheet */ }
        )
    }
) {
    LazyColumn {
        if (uiState.showPartialBanner) {
            item { AskAIResultsBanner() }
        }
        items(uiState.results) { result ->
            FlightCard(result)
        }
    }
}
```

---


## 7. Troubleshooting & FAQ

**Q: My dependencies won't download (401 Unauthorized)**
> **A**: This is almost always a missing or expired GitHub Personal Access Token (PAT). Verify your PAT has the `read:packages` scope and is correctly saved in your `gradle.properties`.

**Q: The "Ask AI" input is blank after submitting**
> **A**: Ensure your `handleSearch` function updates the `query` field in your `SearchUiState`. The sheet is "stateless" and relies entirely on your ViewModel.

---

## 8. TL;DR

Before you ship, verify these 5 points:
- [ ] **Auth**: Is the GitHub PAT working?
- [ ] **Reset**: Does clearing the last chip trigger a standard "Best Sort" search?
- [ ] **Sticky Bar**: Is the `SortFilterRow` always visible at the top?
- [ ] **Warnings**: Does the orange `ParameterChangeWarning` show up when the AI moves dates?
- [ ] **Theme**: Do the colors look correct in both Light and Dark mode?

