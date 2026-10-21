# FlowKit — n8n workflow templates for small businesses

**English** · [Versión en español](README.es.md)

[![Licence: MIT](https://img.shields.io/badge/licence-MIT-blue.svg)](LICENSE)
[![n8n compatible](https://img.shields.io/badge/n8n-compatible-EA4B71.svg)](https://n8n.io)

**n8n workflows you import and run**, built for small businesses without a technical team: clinics, finance and operations teams, agencies, and professional practices.

These are not screenshots or generic tutorials. Every workflow is a **`.json` file you import into n8n in about thirty seconds**, with credential notes, message templates and step-by-step documentation — in English for the clinic and finance workflows, in Spanish for the four sector workflows built with Spanish-speaking practices.

Web version with guides and a free ROI calculator: **https://mrnpvn.github.io/flowkit/en/**

---

## What FlowKit solves

A small business loses hours every week on work that was already solved:

- a contact form **is stored nowhere** and a lead is lost;
- **appointment reminders** are made by hand, one call per patient;
- **bank detail changes** arrive by email and get applied on trust;
- nobody answers the inbound lead until the next morning, and it goes to a competitor.

FlowKit packages that work into tested workflows: import, connect your accounts, and it runs. **No code, no consultant, no monthly SaaS fee.**

## Workflow index

| Workflow | Vertical | Status | File |
|---|---|---|---|
| Vendor bank-change guard — hold, verify out of band, cooling period, audit | Finance / ops | ✅ English | [`workflows/finance/vendor-bank-change-guard.json`](workflows/finance/vendor-bank-change-guard.json) |
| Appointment reminders with confirm/cancel links | Clinic | ✅ English | [`workflows/clinica/appointment-reminders-confirm-cancel.json`](workflows/clinica/appointment-reminders-confirm-cancel.json) |
| Web lead capture (form → spreadsheet → Telegram alert) | General | ✅ Spanish (`captura-leads-web.json`) | [`workflows/general`](workflows/general) |
| Client onboarding + tax deadlines | Accountancy (ES) | ✅ Spanish | [`workflows/asesoria/alta-cliente-plazos.json`](workflows/asesoria/alta-cliente-plazos.json) |
| Appointment reminder + cancellation recovery | Clinic (ES) | ✅ Spanish | [`workflows/clinica/recordatorio-citas.json`](workflows/clinica/recordatorio-citas.json) |
| Lead capture and instant reply | Real estate (ES) | ✅ Spanish | [`workflows/inmobiliaria/captacion-leads-respuesta.json`](workflows/inmobiliaria/captacion-leads-respuesta.json) |
| 💎 Overdue invoices — staged reminders, receivables report, payment link | Accountancy (ES) | 💎 Paid pack | [FlowKit Pack](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) |
| 💎 Document onboarding — checklist, follow-ups, client portal | Accountancy (ES) | 💎 Paid pack | [FlowKit Pack](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) |
| 💎 KPI panel — collections, files, documents, weekly history | Accountancy (ES) | 💎 Paid pack | [FlowKit Pack](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) |

> The three premium workflows are **not in this repository**. They ship inside the [FlowKit Pack](https://mrnpvn.gumroad.com/l/flowkit-pack-asesoria) (€29, first 20 copies), together with guides, a 30-day rollout plan and GDPR templates. Their full documentation stays public here — what each one does, which sheets it needs, what data it touches — so you can judge it before paying. **The pack and its documentation are in Spanish**; the free English workflows below are not a trial version of it.
>
> The free workflows in this repository are MIT licensed, need no signup, and are not a demo.

## The two English workflows in detail

### Vendor bank-change guard (60 nodes, 5 integrations)

A supplier emails new bank details. This workflow does not update anything on trust:

1. **Validates** the IBAN (mod-97 checksum and country) and **scores the risk** — unknown vendor, inactive vendor, foreign country, free-mail domain, change arriving right after an invoice.
2. **Holds** the change through a cooling period (48 h or 72 h depending on the score).
3. **Sends a one-time link** to the finance address already on file — never to the address that requested the change.
4. Stores the token **only as a SHA-256 hash**, so a leaked spreadsheet cannot be replayed.
5. **Refuses early approval** without touching state, answers a second click with "Already handled", and expires the request through a watchdog at 24 / 48 / 72 hours.
6. Writes an **append-only audit row** for denials, duplicates and manual review.

Documentation: [`docs/FLOW-VENDOR-BANK-CHANGE-GUARD.md`](docs/FLOW-VENDOR-BANK-CHANGE-GUARD.md).

### Appointment reminders with confirm and cancel (21 nodes)

Every evening the workflow reads tomorrow's appointments and sends one email per patient with signed confirm and cancel links. Patients update their own status; a cancellation tells the front desk exactly which slot just opened. No health data ever leaves the spreadsheet.

Documentation: [`docs/FLOW-APPOINTMENTS-EN.md`](docs/FLOW-APPOINTMENTS-EN.md).

## Installation (2 minutes)

1. Download the `.json` of the workflow you want.
2. In n8n: **Workflows → Import from File** and pick the file.
3. Open each node flagged in red, choose your credentials (Google Sheets, Telegram, SMTP) and save.

Requirements: **n8n self-hosted or n8n Cloud, version 1.40 or newer**, plus accounts for the services the workflow uses. Both English workflows are tested with a clean import and re-export on n8n 2.43.8.

Detailed guide, requirements and troubleshooting (in Spanish): [`docs/INSTALACION-N8N.md`](docs/INSTALACION-N8N.md).

## Use cases

- **Finance and operations** — a bank detail change cannot be applied by email alone; every request is held, verified through a separate channel and logged.
- **Clinics and dental practices** — reminder 24 hours ahead, recovery of slots freed by cancellation, patient confirms their own appointment.
- **Accountancy practices** — client onboarding without retyping data, automatic deadline alerts, document tracking.
- **Real estate agencies** — instant reply to web leads, matching against the portfolio, assignment by area.
- **Agencies and practices** — form capture, quote follow-up, periodic client reports without exporting CSV by hand.

If your case is missing, [open an issue](https://github.com/mrnpvn/flowkit/issues): requested workflows get built.

## Customisation and support

- Every workflow carries **sticky notes inside n8n** explaining what each step does and what you must configure.
- Message templates are separate from the logic: you edit wording, not code.
- Stuck on something? Open an issue with the workflow name and the error.

## Privacy and GDPR

These workflows run **in your own n8n instance, with your own credentials**. Client data never reaches our servers.

That does not make compliance automatic. When you run them on personal data (GDPR / RGPD, and especially health data in clinics):

- connect only the accounts you need, with minimum permissions;
- tell your clients what you use their data for in your privacy notice;
- keep data in tools that offer a data-processing agreement;
- delete data once it is no longer needed.

## Licence

The code and documentation in this repository are released under the **MIT licence** (see [`LICENSE`](LICENSE)): use them, modify them and use them commercially in your business, including in client work. You may not resell the files as templates.

The **premium pack** (extended versions, complete sector workflows and support) is distributed separately and is not covered by MIT.

## Contributing

A workflow that works in your business? A pull request with the exported `.json` and an entry in `docs/CATALOGO.md` is welcome. Rules in [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

FlowKit is a product of **AIVANTA**.
