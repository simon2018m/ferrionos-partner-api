# FerrionOS Partner API (Sandbox)

FerrionOS is a gate identity and visitor management system for Nigerian sites: estates, schools, corporate offices and hospitality. A tablet at the gate verifies each visitor's identity (NIN lookup plus face and fingerprint capture) and keeps a digital visit log in place of the paper logbook. FerrionOS is built by Hemon Industries.

This repository documents the **partner API**: how a security company, facility manager or estate management platform can read FerrionOS gate data into its own systems.

> **Status:** FerrionOS is in pilot and not yet live at any site. Everything here is a sandbox. All examples use mock data, and no source code, credentials or real visitor data are published.

## Who this is for

- **Security companies** managing guards across several sites, who want gate activity in their own dashboard
- **Facility and estate managers** who want daily visitor counts and on-site lists in their existing tools
- **Integration partners** building reporting or alerting on top of gate data

## What partners can do

The partner API is read-only. Identity capture only ever happens at the gate.

| Method and path | What it returns | Who can call it |
| --- | --- | --- |
| `POST /auth/login` | An access token | Guard, site admin |
| `GET /sites/me` | Site profile (name, address, industry) | Guard, site admin |
| `GET /visitors/open` | Visitors currently on site | Guard, site admin |
| `GET /visitors/stats/today` | Today's sign-in and sign-out counts | Guard, site admin |
| `GET /entries` | The visit log (guards see today only) | Guard, site admin |
| `GET /entries/{id}` | One visit in full | Site admin |
| `GET /billing/me` | Verification credit status (guards see a reduced view) | Guard, site admin |
| `GET /devices/me` | Which gate tablets are online | Site admin |

## Authentication

Call `POST /auth/login` to get a token, then send it on every request:

```
Authorization: Bearer <token>
```

Tokens last 8 hours. A missing or invalid token returns `401`.

## How this kit is being built

- [x] Endpoint inventory and design decisions ([DESIGN-NOTES.md](DESIGN-NOTES.md))
- [ ] OpenAPI specification and hosted docs
- [ ] Postman collection with a mock server
- [ ] Payment webhook demo (Paystack test mode)
- [ ] Quickstart and troubleshooting guide
- [ ] Walkthrough video

## Author

Simon Ode Agbo, founder of FerrionOS · [LinkedIn](https://linkedin.com/in/simonagbo)
