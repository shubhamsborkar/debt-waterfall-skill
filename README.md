# Debt Waterfall Skill

A reusable Claude skill that reads a single SEC filing (10-K or 10-Q) and
extracts a company's full debt structure into four sourced outputs: a debt
master table ordered by ranking, an events ledger, a qualitative-terms note,
and an open-items list.

It works on any filer in any industry, because it encodes how debt is
*disclosed*, not any one company's debt. It reads only the filing you give it,
cites the page and section for every figure, marks anything missing as
NOT DISCLOSED, and routes it to open items. It does not value, recommend, or
opine. It extracts.

Built and explained in this edition of Alpha with AI:
👉 [https://ai.shikshannivesh.com/p/i-taught-claude-to-pull-a-companys]

## What's in this repo

- `debt-waterfall.md` — the skill file. This is the artifact.
- `outputs/` — a real, verified extraction run on Carnival Corporation's
  Q1 FY2026 10-Q, as a worked example.
- `transcript.md` — the full Claude Cowork conversation: defining the job,
  pressure-testing it, writing the skill, running it, and the moment a number
  I thought was wrong turned out to be real and sourced.

## How to use it

1. Open Claude Cowork (or Claude with skills enabled).
2. Add `debt-waterfall.md` to your skills.
3. Drop a company's 10-K or 10-Q into your working folder.
4. Ask Claude to run the debt-waterfall skill on that filing.
5. Take the Markdown output into Excel (Claude in Excel) for analysis.

## Important: verify the output

This skill is built to make verification easy, not to replace it. Every figure
it produces carries a citation back to the page it came from. Always check the
numbers against the filing yourself. AI can miss or misread; the citations are
there so you can confirm every cell in seconds. On material positions, read the
primary documents the open-items list points you to.

## Disclaimer

This skill and its outputs are for educational and research purposes only. They
do not constitute investment advice or a financial recommendation. The skill
reads only the filing provided and cannot capture terms held in indentures,
credit agreements, or other documents a filing merely references. Always verify
disclosures against primary sources and consult qualified professionals before
making investment or business decisions.

---

Built by Shubham Borkar · Alpha with AI · shikshannivesh.com
