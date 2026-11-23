# Study Schedule (Android, C++ core)

A student timetable app: add/edit/delete "pairs" (classes) per day, insert
windows (gaps), autocomplete room/subject names, keep a rolling 30-day
schedule with 7-day auto-cleanup, switch color palettes, and show today's
pairs in a home-screen widget.

## Why C++ *and* Kotlin

Android has no way to draw screens, home-screen widgets, or handle touch
input from pure C++ — there's no "C++ Android UI" in the way there's a
Win32 or Qt UI. The standard, and only real, way to use C++ on Android is
through the **NDK**: C++ owns your real logic, and a thin Java/Kotlin layer
talks to the OS and draws the screen. That's what this project does:

| Layer | Language | Owns |
|---|---|---|
| `app/src/main/cpp/` | **C++17** | Every rule in the spec: adding/editing/deleting pairs, sorting by time, windows, the 30-day window, the 7-day auto-purge, autocomplete history, and all file storage (plain tab-separated files, one per day — same style as your Women's Safety C project). |
| `app/src/main/java/.../ScheduleNative.kt` | Kotlin | A thin JNI facade — no logic, just calls into C++. |
| Everything else in `java/` and `res/` | Kotlin + XML | Screens, navigation, the widget, and the color-palette switcher. This is boilerplate Android, not scheduling logic. |

If anyone asks "is this a C++ app" — yes, in the sense that matters: open
`schedule_core.h/.cpp` and `jni_bridge.cpp` and that's the entire brain of
the app. The Kotlin files are intentionally dumb plumbing.

## Project layout

```
StudySchedule/
├── app/src/main/cpp/          C++ core (schedule_core.*, jni_bridge.cpp, CMakeLists.txt)
├── app/src/main/java/...      Activities, adapter, widget
├── app/src/main/res/          Layouts, strings, color palettes, widget XML
├── app/src/main/AndroidManifest.xml
├── app/build.gradle
├── build.gradle
└── settings.gradle
```

## Feature → file map

- Add/edit/delete pairs → `schedule_core.cpp` (`addPair`/`editPair`/`deletePair`), UI in `AddEditPairActivity.kt`
- Room/name/type/start/end per pair → `Pair` struct in `schedule_core.h`
- Windows (gaps) → `addWindow()`, toggle switch in the add/edit screen
- Sorting by time → `Day::sortByTime()`, called after every change
- 30-day schedule / 7-day auto-delete → `purgeOlderThan()` + `enforceRollingWindow()`, triggered from `MainActivity.onCreate()`
- Day navigation → prev/next buttons in `MainActivity.kt`
- Autocomplete by prefix → `suggestNames()`/`suggestRooms()` in C++, wired to the `AutoCompleteTextView`s
- Color palette → `SettingsActivity.kt` + `themes.xml` (4 palettes; easy to add more)
- Desktop widget → `ScheduleWidgetProvider.kt` + `widget_schedule.xml`

## Building it

This needs the Android SDK + NDK, which aren't available in this chat's
sandbox (no network, no Android build tools installed here), so I wrote
the full source but couldn't compile or run it myself — treat it as a
complete first draft to build and debug on your own machine or in Acode.

1. Install **Android Studio** (easiest — it bundles the NDK and CMake), or
   if you're doing this on-device in **Acode**: install the NDK and CMake
   via its companion toolchain (Acode itself edits files; building still
   needs `gradle` + NDK available on-device, e.g. through Termux).
2. Open the `StudySchedule/` folder as a project.
3. Let Gradle sync — it will prompt to install the NDK/CMake if missing.
4. Run on a device/emulator (minSdk 24, Android 7.0+).

## Known rough edges to tighten next

- No app icon is included yet (`@mipmap/ic_launcher` — Android Studio can
  generate this for you via *New > Image Asset*).
- The widget redraws on a timer (`updatePeriodMillis`) and on app resume;
  if you want it to update the instant a pair is saved, send a manual
  `AppWidgetManager` broadcast from `AddEditPairActivity` after saving.
- Only 4 palettes are wired up; add more by adding a style to `themes.xml`
  and a row to the `palettes` list in `SettingsActivity.kt`.
