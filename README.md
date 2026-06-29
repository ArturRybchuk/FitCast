# FitCast: Smart Weather & Activity Planner

> **Global climate in your pocket.** A native iOS application that bridges hyper-local weather conditions, air quality monitoring, and personalized outdoor activity recommendations — all wrapped inside a beautiful, multi-sensory glassmorphic UI.

[![Platform](https://img.shields.io/badge/Platform-iOS%2018.5+-blue.svg)]()
[![Language](https://img.shields.io/badge/Language-Swift%205-orange.svg)]()
[![Build](https://img.shields.io/badge/Stack-100%25%20Native%20%2F%20Zero%20Dependencies-brightgreen.svg)]()
[![Status](https://img.shields.io/badge/Status-Pre--Release%20%2F%20Commercial-yellow.svg)]()

---

## App Store Panoramic Showcase

FitCast is designed for a premium App Store presence utilizing a seamless, continuous panoramic mockup grid. The UI transitions smoothly across screens, guiding the user's focus from contextual data widgets to high-density personalization flows.

![FitCast Full Panoramic Showcase](assets/full-length-showcase.jpg)

*App Store Screen Modules Breakdown:*
* **01. Live Weather Dashboard:** ![Live Preview](assets/dashboard-screen.jpg)
* **02. Smart Personalized Alerts:** ![Notification Center](assets/notifications-screen.jpg)
* **03. Global Cities & AQI Monitoring:** ![Cities Tracking](assets/cities-screen.jpg)

---

## About This Repository

**Please Note:** This repository serves as a professional product showcase and architectural documentation. 

The application is being actively prepared for an official **commercial App Store release**. Consequently, the core codebase remains **private** to protect proprietary scoring logic, system configurations, and custom visual assets. 

This `README.md` provides an exhaustive technical blueprint, architectural overview, data pipeline map, and explicit code snippets to demonstrate engineering decisions and system-level implementation without exposing the private repository files.

---

## Core Vision & Problem Statement

Standard weather applications function merely as digital thermometers. They report raw atmospheric values (degrees, humidity percentages, wind speeds), leaving it to the user to manually interpret what those metrics mean for their lifestyle and daily routines. 

**FitCast shifts the paradigm from *numbers* to *actions*.** It proactively interprets real-time local conditions against explicit personal parameters, instantly answering the user's actual question: 

> *"Given the exact environmental, temperature, and air quality index right now, what outdoor activities can I safely, comfortably, and optimally enjoy?"*

---

## 🚀 Key Product Features

### 1. Smart Activity Recommendation Engine
Instead of generic, static outdoor tips, FitCast runs a dynamic evaluation matrix. The calculation targets data from the 5-day forecast strip and cross-references it with user interests established during onboarding. It scores parameters including temperature bounds, wind restrictions, and precipitations, applying intelligent fallback workflows (e.g., routing to `.indoorFitness` or home-based tasks) to guarantee the screen never shows a broken or empty state.

### 2. Deep Air Quality Layer (AQI)
FitCast introduces environmental health as a primary metric by coupling standard weather feeds with the **WAQI API**. The application features advanced fallback station discovery: if a primary station’s data payload becomes stale or drops off, the client dynamically traces surrounding bounding boxes to locate the nearest active reporting station.

### 3. Multi-Sensory Immersive UX
The interface is engineered to adapt directly to the climate, allowing users to *feel and hear* conditions through a multi-sensory feedback layer:
* **Tactile Haptics (`CoreHaptics`):** Emits distinctive vibration patterns (e.g., sharp, fast transient impulses for heavy thunderstorms; continuous low-frequency waves for severe wind gusts).
* **Ambient Audio (`AVFoundation`):** Fades in looping environmental sound micro-tracks (rain patterns, wind chimes, distant storms) when inspecting deep weather analytics.
* **Accessibility Asset:** Beyond visual luxury, this serves as an essential feature for **visually impaired or low-vision individuals**, providing direct sensory feedback about local environmental changes without requiring labels to be read.

### 4. Intentional Local Alert Scheduling
FitCast rejects erratic notification cycles, routing all notifications locally via `UserNotifications` synchronized against background execution boundaries:
* **Morning Recommendations (06:00):** A custom activity layout optimized for the upcoming morning window.
* **Preferred Activity Windows:** Timed push alerts triggered exactly 30 minutes before the user's chosen routine workout hours.
* **Night Forecast Digest (21:00):** A strategic evening update evaluating the **next morning’s forecast**. This enables users to seamlessly prep gear, choose sportswear, or alter their alarms before going to sleep.

### 5. Pure Native Implementation (Zero-Dependency)
The architectural baseline relies 100% on native Apple capabilities. It uses no CocoaPods, no Swift Package Manager (SPM) wrappers, and no heavy third-party networking libraries. This completely eliminates supply-chain risks, guarantees an ultra-low binary foot-print, and maximizes hardware optimization.

---

## User Flows & Controller Routing

### Onboarding & Initialization (`FirstScreenViewController`)

```

App Cold Launch ──► Has Completed Onboarding Check? (UserDefaults)
│
├──► [FALSE] ──► Onboarding Flow Launch
│                 ├── Enter User Name
│                 ├── Pick Interests (Max 5 Activities)
│                 ├── Configure Preferred Time Frames (Max 3)
│                 └── Select Fav Weather Profiles (Max 3)
│                      │
│                      └──► Persist to UserData (Singleton)
│                           ├── Request Push Notification Rights
│                           └── Trigger Main Dashboard Route
│
└──► [TRUE]  ──► Route Direct to Weather Dashboard

```

### Dashboard Core Pipeline (`WeatherViewController`)

```

WeatherViewController ──► Check Location Permissions (CoreLocation)
│
├──► [GRANTED] ─► Fetch Live Lat/Lon Coordinates
└──► [DENIED]  ─► Read Last Cached City Name Location
│
▼
Dispatch Async Network Calls via URLSession
├── WeatherManager ──► OpenWeatherMap API
└── AQIManager     ──► WAQI API Endpoint
│
▼
Parse Payloads to Strongly-Typed Codable Structures
│
▼
Initialize WeatherViewModel Presentation Models
                  │
                  ▼
┌─────────────────┴─────────────────┐
▼                                   ▼
Update UIKit Views                   Fire Sensory Managers
├── Render Custom Glass Blurs        ├── SoundManager Loop
├── Populate 5-Day Strip             └── CoreHaptics Wave
└── Run Activity Scoring Engine

```

### Saved Cities Management (`CitiesViewController`)

```

CitiesViewController ──► Preload Display Cache (In-Memory Arrays)
│
├──► Search UI ──► Direct API Query ──► Append to Favorites List
├──► Swipe Cell Gesture ─────────────► Remove from UserDefaults Cache
└──► Tap Grid Card Row ──────────────► Swap Main View Target Location

```

---

## Architectural Blueprint

FitCast embraces a clear, manager-centric **Separation of Concerns (SoC)** approach, thoroughly dividing background resource ingestion, model mapping, scoring business layers, and presentation.


```

┌─────────────────────────────────────────────────────────────┐
│                        UI / View Layer                      │
│  WeatherViewController  ·  CitiesViewController             │
│  ProfileViewController  ·  FirstScreenViewController        │
│  Style.swift  ·  LiquidGlassView  ·  CityTableViewCell      │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                   Presentation & ViewModels                 │
│  WeatherViewModel  ·  CityCellDisplayModel                  │
│  CityWeatherDisplayHelper  ·  ToastPresenter                │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                     Core Business Logic                     │
│  ActivityRecommendationManager  ·  NotificationManager      │
│  UserData (User Profile State)  ·  CitiesStore Persistence  │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                System Services & Network Enablers           │
│  WeatherManager  ·  AQIManager  ·  CityAQIFetcher           │
│  WeatherBackgroundTaskManager  ·  SoundManager  ·  HapticApp│
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                      Data Persistence Layer                 │
│  UserDefaults Engine  ·  LocationCache  ·  Codable Structs  │
└─────────────────────────────────────────────────────────────┘

```

### Design Patterns Enforced
* **Singleton Managers:** Structural core components (`WeatherManager`, `AQIManager`, `NotificationManager`, `UserData`, `HapticManager`) maintain a single-point-of-truth state.
* **Delegate Pattern:** `AQIManagerDelegate` decouples background network data delivery pipelines from view rendering instances.
* **Observer Layouts:** Native `NotificationCenter` broadcasts profile configurations or connectivity state variations across decoupled controllers seamlessly.
* **Hybrid Presentation Interfaces:** Employs `UIHostingController` wrappers to insert interactive SwiftUI layers (`LiquidMenu`) inside UIKit Storyboard lifecycles.

---

## Comprehensive Tech Stack Registry

### Languages & UI Layout Engine
* **Swift 5** — Powers 100% of the core app architecture, business logic engines, network contracts, and widget entities.
* **UIKit** — Governs primary layout structures (onboarding loops, table view arrays, navigation controllers). Implements robust programmatic frame auto-layout logic mixed with stable **Storyboards** constraints.
* **SwiftUI** — Leveraged to build rich, fluid menu animations (`LiquidMenu`).

### System Frameworks (Apple Capabilities)
* **CoreLocation** — Coordinates precise GPS parsing, manages coordinate caches for rapid cold-start rendering, and monitors location parameters to drive automated weather refresh pipelines.
* **UserNotifications** — Registers and targets the local contextual message scheduling pipeline (workout indicators, daily activity suggestions, early-morning forecasts).
* **BackgroundTasks** — Uses `BGAppRefreshTask` routines to systematically trigger automated, background data updates (`com.fitcast.weather.refresh`) before notification delivery.
* **CoreHaptics** — Creates complex haptic transient patterns mimicking precise real-world weather metrics.
* **AVFoundation** — Utilizes `AVAudioPlayer` structures to render custom-mapped looping ambient audio profiles without memory footprint leakage.
* **Network** — Tracks internet link status variations via natively tracked paths to suspend heavy data requests when cellular access drops out.

### Networking & Data Engineering
* **URLSession** — Executes clean async web requests targeting REST endpoints without relying on heavy external clients like Alamofire.
* **Codable** — Enforces type-safe serialization of complex JSON schemas directly into structural Swift entities (`WeatherData`, `ForecastData`, `AQIData`).
* **Swift Concurrency** — Implements thread safety across data tasks using explicit async/await patterns, `@MainActor` UI constraints, and structured `Task` scopes.
* **OperationQueue** — Sequences recurrent weather fetch procedures into structured background execution channels.

---

## File & Directory Module Mapping

```text
📁 FitCast/
 │
 ├── 📁 App/
 │    ├── 📜 AppDelegate.swift       # BGTask registrations & application launch setup
 │    └── 📜 SceneDelegate.swift       # UI Window routing (Onboarding vs Dashboard)
 │
 ├── 📁 Controllers/
 │    ├── 📜 FirstScreenViewController.swift  # Profile initialization & onboarding wizard
 │    ├── 📜 WeatherViewController.swift      # Primary climate dashboard and scoring hub
 │    ├── 📜 CitiesViewController.swift       # City card registry, API lookups & search workflows
 │    └── 📜 ProfileViewController.swift      # Preference adjustment view housing SwiftUI menus
 │
 ├── 📁 Managers/
 │    ├── 📜 WeatherManager.swift             # REST client for OpenWeatherMap integration
 │    ├── 📜 AQIManager.swift                 # Environmental station tracer and WAQI client
 │    ├── 📜 ActivityRecommendationManager.swift # Logic processing engine for sports suitability
 │    ├── 📜 NotificationManager.swift        # Push logic tracker, builder and scheduler
 │    ├── 📜 HapticManager.swift              # CoreHaptics pattern array playback coordinator
 │    └── 📜 SoundManager.swift               # AVFoundation looping ambient audio controller
 │
 ├── 📁 Models/
 │    ├── 📜 UserData.swift                   # Singleton handling persistent profile metrics
 │    ├── 📜 WeatherData.swift                # Codable response structures for weather APIs
 │    ├── 📜 WeatherModel.swift               # Domain-level weather structures used by presentation
 │    ├── 📜 AQIData.swift                    # Codable schemas parsing nested WAQI payloads
 │    └── 📜 SavedCity.swift                  # Persistent custom city metadata blueprints
 │
 └── 📁 Views/
      ├── 📜 Style.swift                      # Central styling engine (glass layers, parallax, effects)
      ├── 📜 LiquidGlassView.swift            # Reusable wrapper utilizing native glass visual matrices
      ├── 📜 LiquidMenu.swift                 # SwiftUI expandable interface components
      └── 📜 CityTableViewCell.swift          # Custom glass-designed list rows for saved cities

```

---

## Deep Technical Implementation Highlights

### 1. Persistent Activity & Scoring Algorithms

The `ActivityRecommendationManager` maps localized, real-time incoming OpenWeatherMap condition codes (`WeatherCategory`) against strict ideal ranges defined within a static activity catalog.

```swift
// Presentation architecture for user activity recommendation pipelines
enum WeatherCategory {
    case sunny, cloudy, rainy, snowy, severe
}

enum ActivityType: String, Codable {
    case running, cycling, yoga, walking, outdoorFitness, skiing, indoorFitness
}

final class ActivityRecommendationEngine {
    static func generateSuggestions(userInterests: [ActivityType], weatherCode: Int, isNight: Bool) -> [ActivityType] {
        let category = mapConditionCode(weatherCode)
        
        let suitableActivities: [ActivityType] = {
            switch (category, isNight) {
            case (.sunny, false): return [.running, .yoga, .cycling]
            case (.sunny, true):  return [.walking]
            case (.cloudy, _):    return [.walking, .outdoorFitness]
            case (.rainy, _):     return [.yoga, .indoorFitness]
            case (.snowy, _):     return [.skiing]
            default:              return [.indoorFitness]
            }
        }()
        
        // Output intersection matching environmental variables vs explicit user preferences
        let output = suitableActivities.filter { userInterests.contains($0) }
        
        // Multi-tier fallback strategy guarantees screen is never left empty
        return output.isEmpty ? [.indoorFitness] : output
    }
    
    private static func mapConditionCode(_ code: Int) -> WeatherCategory {
        switch code {
        case 200...299, 700...781: return .severe
        case 300...599: return .rainy
        case 600...699: return .snowy
        case 800: return .sunny
        default: return .cloudy
        }
    }
}

```

### 2. Multi-Sensory Accessibility Driver

The wrapper module below demonstrates how structural weather identification keys are safely interpreted into ambient looping audio clips and intricate physical haptic event arrays.

```swift
import CoreHaptics
import AVFoundation

final class SensoryFeedbackManager {
    static let shared = SensoryFeedbackManager()
    private var hapticEngine: CHHapticEngine?
    private var audioPlayer: AVAudioPlayer?
    
    private init() {
        guard CHHapticEngine.capabilitiesForHardware().supportsHaptics else { return }
        do {
            hapticEngine = try CHHapticEngine()
            try hapticEngine?.start()
        } catch { print("Sensory haptics dropped initialization: \(error)") }
    }
    
    func executeWeatherFeedback(code: Int) {
        // 1. CoreAudio Pipeline for Low-Vision Accessibility Context
        let audioTrackName = code >= 500 ? "ambient_heavy_rain" : "ambient_clear_sky"
        if let trackAsset = NSDataAsset(name: audioTrackName) {
            do {
                audioPlayer = try AVAudioPlayer(data: trackAsset.data)
                audioPlayer?.numberOfLoops = -1 // Persistent ambient simulation loop
                audioPlayer?.volume = 0.12      // Subtle, non-intrusive backing audio levels
                audioPlayer?.play()
            } catch { print("Audio feedback system failure: \(error)") }
        }
        
        // 2. CoreHaptics Continuous Event Generation
        guard let engine = hapticEngine else { return }
        let intensityValue: Float = code >= 500 ? 0.85 : 0.35
        let sharpnessValue: Float = code >= 500 ? 0.70 : 0.15
        
        let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: intensityValue)
        let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: sharpnessValue)
        
        let continuousEvent = CHHapticEvent(eventType: .hapticContinuous, parameters: [intensity, sharpness], relativeTime: 0, duration: 3.0)
        
        do {
            let pattern = try CHHapticPattern(events: [continuousEvent], parameters: [])
            let dynamicPlayer = try engine.makePlayer(with: pattern)
            try dynamicPlayer.start(atTime: 0)
        } catch { print("Haptic feedback structural run error: \(error)") }
    }
}

```

### 3. Asynchronous Background Scheduling

To fulfill the value proposition of the Night Forecast Digest and Sport Hours alerts, the network engine updates location details and indices quietly via registered task requests.

```swift
import BackgroundTasks

final class WeatherBackgroundSyncCoordinator {
    static let shared = WeatherBackgroundSyncCoordinator()
    private let refreshIdentifier = "com.fitcast.weather.refresh"
    
    func scheduleNextSilentUpdate() {
        let taskRequest = BGProcessingTaskRequest(identifier: refreshIdentifier)
        taskRequest.earliestBeginDate = Date(timeIntervalSinceNow: 3600) // Execute updates recurrently every hour
        taskRequest.requiresNetworkConnectivity = true
        taskRequest.requiresExternalPower = false
        
        do {
            try BGTaskScheduler.shared.submit(taskRequest)
        } catch { print("Background submit rejected: \(error)") }
    }
    
    func executeSilentPipeline(task: BGProcessingTask) {
        scheduleNextSilentUpdate()
        
        let backgroundQueue = OperationQueue()
        backgroundQueue.maxConcurrentOperationCount = 1
        
        let networkOperation = BlockOperation {
            // Quietly invoke endpoints, evaluate alerts parameters and update local cache
            print("Background refresh completed silently.")
        }
        
        task.expirationHandler = { backgroundQueue.cancelAllOperations() }
        networkOperation.completionBlock = { task.setTaskCompleted(success: !networkOperation.isCancelled) }
        backgroundQueue.addOperation(networkOperation)
    }
}

```

---

## Quality & Launch Benchmarks

To adhere strictly to the Apple App Store Review Guidelines, the FitCast architecture layout respects major platform design patterns:

1. **Main Thread UI Routing:** All network response handshakes generated inside `URLSession` blocks or location updates processed via `CoreLocation` delegates are forced back through `DispatchQueue.main` or decoupled safely onto `@MainActor` constraints.
2. **Asset Failure Fallbacks:** The translation layer managing the 53 background layouts and looping sound files handles dictionary lookups defensively. Any missing keys or corrupt API statuses default gracefully to safe fallback assets, completely eliminating runtime errors.
3. **Local Memory Isolation:** User preference sets, favorited tracking spots, and transient coordinates are handled without writing heavy operational databases, allowing the footprint to scale cleanly.

---

## Licensing & Permissions

Developed strictly as a private commercial iOS product. All layout structures, continuous multi-panel mockups, visual illustrations, and underlying logic models are fully protected under global intellectual property frameworks and App Store commercial presentation agreements.

```
