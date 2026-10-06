# strava-connect

A shareable skill for connecting a Strava account via OAuth and reading
training data (activities, laps, heart rate) through the Strava API.
Read-only — it never modifies anything on the account.

## Install

Copy this folder into your Muse skills directory:

```bash
cp -r strava-connect ~/workspace/skills/
chmod +x ~/workspace/skills/strava-connect/bin/strava-api
```

The skill activates when someone asks to link Strava or pull Strava data.

## How it works

1. The user creates a free API application at
   `https://www.strava.com/settings/api` (one-time, ~2 minutes).
2. The agent collects OAuth consent through Muse's Secure Vault — no
   tokens ever appear in chat, logs, or files.
3. `bin/strava-api` reads activities, activity detail, and per-lap splits.

See `SKILL.md` for the agent instructions and
`references/oauth-setup.md` for the app-creation walkthrough.

## Contents

- `SKILL.md` — the skill (purpose, tooling, auth, operating rules)
- `bin/strava-api` — CLI: `athlete`, `activities`, `activity <id>`
- `bin/dynamic_credentials.py` — authd surrogate helper (do not modify)
- `references/oauth-setup.md` — Strava app setup, scopes, token lifecycle
