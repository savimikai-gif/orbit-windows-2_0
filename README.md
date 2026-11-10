# ORBIT 3.1 — Voice + Notes + Windows Control

**Architecture:** .NET 10 / WinForms host + WebView2 HTML/CSS/JS UI. No Firebase, Supabase, user cloud accounts, Google News or sign-in screen.

## What is in this build

- Animated reactive assistant orb (idle, wake armed, listening, speaking); motion-respectful CSS and microphone controls.
- Native opt-in wake grammar for "Hey ORBIT" plus push-to-talk. Built with `System.Speech` and installed Windows recognition language support. Native voice event messages are delivered to the bundled WebView2 UI.
- Voice commands: volume percent, mute/unmute, brightness percent where supported, open Notepad/Calculator/Explorer/Windows Settings, show/hide monitor overlay, open Notes/Reminders/System, create/search/read simple notes.
- Dedicated Notes workspace inspired by the simple two-pane notes-app model: new note, edit, autosave, timestamp, pin, delete, search, microphone dictation. Notes live under `%LOCALAPPDATA%\ORBIT\notes.json`.
- Windows system controls in native C#: NAudio endpoint volume, WMI brightness, fixed allowlisted apps, CPU/RAM monitoring. Existing reminder/tray notifications and user-presets remain.
- Optional draggable, always-on-top corner performance panel.
- GitHub Releases update checking, verified-download flow (SHA-256 sidecar file), user-approved installer launch.
- Existing reminders, timers, themes and profiles continue using persistent WebView2 local storage under `%LOCALAPPDATA%\ORBIT\WebView2` (no data migration).

## Build

Read **START_HERE.txt** for the shortest GitHub Actions route. The `.github/workflows/build-windows.yml` workflow requires `Native`, `Web`, `Assets`, and `Installer` at repo root, builds `Publish/ORBIT.exe` and `Release/ORBIT_3_1_Setup.exe`, produces a SHA-256 sidecar and uploads all 3 files to the Actions artifact.

## Privacy and operational limitations

Speech recognition is done via Windows installed recognizers. The always-on mic is disabled by default and controlled by an explicit switch; opt out in Settings or tray menu. The system does not intentionally upload continuous microphone audio. Language recognition support varies with Windows installation; test with Windows microphone permissions, en-US recognition installed, and optional ro-RO recognition if available.

**AI neural voice status:** This source build ships with Windows TTS fallback and can select natural voices when available among the voices Windows exposes. It does **not** yet offer a hosted neural TTS service to all users: doing so requires a secure developer-operated endpoint to protect API credentials and cover synthesis costs. You should not publish your own Azure key inside `ORBIT.exe`, and there is no magic free public key.

The updater checks a configured public GitHub repository; installer files require matching `.sha256` release assets. SHA-256 does not authenticate a publisher. For wide distribution, digitally sign installers and use a dedicated release pipeline. The update UI is user-confirmed, not an unattended updater.

Windows-only C# code was prepared but **not compiled or hardware-tested** in the environment used to create this source ZIP. Browser UI passed a non-Windows smoke test for navigation and Notes create/edit/save in preview mode. Validate the `.exe`, voice, brightness, installer and update flow on Windows before distribution.

## Structure

- `Native/MainWindow.cs` — privileged WebView2 host and message bridge.
- `Native/SpeechController.cs` — microphone, wake grammar and dictation.
- `Native/WindowsControl.cs` — audio/brightness/metrics/app shortcuts.
- `Native/NotesStore.cs` — local notes JSON store.
- `Native/MiniWidget.cs` — draggable performance overlay.
- `Native/UpdateService.cs` — GitHub Releases update checks and verified download.
- `Web/index.html`, `Web/src/` — UI, Notes, voice states, commands, styling.
- `Installer/ORBIT.iss` — installs into `%LOCALAPPDATA%\Programs\ORBIT`, preserving `%LOCALAPPDATA%\ORBIT` user files.

## If voice does not listen

Open Windows Settings > Privacy & security > Microphone and allow access for desktop apps. Check Windows speech recognition language components. In ORBIT, use push-to-talk first, then optionally enable always-on "Hey ORBIT". If `ro-RO` is unsupported on your Windows version, use an installed recognition language; synthesizer voices and recognizers are separate Windows components.

## Security

Privileged commands are validated by the C# host and accepted only from bundled `https://orbit.local/` UI. Voice launching uses a fixed allowlist; no arbitrary voice commands are passed to `cmd.exe`/PowerShell. Browser-origin operations cannot directly access arbitrary OS APIs, and update repository URLs are constrained to GitHub's public release format.
