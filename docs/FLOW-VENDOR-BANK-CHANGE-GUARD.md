# Vendor bank-change guard with out-of-band approval and cooling period

A payment-detail change is the highest-value request an attacker can send to a finance team. This workflow makes that request **impossible to apply by accident**: every change is held, verified out of band, cooled down, and logged before anyone touches the banking system.

- **File:** `workflows/finance/vendor-bank-change-guard.json`
- **Nodes:** 60 (54 functional + 6 sticky notes)
- **Integrations:** Google Sheets, SMTP email, Telegram, webhook, form trigger
- **Tested on:** n8n 2.43.8 (import + export, no warnings) and 45 logic assertions on the Code nodes
- **Language:** English (ready for the public n8n template library)

---

## The mechanism

Most "approval" workflows are a route: read a row, send an email, done. This one is a state machine with three properties a reviewer or an auditor can test:

1. **State that survives between runs.** Each request lives in `BankChanges` with a status: `PENDING_VERIFICATION`, `APPROVED`, `REJECTED`, `EXPIRED`, `MANUAL_REVIEW` or `DUPLICATE_IGNORED`. The daily watchdog reads that state and advances it.
2. **A decision that changes behaviour.** The same submission takes three different routes depending on vendor match, IBAN validity, duplicate detection and a risk score. High risk never gets an approval link.
3. **An error path that does more than stop.** Refused submissions, duplicate attempts, manual reviews and denied links are written to an append-only `Audit` tab. A failed email stops the state change instead of advancing a hold that nobody was told about.

In one sentence: **it holds every payment-detail change until an out-of-band approver clears a cooling period, and it refuses replays and expired links while keeping an audit trail.**

## Why the link goes to a different address

The verification email is sent to the finance contact **already stored in `Vendors`**, never to the address that submitted the form. If an attacker controls the vendor's mailbox, they still cannot approve their own change: the link lands in the finance team's inbox.

The link carries a random 64-character token. Only its **SHA-256 hash** is stored in the sheet, so a leaked spreadsheet cannot be used to approve anything. The token is single-use, expires after 7 days, and is **never re-sent** — reminders tell the approver to use the original email.

## The three parts

### A. Intake (form → hold)

1. The vendor submits the change through a hosted form (`/form/vendor-bank-change`).
2. The IBAN is normalised and validated: format, **mod-97 checksum**, and the bank country must match the IBAN prefix. A typo is rejected before it reaches a human.
3. The vendor is matched against `Vendors`. The change is scored:

| Condition | Points |
|---|---|
| Unknown vendor | 100 (manual) |
| Vendor not `active` | +40 |
| IBAN country ≠ vendor country | +30 |
| Requester domain ≠ vendor domain | +25 |
| Free mailbox (gmail, outlook, …) | +15 |
| One change in the last 90 days | +15 |
| Two or more changes in the last 90 days | +30 |

   Risk ≥ 70 goes to **manual review** and never gets an approval link. Risk ≥ 40 uses the long cooling period (72 h by default; 48 h otherwise).
4. Duplicates are detected by IBAN hash while a request is pending or approved: the second request is ignored (`DUPLICATE_IGNORED`) and no second email is sent.
5. A row is appended with status `PENDING_VERIFICATION`, a token is generated and hashed, and the verification email is sent to the known finance address.

### B. One-click decision (webhook)

`GET /webhook/vendor-bank-decision?t=<token>&d=approve|reject` returns a small HTML page. The workflow:

- rejects malformed, unknown, expired or already-used tokens (HTTP 400, `LINK_DENIED` in Audit);
- refuses to approve **before** `cooling_until` (HTTP 200, "still in the cooling period", state untouched);
- on approve: sets `APPROVED`, emails finance with the masked IBAN and tells them to apply it and confirm by phone;
- on reject: sets `REJECTED` and tells the requester that nothing was changed;
- on a second click: returns "Already handled" and changes nothing (idempotent).

### C. Watchdog (daily at 07:30)

| Age | Stage | Action |
|---|---|---|
| 24 h | 1 | Reminder to the finance contact |
| 48 h | 2 | Escalation to the finance lead |
| 72 h | 3 | Hold closed as `EXPIRED`; requester told nothing was applied |

`last_stage` in the sheet guarantees a stage is sent once. The same run sends one digest to the finance lead (open holds, high-risk count, oldest hold, last-7-days decisions) instead of one message per vendor.

