# MediTrack - Design & Implementation Spec

> **Single source of truth.** Every decision or change goes in this document. Agents implement from this file.
> Status: **DRAFT** - planning in progress. Anything marked `OPEN` or `RESEARCH` is not decided; do not implement it until resolved. Ask Or.
> "MediTrack" is a temporary working name (see Section 9).

---

## 0. First Task for the Implementing Agent: Audit and Fix the Project Structure

The repo (`github.com/or-dvir/MediTrack`) contains only the project Android Studio generated when it was created (plus a README, `.gitignore` and `LICENSE`). **Assume nothing is set up and nothing matches this spec.**

Before any feature work:
1. Read this entire spec, the global `rules.md`, and the relevant skills (see Section 8).
2. Do a full scan of the project (Gradle files, version catalog, manifest, package names, themes, dependencies, source sets, tests).
3. Fix everything needed to bring the project in line with this spec, the global rules, and the skills. The agent decides what needs fixing; this spec does not list it.
4. Where the spec is silent, unclear, or marked `OPEN`, ask Or. Do not guess.
5. Things the agent cannot do itself (for example, Firebase projects and their `google-services.json` files) must be reported to Or as prerequisites; never commit `google-services.json`.
6. Verify the result builds with `./gradlew assembleDebug`.

---

## 1. Overview and Priorities

- Personal Android app, built from scratch, to track prescriptions and health-related products across multiple family member profiles, including pets.
- Personal-use tool with no release deadline, so modern APIs and clean architecture are favored over broad compatibility.
- **Speed matters.** The app must feel snappy and responsive. It is not the number one priority, but it must not be neglected for other goals.
- **Safety matters most.** Wrong dosage information can cause harm. See Section 2.9 and Section 3.
- Pure Android (no KMP). Native development in Android Studio.

---

## 2. Features

Status tags: **[MVP]** in the minimum viable product; **[MVP, low priority]** in the MVP but built last and the first to be dropped if the MVP grows; **[FUTURE]** later; **[RESEARCH]** needs research before anything is built.
Or has decided the MVP vs future split for every feature below.

General rules:
- Everything the user creates (profiles, items, history entries) must be editable and deletable.
- Do not add fields "for later" just because it is cheap. Only store data that has a use in the current scope.

### 2.1 Profiles
- [MVP] Single account with multiple sub-profiles (family members and pets), each with a profile picture.
- [MVP] Birth date, **optional** (people may not want to provide it). It never changes and is easy to use, for example to show age. Later it can drive dosage selection.
- [MVP] Default/favorite profile: the app opens straight into it instead of the profile selection screen. UI details (how many favorites, separate from the others vs highlighted or sticky at the top) are decided during UI design.
- Weight, **optional**: not in the MVP. It needs tracking over time and how to do that is `OPEN`.
- [FUTURE] Archive profiles. The archive structure is undecided (per-element archive vs one unified archive screen).

### 2.2 Items (medicines / health products)
- [MVP] Fields: name, description, tags, max daily dosage, multiple pictures.
- [MVP] **Dosage must not be a single value.** It must accommodate multiple conditions, for example "1-2 years: 3ml, 3-5 years: 5ml", by age and/or weight. Frequency can also differ per condition, so dosage and frequency are not necessarily separate fields. The data structure and UI component are `OPEN`. Purpose: avoid dosage mistakes (for example, a child got older and the item was not updated).
- [MVP] Active/inactive state. Inactive items are dimmed and placed at the bottom of the list.
- [MVP] Default sorting is "what I should take next (and when)". This depends on structured frequency data, so feasibility depends on the dosage structure (`OPEN`).
- [FUTURE] Archive items: hide an item but keep its data. An active item cannot be archived, and an archived item is automatically unarchived when set to active. Design note (proposed, not yet confirmed): model active/inactive/archived as one status value instead of separate flags.
- [FUTURE] Opened date and expiry date.
- [FUTURE] Priority/highlight for important items (for example, antibiotics): shown on top, color or other emphasis, stronger notifications. Judgment call kept for now; revisit when implementation details are decided.
- [FUTURE] Favorite items (shortcuts).
- [FUTURE, reconsider] Categories: user-named groups of items inside a profile (for example, "fever reducer"), so the structure becomes profile -> category -> items. Can also be created after the fact by selecting existing items or adding them to an existing category. Tags (MVP) may serve the same purpose, so reconsider before building.

