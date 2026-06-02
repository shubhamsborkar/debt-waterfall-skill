# Building the `debt-waterfall` skill — session log

A working session that designed, built, and tested a reusable skill for extracting a company's debt structure from a single SEC filing. Shared for the subscribers as a record of how the skill was scoped and what it does.

**Date:** June 2, 2026
**Participants:** Shubham · Claude (Cowork)
**Artifacts produced:** `debt-waterfall.skill` (the installable skill) · `Carnival_debt_waterfall.md` (a live test run against Carnival's Q1 FY2026 10-Q)

---

## 1. Scoping the job

**Shubham** opened with a one-sentence definition of the skill's job and asked for three sharp questions to expose anything vague:

> "An equity or credit investor, looking at any company with meaningful debt, who needs every instrument, term, and ranking pulled from the filing into one clean, sourced structure before they trust their view of the company."

**Claude** asked three questions targeting the soft spots:

1. **Source scope** — "the filing" is singular, but covenants/security/ranking usually live in indentures and credit agreements, not the 10-K footnote. What is the skill allowed to read?
2. **Field depth** — two instruments with the same coupon and maturity can rank completely differently. How deep does each instrument record need to go to count as "complete"?
3. **Sourcing granularity** — "sourced" can mean one citation per instrument or a page reference behind every value. How traceable must it be?

**Shubham's answers:** 10-K/10-Q only · seniority + security depth · cite + flag conflicts.

---

## 2. Resolving the tension

**Claude** flagged that the three answers were in conflict: a periodic filing alone usually can't support full seniority/security detail or cross-document conflict-flagging — that lives in the indentures. Three ways out were offered:

1. Keep 10-K-only, downgrade the promise (ranking "as labeled," no conflict-flagging).
2. Keep the depth, widen the source to "periodic + key exhibits."
3. Make source a runtime input.

**Shubham chose Option 1**, with two refinements:
- Seniority captured **"as disclosed; route deeper ranking to open items."**
- Conflict-flagging scoped to **"internal to the filing."**

This became the locked spec:

> Pull every debt instrument, its terms, and ranking out of a 10-K/10-Q into one clean, sourced structure. Source: latest periodic filing only. Seniority as disclosed; deeper ranking → open items. Every value carries a citation; flag only contradictions internal to the filing.

---

## 3. First build, then a redesign

Claude first built a skill (`debt-structure-extractor`) producing a single sourced Markdown table. After laying out the full disclosure structure against a real filing, Shubham asked for a cleaner, four-output version named **`debt-waterfall`**, and the earlier skill was retired to avoid trigger overlap.

### What `debt-waterfall` does

Reads **one** SEC filing and produces four sourced outputs:

1. **Debt master table**, ordered by the ranking tiers the filing discloses (secured-guaranteed → unsecured-guaranteed → unsecured-non-guaranteed), closing with the filing's own gross → net → current → long-term reconciliation.
2. **Events ledger** — the period drivers that move the balance (issuance, repayment, conversion, redemption, amendment, FX, amortization), swept from the cash-flow, equity, and footnote sections because no single statement closes the bridge.
3. **Qualitative terms note** — collateral, covenants, cross-default, rate floors, make-whole, change-of-control puts, special interest.
4. **Open-items list** — everything the filing can't establish, phrased as actionable questions pointing at where the answer lives.

### The rules that make it trustworthy

- Reads **only** the filing provided — never outside knowledge about the company.
- Writes **`NOT DISCLOSED`** wherever the filing is silent, and records it in open items. Never fills a gap from memory.
- **Cites page and section** for every figure and every legal term.
- Stays strictly at **extraction** — no valuation, no recommendation, no opinion.
- Works on **any filer in any industry**, because it encodes how debt is *disclosed*, not one company's debt.

---

## 4. Live test — Carnival Q1 FY2026 10-Q

The skill was run against Carnival Corporation & plc's 10-Q (quarter ended Feb 28, 2026). Highlights:

- **19 instruments** mapped across the three disclosed ranking tiers: secured-guaranteed $3,098M → unsecured-guaranteed $21,644M → unsecured-non-guaranteed $1,262M; reconciled gross $26,004M → net $25,290M → long-term $23,788M.
- Because the filing nets unamortized issuance costs **only in aggregate**, per-instrument carrying value was marked `NOT DISCLOSED` rather than back-solved.
- **Events ledger** captured the non-cash items a cash-flow-only read misses — the $1.1B convertible settlement (69.1M shares + $500M cash; $618M to equity) and $27M discount amortization — and honestly marked the FX-vs-repayment split as `NOT DISCLOSED`.
- **Internal conflict flagged:** the convertible is labeled "Dec 2025" in the debt table but "2027 Convertible Notes" in the narrative — both recorded, unreconciled. Everything else tied out.
- **11 open items**, each pointing to the indenture, intercreditor agreement, or the 2025 10-K.

### Provenance check

Shubham challenged whether the "69.1M shares + $500M cash" figure was actually in the filing. It was — in the **Convertible Notes** paragraph of Note 3, page 10, verbatim:

> "In December 2025, we settled $1.1 billion principal amount of the 2027 Convertible Notes, resulting in the issuance of 69.1 million shares of Carnival Corporation common stock and a cash payment of $500 million."

Easy to miss because it's narrative text on page 10, not in the debt schedule table or the cash-flow statement — which is exactly why the skill cites the section, not just the document.

---

## Takeaways for the Subscribers

- The skill is deliberately **narrow**: one filing, extraction only. That's a feature — it's the part you can fully trust without re-reading the document.
- Anything requiring the underlying agreements (lien priority within a tier, call schedules, covenant definitions) is **routed to open items**, never guessed.
- For a complete waterfall with true intercreditor ranking, the open-items list is the checklist of what to pull next (indentures, credit agreement, prior 10-K).
