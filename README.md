# Rivi Android SDK
[![askai](https://img.shields.io/badge/askai-latest-blue)](./askai/ASKAI%20INTEGRATION_GUIDE.md)
[![askai-ui-components](https://img.shields.io/badge/askai--ui--components-latest-blue)](./askai-ui/ASKAI%20UI%20INTEGRATION_GUIDE.md)
[![rivi-ui-core](https://img.shields.io/badge/rivi--ui--core-latest-blue)](./rivi-ui-core/RIVI%20UI%20INTEGRATION_GUIDE.md)

Welcome to the official **Rivi Android SDK** repository. This repository hosts the public Maven registry, package metadata, and integration documentation for Rivi’s AI-powered travel search and UI ecosystem.

## Overview

The Rivi Android SDK helps Android teams integrate AI-powered flight and hotel search into their apps with modular SDK packages and ready-to-use Jetpack Compose UI components.

It is designed to support different levels of integration:
- Use `askai` for core networking, search, ranking, and SSE-based updates.
- Use `askai-ui-components` for plug-and-play Compose screens.
- Use `rivi-ui-core` for Rivi’s shared design system and foundational UI layer.

## Latest Versions

| Package | Latest Version | Artifact |
| :--- | :--- | :--- |
| `askai` | `1.0.8` | `co.rivi:askai` |
| `askai-ui-components` | `1.0.8` | `co.rivi:askai-ui-components` |
| `rivi-ui-core` | `1.0.8` | `co.rivi:rivi-ui-core` |

## Available Packages

| Package | Purpose | Documentation |
| :--- | :--- | :--- |
| **`co.rivi:askai`** | Core Android SDK for AI ranking, SSE subscriptions, and travel search logic. | [Core SDK Guide](./askai/ASKAI%20INTEGRATION_GUIDE.md) |
| **`co.rivi:askai-ui-components`** | Ready-to-use Compose screens such as `Dashboard`, `FlightSearch`, and `HotelSearch`. | [UI Components Guide](./askai-ui/ASKAI%20UI%20INTEGRATION_GUIDE.md) |
| **`co.rivi:rivi-ui-core`** | Rivi design system, including theme, typography, spacing, and shared components. | [UI Core Guide](./rivi-ui-core/RIVI%20UI%20INTEGRATION_GUIDE.md) |

## Installation

Add the Rivi Maven repository to your `settings.gradle.kts`:

> [!IMPORTANT]
> GitHub Packages requires authentication, even for public packages in many client setups. Use your GitHub username and a Personal Access Token with package read access.

### 1. Add Repository

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()

        maven {
            name = "RiviPublic"
            url = uri("https://maven.pkg.github.com/rivi-inc/Rivi-Android-Sdk")
            credentials {
                username = "YOUR_GITHUB_USERNAME"
                password = "YOUR_GITHUB_PERSONAL_ACCESS_TOKEN"
            }
        }
    }
}
```

### 2. Add Dependencies

Choose the modules you need in your `build.gradle.kts`:

```kotlin
dependencies {
    // Core SDK
    implementation("co.rivi:askai:1.0.8")

    // Ready-to-use Compose UI layer
    implementation("co.rivi:askai-ui-components:1.0.8")

    // Shared Rivi design system and foundational components
    implementation("co.rivi:rivi-ui-core:1.0.8")
}
```

## Integration Guides

- [`askai` Integration Guide](./askai/ASKAI%20INTEGRATION_GUIDE.md)
- [`askai-ui-components` Integration Guide](./askai-ui/ASKAI%20UI%20INTEGRATION_GUIDE.md)
- [`rivi-ui-core` Integration Guide](./rivi-ui-core/RIVI%20UI%20INTEGRATION_GUIDE.md)

## Module Guide

### `askai`
Use this module when you need the core SDK layer for AI search, ranking, subscriptions, and travel search orchestration.

### `askai-ui-components`
Use this module when you want ready-made Jetpack Compose screens that speed up integration and reduce UI development effort.

### `rivi-ui-core`
Use this module when you need access to Rivi’s shared theme, design tokens, and reusable UI building blocks.

## Versioning

We recommend keeping all Rivi SDK modules on the same version whenever possible to ensure compatibility across the full SDK stack.

## Support

For setup steps and module-level integration details, start with the package-specific integration guides linked above.

© 2026 Rivi Inc. All rights reserved.