### 2.3 History
(Each time an item is logged as taken.)
- [MVP] Log an item as taken with as few taps as possible. The mechanism is decided during UI design.
- [MVP] Full history of every logged entry per item.
- [MVP] Show last-taken and next-available time per item. Proposed, not yet confirmed: calculate these from the history entries instead of storing them, so edits and deletions correct them automatically.
- [MVP] Edit, delete, and backdate entries. Backdating means adding a manual "taken" entry at a past date and time. Logging in the future is **disabled**. No "edited" marker.
- [FUTURE] Dose warnings: ideally shown before logging if the dose would exceed the max daily dosage or is too soon. The user can still log it; the entry is then flagged (for example, "over limit"). Details when this feature is planned.

### 2.4 Search
- [MVP] Global search across the whole account (not limited to one profile), reachable without extra steps, by name and tag.
- [MVP] Search filters: profile (certain), tag. Other filters are decided when search is designed.
- [FUTURE] Combine search with an archive view (idea, depends on archive).

### 2.5 Account, sync and settings
- [MVP] Firebase Authentication with email verification.
- [MVP] Password reset.
- [MVP] Account deletion: the user can delete their account and all their data from Firebase.
- [MVP] Multi-device sync of all data, offline-first. Firestore provides most of this; images in Firebase Storage need their own loading/caching design.
- [MVP] Disclaimer shown to the user, who must accept it (see 2.9).
- [MVP] Theme selection: system default, always light, always dark.
- [MVP, low priority] App lock (biometric), toggled in settings. If the basic version turns out not to be easy, move it to future.
- [FUTURE] [RESEARCH] Client-side encryption of all data (possibly optional). Note: Firestore already encrypts data at rest and in transit; this would be client-side encryption.

