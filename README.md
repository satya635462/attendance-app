# Attendance

An Android app that tracks class attendance. Set up your timetable, answer a quick "did you attend?" prompt when each class ends, and see your attendance percentage on the home screen. The app is a web page wrapped as an Android app with Capacitor, and GitHub builds it for you.

The app has four tabs: **Today**, **Timetable**, **Graph** and **Report**.

## Features

### Home screen (Today tab)
- A large attendance percentage, with the hours attended out of hours held.
- A target (75% by default, changeable in the Report tab) and a line telling you how many more classes you can miss, or how many you need to attend, to reach it.
- **Low-attendance warning:** when you are below your target, a red card shows how many classes in a row you need to attend to get back to it, and roughly by which date based on your timetable.
- Today's classes with **Attended**, **Missed** and **Cancelled** buttons. A date picker lets you mark or change any other day. Cancelled classes are not counted as held.
- **Mark the whole day** with one tap: Whole day present, Whole day absent, Whole day cancelled, or Clear. If some classes are already marked differently, the app asks before replacing them.

### Timetable editor
- Add classes per weekday with a subject name, start time and end time.
- Tick **Lab session** for a lab. A lab counts as 2 hours: attending it adds 2 attended and 2 held, and missing it adds 2 held. Labs show "lab (counts as 2)" in the lists. Marks you made earlier keep their weight even if you later delete the lab.
- To change a class, delete it and add it again.
- Deleted classes stop sending reminders. Today's list and the daily prompts come from this timetable.

### Class reminders
- A notification at the end of each class asks whether you attended.
- Each reminder has **Attended**, **Missed** and **Cancelled** buttons. Tapping one records the answer for that class (for the most recent time it was held, up to a week back), and the app shows a confirmation card. On Android, tapping a button may open the app briefly to record it.
- Reminders repeat weekly, work when the app is closed, and update whenever you change the timetable. The app replaces its scheduled reminders each time it opens.
- Android may delay them by a minute or two. If they don't arrive, allow notifications and set the app's battery usage to "unrestricted" in phone settings.

### Attendance graph
The **Graph** tab shows a line chart of your running attendance percentage, with a dashed line at your target. The line is green above the target and red below it.
- Switch between the last 30 days and all time.
- Each point is your total after that day. The starting point is your latest report, and later points come from the classes you mark.
- A **By day** list below shows each day's result and the running percentage.
- Days with no marks, and fully cancelled days, don't appear. You need at least two days of data before the line shows.

### Attendance report update (Report tab)
Official attendance reports are issued from time to time, and your own count can drift from them. Enter the classes held, classes attended and the date the report covers. The app uses those numbers as a new starting point and counts only the classes you mark after that date. You can also change your target here.

### Google sign-in and sync
- Sign in with Google from the bottom of the home screen.
- Your timetable, marks, report numbers and target are stored in Firebase under your account and sync across devices.
- The app works offline and syncs when it can.
- Sync keeps the most recently changed copy. Editing on two devices while offline can overwrite one set of changes.

### Updates inside the app
- When the app opens, it checks the latest full GitHub release. If a newer build exists, an **Update available** card appears. Pre-releases are ignored, so only releases you have promoted are offered.
- Tapping the card downloads the new APK in the phone's browser. You then tap the file to install it. Android always asks you to confirm, so updates cannot be fully silent.
- The build number is shown at the bottom of the home screen.

### Push notifications
- Every install joins the `all` topic.
- The owner sends a message from the Firebase console and every phone receives it. If the app is open, it appears as a card on the home screen.

## How it works

| Part | What it does |
|---|---|
| `www/index.html` | The whole app: screens, attendance maths, graph, sync, update check |
| `capacitor.config.json` | App name and package ID (`appId`) |
| `package.json` | Capacitor plugins: local notifications, app info, browser, Firebase authentication and messaging |
| `google-services.json` | Firebase project settings for the Android app |
| `.github/workflows/android.yml` | Builds and signs the APK when you run it, and publishes it as a pre-release |

At build time the workflow also fills in your repository name for the update check, writes your Firebase project ID into the app from `google-services.json`, and adds the notification permission and Firebase sign-in settings.

Attendance is calculated in hours, where a normal class is 1 and a lab is 2. It is calculated as: (attended in the last report + attended since) divided by (held in the last report + held since). Every build gets the next version number, and the release is tagged `v<number>`.

## Setting it up from scratch

