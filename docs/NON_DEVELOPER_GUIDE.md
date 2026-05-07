# Using the Communication Agent (Non‑Developer Guide)

This guide explains **what the system does for you**, what **requires a technical teammate** to configure, and **how you can work with it day to day** without writing code.

---

## What this system is

The **communication agent** is software that helps LongevityInTime.org with:

- **Investor outreach** — Sends personalized introductory emails from a spreadsheet list, tracks who replied, and **labels replies** as interested, not interested, needs more information, or other.
- **Specialist workflows** (grants, preprints, journals, patents, data agreements, FDA packages) — These parts **produce drafts or file packages**; a human still submits or signs off through official portals. They are primarily used when a researcher or program lead asks technical staff to run specific tasks.

If you only care about investor emails and seeing results in a dashboard, focus on **“Investor outreach — what you handle”** below.

---

## Investor outreach — what you handle

### 1. Prepare your contact list in a spreadsheet

Use **Excel**, **Google Sheets**, or similar. When you save the file for the team run, use the **CSV** format (comma‑separated values).

**Required columns** (exact spelling):

| Column        | Example              | Notes                                      |
|---------------|----------------------|--------------------------------------------|
| `name`        | Jane Investor        | Person’s name                              |
| `email`       | jane@example.com     | Must be unique and valid                   |
| `firm`        | Example Capital      | Organization or fund name                   |
| `focus_area`  | Longevity, biotech   | Short phrase about their investing focus    |

**Optional column:**

| Column   | Notes                                      |
|----------|--------------------------------------------|
| `notes`   | Anything the AI should consider when tailoring the message |

**Tips:**

- One row per person; **no duplicate emails** in the same campaign if you expect clean tracking.
- Remove extra spaces; use a **single header row**.
- Anyone **already emailed** by this system before will be **skipped** automatically when the outreach job runs — you do not need to remember that list yourself.

Hand the finished CSV to whoever runs the send (see “One‑time and ongoing technical setup”).

### 2. Review and approve message content (process, not code)

Before the first real send, your team should agree on:

- **Tone and facts** — What the organization promises, links, and disclosure language.
- **Who the “from” address is** — It must match an email domain verified with the email provider (technical setup).

Dry runs can show **subject lines** and **who would receive** mail without delivering it. Ask your technical contact for a **preview / dry run** if you want to validate the list before go‑live.

### 3. Use the dashboard like any web app

After outreach has run at least once, someone starts the **Streamlit dashboard**. You open it in the **browser** (your technical contact shares the URL, often on your internal network).

**What you’ll see:**

- **Numbers at the top** — Totals for people on the list, emails sent, replies, and simplified categories (e.g. interested, needs info).
- **A table** — Names, emails, firms, focus areas, when the email was sent, reply status, and when a reply was received.
- **Filters** — Narrow by reply status, firm, or search by name/email.
- **Refresh** — Click refresh if you expect new data after replies arrive.

**Reply status labels** (plain language):

| Label            | Typical meaning                                       |
|------------------|-------------------------------------------------------|
| Pending          | No reply recorded yet (or system hasn’t classified)   |
| Interested       | Reply suggests they want to continue the conversation|
| Not interested   | Reply declines or pushes back clearly                 |
| Needs info       | They asked questions or requested more detail         |
| Other            | Doesn’t fit the above — worth a human read            |

The system guesses intent from reply text using AI; **treat classifications as helpers**, not legal commitments. Important threads should always get **human judgment**.

---

## What happens when someone replies (conceptual)

1. Investor replies to the email your organization sent.
2. Those messages flow through **Resend** (the email sending service).
3. Resend notifies the **webhook server**, which reads the reply and updates the matching row in the **database**.
4. The **dashboard** reads that database and shows updated status.

You do **not** need to log into Resend or the database yourself for normal monitoring — the dashboard is the user‑facing summary.

---

## One‑time and ongoing technical setup (for your IT / developer partner)

Share this checklist with whoever maintains the software. **You do not need to perform these steps** unless that is explicitly your role.

| Task | Why it matters |
|------|----------------|
| Install Python, create virtual environment, `pip install -r requirements.txt` | Runs the agents |
| Create `.env` from `.env.example` and add keys (`OPENAI_API_KEY`, `RESEND_API_KEY`, `WEBHOOK_SECRET`, `FROM_EMAIL`, etc.) | Connects AI and email |
| Configure Resend domain (DKIM/SPF), verify sender address | Deliverability and compliance |
| Run outreach: `python -m scripts.run_outreach --csv path/to/list.csv` (or `--dry-run` first) | Sends emails and saves records |
| Run webhook server (`uvicorn …`) behind HTTPS and register Resend **inbound webhook** pointing to `/webhook` | So replies update the dashboard |
| Run dashboard: `streamlit run dashboard/app.py` | Gives you the browser UI |

Optional: **`WEBHOOK_API_KEY`** protects some task APIs; Resend’s inbound webhook path does not use that key.

---

## Grants, FDA, patents, and other “roadmap” features

These modules **assist experts** with search, drafting, or building submission **folders**. They do **not** replace:

- Legal review
- Institutional sign‑off
- Official government or publisher websites

If your job is stakeholder communication or fundraising, your main interaction is usually **outreach CSV + dashboard**. For science or regulatory deliverables, you’ll typically **request outputs from a technical owner** and then route them to the right specialist (legal, regulatory, PI).

---

## Glossary

| Term | Simple meaning |
|------|----------------|
| **CSV** | A plain text spreadsheet format; easy for the system to read |
| **Webhook** | A secure “phone call” from the email service to your server when an event happens (e.g. new reply) |
| **Dashboard** | The browser page that shows outreach status |
| **Dry run** | A test that shows what would happen **without** sending real email |
| **Resend** | Third‑party service used to send and receive email for this project |
| **GPT / OpenAI** | AI used to personalize emails and classify reply intent |

---

## If something looks wrong

| Symptom | What to report to technical support |
|---------|-------------------------------------|
| Dashboard says “No outreach records” | Confirm the outreach script ran successfully against your CSV |
| Emails never arrive | Sender domain, spam folder, Resend dashboard logs |
| Replies stay “pending” forever | Webhook URL, HTTPS, `WEBHOOK_SECRET`, inbound email routing in Resend |
| Wrong person linked to a reply | Matching is by sender **email address** — alias or forwarded mail can confuse matching |

Always include **example email address**, **approximate time**, and **screenshot** of the dashboard row when filing an issue — that speeds up diagnosis.

---

## Summary

As a non‑developer, your highest‑value contributions are: **accurate spreadsheets**, **clear approval of messaging**, and **using the dashboard** to prioritize follow‑ups. Technical colleagues own installation, secrets, sending jobs, webhook hosting, and any specialist automation you request beyond standard outreach.
