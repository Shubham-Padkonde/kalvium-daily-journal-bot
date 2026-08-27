# Kalvium Daily Journal Bot

Automatically submits the "Simulated Work Daily Journal" Google Form every
weekday at 4:00 PM IST via GitHub Actions, marking "working day, present"
and filling the four follow-up questions with varied generated content.

## How it works

- `daily_fill.py` reuses a saved, logged-in Google session (no password
  ever stored) to open the form and submit it headlessly.
- The session is stored as the `AUTH_STATE` repo secret and restored at the
  start of each workflow run.
- The schedule lives in `.github/workflows/daily-journal.yml`
  (`30 10 * * 1-5` UTC = 4:00 PM IST, Mon-Fri).

## Setup (for your own account)

Each person needs their **own copy of this repo** and their **own login
session** — the bot submits as whichever Google account you log in with.

### 1. Prerequisites

- [Git](https://git-scm.com/downloads)
- [Python 3.12+](https://www.python.org/downloads/)
- [GitHub CLI](https://cli.github.com/) (`gh`), logged in via `gh auth login`

### 2. Get the code

Fork this repo on GitHub, then clone your fork:

```
git clone https://github.com/<your-username>/<your-fork>.git
cd <your-fork>
```

Make sure the repo is **private** (Settings → General → Danger Zone →
Change visibility), since it will hold a secret tied to your Google login.

### 3. Install dependencies

```
python -m venv venv
venv\Scripts\python.exe -m pip install -r requirements.txt
venv\Scripts\python.exe -m playwright install chromium
```

(On macOS/Linux, use `venv/bin/python` instead of `venv\Scripts\python.exe`.)

### 4. Log in and capture your session

```
venv\Scripts\python.exe discover_form.py
```

This opens a real browser window. Log into **your own** Kalvium Google
account, wait until the form itself is visible, then press Enter in the
terminal. This creates `auth_state.json` locally — it is never committed
(see `.gitignore`); it only ever gets stored as an encrypted GitHub secret.

### 5. Upload the session as a GitHub secret

```
gh secret set AUTH_STATE --repo <your-username>/<your-fork> < auth_state.json
```

### 6. Enable the workflow

Push your clone to GitHub (if you haven't already), then check the
**Actions** tab on your fork — GitHub sometimes disables scheduled
workflows on forks by default, so click "Enable workflow" if prompted.

That's it. The bot will now run automatically every weekday at 4:00 PM IST.

## When the session expires ("logged out")

Google Workspace can force re-authentication after some period. When that
happens, the scheduled run fails and GitHub emails you a "workflow run
failed" notification, with the log saying the session expired.

To fix it, repeat steps 4 and 5 above:

```
venv\Scripts\python.exe discover_form.py
gh secret set AUTH_STATE --repo <your-username>/<your-fork> < auth_state.json
```

## Manually triggering a run

```
gh workflow run daily-journal.yml --repo <your-username>/<your-fork>
```

## Notes

- Holidays/leave days aren't auto-detected — the bot always marks
  "present". Disable the workflow manually (Actions tab → ... → Disable
  workflow) on days you don't want it to run.
- Keep the repo **private** — `auth_state.json` grants access to your
  Google session and should never be committed or shared. It only ever
  lives as the encrypted `AUTH_STATE` secret.
