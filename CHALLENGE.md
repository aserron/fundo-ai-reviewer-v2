# Fundo AI Engineer Take-Home Challenge — Specification

## 1. Background & Business Context

Fundo provides working capital advances to small and medium-sized businesses (SMBs). When a business applies for funding, Fundo ingests **90 days of bank transaction history** via Plaid and runs an automated underwriting pipeline to determine creditworthiness and calculate a funding offer.

Today, transactions are classified by a **legacy deterministic keyword engine** that maps transaction descriptions (`name`) and amounts into **13 risk/revenue groups** (plus `None` for standard unclassified transactions), determines whether each transaction is business-related (`is_business`), and flags whether a credit counts toward operating revenue (`is_revenue`).

### Why the Legacy Engine Needs a Reviewer
In production, bank transaction strings are noisy, truncated, and overloaded. The legacy keyword matcher suffers from three recurring failure modes:

1. **Noisy Bank Descriptions & Rule Precedence Collisions:**
   Payment processors and ACH networks embed generic banking terms inside settlement strings. For example, `ORIG CO NAME:SHOPIFY CO ENTRY DESCR:TRANSFER` contains the keyword `"TRANSFER"`. If the legacy engine evaluates `Internal transfer` before `Revenue verification`, legitimate e-commerce sales are misclassified as internal transfers, wiping out recognized revenue.
2. **Counterparty Overload:**
   A single financial brand often appears across completely different financial products. For instance, `"SQUARE"` or `"SHOPIFY"` can appear in:
   - Daily merchant card processing settlements (true operating revenue).
   - A lump-sum working capital loan disbursement (`SQUARE CAPITAL LOAN` — debt proceeds, **not** operating revenue).
   - Daily or weekly ACH loan repayments (`SQUARE CAPITAL` debit — an `Active advance` obligation).
3. **Regex Word-Boundary & Punctuation Failures:**
   Naive word-boundary regular expressions (e.g., `\bKEYWORD\b`) silently fail when bank descriptors attach non-word punctuation to tokens (such as `*SQ`, `S&P`, `A+`, or `NSF/RET`), causing missed revenue verification, missed MCA debits, or undercounted NSF events.

---

## 2. Domain Rules & Financial Invariants

### 2.1 Plaid Sign Convention (Strict Invariant)
Plaid reports transaction amounts from the perspective of money leaving the account:
- **`amount < 0` (Negative):** **Credit / Inflow** (money entering the account: customer deposits, card processor settlements, loan disbursements received, transfers in).
- **`amount > 0` (Positive):** **Debit / Outflow** (money leaving the account: vendor payments, rent, payroll, NSF fees, daily MCA repayments).
- **`amount == 0` (Zero):** Informational bank event (e.g., fee-free ACH return notices such as `ACH RETURN R01`).

### 2.2 The 13 Keyword Groups (`FundoGroup`)
Every transaction is assigned one of the following 13 groups, or `None`:

1. `Not average monthly revenue` — Non-operating inflows: loan/advance proceeds received, owner capital injections, tax refunds, one-time asset sales.
2. `NSFs` — Non-Sufficient Funds fees, returned item fees, or bounced payment return events (`R01` / `R09`).
3. `Overdraft` — Bank overdraft fees or negative balance penalties.
4. `Internal transfer` — Transfers between the business's own accounts, sweeps, or commingled owner peer-to-peer transfers (e.g., personal Zelle/Venmo transfers).
5. `UCC` — Uniform Commercial Code lien filing fees.
6. `Active advance` — Competing Merchant Cash Advance (MCA) or revenue-based financing daily/weekly ACH debits or split withholdings.
7. `Auto deposit` — Automated check or cash deposits requiring verification.
8. `Revenue verification` — Verified merchant processor settlements, insurance reimbursements, or commercial contract settlements (e.g., Stripe, Square, Toast, Clover, Shopify).
9. `High risk — gambling` — Casinos, sportsbooks, betting platforms (e.g., DraftKings, FanDuel, BetMGM).
10. `High risk — bankruptcy` — Bankruptcy court/trustee fees, restructuring legal payments.
11. `High risk — debt settlement payments` — Debt consolidation or settlement agencies (e.g., Freedom Debt Relief).
12. `High risk — garnishment` — IRS tax levies, state tax liens, court-ordered garnishments.
13. `High risk — other` — Pawn shops, payday lenders, bail bonds, distressed liquidations.
14. `None` — Default category for routine business operating expenses (debits) or standard B2B/B2C customer invoice payments (credits) that do not match any special group above.

### 2.3 The Revenue Invariant
A transaction counts as operating revenue (`is_revenue = True`) **if and only if** all three conditions hold:
1. It is an inflow (`amount < 0`), **AND**
2. It is a business transaction (`is_business == True`), **AND**
3. Its `group` is **NOT** in an excluded credit category (`Not average monthly revenue`, `Internal transfer`, `Active advance`, `NSFs`, `Overdraft`, `UCC`, or any `High risk — *` group).

> **Hard Invariant:** A debit (`amount >= 0`) can **never** be marked as revenue under any circumstances.

---

## 3. Challenge Deliverables

Since real bank data cannot be shared, **you will first generate a realistic synthetic Plaid transaction dataset** (~2,000–2,500 transactions across ~10 small businesses over a 90-day window) junto with a baseline Legacy Keyword Engine that exhibits the flaws described above.

