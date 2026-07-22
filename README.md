## Generate License Information File:
```shell
  ..\gradlew :composeApp:exportLibraryDefinitions
```

## NavGraph:
```kotlin
navigation<Route.LegalNav>(
    startDestination = LegalRoute.Imprint,
    enterTransition = { slideInHorizontally { it } },
    popExitTransition = { slideOutHorizontally { it } },
) {
    composable<LegalRoute.Imprint> {
        ImprintScreen(
            navigateBack = { navController.popBackStack() },
        )
    }
    composable<LegalRoute.Privacy> {
        var htmlData by remember { mutableStateOf<String?>(null) }
        LaunchedEffect(Unit) {
            htmlData = Res.readBytes("files/privacy_policy_de.html").decodeToString()
        }
        PrivacyScreen(
            htmlData = htmlData,
            navigateBack = { navController.popBackStack() },
        )
    }
    composable<LegalRoute.Licenses> {
        val libraries by produceLibraries {
            Res.readBytes("files/aboutlibraries.json").decodeToString()
        }
        LicensesScreen(
            libraries = libraries,
            navigateBack = { navController.popBackStack() },
        )
    }
}
```

## Dependencies:
### VersionCatalog:
```toml
agp = "9.2.1"
kotlin = "2.4.10"
kotlinStdlib = "2.4.10"
composeMultiplatform = "1.11.1"
androidx-activityCompose = "1.13.0"

aboutLibraries = "15.0.4"

[libraries]
androidx-activity-compose = { module = "androidx.activity:activity-compose", version.ref = "androidx-activityCompose" }

ui-backhandler = { module = "org.jetbrains.compose.ui:ui-backhandler", version.ref= "compose-multiplatform" }

kotlin-stdlib = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib", version.ref = "kotlinStdlib" }
aboutlibraries-compose-core = { module = "com.mikepenz:aboutlibraries-compose-core", version.ref = "aboutLibraries" }
aboutlibraries-compose-m3 = { module = "com.mikepenz:aboutlibraries-compose-m3", version.ref = "aboutLibraries" }

[plugins]
# Basics
composeCompiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
# KMP/CMP
androidMultiplatformLibrary = { id = "com.android.kotlin.multiplatform.library", version.ref = "agp" }
composeMultiplatform = { id = "org.jetbrains.compose", version.ref = "composeMultiplatform" }
kotlinMultiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
# About Libraries
aboutLibraries = { id = "com.mikepenz.aboutlibraries.plugin", version.ref = "aboutLibraries" }
```

### root build.gradle.kts
```kts
plugins {
    alias(libs.plugins.androidMultiplatformLibrary) apply false
    alias(libs.plugins.composeMultiplatform) apply false
    alias(libs.plugins.composeCompiler) apply false
    alias(libs.plugins.kotlinMultiplatform) apply false

    alias(libs.plugins.kotlin.serialization) apply false

    alias(libs.plugins.aboutLibraries) apply false
}
```

### app build.gradle.kts
```kts
plugins {
    alias(libs.plugins.kotlinMultiplatform)
    alias(libs.plugins.androidMultiplatformLibrary)
    alias(libs.plugins.composeMultiplatform)
    alias(libs.plugins.composeCompiler)
    
    alias(libs.plugins.kotlin.serialization)

    alias(libs.plugins.aboutLibraries)
}

aboutLibraries {
    export {
        outputFile = file("src/commonMain/composeResources/files/aboutlibraries.json")
    }
}

kotlin {
    [...]
    sourceSets {
        [...]
        commonMain.dependencies {
            [...]
            implementation(projects.legal)
            [...]
            // About Libraries
            implementation(libs.aboutlibraries.compose.m3)
        }
    }
}
```