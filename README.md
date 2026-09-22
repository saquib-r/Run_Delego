# Running Delego Locally

Welcome to the team. This guide takes you from a fresh laptop to a working **Delego** app on your phone/emulator, talking to a **mundra** backend running on your own machine.

It works on **macOS** and **Windows**. Wherever the two differ, you'll see tabs like this:

> 🍎 **macOS** — do this
> 🪟 **Windows** — do that

You do **not** need to know Flutter or FastAPI to get through this. Just follow the steps in order, top to bottom, and don't skip Part 0.

**What you're setting up:**

| Repo | What it is | Language |
|---|---|---|
| `mundra` | The backend — a web API that stores delegates, logins, schedules | Python (FastAPI) |
| `Delego-3.0` | The app itself — what delegates install on their phone | Dart (Flutter) |

They're **two separate repos** that live side by side in one folder. The app can't do anything useful until the backend is running, so we do the backend first.

---

## Part 0: Install everything first

Work through this list before running a single command from Part 1. Installing as you go is how people end up with half-broken setups.

### 0.1 — Things everyone needs

| Tool | What it's for | macOS | Windows |
|---|---|---|---|
| **Git** | Downloading the code | Comes with Xcode CLT (below), or `brew install git` | [git-scm.com/download/win](https://git-scm.com/download/win) — accept all defaults |
| **A code editor** | Reading/writing code | [VS Code](https://code.visualstudio.com/) (recommended) or Android Studio | Same |
| **Python 3.12 or newer** | Runs the backend | Usually preinstalled; check below | [python.org/downloads](https://www.python.org/downloads/) — **tick "Add python.exe to PATH"** on the first installer screen |
| **uv** | Installs the backend's Python packages | `brew install uv` | `winget install --id=astral-sh.uv` |
| **Flutter SDK — 3.47.0 or newer**, stable channel | Builds the app | [docs.flutter.dev/get-started/install/macos](https://docs.flutter.dev/get-started/install/macos) | [docs.flutter.dev/get-started/install/windows](https://docs.flutter.dev/get-started/install/windows) |

**On macOS**, if you don't have Homebrew (the `brew` command), install it first from [brew.sh](https://brew.sh), then run `xcode-select --install`.

**On Windows**, if you don't have `winget`, install uv from PowerShell instead:
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

> ⚠️ **After installing anything that changes your PATH (Python, uv, Flutter), close your terminal and open a new one.** Otherwise the new command "won't exist" and you'll waste an hour. This is the single most common setup problem.

**Check they all work** — open a *new* terminal (macOS: Terminal; Windows: PowerShell) and run:
```bash
git --version
python3 --version      # Windows: python --version   → must be 3.12 or higher
uv --version
flutter --version      # must be 3.47.0 or higher, on the "stable" channel
```
All four should print a version number. If one says "command not found" / "not recognized", that tool isn't installed or isn't on your PATH — fix it before moving on.

> ⚠️ **Flutter must be at least 3.47.** Older versions will fail to build this project. If `flutter --version` shows something lower, or a channel other than `stable`:
> ```bash
> flutter channel stable
> flutter upgrade
> ```
> Then re-run `flutter --version` to confirm. We're on **3.47.5** — anything from 3.47.0 up is fine.

### 0.2 — Pick where you'll run the app

You need **at least one** of these. Pick the one that matches your machine:

| Your machine | Easiest target | What to install |
|---|---|---|
| Mac | **iOS Simulator** | Section 0.3 |
| Mac | Android emulator | Section 0.4 |
| Windows | **Android emulator** (only option) | Section 0.4 |

> 🪟 **Windows can't build iOS apps.** That's an Apple restriction, not a project limitation. Windows users work with Android; the app's code is identical either way.
>
> This app also **can't** run as a desktop app or in Chrome — ignore those targets even if Flutter offers them.

### 0.3 — iOS Simulator setup (macOS only)

1. Install **Xcode** from the Mac App Store (it's large — start this early).
2. Open Xcode once and let it finish installing components.
3. Accept the licence:
   ```bash
   sudo xcodebuild -license
   ```
4. Install **CocoaPods** (fetches the iOS side of Flutter plugins):
   ```bash
   brew install cocoapods
   ```
5. Make sure a simulator runtime is installed: **Xcode → Settings → Platforms → iOS**.
6. Confirm the Simulator opens:
   ```bash
   open -a Simulator
   ```

### 0.4 — Android emulator setup (macOS *and* Windows)

1. Install **[Android Studio](https://developer.android.com/studio)**. You need it for the Android SDK and the emulator even if you never write code in it.
2. Open it and complete the setup wizard (it downloads the SDK — this takes a while).
3. Install the command-line tools: **Android Studio → More Actions (or Tools) → SDK Manager → SDK Tools tab → tick "Android SDK Command-line Tools (latest)" → Apply**.
4. **Create an emulator:** **More Actions → Virtual Device Manager → Create Device →** pick e.g. *Pixel 7* → pick a recent system image (download it if there's an arrow icon) → **Finish**.
5. Accept the Android licences — run this and type `y` at every prompt:
   ```bash
   flutter doctor --android-licenses
   ```
6. 🪟 **Windows only:** turn on **Developer Mode** (Settings → System → For developers). Flutter needs it to build plugins.

### 0.5 — Final toolchain check

```bash
flutter doctor -v
```

Read the output. You need a `[✓]` on:
- **Flutter**
- **Android toolchain** (if you're using Android)
- **Xcode** (if you're on a Mac using iOS)

You can **ignore** warnings about Chrome, Visual Studio (the C++ one), Linux desktop, and "Connected device" for macOS desktop — this app doesn't use those targets.

If something shows `[✗]` or `[!]`, `flutter doctor -v` prints the exact fix underneath it. Do that, then re-run it.

---

## Part 1: Get the code

You work from **your own fork** of each repo — you've already been asked to fork both. You clone *your* fork, not the original, so you always have somewhere to push.

### Step 1 — Make the parent folder

Both repos must sit side by side inside **one parent folder**. The name doesn't matter; `DelegoApp` is what we use.

> 🍎 **macOS**
> ```bash
> mkdir ~/DelegoApp
> cd ~/DelegoApp
> ```

> 🪟 **Windows** (PowerShell)
> ```powershell
> mkdir $HOME\DelegoApp
> cd $HOME\DelegoApp
> ```

### Step 2 — Clone your two forks

Replace `<your-github-username>` with your actual GitHub username in both lines:

```bash
git clone https://github.com/<your-github-username>/mundra.git
git clone https://github.com/<your-github-username>/Delego-3.0.git
```

> 💡 The easiest way to get these URLs right: open your fork on GitHub, click the green **Code** button, and copy the HTTPS URL.

You should now have exactly this:
```
DelegoApp/
├── mundra/        ← backend
└── Delego-3.0/    ← app
```

### Step 3 — Link each fork back to the main repo

Your fork is a snapshot from the moment you forked it. Adding an `upstream` remote lets you pull in the team's newer changes later.

```bash
cd mundra
git remote add upstream https://github.com/saquib-r/mundra.git
cd ../Delego-3.0
git remote add upstream https://github.com/saquib-r/Delego-3.0.git
cd ..
```

Check it worked — `git remote -v` in either folder should show **two** names: `origin` (your fork, where you push) and `upstream` (the team's repo, which you only pull from).

> ⚠️ **Confirm the `upstream` URLs with the team before running these.** The backend in particular may live under the `munsoc-mpstme` org rather than the URL above.

To pull in the team's latest changes later:
```bash
git fetch upstream
git merge upstream/main        # the backend's branch is called `master`, not `main`
```

> 📌 **Branch names differ between the repos:** `Delego-3.0` uses **`main`**, `mundra` uses **`master`**. Not a typo — just how they were set up.

---

## Part 2: Run the backend (`mundra`)

⚠️ **Every command in this part must be run from inside the `mundra` folder.** The backend loads its config, templates and static files relative to wherever you started it, so running from the parent folder will fail in confusing ways.

### Step 1 — Go into the backend folder
```bash
cd mundra
```

### Step 2 — Install the Python dependencies
```bash
uv sync
```
This creates a private virtual environment (`.venv/`) and installs the exact package versions the team uses. You never need to `pip install` anything by hand, and you never need to "activate" the venv — `uv run` handles it.

### Step 3 — Create your config file (`.env`)

The backend refuses to start without a config file holding a secret key.

> 🍎 **macOS**
> ```bash
> cp .env.example .env
> ```

> 🪟 **Windows** (PowerShell)
> ```powershell
> Copy-Item .env.example .env
> ```

Now open `.env` in your editor and fill in the **two required values**:

1. **`SECRET_KEY`** — this signs login tokens. Generate a random one:
   ```bash
   uv run python -c "import secrets; print(secrets.token_urlsafe(48))"
   ```
   Paste the output after `SECRET_KEY=` (no quotes, no spaces around the `=`).

2. **`MAIL_SERVER`** — set it to `localhost`:
   ```
   MAIL_SERVER=localhost
   ```

Leave everything else in the file exactly as it is.

> ⚠️ **Never leave a setting present but blank.** A line like `DOCS_URL=` means *"set this to an empty string"*, not *"use the default"* — it will break things. If you don't want to set something, delete the line or leave it commented out with `#`.
>
> 🔒 **`.env` is a secrets file. Never commit it, never paste its contents into chat.** It's already in `.gitignore`.

### Step 4 — Check everything works
```bash
uv run pytest
```
You should see **`15 passed`**. These tests use a throwaway database, so they won't touch your real data.

If this fails, stop here and fix it — the server won't work either. Send the error output to the team.

### Step 5 — Start the server
```bash
uv run fastapi dev app.py
```

The backend is now live at **http://127.0.0.1:8000**.

**Leave this terminal window open and running.** It's your server. Open a *second* terminal tab/window for everything in Part 3.

> 📱 **Testing on a real phone instead of an emulator?** Start it this way instead, so other devices on your Wi-Fi can reach it:
> ```bash
> uv run fastapi dev app.py --host 0.0.0.0
> ```

### Step 6 — Confirm it's alive

Open **http://127.0.0.1:8000/swagger** in your browser. You should see **Swagger UI** — an interactive list of every API endpoint, where you can click "Try it out" and call them directly. This is your best friend for debugging the backend.

Or from a terminal:

> 🍎 **macOS**
> ```bash
> curl http://127.0.0.1:8000/
> ```

> 🪟 **Windows** (PowerShell — use `curl.exe`, not `curl`, which is an alias for something else)
> ```powershell
> curl.exe http://127.0.0.1:8000/
> ```

Expected: `{"message":"Server is up and running"}`

### 📧 Heads-up: email doesn't work locally — and that's fine

The backend sends real emails through a mail server that only exists in production. Locally, any route that sends email (**register**, **forgot password**, **resend verification**) will return a **`500` error**.

**This is expected and mostly harmless.** The important detail:

> When you register from the app's own sign-up screen, the account **is created and is already verified**, even though the app shows an error. Just go back to the login screen and log in with the email and password you typed. Password must be at least 8 characters.

That's how you'll get a test account in Part 3 — no database seeding needed.

---

## Part 3: Run the app (`Delego-3.0`)

Open a **new terminal window** (leave the backend running in the old one) and go to the app folder:

```bash
cd ../Delego-3.0      # or: cd ~/DelegoApp/Delego-3.0
```

### Step 1 — Download the app's packages
```bash
flutter pub get
```

### Step 2 — Platform-specific one-time setup

> 🍎 **iOS (macOS only)** — let CocoaPods fetch the native plugin code:
> ```bash
> cd ios
> pod install
> cd ..
> ```

> 🤖 **Android (macOS or Windows)** — two things:
>
> **a) Allow plain HTTP in debug builds.** Android blocks unencrypted `http://` by default, and your local backend isn't `https://`. Open `android/app/src/debug/AndroidManifest.xml` and add an `<application>` block so the file looks like this:
> ```xml
> <manifest xmlns:android="http://schemas.android.com/apk/res/android">
>     <uses-permission android:name="android.permission.INTERNET"/>
>     <application android:usesCleartextTraffic="true" />
> </manifest>
> ```
> This is the **debug** manifest only, so release builds stay secure. Don't make this change in `main/` or `profile/`.
>
> **b) If (and only if) a build later fails with a Gradle/Java version error**, point Flutter at the JDK bundled with Android Studio:
> ```bash
> # macOS
> flutter config --jdk-dir="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
> ```
> ```powershell
> # Windows
> flutter config --jdk-dir="C:\Program Files\Android\Android Studio\jbr"
> ```
> Then run `flutter clean` and try again. You don't need this step unless you hit that error.

### Step 3 — Start your emulator/simulator

> 🍎 **iOS Simulator**
> ```bash
> open -a Simulator
> ```

> 🤖 **Android emulator** — list your emulators, then launch one by name:
> ```bash
> flutter emulators
> flutter emulators --launch Pixel_7_API_35      # use YOUR emulator's id from the list
> ```
> (You can also just press ▶ next to the device in Android Studio's Virtual Device Manager.)

Wait until the emulator has fully booted to its home screen, then confirm Flutter can see it:
```bash
flutter devices
```
Your device should be listed. Note the name — you'll use it in the next step.

### Step 4 — Run the app, pointed at your local backend

This is the key step. The `--dart-define=API_BASE_URL=...` part tells the app to talk to *your* backend instead of the production one. **The address is different for each target** — this trips everybody up:

> 🍎 **iOS Simulator** — it shares your Mac's network, so `127.0.0.1` works:
> ```bash
> flutter run -d "iPhone 17" --dart-define=API_BASE_URL=http://127.0.0.1:8000
> ```
> (replace `"iPhone 17"` with whatever `flutter devices` showed)

> 🤖 **Android emulator** — it's a separate virtual machine, so `127.0.0.1` would mean *the emulator itself*. Use the special address `10.0.2.2`, which means "the computer I'm running on":
> ```bash
> flutter run -d emulator-5554 --dart-define=API_BASE_URL=http://10.0.2.2:8000
> ```
> (replace `emulator-5554` with your device id from `flutter devices`)

> 📱 **A real phone** (backend started with `--host 0.0.0.0`) — use your computer's address on the Wi-Fi network. Find it with `ipconfig getifaddr en0` on macOS, or `ipconfig` on Windows (look for "IPv4 Address" under your Wi-Fi adapter). Phone and computer must be on the **same Wi-Fi**:
> ```bash
> flutter run -d <device-id> --dart-define=API_BASE_URL=http://192.168.1.42:8000
> ```

First build takes several minutes. Later runs are much faster.

> 💡 While `flutter run` is going, press **`r`** in that terminal for hot reload (instant UI changes), **`R`** for a full restart, and **`q`** to quit.

### Step 5 — Create an account and smoke-test

Arrange your windows so you can see the **backend terminal** while using the app — you'll watch requests arrive live.

1. In the app, tap **Register**. Fill in first name, last name, email, and a password of **8+ characters**.
2. You'll get an error about email — **expected** (see Part 2). Your account exists and is verified.
3. Go back and **log in** with that email and password. In the backend terminal you should see `POST /login` and `GET /delegates/me` come through.
4. Open **Rooms** and **Schedule** → backend shows `GET /rooms`, `GET /schedule`.
5. Open **Profile** and edit a field → backend shows `PATCH /delegates/{id}`.
6. Open the **QR** screen → backend shows `GET /qr?id=`.

If those requests appear with `200`/`201` status codes, **you're fully set up.** 🎉

---

## Everyday workflow (after setup)

Two terminals, every time:

```bash
# Terminal 1 — backend
cd DelegoApp/mundra
uv run fastapi dev app.py

# Terminal 2 — app
cd DelegoApp/Delego-3.0
flutter run -d <your-device> --dart-define=API_BASE_URL=http://10.0.2.2:8000   # or 127.0.0.1 for iOS
```

Both auto-reload when you save a file — the backend restarts itself, and the app hot-reloads (press `r` if it doesn't).

Before you push code:
```bash
cd mundra      && uv run ruff check . && uv run pytest
cd Delego-3.0  && flutter analyze
```

---

## Troubleshooting

| Symptom | Why | Fix |
|---|---|---|
| `uv` / `flutter` / `python` — "command not found" or "not recognized" | PATH not refreshed after install | **Close the terminal and open a new one.** Still broken → reinstall and confirm the installer's "add to PATH" option was ticked |
| Build fails with Dart/Flutter SDK version errors | Flutter older than 3.47 | `flutter channel stable && flutter upgrade`, then `flutter clean` and run again |
| `git push` rejected with "permission denied" | Pushing to the team repo instead of your fork | You should be pushing to `origin` (your fork). Check with `git remote -v` — see Part 1, Step 3 |
| App shows a network/connection error on every screen | Wrong `API_BASE_URL`, or the backend isn't running | Android emulator needs `10.0.2.2`, **not** `127.0.0.1`. Check Terminal 1 is still running. Re-run with the right `--dart-define` |
| Android: connection refused even with `10.0.2.2` | Cleartext HTTP still blocked | Re-check Part 3 Step 2a, then `flutter clean` and run again |
| Register / forgot-password returns `500` | Expected locally — no mail server | Ignore it; the account was created and verified. Just log in |
| Backend returns `500` where you'd expect `401`/`404` | Known quirk: routes wrap their own errors | Read the `detail` text — the real status code is the number inside the message |
| App shows old/production Rooms or Schedule data | Those are cached on the device for 2 hours | Uninstall the app from the emulator, or `xcrun simctl erase all` (iOS) / wipe data in Android Device Manager |
| `flutter doctor` shows `[✗] Android toolchain — licences not accepted` | Licences not accepted | `flutter doctor --android-licenses`, type `y` to each |
| Android build fails on a Gradle/Java version | Wrong JDK picked up | Part 3, Step 2b |
| iOS Simulator missing from `flutter devices` | Simulator not open, or no runtime installed | `open -a Simulator`; add a runtime in Xcode → Settings → Platforms |
| Emulator very slow / won't boot (Windows) | Hardware acceleration off | Enable virtualisation (VT-x / AMD-V) in BIOS; install "Android Emulator hypervisor driver" via SDK Manager → SDK Tools |
| `pod install` fails on macOS | CocoaPods out of date | `brew upgrade cocoapods`, then delete `ios/Podfile.lock` and retry |
| Builds fail in strange ways after pulling new code | Stale build cache | `flutter clean && flutter pub get`, then run again |

---

## Ground rules

- **Never commit** `mundra/.env`, `Delego-3.0/android/key.properties`, or `upload-keystore.jks`. They hold secrets.
- **Push to your own fork (`origin`), never to `upstream`.** Work on a branch, push it to your fork, then open a pull request against the team's repo.
- **Be careful with `git stash` / `git checkout .` / `git reset --hard`** — they silently throw away uncommitted work, including yours. Commit first, ask if unsure.
- **Don't change `pubspec.yaml` or package versions** on your own. Dependency bumps affect everyone; raise it with the team first.
- Keep **setup fixes separate from feature work** in your commits.

Stuck for more than 30 minutes? Ask in the team chat and paste the **full** error output plus the output of `flutter doctor -v`. That's not a bother — it's faster for everyone.
