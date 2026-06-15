# Horizen AI · Lead Gen & Cold Email Automation

![n8n](https://img.shields.io/badge/built%20with-n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4.1--mini-412991?style=flat-square&logo=openai&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-leads-34A853?style=flat-square&logo=googlesheets&logoColor=white)
![Resend](https://img.shields.io/badge/Resend-send-000000?style=flat-square&logo=resend&logoColor=white)

Fully automated outbound system that reads leads from Google Sheets, generates a personalized cold email in Serbian for each contact using GPT-4.1-mini, and sends it from your own domain via the Resend API — 5 times per day, at different hours, zero manual work.

Built for [Horizen AI](https://horizen.rs) to generate outbound interest in the AI Receptionist product.

---

## Why plain text, not HTML?

This system deliberately sends **plain-text emails** — no logo, no buttons, no styled template. For cold outreach this is a feature, not a limitation:

| | Plain text | Branded HTML |
|---|---|---|
| Deliverability | High — looks like 1:1 mail | Lower — bulk-mail fingerprint |
| Spam filters | Rarely triggered | Triggered far more often |
| First impression | "A person wrote this to me" | "A company is marketing at me" |
| Reply rate | Higher | Lower |

A local business owner who receives a polished branded email knows instantly it's a mass campaign. The same message as plain text reads like the founder personally sat down and typed it — because the AI personalization makes each one genuinely unique. HTML templates belong in newsletters and onboarding sequences, not in a first cold touch.

No links in the body either (links are a spam signal in cold email) — only the bare domain in the signature.

---

## How it works

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   ⏰  5 Schedule Triggers     09:00 · 11:00 · 13:00 · 15:00 · 17:00 │
│            │                   Monday – Friday                      │
│            ▼                                                        │
│   📊  Google Sheets            Read all rows from leads sheet       │
│            │                                                        │
│            ▼                                                        │
│   🔍  Filter                   Skip rows where:                     │
│            │                   · Email is empty                     │
│            │                   · Kontaktiran = true                 │
│            ▼                                                        │
│   1️⃣  Take 1 Lead              Process exactly 1 lead per trigger   │
│            │                                                        │
│            ▼                                                        │
│   🤖  GPT-4.1-mini             Personalized email in Serbian        │
│            │                   Industry pain point · 5-part         │
│            │                   structure · few-shot guided          │
│            ▼                                                        │
│   ⚙️  Parse Response           Extract subject + body from JSON     │
│            │                   Fallback on parse error              │
│            ▼                                                        │
│   ✍️  Build Email              Append signature + opt-out line      │
│            │                   Pure plain text, no HTML             │
│            ▼                                                        │
│   📨  Send via Resend          POST to Resend API, sent from        │
│            │                   matija@horizen.rs (verified domain)  │
│            ▼                                                        │
│   ✅  Mark as Contacted        Updates Kontaktiran = true in sheet  │
│            │                                                        │
│            ▼                                                        │
│   📝  Log                      Timestamp · company · status         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Cost:** ~$0.01 per email (GPT-4.1-mini) + Resend free tier (3,000 emails/mo)
**Daily volume:** 5 emails/day — one per trigger, spread across working hours
**Delivery:** Sent from your own domain via Resend; replies land in your normal inbox (domain MX is unchanged)

---

## AI Personalization

The system prompt maps each lead's category to a specific pain point so the first sentence of the email feels hand-written, not templated.

### Email structure (enforced via prompt)

```
1. POZDRAV — "Dobar dan," — neutral, natural Serbian opener
2. HOOK    — category-specific industry stat, never a claimed internal fact
3. BRIDGE  — connect the pain to the solution
4. PROOF   — one number with context, not one adjective
5. CTA     — low-friction yes/no question, never "zakažite meeting"
```

The signature and opt-out line are **not** written by the AI — they're appended in code, so they're identical in every email and the model can't mangle them.

### Category pain point map

| Category | Hook |
|---|---|
| Dentist / Dental clinic | ordinacije propuste svaki peti poziv dok je tim zauzet s pacijentom — svaki je termin od najmanje 50€ |
| Doctor / Medical | kad je recepcija preopterećena, pacijenti ne čekaju — zovu sledeću kliniku na Google Maps-u |
| Health resort / Spa | najviše poziva stiže upravo kad je osoblje usred tretmana |
| Automotive | rezervacije se gube svaki put kad telefon zazvoni dok je mehaničar pod autom |
| All other categories | propušten poziv dok je osoblje nedostupno = izgubljen lead |

### Prompt design

- **Forbidden phrases** — "Nadam se da ste dobro", "Stupam u kontakt" and others explicitly blocked
- **Rating-aware** — 4.5★+ leads get a "reputation worth protecting" angle; low/few reviews are never mentioned
- **Few-shot example** — a complete high-quality email shown inside the prompt so the model has a concrete target, not just rules
- **Counter-example** — a bad email with an explanation of *why* it's bad, which steers the model away from spam patterns
- **Strict JSON output** — `{ "subject": "...", "body": "..." }` with markdown fence stripping and fallback parser

---

## Sample output

**Input:** `Dental Centar Jovanović · Dentist · Belgrade · 4.9★ (224 reviews)`

---

> **Subject:** termini koje dental centar jovanović gubi
>
> Dobar dan,
>
> Stomatološke ordinacije u proseku propuste svaki peti dolazni poziv dok je tim zauzet s pacijentom. Svaka ta propuštena slušalica je po pravilu izgubljen termin.
>
> Dental Centar Jovanović ima 4.9 zvezda. Šteta bi bilo da pacijent ode konkurenciji samo zato što niko nije stigao da se javi.
>
> Zato smo napravili AI Recepcionera koji odgovori na svaki poziv za manje od 2 sekunde, kvalifikuje pacijenta i direktno zakaže termin, 24/7.
>
> Ordinacije koje ga koriste beleže 38% više zakazanih termina u prvih 30 dana.
>
> Ima li smisla da vam pokažem kako bi izgledalo za vašu ordinaciju? Demo traje 8 minuta.
>
> Srdačan pozdrav,
> Matija Radulović
> Horizen AI · horizen.rs
>
> P.S. Ako vam ovo nije relevantno, samo odgovorite "ne" i neću vas više pisati.

---

The P.S. opt-out line matters: it dramatically reduces spam complaints (people reply "ne" instead of clicking *Report spam*), and spam complaints are what actually kill a sending domain.

---

## Workflow nodes

| Node | Type | Purpose |
|---|---|---|
| 09:00 / 11:00 / 13:00 / 15:00 / 17:00 | Schedule Trigger | Fire once per hour slot, Mon–Fri |
| Get Leads | Google Sheets | Read all rows from leads sheet |
| Filter | n8n Filter | Skip empty emails and already-contacted leads |
| Take 1 Lead | Limit | Exactly 1 lead per workflow run |
| Generate Email | OpenAI (LangChain) | GPT-4.1-mini writes personalized email |
| Parse Response | Code (JS) | Extract JSON, handle parse errors |
| Build Email | Code (JS) | Append signature + opt-out, output plain text |
| Send via Resend | HTTP Request | POST to Resend API — sends from your verified domain |
| Mark as Contacted | Google Sheets | Set `Kontaktiran = true` for the processed row |
| Log | Code (JS) | Write result to n8n execution log |

---

## Google Sheets format

The workflow reads from a sheet with these columns (see [`leads_template.csv`](leads_template.csv)):

| Column | Description |
|---|---|
| `Naziv firme` | Company name |
| `Kategorija` | Business category (e.g. Dentist, Automotive) |
| `Email` | Email address — **must be filled for row to be processed** |
| `Telefon` | Phone number |
| `Website` | Website URL |
| `Grad` | City |
| `Ocena` | Star rating |
| `Broj recenzija` | Number of reviews |
| `Kontaktiran` | Leave empty — automatically set to `true` after email is sent |

---

## Setup

Requires an active n8n instance with three credentials:

| Credential | Used in |
|---|---|
| Google Sheets OAuth2 | Get Leads · Mark as Contacted |
| OpenAI API | Generate Email |
| HTTP Header Auth (Resend) | Send via Resend |

### Resend setup (one-time)

**1.** Create an account at [resend.com](https://resend.com) and add your domain (`horizen.rs`)

**2.** Resend gives you DNS records (SPF, DKIM, DMARC) — add them at your domain registrar and wait for verification (usually minutes). Without this, `from: matija@horizen.rs` will be rejected.

**3.** Create an API key (starts with `re_`)

**4.** In n8n, create a **Header Auth** credential named `Resend API`:
- **Name:** `Authorization`
- **Value:** `Bearer re_your_api_key_here`

### Workflow setup

**5.** Import `workflow.json` into n8n

**6.** In **Get Leads** and **Mark as Contacted** — select your spreadsheet and sheet tab

**7.** Connect the three credentials to their nodes (the Resend key goes on **Send via Resend**)

**8.** Add a `Kontaktiran` column to your Google Sheet (leave all rows empty initially)

**9.** Make sure the `Email` column is populated — rows without an email are skipped

**10.** Activate the workflow

> **Deliverability tip:** domain verification (SPF/DKIM/DMARC) is mandatory for Resend, not optional. Keep volume at 5–10/day for the first few weeks while the domain warms up.

---

## Cost

| | |
|---|---|
| Per email | ~$0.01 (GPT-4.1-mini) |
| Resend | Free up to 3,000 emails/month |
| 5 emails/day · 20 workdays | **~$1/month** |

---

Built by [Horizen AI](https://horizen.rs) — AI systems that grow your business while you focus on the work only you can do.
