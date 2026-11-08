# Fundo AI Reviewer (v2 Parallel Track)

Clean-slate specification repository for the **Fundo AI Engineer Challenge**.

## Documents

- **[`CHALLENGE.md`](CHALLENGE.md):** Complete problem statement, Plaid sign conventions, the 13 `FundoGroup` definitions, revenue & credit offer formulas, sensitivity analysis requirements (2%, 5%, 10%), risk questions, and production rollout prompts.
- **[`GEMINI.md`](GEMINI.md):** Agent instructions, domain invariants, and simplicity constraints for autonomous or pair-programmed implementation.

## Implementation Scope (Tracked in GitHub Project)

0. **Step 0 — Architecture, Stack & File Layout Blueprint:** Decide and document the repository file structure, minimal dependencies, caching/offline-replay persistence strategy, Code vs. Model boundary, and key tradeoffs (ADRs) before coding.
1. **Step 1 — Synthetic Benchmark & Legacy Keyword Engine:** Generate a realistic 90-day Plaid dataset (~10 businesses, ~2,000+ transactions) and baseline keyword matcher exhibiting description collisions, counterparty overload, and punctuation edge cases.
2. **Step 2 — Part 1: Hybrid Transaction Reviewer:** Audit legacy labels, enforce deterministic financial invariants & prompt-injection defenses, output calibrated confidence + 5-second explanations, and support zero-key cached replay (`<$10` budget).
3. **Step 3 — Part 2: Credit Impact & Sensitivity Analysis:** Compute before/after underwriting features, apply the offer formula & `NSF > 5` cliff, quantify Revenue & Offer MAE vs. Ground Truth, run 2%/5%/10% mislabel sensitivity analysis, and address the neobank (`R01`) and 61-day window edge cases.
4. **Step 4 — Part 3: Production Rollout, CLI & Executive Deliverable:** Provide one-command CLI execution (including external file review for live debriefs) and author the technical solution report covering shadow promotion SLOs, silent drift detection, bit-for-bit decision reproducibility, and HITL underwriter feedback.