### 2.6 Images
- [MVP] Images are uploaded to Firebase (Firebase Storage; Firestore cannot hold images) so they sync across devices.
- [MVP] Profile pictures **must** be encrypted before upload (they can show people's faces, including children).
- [MVP] Multiple pictures per item, encrypted with the same logic. Whether encryption is a global setting or a per-item/per-profile choice is decided later.
- Rejected alternative: a pointer to a local device image (simpler, but breaks multi-device sync).

### 2.7 Notifications
- [FUTURE] Reminders with options: completely off; only when the item is actively taken; at a specific preset time; custom times. Toggle per item, per profile, or globally (exact structure `OPEN`). Some medicines must be taken at the same time each day; take this into account.
- [FUTURE] Mark an item as taken directly from the notification.
- [FUTURE] Priority items get stronger notifications (for example, non-dismissable). Android may limit non-dismissable notifications; verify before relying on it.
- [FUTURE] Launcher shortcut (Android "app shortcuts", long-press the app icon) to log a dose.

### 2.8 Other future features
- [FUTURE] Export history. Import data.
- [FUTURE] Cross-account family sharing.
- [FUTURE] Multi-language support. Translations are future, but the code must use string resources from day one (no hardcoded user-facing strings).
- [FUTURE] [RESEARCH] Medicine information from official REST APIs, and barcode scanning to auto-fill an item's details. See 2.9 for requirements.

### 2.9 Safety, disclaimer and external data
- The app only helps the user organize things. It is **not medical advice** and the app's author takes no responsibility for what the user does based on it. A disclaimer the user must accept is required (MVP).
- Any external data source (REST API or similar) **MUST be official and reliable** (health authorities such as the FDA or the European equivalents, or the manufacturer). No random open-source libraries or unofficial sources. Several sources, with the user choosing the one relevant to their country, would be best. Which sources exist and are usable is `RESEARCH`.
- If the app ever recommends or highlights a dose (for example, a dosage row matched to a profile's age or weight), it must show a warning and keep on-screen information minimal, so the user is not confused into taking a wrong dose.
- Regulation: an app that provides medical or dosing information may fall under medical-device regulation. This may apply even if the app is only distributed through GitHub and not an app store. `RESEARCH` before relying on any external medical data. Idea to evaluate later: link users to the official source (for example, the FDA page for a medicine) instead of showing the information in the app. Whether that avoids the problem is unknown.

---

## 3. Tech Stack

| Concern | Decision |
|---|---|
| Language | Kotlin, Coroutines + Flow |
| UI | Jetpack Compose, Material Design 3 |
| Architecture | MVVM |
| Dependency injection | Koin |
| Auth | Firebase Authentication (email verification, password reset) |
| Database | Firebase Firestore (offline-first, multi-device sync). **Revisit after the spec is complete and before implementation starts** (is a document store the right fit vs a DB-style product?) |
| File storage | Firebase Storage (profile pictures encrypted client-side) |
| Local DB | None (no Room) |
| Networking | REST APIs are wanted eventually (official sources only, see 2.9). Not in the MVP (future, research needed). Library choice `OPEN`. |
| Build config | Gradle Version Catalog (`gradle/libs.versions.toml`), BOMs where available |
| minSdk | 30 (Android 11, about 87% device coverage) |
| compileSdk / targetSdk | Latest (35 or 36) |

### Project structure
- Flat single module (small project). No feature modules.

---

## 4. Conventions Inherited from Global `rules.md`

Source: `github.com/or-dvir/agents-config/rules.md`.

**Conflict rule:** if this spec or any task contradicts `rules.md`, ask Or. A spec entry may override `rules.md` only after Or has approved it, and the entry must say so (for example, "approved by Or").

- Base package: `com.hotmail.or_dvir.meditrack` (may change if the app is renamed; ask before renaming).
- `debug` build type uses `applicationIdSuffix = ".debug"` and `versionNameSuffix = "-DEBUG"`.
- Separate Firebase projects: `app/src/debug/google-services.json` (dev) and `app/src/release/google-services.json` (prod).
- Compose + Material 3 only. No XML layouts, Views, or Fragments.
- `enableEdgeToEdge()` in `MainActivity`; respect scaffold `innerPadding` / `WindowInsets`.
- Theme: System Default / Always Light / Always Dark, persisted with Jetpack DataStore.
- Material 3 expressive styling: tonal containers (`surfaceContainer*`), tinted icon badges, `tertiary` accents. Approved by Or.
- Experimental APIs: a handful is acceptable, but the project must not be flooded with them. The agent lists each experimental API it uses; anything beyond a handful needs Or's approval. Whether to adopt the actual Material 3 Expressive APIs is `OPEN`.
- `ScreenState` data class + `ScreenAction` sealed interface + `onAction(action)` in the ViewModel; state collected with `collectAsStateWithLifecycle()`.
- One-time UI events via `Channel<UiEvent>` only when the ViewModel must push a side effect.
- Previews: combine component states in one preview. Full screens get previews for all relevant states: content, loading, empty, error, and any other relevant state (for example, a single profile vs multiple profiles). Include edge cases such as enough items to fill the screen, so long and short lists can both be seen.
- Gradle properties: `org.gradle.caching=true`, `org.gradle.parallel=true`, `org.gradle.configuration-cache=true`.
- A project-root `AGENTS.md` must exist and inherit global rules (see Section 8).
- Do not refactor existing code unprompted. The agent may suggest changes to existing code; Or explicitly approves or denies each one.

---

## 5. Design Principles

- Modern and not boring: use icons, color, borders, cards and subtle motion so screens do not look like bare default components (for example, two plain text fields on a login screen).
- Within reason: not everything flashy or multicolored, and no composable used just for variety. Simple is sometimes better than flashy. The agent designing the UI judges this case by case, with Or.
- Animations are short. Nothing jumps around, and the user never waits on long animations.
- Fewest possible taps for frequent actions, especially logging an item as taken.

---

## 6. Architecture

- MVVM.
- `OPEN`: whether to adopt the `android-presentation-mvi` skill conventions (State / Action / optional Event, Root + Screen composable split in one file, UI models suffixed `Ui`, `UiText` for resource-based strings).
- `OPEN`: package layout inside the single module.

---

## 7. Data Model and Image Encryption

### 7.1 Data model (Firestore)
`OPEN` - not yet designed. To decide:
- Collections and document structure (account, profiles, items, intake log entries).
- Intake log storage (subcollection under each item vs top-level collection).
- How multi-condition dosage is stored (see 2.2).
- Global search across all profiles of the account (client-side over cached data is the working assumption; Firestore has no full-text search).
- MVP fields to cover: active/inactive state, optional profile birth date, default-profile marker. Future features (archive, opened/expiry dates, weight, categories) must not be stored yet, but the model should not make them hard to add.
- Security rules.

### 7.2 Image encryption
`OPEN` - to decide:
- Key management and where the key lives.
- How multiple devices (and later, family members) obtain the key.
- Shared encryption logic for profile and item pictures.

---

## 8. Agent Setup

- Project-root `AGENTS.md` required, following the template in global `rules.md` (identity, tech stack and data sources, key features and screens).
- Related skills live in `github.com/or-dvir/agents-config` (`skills/`). Some skills assume Room, Ktor, REST, multi-module structure, or KMP. **Those parts do not apply to MediTrack.**
- **Read before implementing:** global `rules.md` and the skills in `github.com/or-dvir/agents-config/skills` that are relevant to the work. Where a skill contradicts this spec (or `rules.md`), **stop and ask Or to decide**. Neither side wins automatically. Once Or decides, record the decision in this spec.
- Skill conventions that are **not decided yet** (see Section 9) must not be applied until decided: error handling and navigation.
- Several skills reference skills that are not in the repo (`android-background`, `android-compose-architecture`, `figma-to-android-vector`, etc.). Handle when relevant.

---

## 9. Open Decisions / To Revisit

| Topic | Status |
|---|---|
| Firestore vs DB-style alternative | Revisit after spec is complete, before implementation |
| Error handling (Kotlin `Result<T>` vs custom typed `Result<D, E>`) | Deferred; decide when error handling is discussed |
| Navigation (type-safe Navigation Compose vs Navigation 3) | Deferred until after the spec is complete |
| Firestore data model | Open |
| Image encryption design | Open |
| Dosage structure (multiple conditions, frequency per condition) and its UI component | Open |
| Dose recommendation by profile age/weight, and its warning design | Open, needs careful design |
| Dose warnings (before logging, flag when over limit) | Details when planned |
| Notifications design | Open (Future feature) |
| Priority/highlight design | Open |
| Official APIs and barcode scanning: which sources exist | Research needed |
| Regulation (medical-device rules, also for GitHub-only distribution); link-to-official-source idea | Research needed |
| UI screens and navigation flow | Open |
| Auth flow and Firestore security rules | Open |
| Testing approach | Open |
| Image loading library | Open |
| Adopt actual Material 3 Expressive APIs | Open |
| App name (MediTrack is temporary); package and applicationId depend on it | Open |
| Weight tracking over time (optional profile weight) | Open, not in MVP |
| Categories vs tags | Reconsider before building (future) |
| Priority/highlight | Revisit when implementation details are decided |
| Default profile UI (how many, sticky vs separate) | Decided during UI design |
| Quick "log as taken" mechanism | Decided during UI design |
| Client-side encryption of all data | Research (future) |
| Further features | May be added, removed or changed later |

---

## 10. Change Log

- Initial draft created from planning decisions so far.
- Spec moved into the MediTrack GitHub repo at docs/SPEC.md (public). The repo copy is the source of truth.
- Added Section 0: repo contains only the Android Studio generated project; first agent task is a full audit and fix against this spec, global rules and skills.
- Conflict rule: when a skill contradicts the spec, the agent asks Or to decide instead of assuming either side wins.
- Major update: features rewritten as a tagged list (MVP / FUTURE / UNDECIDED / RESEARCH) with many new features; removed owner location; added speed and safety priorities, design principles, disclaimer and regulation notes; images uploaded to Firebase; REST APIs wanted eventually; password reset and account deletion added; rules.md conflicts require asking Or; renamed "dose history" to "intake log" and "daily dosage enforcement" to "dose warnings".
- MVP vs future split completed for all features. Renamed "intake log" to "History". Optional birth date added (MVP); weight left open; opened/expiry dates, archive, priority, favorites, categories and dose warnings moved to future; app lock set to MVP low priority; client-side encryption of all data added as future research.
