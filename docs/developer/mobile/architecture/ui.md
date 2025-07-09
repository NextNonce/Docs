

The user interface for the NextNonce application is built entirely with **Compose Multiplatform**, JetBrains' modern declarative UI toolkit for Kotlin. This strategic choice allows for a single, unified UI codebase that runs natively on both Android and iOS, providing a consistent user experience across platforms.

### Core Philosophy

The fundamental principle of our UI architecture is **100% shared code**. All UI-related code, from individual components like buttons and text fields to entire screens, is located in the `composeApp/src/commonMain` source set.

This approach means the project contains **absolutely no platform-specific UI code**. You will not find any Android XML layouts or SwiftUI views. Every pixel rendered on the screen is drawn by Compose Multiplatform, ensuring that the look, feel, and behavior of the app are identical on both Android and iOS.

The application adheres to **Material Design 3** principles, providing a modern, clean, and intuitive interface that supports both light and dark themes out of the box.


<div class="screenshots-row-with-text">
  <div class="screenshot">
    <img src="/images/ui/portfolio/android_portfolio_light.jpg" alt="Android Light">
    <div class="caption">Android — Light Theme</div>
  </div>
  <div class="screenshot">
    <img src="/images/ui/portfolio/android_portfolio_dark.jpg" alt="Android Dark">
    <div class="caption">Android — Dark Theme</div>
  </div>
</div>

<div class="screenshots-row-with-text">
  <div class="screenshot">
    <img src="/images/ui/portfolio/iphone_portfolio_light.png" alt="iOS Light">
    <div class="caption">iOS — Light Theme</div>
  </div>
  <div class="screenshot">
    <img src="/images/ui/portfolio/iphone_portfolio_dark.png" alt="iOS Dark">
    <div class="caption">iOS — Dark Theme</div>
  </div>
</div>
### Key Libraries

* **Compose Multiplatform (`org.jetbrains.compose`)**: The core framework used to build the entire user interface.
* **Kamel (`media.kamel:kamel-image-default`)**: The library responsible for asynchronous image loading, chosen specifically for its excellent cross-platform performance.

---

### A Note on Image Loading: From Coil to Kamel

During development, image loading presented a significant cross-platform challenge. The project initially integrated **Coil**, a popular and powerful image loading library for Android.

While Coil performed adequately on Android, it exhibited persistent and critical issues on iOS, most notably rendering images in a noticeably **low quality**. Despite a significant amount of time invested in troubleshooting, testing various configurations, and attempting different implementation strategies, the image quality on iOS remained substandard.

To resolve this, the decision was made to replace Coil with **Kamel**, an image loading library designed from the ground up for Kotlin Multiplatform. The migration to Kamel immediately solved the image quality problems on iOS while maintaining excellent performance on Android. It is now the sole library used for displaying all remote images, such as token and network icons.