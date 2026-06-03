# Horizen AI · Lead Gen & Cold Email Automation

![n8n](https://img.shields.io/badge/built%20with-n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4.1--mini-412991?style=flat-square&logo=openai&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-leads-34A853?style=flat-square&logo=googlesheets&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-drafts-EA4335?style=flat-square&logo=gmail&logoColor=white)

Fully automated outbound system that reads leads from Google Sheets, generates a personalized cold email in Serbian for each contact using GPT-4.1-mini, and saves it as a branded HTML draft in Gmail — 5 times per day, at different hours, zero manual work.

Built for [Horizen AI](https://horizenai.com) to generate outbound interest in the AI Receptionist product.

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
│            │                   Industry pain point · 4-part         │
│            │                   structure · few-shot guided          │
│            ▼                                                        │
│   ⚙️  Parse Response           Extract subject + body from JSON     │
│            │                   Fallback on parse error              │
│            ▼                                                        │
│   🎨  Build HTML Email         Branded Horizen AI template          │
│            │                   Fully inline CSS (Gmail-compatible)  │
│            ▼                                                        │
│   📨  Gmail: Create Draft      Saved to Drafts with recipient       │
│            │                                                        │
│            ▼                                                        │
│   ✅  Mark as Contacted        Updates Kontaktiran = true in sheet  │
│            │                                                        │
│            ▼                                                        │
│   📝  Log                      Timestamp · company · status         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Cost:** ~$0.01 per email (GPT-4.1-mini)
**Daily volume:** 5 emails/day — one per trigger, spread across working hours
**Output:** Gmail Drafts — review before sending, or switch to Send Email node

---

## AI Personalization

The system prompt maps each lead's category to a specific pain point so the first sentence of the email feels hand-written, not templated.

### Email structure (enforced via prompt)

```
1. HOOK    — category-specific pain point that makes them think "kako znaju?"
2. BRIDGE  — connect the pain to the solution
3. PROOF   — one number, not one adjective
4. CTA     — low-friction ask, never "zakažite meeting"
```

### Category pain point map

| Category | Hook |
|---|---|
| Dentist / Dental clinic | 35–40% poziva propušteno tokom zahvata — svaki je izgubljen termin vredan 50–500€ |
| Doctor / Medical | klinike gube rezervacije kad je recepcija preopterećena — pacijenti zovu sledećeg |
| Health resort / Spa | gube rezervacije i upite dok je osoblje zauzeto sa tretmanima |
| Automotive | rezervacije se gube svaki put kad mehaničar ne može da ostavi auto |
| Sve ostale kategorije | propušteni dolazni pozivi koštaju leadove dok je osoblje nedostupno |

### Prompt design

- **Forbidden phrases** — "Nadam se da ste dobro", "Stupam u kontakt" and others explicitly blocked
- **Few-shot example** — a complete high-quality email shown inside the prompt so the model has a concrete target, not just rules
- **Strict JSON output** — `{ "subject": "...", "body": "..." }` with markdown fence stripping and fallback parser

---

## Sample output

**Input:** `Dental Centar Jovanović · Dentist · Belgrade · 4.9★ (224 reviews)`

---

> **Subject:** propušteni pozivi u dental centru jovanović
>
> Dental Centar Jovanović propusti svaki treći poziv dok je tim zauzet s pacijentom — to nije samo propušten poziv, to je izgubljen termin.
>
> Napravili smo AI Recepcionera koji odgovori na svaki poziv za manje od 2 sekunde, kvalifikuje pacijenta i direktno zakaže termin — 24/7, bez čekanja.
>
> Ordinacije koje ga koriste beleže 38% više zakazanih termina u prvih 30 dana.
>
> Ima li smisla pokazati vam kako bi to izgledalo za vašu ordinaciju?

---

## Email template

Fully inline CSS — renders correctly in Gmail, Outlook, and Apple Mail.

```
╔══════════════════════════════════════╗
║  [Horizen AI logo]  (black · #0a0a0a)║
╠══════════════════════════════════════╣  ← purple gradient bar
║                                      ║
║  [AI-generated email body]           ║
║                                      ║
║  [ Pogledaj kako funkcioniše → ]     ║  ← purple CTA #7c6aff
╠══════════════════════════════════════╣
║  Horizen AI · email · unsubscribe    ║
╚══════════════════════════════════════╝
```

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
| Build HTML Email | Code (JS) | Inline CSS branded template |
| Create Draft | Gmail | Save to Drafts with recipient, subject, HTML body |
| Mark as Contacted | Google Sheets | Set `Kontaktiran = true` for the processed row |
| Log | Code (JS) | Write result to n8n execution log |

---

## Google Sheets format

The workflow reads from a sheet with these columns:

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
| `Kontaktiran` | Leave empty — automatically set to `true` after email is drafted |

---

## Setup

Requires an active n8n instance with three credentials:

| Credential | Used in |
|---|---|
| Google Sheets OAuth2 | Get Leads · Mark as Contacted |
| OpenAI API | Generate Email |
| Gmail OAuth2 | Create Draft |

**1.** Import `workflow.json` into n8n

**2.** In **Get Leads** and **Mark as Contacted** — select your spreadsheet and sheet tab

**3.** Connect the three credentials to their nodes

**4.** Add a `Kontaktiran` column to your Google Sheet (leave all rows empty initially)

**5.** Make sure the `Email` column is populated — rows without an email are skipped

**6.** Activate the workflow

---

## Cost

| | |
|---|---|
| Per email | ~$0.01 (GPT-4.1-mini) |
| 5 emails/day · 20 workdays | **~$1/month** |

---

Built by [Horizen AI](https://horizenai.com) — AI systems that grow your business while you focus on the work only you can do.
