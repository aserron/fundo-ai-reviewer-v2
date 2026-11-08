# Fundo AI Reviewer (v2 Parallel Track)

We are building an AI-assisted transaction auditing and credit impact pipeline for **Fundo**, a working capital lender for small and medium-sized businesses (SMBs). When a business applies for funding, Fundo analyzes 90 days of Plaid bank transactions using a legacy deterministic keyword engine to classify cash flows into 13 risk and revenue groups, estimate monthly operating revenue, detect competing debt obligations, and compute a credit offer. Because raw bank memos are noisy, overloaded, and prone to regex edge cases, the legacy matcher frequently misclassifies loan disbursements as revenue, wipes out legitimate card settlements due to keyword collisions, or misses non-standard overdraft and Merchant Cash Advance (MCA) signals.

The high-level goal of this project is to design and build a hybrid **Deterministic + LLM Reviewer** that audits the legacy engine's classifications, enforces strict accounting invariants, and quantifies how label corrections shift real underwriting offers and risk exposure. This repository serves as a clean-slate starting point—containing only the domain rules, mathematical formulas, and challenge requirements without pre-baked architectural decisions—so a new implementation track can independently design its stack, generate a synthetic benchmark dataset, and deliver a reproducible, zero-key evaluable solution under a strict `<$10` LLM budget.

## Documents

- **[`CHALLENGE.md`](CHALLENGE.md):** Complete problem statement, Plaid sign conventions, the 13 `FundoGroup` definitions, revenue & credit offer formulas, sensitivity analysis requirements (2%, 5%, 10%), risk questions, and production rollout prompts.
- **[`GEMINI.md`](GEMINI.md):** Agent instructions, domain invariants, and simplicity constraints for autonomous or pair-programmed implementation.

## Implementation Scope (Tracked in GitHub Project)

0. **Step 0 — Architecture, Stack & File Layout Blueprint:** Decide and document the repository file structure, minimal dependencies, caching/offline-replay persistence strategy, Code vs. Model boundary, and key tradeoffs (ADRs) before coding.
1. **Step 1 — Synthetic Benchmark & Legacy Keyword Engine:** Generate a realistic 90-day Plaid dataset (~10 businesses, ~2,000+ transactions) and baseline keyword matcher exhibiting description collisions, counterparty overload, and punctuation edge cases.
2. **Step 2 — Part 1: Hybrid Transaction Reviewer:** Audit legacy labels, enforce deterministic financial invariants & prompt-injection defenses, output calibrated confidence + 5-second explanations, and support zero-key cached replay (`<$10` budget).
3. **Step 3 — Part 2: Credit Impact & Sensitivity Analysis:** Compute before/after underwriting features, apply the offer formula & `NSF > 5` cliff, quantify Revenue & Offer MAE vs. Ground Truth, run 2%/5%/10% mislabel sensitivity analysis, and address the neobank (`R01`) and 61-day window edge cases.
4. **Step 4 — Part 3: Production Rollout, CLI & Executive Deliverable:** Provide one-command CLI execution (including external file review for ad-hoc validation) and author the technical solution report covering shadow promotion SLOs, silent drift detection, bit-for-bit decision reproducibility, and HITL underwriter feedback.
