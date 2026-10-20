# Attendance

An Android app that tracks class attendance. Set up your timetable, answer a quick "did you attend?" prompt when each class ends, and see your attendance percentage on the home screen. The app is a web page wrapped as an Android app with Capacitor, and GitHub builds it for you.

## Features

### Home screen
- A large attendance percentage, with the number of classes attended out of classes held.
- A target line (75% by default): how many more classes you can miss, or how many you need to attend, to reach the target.
- Today's classes with **Attended**, **Missed** and **Cancelled** buttons. A date picker lets you mark or change any other day.
- Cancelled classes are not counted as held.

### Timetable editor
- Add classes per weekday with a subject name, start time and end time.
- Delete classes. Deleted classes stop sending reminders.
- Today's list and the daily prompts are generated from this timetable.

### Class reminders
- A notification at the end of each class asks whether you attended.
- Reminders repeat weekly and work when the app is closed.
- They update whenever you change the timetable.
- Android may delay them by a minute or two. If they don't arrive, set the app's battery usage to "unrestricted" in phone settings.

### Attendance report update
Official attendance reports are issued from time to time, and your own count can drift from them. Enter the classes held, classes attended and the date the report covers. The app uses those numbers as a new starting point and counts only the classes you mark after that date.

### Google sign-in and sync
- Sign in with Google from the bottom of the home screen.
- Your timetable, marks, report numbers and target are stored in Firebase under your account and sync across devices.
- The app works offline and syncs when it can.
- Sync keeps the most recently changed copy. Editing on two devices while offline can overwrite one set of changes.

### Updates inside the app
- When the app opens, it checks the latest GitHub release. If a newer build exists, an **Update available** card appears.
- Tapping it downloads the new APK in the phone's browser. You then tap the file to install it. Android always asks you to confirm, so updates cannot be fully silent.
- The build number is shown at the bottom of the home screen.

### Push notifications
- Every install joins the `all` topic.
- The owner sends a message from the Firebase console and every phone receives it. If the app is open, it appears as a card on the home screen.

## How it works

| Part | What it does |
|---|---|
| `www/index.html` | The whole app: screens, attendance maths, sync, update check |
| `capacitor.config.json` | App name and package ID (`appId`) |
| `package.json` | Capacitor and Firebase plugins |
| `google-services.json` | Firebase project settings for the Android app |
| `.github/workflows/android.yml` | Builds, signs and publishes the APK on every commit |

Attendance is calculated as: (attended in the last report + attended since) divided by (held in the last report + held since). Every build gets the next version number, and the release is tagged `v<number>`.

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
6. Commit. The **Actions** tab builds the app, and the APK appears under **Releases** as `attendance.apk`.

## Releasing an update

1. Change files in the repository, for example `www/index.html`, and commit.
2. Wait for the green tick in **Actions**.
3. Installed apps show **Update available** the next time they open.

Always install new builds over the old one. Uninstalling first deletes local data. Cloud data is kept for signed-in users.

## Sending a notification to everyone

In the Firebase console open **Messaging**, create a campaign of type Firebase Notification messages, set the target to **Topic** `all`, and publish. The first message may take a few minutes to arrive.

## Troubleshooting

- **App closes on launch:** run a crash test (a GitHub workflow that runs the APK in an emulator) and read the `FATAL EXCEPTION` lines.
- **Sign-in fails with code 10 or 12500:** the signing key's fingerprint is missing in Firebase, or `google-services.json` was downloaded before Google sign-in was enabled.
- **Reminders don't arrive:** allow notifications and set battery usage to unrestricted.
- **Build fails:** open the red step in **Actions** and read the lines under "What went wrong".

## Privacy and security

- Signed-in users' attendance data is stored in your Firebase project, and the project owner can see it in the console. Tell other users if you share the app.
- `google-services.json` is public if the repository is public. This is normal for Firebase client settings. Access is controlled by the Firestore rules above, so keep them in place.
- Never commit the keystore or its password. They belong in repository secrets only.

## Known limitations

- Attendance is calculated overall, not per subject.
- The app is Android only.
- Updates always need a confirmation tap from the user.
- Sync is last-write-wins.
