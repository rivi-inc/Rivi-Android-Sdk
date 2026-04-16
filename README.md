# Rivi Android SDK

[![askai](https://img.shields.io/badge/askai-1.0.9-blue)](./askai/ASKAI%20INTEGRATION_GUIDE.md)
[![askai-ui-components](https://img.shields.io/badge/askai--ui--components-1.0.9-blue)](./askai-ui/ASKAI%20UI%20INTEGRATION_GUIDE.md)
[![rivi-ui-core](https://img.shields.io/badge/rivi--ui--core-1.0.9-blue)](./rivi-ui-core/RIVI%20UI%20INTEGRATION_GUIDE.md)

Maven packages and documentation for the Rivi Ask AI Android SDK.

## Packages

| Package | Purpose | Guide |
|---------|---------|-------|
| `co.rivi:askai` | Core Ask AI search, ranking, SSE updates | [Guide →](./askai/ASKAI%20INTEGRATION_GUIDE.md) |
| `co.rivi:askai-ui-components` | Ask AI UI components (`SortFilterRow`, `AskAiSheet`) **_requires rivi-ui-core_** | [Guide →](./askai-ui/ASKAI%20UI%20INTEGRATION_GUIDE.md) |
| `co.rivi:rivi-ui-core` | **Required** theme and design system for Ask AI UI components | [Guide →](./rivi-ui-core/RIVI%20UI%20INTEGRATION_GUIDE.md) |

**Full Ask AI experience**: `askai` + `askai-ui-components` + `rivi-ui-core`

## Installation

### Repository (`settings.gradle.kts`)
```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            name = "Rivi"
            url = uri("https://maven.pkg.github.com/rivi-inc/Rivi-Android-Sdk")
            credentials(PassowrdCredentials::class) {
                username = "YOUR_GITHUB_USERNAME"
                password = "YOUR_GITHUB_PAT"
            }
        }
    }
}


```

### Dependencies (`build.gradle.kts`)
```kotlin
// Required for full Ask AI experience
implementation("co.rivi:askai:$latestVersion")
implementation("co.rivi:askai-ui-components:$latestVersion")
implementation("co.rivi:rivi-ui-core:$latestVersion")
```

> [!NOTE]  
> [GitHub Packages requires authentication even for public packages](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry#authenticating-to-github-packages).  
> Use PAT with `read:packages` scope.

## Quick Usage

**Core SDK:**
```kotlin
val askAI = AskAI.create("your-tenant-token")
val result = askAI.flights.filter("cheap flights with meals")
```

**UI Components:**
```kotlin
RiviTheme {
    SortFilterRow(
        chips = uiState.activeChips,
        onAskAiClick = { showAskAiSheet() },
        ...
    )
}
```

## Documentation

**Start here for Ask AI:**
1. [AskAI Core →](./askai/ASKAI%20INTEGRATION_GUIDE.md)
2. [AskAI UI Components →](./askai-ui/ASKAI%20UI%20INTEGRATION_GUIDE.md) 
3. [Rivi UI Core (required for UI) →](./rivi-ui-core/RIVI%20UI%20INTEGRATION_GUIDE.md)

**Keep all modules at `1.0.9`** for compatibility.

---

© 2026 Rivi Inc.