1. **Repository.** Create a GitHub repository and upload the files above. The workflow file lives at `.github/workflows/android.yml`. If your computer won't upload the hidden `.github` folder, create the file on GitHub with **Add file, Create new file**.
2. **Package ID.** Choose `appId` in `capacitor.config.json` once. Each part must start with a letter. Changing it later makes Android treat it as a different app.
3. **Signing key.** Generate a keystore (alias `attendance`) and add two repository secrets: `KEYSTORE_BASE64` (the keystore, base64 on one line) and `KEY_PASSWORD`. Keep a copy of the keystore outside GitHub. If it is lost, installed copies can never be updated.
4. **Firebase.**
   - Create a project and add an Android app with the same package ID.
   - Add the signing key's SHA-1 and SHA-256 fingerprints.
   - Enable Google sign-in and create a Firestore database.
   - Download `google-services.json` after the steps above and place it in the repository root.
   - Publish these Firestore rules:
     ```
     rules_version = '2';
     service cloud.firestore {
       match /databases/{database}/documents {
         match /users/{uid} {
           allow read, write: if request.auth != null && request.auth.uid == uid;
         }
       }
     }
     ```
5. **Settings.** Under repository Settings, Actions, General, set Workflow permissions to read and write. Make the repository public if you want in-app update checks to work.
6. **First build.** Builds only run when you start them. Open **Actions**, choose **Build APK**, and click **Run workflow**. The APK appears under **Releases** as `attendance.apk`.

## Releasing an update

Builds are published as pre-releases first, so you can test before anyone else gets them.

1. Change files in the repository, for example `www/index.html`, and commit. Nothing is built yet, so you can make as many edits as you like.
2. Open **Actions**, choose **Build APK**, and click **Run workflow**. Wait for the green tick.
3. Open **Releases**. The new build is marked **Pre-release**. Download `attendance.apk` and install it over your current app. Other users are not offered it.
4. Test it on your phone.
5. If it is good, promote that same build: open the release, click the pencil (Edit), untick **Set as a pre-release**, tick **Set as the latest release**, and click **Update release**. Users' apps then show **Update available** the next time they open.
6. If it is not good, fix it and run the build again.

Notes:
- Always install new builds over the old one. Uninstalling first deletes local data. Cloud data is kept for signed-in users.
- Android will not let you install an older build over a newer one. Promote the newest build you tested, or a later one.
- Test builds use the same Firebase data as the real app, so be careful when testing changes to sync.

## Sending a notification to everyone

In the Firebase console open **Messaging**, create a campaign of type Firebase Notification messages, set the target to **Topic** `all`, and publish. The first message may take a few minutes to arrive. Close the app first if you want to see it as a normal notification, because an open app shows it as a card instead.

## Troubleshooting

- **App closes on launch:** run a crash test (a GitHub workflow that runs the APK in an emulator) and read the `FATAL EXCEPTION` lines. The crash test downloads the latest full release, so to test a pre-release give it that release's tag, for example `v25`.
- **Sign-in fails with code 10 or 12500:** the signing key's fingerprint is missing in Firebase, or `google-services.json` was downloaded before Google sign-in was enabled.
- **Reminders don't arrive:** allow notifications and set battery usage to unrestricted.
- **Build fails:** open the red step in **Actions** and read the lines under "What went wrong".
- **Warnings in the Sign APK step:** Java warnings from the signing tool are usually harmless. If the step has a green tick, the APK is fine.
- **Firebase Dynamic Links notice:** it concerns email-link and Cordova sign-in, which this app doesn't use, so no action is needed.

## Privacy and security

- Signed-in users' attendance data is stored in your Firebase project, and the project owner can see it in the console. Tell other users if you share the app.
- `google-services.json` is public if the repository is public. This is normal for Firebase client settings. Access is controlled by the Firestore rules above, so keep them in place and confirm they are not in test mode.
- Anyone who controls the GitHub repository can push an update that users are offered, so turn on two-factor authentication for GitHub and for the Google account that owns the Firebase project.
- Never commit the keystore or its password. They belong in repository secrets only.
- Push messages are not private. Don't put anything sensitive in them.

## Known limitations

- Attendance is calculated overall, not per subject.
- A lab always counts as 2 hours.
- Classes can't be edited, only deleted and re-added.
- Reminder buttons may open the app briefly instead of working fully in the background.
- The app is Android only.
- Updates always need a confirmation tap from the user.
- Sync is last-write-wins.

## Ideas not built yet

Per-subject attendance, a holiday calendar, editing classes, a list of unanswered past classes, CSV or PDF export, timetable sharing, an app lock, and moving the data from Firestore to another database such as Supabase.
