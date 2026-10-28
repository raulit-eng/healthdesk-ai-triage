# HealthDesk AI Triage

An IT service desk prototype for **Meridian Valley Health**, a fictional community hospital system. It explores one question: *how many tickets can be solved by sending the right knowledge base article, without a technician touching them — and how do you make sure nobody falls through the cracks when the AI guesses wrong?*

Built by Raul Astorga as a portfolio project.

## What it does

| Area | What happens |
|---|---|
| **Knowledge base** | 20 step-by-step articles written for a hospital environment: password reset, MFA, badge tap-in, EHR freezes, wristband label printers, dictation mics, VPN, Wi-Fi, shared mailboxes, encrypted email, phishing, access requests, and more. |
| **Get help (portal)** | Employees submit a ticket. As they type, matching articles are suggested before they even submit. |
| **AI triage** | Every ticket is created first, then triaged. Triage assigns priority (P1–P4), category, and matching KB articles with confidence scores. If a self-service article confidently fits, the employee immediately gets a reply with the link. |
| **IT queue** | Every ticket lands here, including the ones the AI answered. Technicians can assign, add notes, escalate, and resolve. Each ticket shows the AI's reasoning and the exact reply the employee received. |
| **Follow-ups** | When the AI sends a KB link, a follow-up reminder is set (P2: 4h, P3: 24h, P4: 48h). Overdue follow-ups are flagged until the employee confirms the fix or a technician takes over. |
| **Insights** | Deflection rate, how often a KB link actually fixed the problem, which articles get sent most, and how many tickets needed a tech after all. |

## Guardrails built into triage

The AI is deliberately conservative, because a wrong deflection wastes a clinician's time:

- **Patient care blocked** (checkbox or language like "can't chart", "whole unit") → P1, technician paged, never deflected.
- **Security incidents** (clicked a phishing link, entered a password, lost device) → P1, never deflected.
- **Access requests, broken hardware, anything needing admin rights** → always routed to a human.
- A deflection needs a self-service article **and** ≥ 60% confidence.
- The ticket always stays open in the IT queue. The employee can click **Still need help** at any time to escalate.

## Two triage engines

- **Claude** — when the page runs as a Claude artifact, triage calls Claude with the KB catalog and the ticket and asks for structured JSON. The response is validated and the guardrails above are enforced in code regardless of what the model returns.
- **Keyword matcher** — a fallback that scores tickets against article tags, titles, and summaries. Used automatically when Claude isn't available (for example, when you open `index.html` directly).

## Running it

It's a single self-contained file. Open `index.html` in a browser.

- **As a standalone file**: tickets are saved in your browser's localStorage ("Demo mode") and triage uses the keyword matcher. Use **Load example tickets** on the IT queue to see a populated desk.
- **As a Claude artifact**: tickets live in a shared database so everyone with the link sees the same queue, and triage uses Claude.

## Structure

`index.html` contains three parts:

1. **Styles** — design tokens for light and dark themes.
2. **Engine** (`<script id="engine">`) — KB content, the keyword matcher, the AI prompt, response validation, and follow-up scheduling. It never touches the DOM, so it can be loaded in Node for testing.
3. **App** — rendering, routing (`#portal`, `#kb`, `#kb101`, `#queue`, `#followups`, `#insights`), and the storage layer.

## Ideas for next steps

- Email or Teams notifications when a follow-up goes overdue
- Track whether the employee actually opened the article
- Let technicians promote a resolved ticket into a new KB article
- Export tickets to CSV for reporting
