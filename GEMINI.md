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
