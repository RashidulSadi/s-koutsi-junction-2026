# s-koutsi-junction-2026
S-Koutsi: Generative UI Financial Co-Pilot &amp; Three Baskets Trainer for S-mobiili (Junction 2026 S-Pankki Challenge)
# S-Koutsi · Your AI Financial Survival & Habit Strategist

> **Junction 2026 · S-Pankki challenge: Personal Economic Trainer**
> A conversational money coach for S-mobiili that turns 24 months of banking data into a plan for *this week*, not a report on last month.

**▶ Live demo:** open `index.html` (or the GitHub Pages URL for this repo) and press **🎬 Auto-Play Demo (45s)**.

> ⚠️ Unofficial hackathon concept. Not an S-Bank service and not affiliated with S-Pankki Oy or S Group. Uses the challenge's sample dataset. No real accounts, payments or bank connections.

---

## Contents

1. [The problem](#the-problem)
2. [What S-Koutsi does](#what-s-koutsi-does)
3. [Run it](#run-it)
4. [Architecture](#architecture)
5. [The Three Baskets: categorizing CUST00001's 581 transactions](#the-three-baskets-categorizing-cust00001s-581-transactions)
6. [Zero-hallucination AI: deterministic function calling](#zero-hallucination-ai-deterministic-function-calling)
7. [S-Group retail synergy](#s-group-retail-synergy)
8. [Prototype vs production](#prototype-vs-production)
9. [Data notes & assumptions](#data-notes--assumptions)

---

## The problem

Bank statements and pie charts are a rear-view mirror: they show that the money is gone after it has gone. When an unexpected bill lands, young adults and fixed-income customers often panic and reach for expensive quick loans, even when a cheaper, legal option exists.

**Persona (real dataset row):** Customer `CUST00001`, *Aino, 26, Helsinki*

| Metric | Value |
|---|---|
| Transactions analyzed | **581** over 24 months (Oct 2024 – Sep 2026) |
| Net income | €2,394.09/mo average · €2,447.81 in Sep 2026 |
| Account balance | €4,723.58 |
| Auto-invest | €230.55/mo |
| Average monthly spend | €2,097.58 |
| Flex spending, Sep 2026 | **€611.76** (+49% vs her €410.15 average) |

**Demo scenario:** a €231.82 appliance repair lands just before a €150 Helen electricity bill is due. The repair is a scenario for the demo, not a transaction in her history.

---

## What S-Koutsi does

Three tabs inside an S-mobiili-style shell:

| Tab | What it shows |
|---|---|
| **3 Baskets** | **Safe-to-Spend this week (€122)**, the three baskets with breakdowns, a grocery benchmark badge (€556/mo vs a €380 single-adult reference), and the S-Bonus level in the header (2.0%, €95 from 2.5%). |
| **S-Koutsi AI** | A chat that answers with **generative UI**: interactive widgets rendered inside the conversation, built from Aino's numbers. |
| **Full Picture (PSD2)** | A simulated Nordea + OP connection that updates total balance (€6,886) and Safe-to-Spend (€96/wk), and finds a **duplicate Netflix subscription** (€13.99/mo). |

**Chat widgets**

- **⚡ Tactical Liquidity Slider:** compares *Pay now* with a *14-day reminder buffer*. Under Finnish debt-collection rules a payment reminder can only be sent 14+ days after the due date and its fee is capped at €5. A late bill or reminder alone does not create a payment default (*maksuhäiriömerkintä*). That takes 60+ days of non-payment plus a formal notice, or a court ruling. The widget shows the real cost of waiting next to a quick loan.
- **🎯 9 PM Punalappu Sniper:** shows how buying red-label (expiring) food after 21:00, when the discount rises from −30% to −60% in participating stores, stretches a food budget.
- **☕ Wolt & Café Cutter:** a slider showing what trimming restaurant and delivery spend compounds into over a year, plus the extra S-Bonus earned when that money is spent at S-Group instead.

Typed questions get an answer based on her numbers (groceries, savings, health, investing, subscriptions, travel, S-Bonus and more).

**🎬 Auto-Play Demo (45s):** a hands-free walkthrough with an on-screen cursor, highlights and step captions, made for screen recording. **↺ Reset Demo** restarts it.

---

## Run it

No build step. Everything is in one file.

```bash
# Option 1: just open it
open index.html            # macOS  (or double-click the file)

# Option 2: local server
python3 -m http.server 8000   # then visit http://localhost:8000
```

**GitHub Pages:** *Settings → Pages → Deploy from a branch → `main` / root*. The demo is then served at `https://<user>.github.io/<repo>/`.

`index.html` loads its libraries from public CDNs, so it needs an internet connection:

| Library | Source |
|---|---|
| React 18.3.1 + ReactDOM | `unpkg.com/react@18.3.1`, `unpkg.com/react-dom@18.3.1` |
| Babel standalone 7 | `unpkg.com/@babel/standalone@7` (compiles the JSX in the browser) |
| Tailwind CSS | `cdn.tailwindcss.com` (Play CDN) |
| Lucide icons | `unpkg.com/lucide-react@0.460.0` (UMD, global `LucideReact`) |

> Lucide's UMD build expects a lowercase `react` global, so `index.html` sets `window.react = window.React` before loading it.

---

## Architecture

```
┌───────────────────────────────────────────────────────────────────────────┐
│ 1 · DATA & CATEGORIZATION (deterministic)                                 │
│   S-Bank transactions ─┐                                                  │
│   PSD2 feeds (AIS) ────┼─► MCC + merchant rules ─► Three Baskets ledger   │
│   StatFin open data ───┘        (Python / DuckDB, auditable SQL)          │
└──────────────────────────────────────┬────────────────────────────────────┘
                                       │ typed JSON facts
┌──────────────────────────────────────▼────────────────────────────────────┐
│ 2 · LLM ORCHESTRATION                                                     │
│   Intent → tool selection → deterministic tool call → grounded wording    │
│   The model never computes a number. It only chooses tools and explains.  │
└──────────────────────────────────────┬────────────────────────────────────┘
                                       │ { widget, props }
┌──────────────────────────────────────▼────────────────────────────────────┐
│ 3 · GENERATIVE UI                                                         │
│   Tool output hydrates a pre-approved component library inside the chat   │
│   (LiquiditySlider · PunalappuSniper · CutSavings · BasketCard …)         │
└──────────────────────────────────────┬────────────────────────────────────┘
                                       │
┌──────────────────────────────────────▼────────────────────────────────────┐
│ 4 · S-MOBIILI INTEGRATION                                                 │
│   New tab in the existing bottom navigation · WebView / React Native      │
└───────────────────────────────────────────────────────────────────────────┘
```

**Design principles**

- **Numbers come from code, words come from the model.** Every figure on screen can be traced to a deterministic function and the input rows behind it.
- **A closed widget set.** The model can only request components that compliance has already reviewed. It cannot invent new UI.
- **Coaching, not lending.** The cheapest legal option comes first. Credit products are shown only as a cost comparison.
- **Zero shame.** Copy is written to reduce anxiety ("let's sort it out calmly"), and every insight ends in a concrete action.

---

## The Three Baskets: categorizing CUST00001's 581 transactions

Each card transaction carries a **Merchant Category Code (MCC)**. Categorization runs in two passes:

1. **MCC → category.** A lookup table maps each MCC to one of 12 spending categories.
2. **Category → basket**, decided by *mandatory vs discretionary* and *regular vs irregular*:

| Basket | Rule | Categories |
|---|---|---|
| **1 · The Bedrock** | Mandatory & regular | Housing, Groceries, Utilities, Transportation |
| **2 · Life Happens** | Mandatory & irregular | Health, Home & Tech |
| **3 · The Flex Zone** | Discretionary | Lifestyle, Entertainment, Travel, Restaurants |
| *Savings & Investments* | Excluded from spend | Investment deposits |

**MCC codes seen in CUST00001's data**

| Category | MCC codes | Example merchants |
|---|---|---|
| Groceries | 5411, 5422, 5451, 5499 | S-market Vallila, Prisma Tripla, Alepa, K-Group, Lidl |
| Health | 8011 (doctors), 5912 (pharmacies) | Terveystalo Kamppi, Yliopiston Apteekki, Apteekki Hakaniemi |
| Restaurants | 5812, 5814 | Restaurants, fast food |
| Entertainment | 7922, 7996, 7999 | Events, amusement, recreation |
| Transportation | 4111, 5541, 7523 | HSL transit, fuel, parking |

Housing, utilities, lifestyle, travel and home & tech follow standard MCC groups (for example utilities 4900, telecom 4814, insurance 6300). Rent and other bank transfers without an MCC are classified with merchant and recurrence rules.

**Result for CUST00001 (monthly averages over 24 months)**

| Basket | Avg / month | Share | Breakdown |
|---|---|---|---|
| 🪨 Bedrock | **€1,309.33** | 62.4% | Housing €520.11 · Groceries €555.66 · Utilities €118.03 · Transport €115.53 (fixed bills only: €638.14) |
| 🩹 Life Happens | **€378.10** | 18.0% | Health €329.80 · Home & Tech €48.29 (Aug 2026 spike: €902.48) |
| 🌊 Flex Zone | **€410.15** | 19.6% | Lifestyle €155.26 · Entertainment €135.01 · Travel €71.40 · Restaurants €48.48 (Sep 2026: €611.76) |

**Safe-to-Spend this week**

```
(latest income − Bedrock − Life Happens reserve − investing) ÷ 4.33 weeks
(€2,447.81 − €1,309.33 − €378.10 − €230.55) ÷ 4.33 ≈ €122 / week
```

With PSD2 enabled, recurring outflows found at other banks are subtracted too (€112.88/mo of simulated bills → **€96/week**).

---

## Zero-hallucination AI: deterministic function calling

The LLM is an **orchestrator and explainer**, never a calculator. It receives the user's message and a set of strictly typed tools, picks one, and turns the returned JSON into plain language. The UI renders the tool's numbers directly, not the model's restatement of them.

```ts
get_basket_summary(customer_id: string, months?: number)
  → { bedrock, life_happens, flex, shares, latest_month, fixed_bills }

get_safe_to_spend(customer_id: string, include_psd2?: boolean)
  → { weekly, monthly_free, components: { income, bedrock, reserve, investing, external_bills } }

simulate_14day_delay(bill_amount: number, shock_amount: number, delay_days: number)
  → { buffer_if_paid_now, buffer_if_delayed, late_interest_eur, reminder_fee_eur /* 0 or max 5 */,
      quick_loan_cost_eur, credit_record_impact: "none" }

calc_sgroup_evening_savings(budget_eur: number, red_label_share: number, after_21: boolean)
  → { food_value_eur, savings_eur, multiplier }

calc_bonus_level(monthly_sgroup_purchases_eur: number)
  → { level_pct, next_tier_eur, next_level_pct, est_bonus_eur, payment_benefit_eur }

detect_duplicate_subscriptions(accounts: string[])
  → [{ merchant, amount_eur, accounts, annual_saving_eur }]
```

**Example flow:** *"How do I survive this €231 repair and still pay my €150 Helen electricity bill?"*

1. The model classifies the intent as a liquidity shock and extracts the amounts (`shock=231`, `bill=150`).
2. It calls `simulate_14day_delay(150, 231, 14)`, which returns buffer −€2.90 if paid now vs €147.10 kept until payday, a reminder fee of at most €5, and no credit-record impact.
3. It responds with `{ widget: "LiquiditySlider", props: {...} }` plus two empathetic sentences grounded in those values.
4. The client renders the reviewed `LiquiditySlider` component, and the user explores it hands-on.

**Guardrails:** schema-validated tool I/O · numbers rendered only from tool output · a closed widget set · factual rule text (14-day reminder, €5 cap, credit-record thresholds) stored as reviewed copy, not generated · no product sales in crisis flows.

---

## S-Group retail synergy

A financial coach is also a retail growth engine when the bank and the stores share one customer.

**CUST00001's grocery wallet (24 months: €13,335.79)**

| | €/month | Share |
|---|---|---|
| S-market | 140.04 | |
| Prisma | 113.51 | |
| Alepa | 51.65 | |
| **S-Group total** | **305.21** | **54.9%** |
| K-Group, Lidl & specialty stores | **250.45** | **45.1%** (€6,010.80 over 24 months) |

**Coaching levers**

- **9 PM Punalappu strategy.** When cash is tight, S-Koutsi suggests moving grocery runs to Prisma or Alepa after 21:00, when red-label discounts double to −60% in participating stores. Shifting the €250.45 competitor spend this way saves **up to ~€150/mo** if every switched purchase is a red-label item. It also cuts food waste.
- **S-Bonus level-up.** Under the standard S Group Bonus table (1% from €50/mo up to 5% from €900/mo, in €100 steps), Aino is at **2.0%**. Moving €95 lifts her to 2.5%. Moving the full €250.45 lifts her to **3.0%**, with no extra spending.
- **Win-back value:** up to **€3,005/yr** of grocery spend recaptured from a single customer like Aino.

---

## Prototype vs production

| Layer | This repo (hackathon prototype) | Production design |
|---|---|---|
| Data | Pre-aggregated CUST00001 figures embedded in `index.html` | Live S-Bank ledger + PSD2 AIS feeds + StatFin API |
| Categorization | Results of the MCC → basket mapping | Python/DuckDB pipeline with versioned rule tables |
| AI | Front-end keyword intent matching that calls the same deterministic calculations | LLM with function calling over the tools above |
| UI | React 18 + Tailwind, single file | Pre-approved component library in S-mobiili (WebView / React Native) |
| PSD2 | Simulated Nordea & OP sandbox data | Licensed AISP consent flow (180-day renewal) |

The prototype keeps the production principle: **every number shown is computed by deterministic code from the dataset, never generated.**

---

## Data notes & assumptions

- **Source:** the S-Pankki challenge dataset, customer `CUST00001`, 581 transactions, 2024-10-03 → 2026-09-30.
- **Simulated:** the €231.82 appliance repair (demo scenario), the Nordea & OP balances and bills, and the duplicate Netflix subscription.
- **Illustrative assumptions:** payday 10 days after the bill's due date · late interest ≈ 9% p.a. (reference rate + 7 pp, estimated) · quick-loan comparison uses fees + the 15% p.a. interest cap · fund return 5% p.a. (not guaranteed).
- **Grocery benchmark:** the €380/mo single-adult reference should be checked against the latest Statistics Finland household budget data before production use.
- **Red-label discounts:** timing and depth vary by store. The app tells users to check the shelf label.
- **S-Bonus:** the level is estimated from S-Group grocery spend in the dataset. Other S Group purchases (ABC, restaurants, hotels) would raise it.

---

## Repository

```
.
├── index.html   # complete interactive prototype (React 18 + Babel + Tailwind + Lucide via CDN)
└── README.md
```

Built at **Junction 2026** for the **S-Pankki Personal Economic Trainer** challenge.
