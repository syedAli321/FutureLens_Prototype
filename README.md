# Leasepac FutureLens — Prototype

**Intelligent financial scenario simulation and explainable deal analysis.**

FutureLens is a proposed, read-only decision-support enhancement for Leasepac. It lets a credit or relationship officer examine a financing agreement, see what is wrong with it, simulate alternative repayment arrangements, and produce a decision brief — *before* any operational change is made.

> **Understand today, simulate tomorrow, decide with confidence.**

This repository contains a single-file, working prototype built for the Fintech Group Hackathon 2026. It runs entirely in the browser using fabricated data. It is **independent of Leasepac's source code, database and APIs**.

---

## Contents

- [The problem and the idea](#the-problem-and-the-idea)
- [Quick start](#quick-start)
- [Using the prototype](#using-the-prototype)
- [Features](#features)
- [How the calculations work](#how-the-calculations-work)
- [Deal X-Ray rules](#deal-x-ray-rules)
- [Sample data](#sample-data)
- [Project structure](#project-structure)
- [Extending the prototype](#extending-the-prototype)
- [Hosting it (GitHub Pages)](#hosting-it-github-pages)
- [Implementation plan for real Leasepac integration](#implementation-plan-for-real-leasepac-integration)
- [Limitations and honest boundaries](#limitations-and-honest-boundaries)
- [Privacy and data handling](#privacy-and-data-handling)

---

## The problem and the idea

Leasepac already provides financial calculations, agreement management, credit assessment and restructuring. But comparing financing alternatives can mean moving between separate workflows and interpreting the financial consequences by hand.

FutureLens adds one unified, **read-only** workspace with two complementary capabilities:

| Capability | What it does |
|---|---|
| **Deal X-Ray** | Explainable diagnostics: repayment exposure, coverage inconsistencies, missing credit evidence, incomplete documentation. Every warning shows the rule, the evidence and a suggested human action. |
| **FutureLens Simulator** | Interactive what-if scenarios (term extension, upfront principal prepayment), saved and compared side by side against the untouched original agreement. |

It then produces a **Decision Brief** that is explicitly *not* an approval and is meant to feed existing review and authorization workflows.

---

## Quick start

There is nothing to install or build.

1. Download or clone the repository:
   ```bash
   git clone https://github.com/syedAli321/FutureLens_Prototype.git
   cd FutureLens_Prototype
   ```
2. Open `FutureLens_Prototype.html` in any modern browser (Chrome, Edge, Firefox, Safari).
   - Double-click the file, **or**
   - serve it locally if you prefer:
     ```bash
     python3 -m http.server 8000
     # then open http://localhost:8000/FutureLens_Prototype.html
     ```
3. JavaScript must be enabled. No internet connection, packages or accounts are needed.

**Requirements:** a current desktop or mobile browser. No Node, no build step, no external libraries or network calls.

---

## Using the prototype

The left sidebar is the main navigation. A good end-to-end path (this matches the recorded demo):

1. **Overview** — see remaining principal, current monthly installment and the number of diagnostic warnings for the active agreement.
2. **Agreement explorer** — search or filter the three synthetic agreements and select **DEMO-LEASE-001 (Amani Logistics Ltd)**.
3. **Deal X-Ray** — review the two findings (repayment stress, insurance gap). Click **View evidence and action** on a finding to see its rule, source evidence and suggested review action.
4. **Scenario lab** — use presets (**+6 months**, **+12 months**, **Prepay 10%**) or the slider and prepayment field. The live result and the repayment-trajectory chart update immediately.
5. **Save scenarios** — name a scenario, optionally add an officer note, and click **Save to comparison**.
6. **Compare outcomes** — see the baseline and saved scenarios side by side: monthly payment, immediate prepayment, total remaining outflow, difference versus baseline, and modeled interest.
7. **Decision briefs** — enter the reviewing team and rationale, select up to three saved scenarios, and click **Generate / refresh preview**. Copy the text, download an HTML report, or print/save as PDF.
8. **Method & limitations** — read the formulas, what is and is not implemented, run the built-in calculation self-checks, view the local activity log, or clear the demo workspace.

You can also click **▶ Start guided demo** at the bottom of the sidebar for an in-app tour.

---

## Features

- **Read-only environment.** The baseline agreement is immutable. Simulations never alter it.
- **Agreement explorer** with search, status filter (healthy / stressed / exception) and a per-agreement detail panel.
- **Deal X-Ray** with four deterministic, transparent rules and evidence-linked explanations.
- **Scenario lab** for term extension (0–24 months) and immediate principal prepayment, with input validation and a live recalculation.
- **Repayment trajectory chart** comparing baseline and scenario balances.
- **Scenario portfolio and comparison table** with a CSV export.
- **Decision brief generator** with copy, HTML download and print-to-PDF; always labelled *"Not an approval"*.
- **Activity log** and snapshot export for traceability.
- **Self-checks** for the calculation engine (Method & limitations page).
- **Responsive layout** with a collapsible menu on small screens.

---

## How the calculations work

The prototype uses a standard level-payment amortization model with a fixed nominal annual rate and payments in arrears.

```
r        = annual nominal rate / 12
n        = remaining months
payment  = P × r / (1 − (1 + r)^−n)        (if r = 0:  payment = P / n)
```

For a scenario with a term extension `e` and an immediate prepayment `X`:

```
Revised principal   = P − X
Revised term        = n + e
Revised installment = payment(P − X, rate, n + e)
Total outflow       = revised installment × (n + e) + X
Difference vs base  = total outflow − (original installment × n)
```

**Validation rules**

- Extension must be a whole number from 0 to 24 months.
- Prepayment must be ≥ 0 and strictly less than the remaining principal.
- Rate must be between 0 and 100%, and term between 1 and 360 months.

**Not modeled:** fees, taxes, penalties, capitalized arrears, residual/balloon values, moratoria or any product-specific policy. See [Limitations](#limitations-and-honest-boundaries).

---

## Deal X-Ray rules

| Code | Rule | Triggers when | Evidence field |
|---|---|---|---|
| DX-01 | Repayment stress | `missed installments > 0` (High if ≥ 2, otherwise Medium) | `missedInstallments` |
| DX-02 | Potential insurance coverage gap | insurance expiry is before agreement maturity | `insuranceEnd`, `agreementEnd` |
| DX-03 | Credit evidence unavailable | credit grade is missing | `creditGrade` |
| DX-04 | Documentation incomplete | documents verified is false | `documentsVerified` |

Every rule is deterministic and tied to a field in the snapshot. Findings are review signals for a human, **not** automated credit decisions or default predictions.

---

## Sample data

All records are fictitious.

| ID | Customer | Facility | Principal (KES) | Rate | Term | Profile |
|---|---|---|---|---|---|---|
| DEMO-LEASE-001 | Amani Logistics Ltd | Commercial vehicle finance | 1,200,000 | 14% | 36 mo | Stressed: 2 missed installments, insurance ends before maturity |
| DEMO-LEASE-002 | Meridian Agro Services | Agricultural equipment lease | 820,000 | 12% | 30 mo | Healthy: no findings |
| DEMO-LEASE-003 | Coastal Transit Co. | Fleet vehicle finance | 1,700,000 | 16% | 48 mo | Exception: missing credit grade, incomplete documents, coverage issue |

---

## Project structure

```
FutureLens_Prototype/
├── FutureLens_Prototype.html   # The entire app: HTML, CSS and JavaScript in one file
└── README.md
```

Inside the HTML file, the JavaScript is organized as follows (search for these names):

| Name | Purpose |
|---|---|
| `DATA` | Synthetic agreement records |
| `RULES` | Catalog of Deal X-Ray rules shown in the UI |
| `calcPmt(p, rate, term)` | Amortization payment formula with input checks |
| `evaluate(a, extension, prepay)` | Baseline vs scenario results and totals |
| `diagnose(a)` | Runs the X-Ray rules and returns findings |
| `state` / `persist()` | App state and browser-local persistence (`localStorage` key `futurelens-studio-v2`) |
| `render*()` functions | Page renderers (overview, agreements, X-Ray, simulator, comparison, briefs) |
| `briefData()` / `briefMarkup()` | Decision brief content and layout |

---

## Extending the prototype

**Add an agreement.** Append an object to `DATA`:

```js
{id:'DEMO-LEASE-004', customer:'Example Ltd', facility:'Vehicle finance',
 description:'Short description.', principal:900000, rate:13, months:24,
 missed:0, insuranceEnd:'2028-01-01', agreementEnd:'2028-06-01',
 creditGrade:'A3', documents:true, kind:'healthy',
 lastReview:'2026-11-01', product:'Vehicle lease'}
```

`kind` must be `healthy`, `stressed` or `exception` for the filter to work. Update the count badge in the sidebar if you want it to show the new total.

**Add a diagnostic rule.**
1. Add an entry to `RULES` (code, name, basis, field, limitation).
2. Add the matching check inside `diagnose(a)` that pushes `{code, severity, name, text, evidence, action}`.
3. Update the "From 4 deterministic checks" / "4 of 4" labels in `renderXray()`.

**Add a scenario lever** (for example, a rate change): extend `evaluate()` and its validation, add the input in the Scenario lab markup, include it in the saved scenario object and comparison table, and add a self-check.

---

## Hosting it (GitHub Pages)

Because the app is a single static file, it can be published for free:

1. Rename `FutureLens_Prototype.html` to `index.html` (or keep the name and link to it directly).
2. In the repository go to **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**, select `main` and `/ (root)`, then save.
4. After a minute the demo is live at `https://<your-username>.github.io/FutureLens_Prototype/`.

---

## Implementation plan for real Leasepac integration

This prototype shows the user experience and the logic. A production version would be built in phases:

1. **Read-only data access.** Connect to Leasepac agreement, schedule, arrears, insurance and document data through approved read APIs or a reporting replica, with Leasepac authentication and role-based access.
2. **Replace the model with Leasepac's calculation engine.** Call or mirror the product-specific amortization, fee, tax, balloon and arrears logic instead of the generic formula, and reconcile results against Leasepac schedules.
3. **Expand and govern the rule catalog.** Move rules to configuration, add institution-specific policy checks, version them, and have credit policy owners sign them off.
4. **Persist scenarios and briefs server-side.** Replace browser `localStorage` with an authenticated store and a real audit trail (who, when, which inputs).
5. **Hand off into existing workflows.** Link an approved brief to Leasepac's restructuring and approval process. FutureLens stays read-only and never posts changes.
6. **Validation and rollout.** Parallel-run against known restructurings, security review, user acceptance testing with officers, then a pilot.

---

## Limitations and honest boundaries

- Uses **synthetic data only**; no real customer, balance or transaction is involved.
- The amortization model is **illustrative and has not been certified** against Leasepac's production calculations.
- **No** Leasepac authentication, live database, API, network access or financial posting.
- **No** balloon payments, taxes, fees, penalties, capitalized arrears, residuals or product-specific rules.
- **No** automated underwriting, credit approval or default-prediction model. Diagnostics and briefs require independent human review.
- Scenarios are stored in the browser only and are lost if site data is cleared.

---

## Privacy and data handling

- Everything runs client-side. There are no external packages, trackers or outbound requests.
- Names and amounts are fictitious.
- Saved scenarios, selections, draft context and the activity log live in your browser's `localStorage`. Use **Method & limitations → Clear demo workspace** to reset.

---

## Context

Prepared for the Fintech Group Hackathon 2026 as an independent innovation proposal for Leasepac. FutureLens complements, and does not replace, Leasepac's established approval and posting processes.
