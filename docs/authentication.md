# Authentication

Every partner request carries a short-lived access token. You get one by logging in, then send it in the `Authorization` header until it expires.

> **Sandbox note:** all examples use mock data. FerrionOS is in pilot and not yet live at any site.

## How it works today

1. **Log in** with a site user's role and 4-digit PIN. FerrionOS returns a token.
2. **Send the token** on every request: `Authorization: Bearer <token>`.
3. **Log in again** when the token expires. Tokens last **8 hours**, and there is no refresh token.

Each token is tied to one site and one role, so a partner only ever sees data for the site it logged in to.

## 1. Log in

`POST /auth/login`

**Request body**

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `role` | string | Yes | `guard` or `site_admin` |
| `code` | string | Yes | The user's 4-digit PIN, sent as a string, such as `"0427"` |

```http
POST /auth/login
Content-Type: application/json

{
  "role": "site_admin",
  "code": "0427"
}
```

**Success: `200 OK`**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.mock.token",
  "user": {
    "id": "7f3c2a10-0000-4000-8000-000000000001",
    "name": "Adaeze Okafor",
    "forceId": null,
    "role": "site_admin",
    "siteId": "c41d9b20-0000-4000-8000-000000000009",
    "siteName": "Palm Grove Estate"
  }
}
```

Store the `token` securely on your server. Never put it in a browser, a mobile app bundle or a URL.

## 2. Send the token

```http
GET /visitors/open
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.mock.token
```

To check that a token works, call `GET /sites/me`. A `200` with your site's profile means you're authenticated.

## 3. Roles

| Role | Who it is | What it can reach |
| --- | --- | --- |
| `guard` | Gate or front-desk staff | Live gate data, with reduced views: today's visit log only, and a credit gauge without financial figures |
| `site_admin` | The site's manager | Everything a guard can see, plus full visit history, single-visit detail and device status |

The [README](../README.md) lists which role can call each endpoint.

## Errors

| Status | When it happens | Example body | What to do |
| --- | --- | --- | --- |
| `400` | `role` isn't `guard` or `site_admin`, or `code` isn't a 4-digit string | `{"error": "role must be one of ... and code must be a 4-digit string"}` | Send `code` as a string of exactly 4 digits, such as `"0427"`, not the number `427` |
| `401` | Wrong PIN at login | `{"error": "Invalid code"}` | Check the PIN with the site admin |
| `401` | No `Authorization` header, or it doesn't start with `Bearer ` | `{"error": "Missing bearer token"}` | Add `Authorization: Bearer <token>` |
| `401` | Token expired (after 8 hours) or malformed | `{"error": "Invalid or expired token"}` | Log in again for a fresh token |
| `403` | Your role can't call this endpoint | `{"error": "Requires one of: site_admin"}` | Log in as a `site_admin`, or use an endpoint your role allows |
| `429` | Too many failed logins in a short time | `{"error": "Too many failed attempts", "retryAfterSeconds": 24}` | Wait `retryAfterSeconds`, then retry. Don't loop |

## Good practice for integrations

- **Re-authenticate on `401`, not on a timer.** When a call returns `401` with "Invalid or expired token", log in once and retry that call once.
- **Never retry a failed login in a loop.** Repeated failures trigger a temporary lockout, and the response tells you how long to wait.
- **Don't send an `X-Device-Id` header.** FerrionOS uses it to track gate tablets, so a partner server sending it would show up as a gate device.

## Coming next: partner API keys

Today a partner signs in as a site user, which suits a pilot but not a long-running integration. The planned improvement is a dedicated, revocable API key per partner, so a site can see and cut off each integration separately without sharing staff PINs. This is planned, not built.
