# Three-Perspective Analysis Framework

Used in **Phase 4** of the Daily Business Research skill.
Each news story is analyzed through three distinct lenses before the final article rewrite.

---

## Perspective 1: CEO

**Who**: Chief Executive Officer of a mid-to-large Thai fabric/yarn buying company or manufacturer.

**Mindset**: Strategic risk management, shareholder value, long-term market positioning, board-level narratives.

**Key questions to answer**:
1. Does this news change our **market position** relative to competitors?
2. What is the **financial exposure** (revenue, margin, working capital)?
3. Should we **accelerate or delay** major procurement contracts?
4. Is there a **reputational or ESG angle** the board needs to know?
5. What should we **communicate** to investors, the board, or major customers?

**Output format** (3–5 bullets):
```
• [CEO] {Insight tied to a specific story} — {strategic implication in ≤ 20 words}
```

**Tone**: Concise, risk-aware, forward-looking. Avoids operational detail. Focuses on decisions only a CEO can make.

---

## Perspective 2: Corporate Strategist

**Who**: VP Strategy or Head of Procurement Strategy at the same company.

**Mindset**: Competitive intelligence, supplier ecosystem, scenario planning, cost-to-serve optimization.

**Key questions to answer**:
1. How does this news affect our **supplier negotiation leverage**?
2. Should we **diversify sourcing geography** in response?
3. What **hedging or contractual mechanisms** (FX, price-lock, volume commitment) apply?
4. Which **competitors are better or worse positioned** given this news?
5. What **3–6 month scenario** does this create (base / upside / downside)?

**Output format** (3–5 bullets):
```
• [Strategist] {Insight tied to a specific story} — {tactical implication in ≤ 20 words}
```

**Tone**: Analytical, scenario-driven, supplier-relationship-aware. References frameworks like Porter's Five Forces, VUCA, supply chain resilience models when appropriate.

---

## Perspective 3: Competitor

**Who**: The strategic planning team at a rival fabric/yarn buyer or manufacturer.

**Mindset**: Exploit market disruption, capture market share, identify vulnerabilities in incumbent players.

**Key questions to answer**:
1. How can a nimble competitor **exploit this news faster** than incumbents?
2. Does this create a **window to poach customers or suppliers**?
3. What **pricing moves** might rivals make in the next 30–60 days?
4. Are there **geographic or product niches** this news opens up?
5. What **signals should we watch** to know if a competitor is moving?

**Output format** (3–5 bullets):
```
• [Competitor] {Threat or opportunity a rival sees} — {counter-move to consider in ≤ 20 words}
```

**Tone**: Provocative, adversarial-thinking. Forces the reader to think from outside their own company.

---

## Synthesis Rules (Phase 5 Rewrite)

When merging all three perspectives into the final article:

1. **Do not use perspective labels** in the final text. Perspectives are woven into prose, not listed as headers.
2. **Prioritize convergence**: if all three perspectives agree on an implication, lead with it — it is the most certain signal.
3. **Highlight divergence**: if CEO and Strategist disagree on action timing, or if Competitor analysis contradicts what the CEO would assume, call it out explicitly as a tension point.
4. **Action Items** (end of article) should reflect the union of all three perspectives — some items will be CEO-level decisions, others procurement-team execution tasks.

---

## Impact Score Rubric

Used in Phase 2 and Phase 1 deduplication.

| Score | Criteria |
|---|---|
| 9–10 | Global supply disruption, >10% price movement, affects multiple countries/materials |
| 7–8 | Regional disruption, 5–10% price movement, multiple suppliers affected |
| 5–6 | Domestic impact, 2–5% price movement, single material or geography |
| 3–4 | Trend signal, <2% movement, single supplier or niche material |
| 1–2 | Background noise, no actionable data, speculation only |

A story may be re-published from a previous day if its current `impact_score` exceeds the previously published score by **≥ 2 points**.

---

## Action Items Format

Each item in the `## Action Items` section follows this template:

```
[ ] {Verb phrase describing action} — {Suggested owner role} — {Deadline or trigger condition}
```

Examples:
```
[ ] Lock in Q3 yarn contract before price index resets — Head of Procurement — by end of this week
[ ] Request FX rate lock from treasury for USD-denominated yarn imports — CFO / Treasury — within 48h if THB weakens past 36.5
[ ] Survey top-3 fabric suppliers on inventory levels — Procurement Analyst — rolling weekly until disruption resolves
[ ] Brief CEO on margin exposure from fuel surcharge increase — Procurement Director — before next board meeting
[ ] Evaluate Vietnam as secondary sourcing country for polyester yarn — Category Manager — within 2 weeks
```
