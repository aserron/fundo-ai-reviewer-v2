# Agent & Engineering Guidelines — Fundo AI Reviewer (v2 Parallel Track)

## Purpose of This Repository
This repository is a clean-slate starting point for implementing the **Fundo AI Engineer Challenge** ([`CHALLENGE.md`](CHALLENGE.md)).

It intentionally contains **only the problem specification and domain rules**—with zero pre-decided code architecture, module layout, or synthetic data—so that an independent engineering track can design, test, and benchmark a solution from scratch.

## Core Constraints & Guardrails for Agents

1. **Strict Domain Invariants First:**
   - **Plaid Sign Convention:** `amount < 0` is an inflow/credit; `amount > 0` is an outflow/debit.
   - **Revenue Invariant:** A debit (`amount >= 0`) can **never** count as operating revenue. Inflows only count as revenue when `is_business == True` and `group` is not in an excluded credit category.
   - **No Ground-Truth Leakage:** If you embed ground-truth labels in your synthetic dataset for evaluation, the Legacy Engine and the Reviewer must **never** read the ground-truth fields during classification or review.

2. **Radical Simplicity (Anti-Overengineering):**
   - Do **not** add web UIs, authentication/login systems, external database daemons, Docker containers, or heavy LLM wrapper frameworks unless strictly justified by a functional requirement in [`CHALLENGE.md`](CHALLENGE.md).
   - Prioritize fast, deterministic testability and one-command CLI execution.

3. **LLM Cost & Offline Reproducibility:**
   - Keep total API spend well below the **$10.00 USD** limit.
   - Persist/cache LLM evaluation results so a reviewer can reproduce all outputs and metrics offline without an API key.

4. **Controlled Technical Communication (Anti-Slop):**
   - Back every claim in your deliverable report with measured numbers from your pipeline outputs (MAE, precision/recall, dollar offer shift, token spend).
   - Avoid vague marketing adjectives; state exact mechanisms, formulas, and failure modes.

## Recommended Execution Sequence

0. **Step 0 — Architecture & Tradeoff Blueprint (`ARCHITECTURE.md` / ADRs):** Before writing code, define the file layout, dependency choices, caching/persistence mechanism (e.g., SQLite vs. JSONL vs. custom store), batching/deduplication strategy, and TDD boundaries. Document pros and cons for each choice.
1. **Step 1 — Synthetic Benchmark & Baseline Keyword Engine:** Generate the 90-day Plaid dataset covering all required failure modes and edge cases, plus the baseline 13-group keyword engine.
2. **Step 2 — Hybrid Transaction Reviewer (Part 1):** Implement pre-triage, prompt-injection sanitization, LLM review with caching/telemetry, and deterministic post-LLM financial guardrails.
3. **Step 3 — Credit Impact & Sensitivity Engine (Part 2):** Compute Before/After/Ground-Truth features, offer shifts, MAE reductions, and 2%/5%/10% mislabel simulations.
4. **Step 4 — CLI Runner, Production Rollout & Solution Report (Part 3):** Expose one-command offline/live execution + external file review, and author the final technical deliverable.
