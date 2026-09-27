# Vaxin: COVID-19 Vaccine Registration App (Android)

Vaxin is an Android app for managing COVID-19 vaccination in Bangladesh, built as a university project. Citizens register for a vaccine and check their vaccination date. Health workers with three levels of admin access manage registrations, assign dates and record doses. The app also bundles health information: live COVID-19 statistics, hospitals and ambulances by division, other vaccines, a BMI calculator and a news desk. The interface mixes Bangla and English.

**Tech stack:** Java · Android SDK · Firebase Realtime Database · Material Components · RecyclerView · WebView · DownloadManager

## Features

### For citizens

| Feature               | Description                                                                                                                                                                                                                           |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Registration          | Name, birth year, NID or birth certificate number, mobile number, division, priority category (front-line worker, doctor, teacher, student, MP, 18+ citizen) and a simple captcha. The app rejects duplicate NIDs and mobile numbers. |
| Login                 | Mobile number and password. Shows the vaccination date the admins assigned and downloads the vaccine card as a PDF.                                                                                                                   |
| COVID-19 status       | Live national statistics from corona.gov.bd in a WebView                                                                                                                                                                              |
| Important information | Hospitals and ambulances for a chosen division, loaded from Firebase                                                                                                                                                                  |
| Other vaccines        | Information and registration for rabies, routine immunisation, Japanese encephalitis and yellow fever                                                                                                                                 |
| BMI calculator        | Choose gender, height and weight to get your BMI and its WHO category                                                                                                                                                                 |
| News desk             | Five Bangladeshi newspapers (Prothom Alo, The Daily Star, Naya Diganta, Manab Zamin, Kaler Kantho)                                                                                                                                    |

### For administrators (PIN-protected roles)

| Role         | Can do                                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| High level   | Browse and edit registrations by category, assign vaccination dates, add hospitals and ambulances, view other-vaccine requests |
| Mid level    | Search vaccine takers by NID, by division or by vaccination date                                                               |
| Bottom level | Record first and second doses (vaccine name, date, division), view the list of assigned dates and other-vaccine requests       |

## How it works

Every screen is an `Activity` that reads and writes Firebase Realtime Database directly, and lists are shown with `RecyclerView` adapters in [`reg/`](app/src/main/java/com/mahirshadid/vax/reg). The database is organised as:

```
Database/<category>/<pushId>          full registration record
Reference/<nid>                       NID index, used to reject duplicate registrations
Reference2/<mobile>                   login credentials
ReferenceDivision/<division>/<nid>    registrations grouped by division
Datebase/<mobile>                     vaccination date assigned by an admin
vaccinetakers/<nid>                   doses given (vaccine, date 1, date 2, division)
datesearchdb/<d-m-yyyy>/<nid>         doses indexed by day, for date-wise search
H&A/<division>/<type>/<phone>         hospitals and ambulances
OtherVaccineDatabase/<vaccine>/<id>   other-vaccine requests
```

## Running the app

1. Open the project in **Android Studio** (Koala or newer) and let Gradle sync. It uses Gradle 8.7, AGP 8.5, compile SDK 34 and min SDK 21.
2. Create a project in the [Firebase console](https://console.firebase.google.com/), add an Android app with the package name `com.mahirshadid.vax`, and download its `google-services.json` into the `app/` folder. The file is git-ignored.
3. In Firebase, create a **Realtime Database**. Test-mode rules are enough for a demo.
4. Run the app on an emulator or device.

The admin PINs are hard-coded for the demo: `123` for high level, `1234` for mid level and `12345` for bottom level.

## Project history and fixes

The repository originally held only the `java/` and `layout/` folders of the project, so it could not be opened or built. It now has:

- **A standard Android Studio project.** Gradle build files, the wrapper, and an `AndroidManifest.xml` registering all 41 activities with the internet permission.
- **Missing resources.** Colours, a Material theme, shape backgrounds and Material Design vector icons. The original image assets were not in the repository, so the app logo and newspaper logos are placeholder icons.
- **List item layouts.** The seven RecyclerView item layouts the adapters inflate (`reg_items*.xml`) were missing and have been recreated from the adapter code.

Bugs fixed along the way:

- **Registration.** It wrote an extra database node keyed by the user's **password** into the login table. That node is gone.
- **Admin lists.** They displayed every user's password. They no longer do.
- **Crashes.**
  - The _call admin_ button used `ACTION_CALL` without the `CALL_PHONE` runtime permission and crashed on every tap. It now opens the dialer.
  - Date search, first-dose entry, second-dose entry and date assignment crashed with a `NullPointerException` when no date (or no NID search) had been made first. They now show a message instead.
  - Login crashed when a user record had no password.
- **BMI categories.** Values in the gaps between bands, such as exactly 16, 16.95, 25 or 29.5, all fell through to _Obese_. The bands now follow the WHO cut-offs (16 / 17 / 18.5 / 25 / 30), and the BMI is shown to one decimal place.
- **Validation.** The other-vaccine form's mobile and birth-year length checks used an impossible condition (`x > 11 && x < 11`), so they never ran.
- **Navigation.** The bottom-level admin's _date list_ card opened the wrong screen, which left the date list unreachable.
- **Assigned dates.** The date string was built before the dose number was chosen, so it could be saved as `null : 5/6/2021`.
- **Splash screen.** It stayed on the back stack after launch.

## Limitations

This is a student prototype, not production software:

- Passwords are stored in plain text in the database, and there is no Firebase Authentication.
- Admin access relies on fixed PINs inside the app.
- All validation happens on the client, so database security rules must be locked down before any real use.
- The _vaccine card_ download is a fixed PDF hosted on Google Drive, not a card generated for each person.
