# Development setup

[Back to the README](../README.md) · [Architecture](ARCHITECTURE.md)

This guide describes the configuration expected by the checked-in source. Use a backend you control and test accounts while exploring the app.

## 1. Prepare the Android environment

Open the repository root in Android Studio and install Android SDK Platform **36** through the SDK Manager. Run the app on Android **8.0 / API 26** or newer. The library modules declare API 24, but the application itself requires API 26.

| Setting | Checked-in value | Source |
| --- | --- | --- |
| Application ID | `com.yoav_s.tashtit` | [app/build.gradle.kts](../app/build.gradle.kts) |
| Compile SDK | 36 | [app/build.gradle.kts](../app/build.gradle.kts) |
| Target SDK | 34 | [app/build.gradle.kts](../app/build.gradle.kts) |
| Minimum app SDK | 26 | [app/build.gradle.kts](../app/build.gradle.kts) |
| Java source / target | 17 | [app/build.gradle.kts](../app/build.gradle.kts) |
| Gradle | 8.13 | [Wrapper configuration](../gradle/wrapper/gradle-wrapper.properties) |
| Android Gradle Plugin | 8.13.1 | [Version catalog](../gradle/libs.versions.toml) |
| Gradle daemon JVM | JetBrains JDK 21 | [Daemon configuration](../gradle/gradle-daemon-jvm.properties) |

The daemon JVM and the Java language level serve different purposes. Gradle's daemon criteria select the runtime used for the build; `compileOptions` controls the language and bytecode level. Gradle may download the selected daemon runtime on the first build, so network access is needed.

## 2. Connect your own Firebase project

