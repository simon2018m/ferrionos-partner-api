# Design notes

Decisions behind the FerrionOS partner API: what was chosen, what was rejected, and why.

## 1. The partner API is read-only

**Decision:** partners can read gate data but cannot create, edit or close a visit.

**Alternative considered:** letting partners sign visitors in or out from their own systems.

**Why:** a FerrionOS visit is evidence that a verified person was physically at the gate. Sign-in and sign-out both require a live face and fingerprint capture on the gate tablet. If a partner system could create or close visits remotely, that guarantee would break. So the backend's routes split into two groups: kiosk-only routes that need biometric capture, and 8 read-only routes that are safe to expose to partners.

## 2. Partners do not receive NIN

**Decision:** the partner view of a visit leaves out the visitor's National Identification Number (NIN).

**Alternative considered:** returning the full visit record, including NIN, because the gate app already stores it.

**Why:** data minimisation. A partner needs to know who visited, when, whom they came to see, and whether their identity was verified. It does not need the NIN itself. Every extra copy of a NIN is extra risk. FerrionOS already purges raw NIN and biometric references from visit records after 6 months, keeping only a permanent audit trail of reference number, timestamps and verification outcome. The partner API follows the same principle from the start.

## 3. Guards see less than site admins

**Decision:** the same endpoint returns a reduced view to a guard. On the visit log, guards see today's entries only. On billing, they see the percentage of credit used and a low-credit flag, not the site's financial figures.

**Alternative considered:** blocking guards from these endpoints entirely.

**Why:** front-desk staff need enough to do the job, such as signing out today's visitors and flagging low credit before it runs out. They don't need a site's full history or spending. Shaping the response by role keeps one endpoint per resource while respecting least privilege.

## 4. The partner contract is designed first, ahead of the code

**Decision:** `openapi.yaml` describes the response partners *should* get: camelCase field names throughout, and no NIN, raw identity record, biometric reference, phone number or home address. It is the contract the partner endpoints will be built to.

**What the code does today:** the internal endpoints the gate app uses return more than this. The visit log and on-site list return raw database rows in snake_case, including NIN, phone number, address and, on single-visit detail, the full identity record from the national database and biometric references. Other endpoints, like site profile and billing, already return clean camelCase.

**Why design first:** exposing the internal endpoints to partners as they are would publish NIN and other sensitive fields, and lock partners into inconsistent field names. Writing the contract first makes the gap explicit. Closing it means adding a partner response mapper that selects and renames fields, so the internal gate app is untouched.

## Open questions

- **Partner API keys.** Today every caller logs in as a user and gets an 8-hour token. A dedicated, revocable key per partner integration would be better for long-running integrations. This is planned, not built.
- **Webhooks for gate events.** Partners currently have to poll for new visits. A "visitor signed in" webhook would remove that need.
