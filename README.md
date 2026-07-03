# Sairam Institutions — Smart Driver Trip Sheet (Android Scaffold)

A Kotlin + Jetpack Compose scaffold digitizing the **College Owned Car Trip Sheet** paper form
used by Sai Ram Group of Institutions, built per the reference photo and full feature spec.

## Stack
- **UI:** Jetpack Compose + Material 3, MVVM
- **DI:** Hilt
- **Local DB:** Room (offline-first — every screen writes locally first)
- **Networking:** Retrofit + OkHttp (stubbed, points at a placeholder `BASE_URL`)
- **Auth backend:** Firebase Auth/Firestore dependencies included but not wired — `AuthRepository`
  currently does a **local Room lookup** so you can build/test the full flow before a backend exists
- **Location:** FusedLocationProviderClient (`utils/LocationHelper.kt`)
- **Photos:** CameraX dependency added; capture buttons are wired as `TODO` stubs — see below
- **Permissions:** Accompanist Permissions for runtime GPS/camera requests
- **Branding:** Sairam logo copied to `app/src/main/res/drawable/sairam_logo.png`, blue (#0D47A1)
  + green (#4CAF50) theme in `ui/theme/Color.kt` matching the logo

## What's implemented (v1 scope you picked)
1. **Login + Driver Dashboard** — Staff ID/password login (default `Welcome@123`), routes drivers
   to their dashboard and `ADMIN001` to the admin view. Dashboard shows assigned vehicle, status,
   today's trip count, and 6 quick-action tiles.
2. **Start/End Trip with GPS + odometer** — `StartTripScreen`/`EndTripScreen` capture a GPS fix
   + reverse-geocoded location name, starting/closing KM (closing KM validated against starting
   KM), vehicle condition, and auto-calculate total KM/duration/avg speed (`Trip.kt` computed
   properties) — directly mirroring the paper sheet's Starting/Closing/Total Km columns.
3. **Fuel filling entry** — Pump name, current KM, litres, cost, bill number, payment method,
   GPS tag — mirrors the paper sheet's "...Ltrs. / Rs...." fields plus the spec's extra fields.
4. **Admin analytics dashboard** — KPI grid (vehicles, drivers, running trips, fuel cost today,
   active/inactive drivers, monthly KM) + a dependency-free Compose Canvas bar chart of the last
   7 days' distance, so you're not locked into MPAndroidChart/Vico this early.

Also scaffolded: Trip History list, Room schema for Driver/Vehicle/Trip/FuelEntry, a photo
watermark utility (`utils/PhotoWatermark.kt`) that stamps date/time/GPS/vehicle/driver/trip
onto captured images per the spec's Camera Features section.

## Explicitly NOT built yet (out of scope for this pass)
This request is a large, genuinely enterprise-grade multi-platform spec. To keep this scaffold
honest and actually working, these were **left as clearly marked `TODO`s** rather than faked:
- iOS app, Admin **Web** Dashboard (separate codebases)
- Real backend (Firebase project / Supabase / Postgres + JWT) — `NetworkModule.kt` and
  `AuthRepository` have TODOs marking exactly where to plug it in
- CameraX capture UI (buttons exist, wired to `TODO` — camera preview + capture flow is its own
  chunk of work)
- Offline→online sync worker (Room already tracks `isSynced`, needs a WorkManager job)
- Push notifications, biometric login, QR codes, AI mileage/fuel-anomaly detection, geofencing
- PDF/Excel export, admin driver CRUD screens, vehicle maintenance module

## Running it
This was written as source (no network access in this sandbox to actually run a Gradle build),
so before opening in Android Studio:
1. `File > Sync Project with Gradle Files`
2. Add a `google-services.json` from your Firebase console into `app/` and un-comment the
   `com.google.gms.google-services` plugin in `app/build.gradle.kts` once you're ready to wire
   Firebase Auth in for real (right now login works fully offline against Room).
3. Seed a test driver + vehicle — easiest way is a small `DebugSeeder` that calls
   `driverRepository.save(...)` / `vehicleRepository.save(...)` once on first app launch, or
   insert rows via Android Studio's App Inspection → Database Inspector.

## Suggested next build
Given the four features you picked are in, the next highest-value chunk is probably either:
(a) the CameraX capture screens (dashboard meter / vehicle photo / fuel bill), or
(b) the sync worker + a minimal REST backend so multi-device / admin data isn't just per-phone Room.