1. Follow the official [Firebase Android setup guide](https://firebase.google.com/docs/android/setup).
2. Register an Android app with package name **`com.yoav_s.tashtit`**.
3. Replace `app/google-services.json` with the configuration downloaded for your own project.
4. Enable **Cloud Firestore** and **Cloud Storage** for the data and photo features you want to test.
5. Review [FirebaseInstance.java](../REPOSITORY/src/main/java/com/yoav_s/repository/BASE/DB/FirebaseInstance.java). It contains a manual fallback configured for the original Firebase project. If you retain that fallback, update its project ID, Firebase app ID, API key, and bucket to match your own project.

The Google Services Gradle plugin is already applied. With a valid `google-services.json`, Firebase normally initializes its default app from generated resources. The fallback is used if the repository cannot obtain that default instance.

The existing configuration is a reference to the original project, not a promise of access to its backend. Do not use the original backend for development. Firebase configuration identifies a project; it does not replace database or storage authorization rules.

### Data and access rules

The repositories use these case-sensitive collection names:

```text
Users, Plants, Species, CareTasks, HistoryNotes,
Guides, GuideInteractions, Settings, Reminders
```

Firestore creates collections when documents are first written. A fresh backend will have no community guides until you publish them through a content-creator account. Species browsing uses Perenual; the `Species` model and repository also exist for Firestore data.

**Authentication constraint:** the current code does not use Firebase Authentication. It queries user documents and checks a password hash on the device. A UI sign-in therefore does not establish a Firebase-authenticated identity for Security Rules. Rules based on `request.auth.uid` will not grant access merely because the app says a user is signed in. Do not make the database publicly readable or writable to work around this. A secure shared deployment requires implementing trusted authentication and matching authorization rules first; until then, keep exploration isolated with synthetic data.

Firestore rules, Storage rules, index definitions, and a seed dataset are not supplied in this repository. If Firestore reports a missing index, inspect the specific query and the suggested index in your own Firebase project.

## 3. Configure local API keys

Android Studio normally creates `local.properties` at the repository root with your local `sdk.dir`. Keep that entry and add the following properties, without quotation marks:

```properties
# Existing sdk.dir entry remains here.
PERENUAL_API_KEY=replace_with_your_perenual_key
GEMINI_API_KEY=replace_with_your_gemini_key
CHATGPT_API_KEY=
```

| Property | Purpose |
| --- | --- |
| `PERENUAL_API_KEY` | Species list and details requests through the [Perenual API](https://perenual.com/docs/api). |
| `GEMINI_API_KEY` | The active AI assistant integration. Obtain a key through [Google AI Studio](https://aistudio.google.com/apikey). |
| `CHATGPT_API_KEY` | An existing build configuration field. It is not used by the current Gemini assistant and can remain empty. |

[REPOSITORY/build.gradle.kts](../REPOSITORY/build.gradle.kts) reads these exact property names and generates `BuildConfig` fields. Missing properties become the literal string `"null"`; they are not validated by the build. A successful build alone does not confirm that species search or AI requests are configured correctly.

Keep `local.properties` out of version control. The new root `.gitignore` excludes it. `google-services.json` and the fallback source are already tracked: do not include your local replacements in a pull request.

The checked-in Gemini integration uses the `generativeai` Android SDK **0.9.0** and requests **`gemini-2.5-flash`**. Model access and availability can change independently of this repository. Inspect [GeminiRepository.java](../REPOSITORY/src/main/java/com/yoav_s/repository/AI/GeminiRepository.java) if the provider reports a model or SDK error. Perenual plan limits can also affect the fields and results returned.

## 4. Build and run

Sync Gradle, choose the **app** run configuration, and launch the app on a device or emulator. From the project root:

```bash
./gradlew :app:assembleDebug
./gradlew :app:lintDebug
```

Windows equivalents:

```powershell
.\gradlew.bat :app:assembleDebug
.\gradlew.bat :app:lintDebug
```

The debug APK is generated at `app/build/outputs/apk/debug/app-debug.apk`. Release signing credentials and a distribution workflow are not provided.

## 5. Walk through the main features

1. Open the guest experience and search for a plant species.
2. Register a synthetic test account. Choose **User** for personal care or **Content creator** for guide publishing.
3. Add a plant with a nickname and location, then review and adjust its recurring care schedule.
4. Open plant details and the calendar. Mark a task done or skipped and check the history and next scheduled occurrence.
5. Add a history note and photo. Allow camera or image access when needed.
6. Enable reminders in Settings and grant notification permission on Android versions that request it. Set a near-term reminder to check delivery.
7. Read or publish a guide, then try ratings, comments, and creator analytics with synthetic data.
8. Open the AI assistant, ask a general or plant-specific question, and optionally attach a plant photo.

The assistant sends the question, selected plant context, and any attached image to Google's service. Use non-sensitive sample content during development.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| SDK location or platform error | Confirm `sdk.dir` in `local.properties` and install SDK Platform 36. |
| Gradle daemon download or JVM error | Check the daemon JVM criteria, internet access, and JDK compatibility. |
| Google Services configuration error | Confirm `app/google-services.json` exists and its package name matches the application ID. |
| Firestore or Storage permission denied | Confirm your own project is selected and inspect its access rules. UI sign-in is not Firebase Authentication. |
| Empty guide list | A new backend has no guides until a content creator publishes one. |
| Species request fails or details are incomplete | Check `PERENUAL_API_KEY`, network access, API response, and account plan limits. Some missing care fields use application fallbacks. |
| AI request fails | Check `GEMINI_API_KEY`, quota, network access, and the model configured in `GeminiRepository`. |
| No reminder appears | Check app notification permission, notification settings, reminder time, scheduled tasks, and device battery restrictions. |
| Snooze fails on a newer Android version | Review the exact-alarm calls in `TaskSnoozeReceiver` against that Android version's permission requirements. |

## Development notes

The following notes explain how the current implementation works and provide context for extending the project.

- **User accounts:** Accounts are stored in Firestore, with password verification performed by the app. When preparing a public deployment, connect account verification to a trusted authentication service and enforce permissions through backend access rules.
- **API configuration:** API keys are read from `local.properties` and included in the application build. Keep this file outside version control. For public distribution, handle privileged credentials through a backend or an appropriate provider-supported integration.
- **Notifications:** Care reminders use Android’s local alarm and notification services. Check reminder delivery and snooze behavior on your intended devices. Automatic restoration of scheduled reminders after a device restart is not currently included.
- **External services:** Species discovery, cloud storage, and AI assistance require network access and configured service accounts. Available data and responses depend on the providers’ access and usage limits.
- **Development checks:** The Gradle configuration includes JUnit and Espresso dependencies. Automated test sources are not currently included; the walkthrough above provides steps for checking the main features.
- **Care guidance:** Suggested care intervals can be adjusted to suit each plant. Review species information and AI suggestions alongside the plant’s condition and growing environment.
