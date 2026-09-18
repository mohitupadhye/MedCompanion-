# Medicine Companion

**A medication-literacy app for patients managing multiple prescriptions —
built to catch dangerous overlaps (like two brands unknowingly sharing the
same active ingredient) before they reach a doctor, and to explain
medicines in plain, non-clinical language for elderly and low-literacy
users.**

Native Android (Kotlin + Jetpack Compose), fully offline-first, built
around a real problem in Indian healthcare: without electronic medical
records, doctors often can't see a patient's full medication history in a
short consult, and patients frequently don't know they're duplicating an
ingredient across different brand names.

**⚠️ Project status: active personal project / prototype.** The app is
functional end-to-end (search, scan, interaction checking, persistence),
but the underlying clinical content — drug interactions, side effects,
severity ratings — is explicitly marked as pending clinical review and has
not yet been verified against an authoritative source. This is a portfolio
and engineering showcase, not a validated medical product.

**Core features:**
- Search by brand name, generic name, alternate/international name, or
  ingredient combination (e.g. "metformin + glimepiride")
- Camera-based strip scanning using on-device text recognition — no
  internet required
- Automatic duplicate-ingredient and drug-interaction detection, with
  color- and shape-coded severity (never color alone, for accessibility)
- Tracks chronic (ongoing) vs. temporary (short-course) medicines
  separately, so a finished 5-day antibiotic doesn't sit on the list
  forever
- A distinct, simplified "Doctor View" screen designed to be handed over
  in a consultation
- 100% offline — the full 78-generic / 200-brand reference database ships
  with the app

---

## Technical README (for anyone building or extending this)

database. This version adds synonym/combination search, camera-based strip
scanning, and a cleaner active/past medicines split.

## What's new in this version

- **Smarter search.** One search box now handles: brand name ("Dolo"),
  generic/INN name ("metformin"), alternate/international name
  ("acetaminophen", "Tylenol"), and combination search
  ("metformin + glimepiride" returns brands containing BOTH). A new
  `synonym` table in the database maps alternate names to their generic.
- **Scan the strip.** A camera screen (CameraX + on-device ML Kit text
  recognition) reads the printed name off a medicine strip and shows
  candidate matches to confirm — deliberately not auto-adding a guess,
  since OCR on foil packaging under real lighting is genuinely unreliable.
  Runs fully on-device, no internet required, consistent with the app's
  offline-first design. Camera permission is requested at runtime.
- **Active vs. Past medicines, cleaned up.** "Past medications" is now a
  single collapsed section (tap to expand) containing everything no longer
  being taken — finished short courses AND anything manually marked
  stopped, including a discontinued chronic medicine. This is a slightly
  different split than chronic-vs-temporary: a stopped chronic medicine
  belongs in Past too.
- **Honest sync scaffold added, not a working sync.** `RemoteCatalogSource.kt`
  documents the intended architecture for periodically refreshing the
  bundled catalog in the background when online — but there is currently no
  free, authoritative live data source to connect it to (confirmed by
  research: CDSCO doesn't expose one; commercial Indian drug databases
  exist but require a paid data license). The only implementation right
  now (`NoOpRemoteCatalogSource`) always reports nothing to sync. Wiring
  this up for real is future work, not something faked here.

## How to open it
1. Install Android Studio (latest stable).
2. `File > Open` and select the `MedCompanion` folder (the one containing
   `settings.gradle.kts`).
3. Sync. This version adds CameraX + ML Kit as new dependencies on top of
   Room from the last version — expect another meaningful download on
   first sync. Same troubleshooting playbook as before applies if anything
   goes red.
4. Run on an emulator (API 24+) or, for the scan feature specifically, a
   physical device is strongly preferred — most emulators either lack a
   usable virtual camera or provide only a low-quality synthetic feed that
   won't give OCR anything real to read.

## What's NOT implemented yet (honest gaps)
- **Sync is a scaffold, not a working feature** — see above. Nothing is
  currently fetched from the internet; the app is 100% offline in practice.
- **OCR match quality is unproven** — `matchOcrText` does simple substring
  matching between recognized text and known brand names. This needs real
  testing against real strips in real lighting before trusting it; expect
  to tune this once you see how it performs on an actual device.
- **No localization wiring in the UI** — schema supports it, `MedRepository`
  still hardcodes `lang = 'en'`.
- **No launcher icon design, placeholder severity shape** — same as before.
- **Content is still `needs_clinical_review = 1`** on every interaction row.
  Nothing here should reach a real patient until that review pass (tracked
  in `clinical_review.xlsx`) is done. Scanning and synonym search make the
  app easier to use — they don't make the underlying clinical data any more
  verified.
- **Only ~23 synonyms seeded** (common international/US brand names for
  drugs already in the 78-generic catalog). This is a starting set, not
  comprehensive — extending it is a data task, not a code change.

## Project structure (new/changed files marked)
```
MedCompanion/
  app/
    build.gradle.kts                        — now also includes CameraX + ML Kit
    src/main/
      AndroidManifest.xml                   — now requests CAMERA permission
      assets/meds.db                        — now includes a `synonym` table
      java/com/medcompanion/app/
        MedCompanionApp.kt                  — now routes to ScanScreen
        data/
          MedRepository.kt                  — search rewritten: synonym +
                                               combination + OCR matching
          RemoteCatalogSource.kt  [NEW]     — honest sync scaffold, no-op today
        ui/screens/
          AddMedicineScreen.kt              — now has a "Scan" button
          ScanScreen.kt           [NEW]     — camera + on-device OCR
          MyMedicinesScreen.kt              — Past Medications now collapsible,
                                               shows ALL inactive, not just recent
```

## Suggested next steps, in order
1. Build and run on a **physical device** if at all possible — the scan
   feature genuinely cannot be evaluated on most emulators.
2. Test the scan feature against real medicine strips in real lighting.
   Expect the OCR matching to need tuning — this is the least proven part
   of this version.
3. Test combination search ("metformin + glimepiride") and synonym search
   ("acetaminophen") against the seeded data.
4. When ready to pursue real online sync: evaluate the commercial Indian
   drug database providers found during research, or budget for building
   your own backend — `RemoteCatalogSource` is ready to receive a real
   implementation once one exists.