## Setup (10 minutes)

### 1. Google Sheet with three tabs

**Vendors**

```
vendor_code | vendor_name | country | vendor_domain | finance_email | finance_lead_email | status
```

`finance_email` must be the address you trust **today** for that vendor, not one copied from a recent email.

**BankChanges**

```
change_id | vendor_code | vendor_name | requested_by | finance_email | finance_lead_email |
new_iban | new_iban_masked | iban_hash | bank_country | risk | risk_flags | status | token_hash |
created_at | cooling_until | expires_at | last_stage | last_action_at | decided_by | note
```

**Audit**

```
at | event | change_id | actor | detail
```

### 2. Configure the three Config nodes

Open `Config (intake)`, `Config (approval)` and `Config (watchdog)` and set:

- `base_url` — your public n8n URL, no trailing slash (used to build the links)
- `company` — signature in the emails
- `finance_lead` — fallback recipient for escalations and the digest
- `cooling_hours_standard` (48), `cooling_hours_high` (72), `link_ttl_hours` (168)

Keep the three copies identical; the sticky note in the canvas says so.

### 3. Credentials

| Node | Credential |
|---|---|
| Google Sheets (11 nodes) | Google Sheets OAuth2 |
| Email (6 nodes) | SMTP |
| Telegram (6 nodes) | Telegram bot + chat ID |
| Form / webhook | none (uses your n8n URL) |

Restrict access to the spreadsheet: it holds bank details and should be treated as confidential data.

### 4. Activate and test

1. Add yourself to `Vendors` with a test `vendor_code` and your own `finance_email`.
2. Submit the form with a valid IBAN (for example `ES9121000418450200051332`).
3. Check that the row is `PENDING_VERIFICATION`, that the email arrives at the address on file, and that the approve link returns "Still in the cooling period" if you click it too early.
4. Set `cooling_hours_standard` to `0` for a one-off test if you want to see the approve path immediately, then restore it.

## What it does not do

- It does not verify that the IBAN belongs to the vendor. Banks do not expose that; the workflow forces a **human phone confirmation** with the number already on file before the change is applied.
- It does not apply the change in your banking or ERP system. Finance does that after the approval email.
- It does not screen sanctions or PEP lists.
- It does not prove the identity of the person who submitted the form. It proves that the **finance team** approved the change after a cooling period.

## Data protection notes

- Personal data processed: name/email of the requester, vendor contacts, bank details. Legal basis is usually contract performance plus legitimate interest in fraud prevention; document it in your records of processing.
- The full IBAN is stored in `BankChanges` because finance needs it to apply the change. Restrict the sheet to the finance role, enable 2-step verification on the Google account, and set a retention period (for example, delete closed rows after 12 months).
- The Audit tab is append-only in normal operation. Do not edit it by hand; export it if you need evidence.
- Emails contain the masked IBAN, not the full number.

## Verification performed

- **Import + export in n8n 2.43.8** with no errors: 60/60 nodes and 44 connection groups preserved.
- **45 logic assertions** on the seven Code nodes: IBAN checksum, country mismatch, unknown vendor, duplicate detection, change velocity, token expiry, cooling refusal, replay, malformed/unknown token, bad decision, 24/48/72 h stages, stage de-duplication, expiry email recipient, digest counts.
- Every `$('Node')` reference in the workflow resolves to an existing node (checked mechanically).
- The three Config nodes pass incoming fields through (checked mechanically).
- No secrets, no credentials, no external paid APIs.

## FAQ

**Can I use a different sheet?** Yes — any Google Sheet with the headers above. The workflow only uses read, append and update by `change_id`.

**What if the approver leaves the company?** Update `finance_email` in `Vendors`; the next request goes to the new address. Existing holds can be rejected from the old link or closed by the watchdog.

**Why not store the token so reminders can include the link?** Because a reminder is then a second copy of the credential. The email already contains the link; re-sending it increases the attack surface for no operational gain.

**Can two people approve?** The design is "one approver plus a cooling period". Dual approval would need a second token and a second column; the state machine already supports it if you want to extend it.

**Does it work with Outlook instead of SMTP?** Yes — replace the six email nodes with the Microsoft Outlook node; the fields `to`, `subject` and `body` are already prepared per item.
