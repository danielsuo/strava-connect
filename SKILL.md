---
name: "strava-connect"
description: "Connect a Strava account via OAuth and read training data (activities, laps, heart rate) through the Strava API. Use when someone wants to link Strava or pull Strava workout data."
---

# Strava Connect

## Purpose
Set up Strava OAuth for a user and read their training data: recent
activities, per-activity detail, and per-lap distance, time, speed, and
heart rate. Read-only.

## Tooling
`bin/strava-api` — CLI wrapper around `https://www.strava.com/api/v3`:

- `strava-api athlete` — authenticated athlete profile (verifies the credential)
- `strava-api activities [--per-page N] [--page N]` — recent activities,
  summarized (id, name, date, distance, moving time, avg/max HR, avg speed)
- `strava-api activity <id>` — full activity detail with per-lap distance,
  time, speed, and HR

`bin/dynamic_credentials.py` — bundled authd surrogate helper. Do not modify.

The credential name defaults to `custom.strava`; override with the
`STRAVA_CREDENTIAL` environment variable.

## Auth
OAuth 2.0 authorization-code flow, collected with
`credentials.request_api_access` — never ask the user for client IDs,
secrets, or tokens in chat.

Setup order:
1. The user creates an API application at
   `https://www.strava.com/settings/api` (Strava offers no dynamic client
   registration, so this step is manual). See
   `references/oauth-setup.md` for the exact fields.
2. Call `credentials.request_api_access` with:
   - `provider: "strava"`
   - `auth_scheme: "oauth2_code"`
   - `api_hosts: ["www.strava.com"]`
   - `authorization_url: "https://www.strava.com/oauth/authorize"`
   - `token_url: "https://www.strava.com/oauth/token"`
   - `issuer_domain: "strava.com"`
   - `scopes: "read activity:read_all profile:read_all"`
   - `token_endpoint_auth_method: "client_secret_post"`
3. Verify with `bin/strava-api athlete` — the first real API call is what
   proves the credential works.

The CLI exchanges a surrogate with authd at request time; the real token
never appears in logs or files. Allowed hosts: `www.strava.com`.

## Operating Rules
1. Read-only: never kudos, comment, follow, upload, or edit
   activities/profile.
2. Restrict authenticated requests to `www.strava.com`.
3. Do not print, log, or persist raw credentials or surrogates.
4. Transient auth: Strava rotates tokens roughly every 6 hours and a call
   can 401 mid-rotation. On a 401, wait 60 seconds and retry once before
   concluding auth failed. Only a twice-rejected credential-carrying
   request means the credential needs replacing (via
   `credentials.request_api_access` with `reconnect: true`).
5. A 401 or 403 is a question about the request before it is a question
   about the key. Check that the request was built with the bundled
   helper — a request without it carries nothing and looks exactly like a
   wrong token.
