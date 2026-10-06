# Strava API application setup

Strava does not support dynamic client registration, so the user creates
the API application manually once. After that, the agent handles OAuth
through the Secure Vault (`credentials.request_api_access`).

## Create the application

1. Go to `https://www.strava.com/settings/api` while signed in to Strava.
2. Fill in:
   - **Application Name:** anything recognizable, e.g. `Coach`
   - **Category:** pick the closest (e.g. `Training`)
   - **Website:** the user's site, or `https://www.strava.com` as a placeholder
   - **Authorization Callback Domain:** the domain the OAuth flow returns
     to. For Muse's Secure Vault flow this is `agent.meta.ai`.
3. Note the **Client ID**. The **Client Secret** is shown on the same page —
   keep it in the Secure Vault flow, never in chat.

## Scopes

Request the least needed. For read-only training data:

- `read` — public activities
- `activity:read_all` — private activities too (most training logs need this)
- `profile:read_all` — full athlete profile

Space-separated in the connector setup: `read activity:read_all profile:read_all`.

## Token lifecycle

- Access tokens live ~6 hours. The Secure Vault handles refresh
  automatically; the CLI just makes calls.
- If a call returns HTTP 401, wait 60 seconds and retry once — a refresh
  can race the request. Only treat auth as broken if the retry also 401s.

## Rate limits

Strava allows 100 requests per 15 minutes, 1,000 per day per application.
The three CLI commands stay far under this in normal use; do not poll in a
tight loop.