### Part 1 — The Reviewer & The Code/Model Boundary
Build a **Reviewer** that audits the Legacy Keyword Engine's labels on your dataset:
1. **Flagging Doubts:** Identify transactions where the legacy classification (`group`, `is_business`, or `is_revenue`) is likely wrong or ambiguous.
2. **Structured Output:** For each flagged transaction, output:
   - Proposed corrected label (`group`, `is_business`, `is_revenue`).
   - Calibrated confidence score.
   - A concise **5-second underwriter explanation** (why it was flagged and its dollar impact).
3. **Code vs. Model Boundary:** Explicitly define and enforce what stays in deterministic code vs. what is delegated to the LLM. Consider:
   - How to avoid sending every routine transaction to the LLM.
   - How to guarantee financial invariants (such as sign rules and revenue formulas) even if the LLM hallucinates.
   - How to defend against **adversarial prompt injection** inside untrusted bank counterparty memos.
   - How to avoid false alarms on **hard negatives** (e.g., legitimate B2B customer invoices from clients whose company name happens to contain words like `"Capital"`, `"Advance"`, or `"Transfer"`).
   - How to handle **residual uncertainty** when a large inflow memo is genuinely ambiguous (e.g., an unspecified wire or peer-to-peer payment without an invoice reference).
4. **Budget Constraint:** Total LLM API spend across your development and full evaluation run must stay **strictly under $10.00 USD** (ideally orders of magnitude lower via triage, batching, and caching).

---

### Part 2 — Credit Impact & Risk Questions

#### 2.1 Feature Engineering & Offer Calculation
For each business in your dataset, compute the following underwriting features **Before (Legacy Baseline)**, **After (Reviewer Corrected)**, and against **Ground Truth**:
- `avg_monthly_revenue`: Average monthly operating revenue over the statement window.
- `revenue_share_of_deposits`: Ratio of operating revenue to total credit inflows.
- `nsf_overdraft_count`: Total count of `NSFs` and `Overdraft` events.
- `high_risk_share_of_debits`: Share of total debit dollars falling into `High risk — *` categories.
- `other_funders_daily_payment`: Estimated daily payment to competing MCA funders (total `Active advance` debits divided by active banking days in the window, typically `65` banking days per 90 calendar days).

Calculate the **Fundo Credit Offer** for each state using the policy rule:
- If `nsf_overdraft_count > 5`:
  $$\text{Offer} = \$0.00 \quad (\text{Hard rejection cliff})$$
- If `nsf_overdraft_count <= 5`:
  $$\text{Offer} = \max\left(0,\; 1.2 \times \text{avg\_monthly\_revenue} - 20 \times \text{other\_funders\_daily\_payment}\right)$$

#### 2.2 Error Sensitivity Analysis (2%, 5%, 10% Mislabel Rates)
Simulate or analyze how sensitive the underwriting features and final credit offers are when **2%, 5%, and 10%** of transaction labels are corrupted/mislabeled. Compare the asymmetric financial cost of:
- False `revenue` errors (inflating or deflating monthly revenue).
- False `Active advance` errors (missing debt stacking vs. false MCA flags).
- False `NSFs` errors (crossing the binary `NSF > 5` rejection cliff).

#### 2.3 Two Short Risk Questions
Answer both questions in your technical write-up (and reflect them in your dataset/features where appropriate):
1. **Zero-NSF Fee Banks (Neobanks):** Many modern business bank accounts (e.g., Mercury, Relay, Novo) do not charge $35 NSF fees. How does `nsf_count = 0` bias a risk model trained on traditional banks, and how would you detect cash-flow distress and bounced payments in ingestion/feature engineering?
2. **61-Day Statements on a 90-Day Model:** What breaks if an applicant submits only 61 days of bank history into a pipeline that assumes a 90-day window (e.g., dividing total revenue by 3.0 months or counting raw NSF totals), and how should the data contract / feature engine handle it?

---

### Part 3 — Production Rollout & Governance (1 Page)
Write a concise 1-page production design covering:
1. **Shadow Architecture & Promotion Gate:** How to deploy the Reviewer alongside the legacy engine without disrupting live underwriting, and the quantitative SLOs required to promote it.
2. **Silent Input Drift:** How to detect when bank descriptor formats shift or new LLM/keyword versions degrade category distributions over time.
3. **Decision Reproducibility:** How to guarantee that any historical underwriting offer can be audited and reproduced bit-for-bit months later.
4. **Underwriter Human-in-the-Loop (HITL) Feedback:** How to prioritize underwriter attention (so they aren't flooded with low-impact flags) and feed human overrides back into the system.

---

## 4. Engineering & Evaluation Principles

1. **One-Command Reproducibility:** The evaluator must be able to run your full pipeline, tests, and evaluation in **one or a few copy-paste commands** from the terminal, including an offline/cached mode that does not require a live API key.
2. **Simplicity & Focus on Real Business Requirements:** Discard any technology, framework, infrastructure daemon, GUI, authentication layer, or design pattern whose only justification is making the code look more "enterprise" or modular. Every line of code should directly serve transaction auditing accuracy, financial invariant safety, credit impact analysis, or testability.
3. **AI Tool Disclosure:** Include a brief note in your deliverable disclosing what AI coding assistants/models you used and how you verified correctness.
