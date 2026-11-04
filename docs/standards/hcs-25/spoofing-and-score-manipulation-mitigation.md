# HCS-25 Spoofing and Score Manipulation Mitigation

*Working analysis for Agent Trust Scores Discussion*

## 1. Objective and Scope

**Objective.** Define practical mitigation mechanisms for score manipulation driven by massive fake reviews, fabricated or multiplied evidence, bots, spam, Sybil identities, and coordinated activity. The analysis is intentionally narrow: it identifies the main attack patterns, the properties needed to resist them, and candidate controls that HCS-25 or its adapters can require or recommend.

**Scope boundary.** This is not a rewrite of HCS-25 and does not attempt to solve adjacent WG deliverables such as the full trust-signal taxonomy, score-calculation guidance, minimum evidence requirements, decay/appeal handling, or registry display policy. It only addresses how score-impacting signals can be cheaply manufactured, duplicated, coordinated, or amplified and how that influence can be constrained. The current HCS-25 draft already recognizes that trust scores can be gamed, calls for Sybil resistance for reputation and feedback signals, and recommends limits, validation, and normalization patterns designed to resist gaming. The current ERC-8004 feedback adapter also combines rating sentiment with feedback volume, making manipulation of review volume directly relevant to score integrity. [1][2]

## 2. Threat Model

An attacker may be able to create or control identities, agents, reviewer accounts, interactions, evidence artifacts, timing, or upstream data sources. The central failure mode is not merely false content; it is cheap multiplication of score influence.

**The analysis therefore separates five properties:** authenticity (did the claimed source produce it), eligibility (may it affect this score), uniqueness (is it a distinct underlying event), independence/correlation (does it add distinct evidentiary support), and influence (how much effect may it have). A signature can establish authenticity without establishing independence. A real transaction can be self-dealing. Ten artifacts can still represent one event.

**Important note:** independence must not be approximated by novelty, rarity, or distance from prior observations. An attacker could pad the history with mediocre observations and then inject one extreme event that appears highly “informative,” or flood the history to dilute a legitimate high-impact event. Severity should be policy-defined; independence should be grounded in source/control/event lineage. Robust-statistics work similarly treats unbounded influence from contaminated observations as a failure mode. [5]

## 3. Primary Attack Surface and Mitigation Mechanisms

| Attack pattern | Failure mode | Mitigation mechanisms | Residual limitation |
| --- | --- | --- | --- |
| Fake reviews / Sybil review farms | One controller creates many reviewer identities or feedback records so volume appears to represent broad support. | Bind feedback to a qualifying interaction where feasible; define one score-influencing observation unit; do not treat distinct accounts/keys as proof of independence; cap or diminish repeated-source/repeated-counterparty influence; use identity or credential assurance as an optional strengthening signal, not a universal requirement. | Common control cannot be proven generically. Controls should limit the benefit of cheap multiplicity rather than claim perfect Sybil detection. |
| Bots and spam flooding | Automated agents generate large volumes of low-cost observations, interactions, or feedback to raise a score or dilute adverse evidence. | Rate limits for infrastructure plus separate score-influence limits; qualifying-event criteria; deduplication; per-source/per-window caps; diminishing returns; deterministic admission rules. | Rate limiting alone is anti-abuse, not trust assurance. A well-funded attacker may still generate valid events. |
| Replay and duplicate representations | The same review, transaction, event, or evidence is resubmitted or imported through multiple paths. | Stable event/observation identifiers where available; canonical deduplication keys; deterministic equivalence rules; provenance linking to the underlying event; reject replay as additional score influence. | Cross-system equivalence can be difficult when identifiers or lineage are missing. |
| Fake or multiplied evidence | An attacker creates fabricated evidence or fragments one real event into many receipts, attestations, files, or claims so artifact count appears to equal evidence count. | Verify authenticity/provenance where possible; define the natural evidence unit; group artifacts by underlying event/claim; require references to source event or evidence lineage; count groups rather than artifacts when appropriate. | Authentic artifacts may still be self-generated or correlated; provenance does not itself establish independence. |
| Wash activity / self-dealing | Controlled or colluding parties conduct genuine interactions primarily to manufacture eligible feedback or evidence. | Interaction binding plus repeated-pair limits; counterparty-diversity measures; economic or minimum-substance requirements where domain-appropriate; graph/correlation analysis as implementation guidance. | The interaction can be real even when the motive is manipulation; intent is generally not directly observable. |
| Coordinated review or evidence rings | Several real or apparently distinct parties coordinate to produce mutually reinforcing observations. | Do not infer independence from identity count; group or bound correlated sources; detect repeated reciprocal patterns; preserve source lineage; allow independence assurance to remain unknown when it cannot be established. | Legitimate users may also be correlated, so correlation should constrain influence rather than automatically invalidate evidence. |
| Cross-provider amplification | Multiple providers expose ratings or aggregates derived from the same underlying population, creating false corroboration. | Require or recommend source-lineage / population-basis metadata for aggregates; identify known overlap; group dependent providers or cap combined influence rather than count each as independent support. | Raw populations may be unavailable or private; overlap may remain uncertain. |
| Padding, dilution, and extreme-event injection | Attacker first creates many mediocre observations to alter the baseline, then introduces an extreme malicious event; or floods ordinary observations to dilute a legitimate severe event. | Do not reward novelty by itself; use bounded per-observation influence; policy-defined severity classes; grouped/diminishing repeated observations; robust aggregation where appropriate. | Rare catastrophic events still need a deterministic path to high influence when policy says they are severe. |

## 4. Mitigation Model

- **Define the observation unit.** If quantity can increase trust, the system must define what counts once. Review records, signatures, transactions, and artifacts should not automatically be treated as independent observations.
- **Bind observations to underlying events where feasible.** Interaction or event binding raises the cost of fabricated feedback and supports uniqueness checks, but must not be treated as proof of independence.
- **Make replay and deduplication deterministic.** The same underlying event should not gain additional influence because it is replayed, repackaged, or received through another path.
- **Do not equate identity multiplicity with independence.** Different accounts, keys, agents, or signatures prove distinct identifiers or control points, not necessarily distinct controllers. [3]
- **Bound the influence of multiplicity and correlation.** Caps, grouping, diminishing returns, repeated-counterparty limits, and robust aggregation can ensure that inexpensive repetition does not create unbounded trust.
- **Preserve provenance and uncertainty.** Where source lineage, uniqueness, or independence cannot be established, that state should remain unknown rather than silently treated as verified. Multiple providers exposing signals derived from overlapping populations should not automatically be treated as independent corroboration. [4]

## 5. Candidate HCS-25 Requirements / Guidance for Review

- Where observation quantity affects a trust component, the adapter or configuration should define the score-influencing observation unit.
- Replay or duplicate representations of the same underlying observation should not independently increase score influence.
- Distinct identifiers, accounts, keys, signatures, or providers should not by themselves be treated as evidence of independence.
- Interaction-bound feedback may be used to establish eligibility and uniqueness, but should not by itself establish reviewer independence.
- Repeated or correlated observations should be subject to bounded, grouped, or diminishing influence where multiplicity can otherwise materially change the score.
- Aggregate providers should expose enough lineage or population-basis information for consumers to identify known overlap or uncertainty where feasible.
- Admission, deduplication, grouping, and influence-limiting rules that affect the score should be deterministic and versioned with the applicable scoring configuration.

## 6. Questions for the Working Group

- Which of the above controls should be normative HCS-25 requirements versus adapter/provider guidance?
- Should HCS-25 explicitly require an observation unit whenever volume or count can raise a score?
- Should replay resistance and duplicate suppression be normative for score-impacting observations?
- How should HCS-25 represent unknown versus verified independence or source-lineage assurance?
- What minimum metadata should an aggregate provider expose so dependent or overlapping populations are not silently double-counted?

## References

[1] Hashgraph Online, “HCS-25: AI Trust Score Methodology,” Draft, https://hol.org/docs/standards/hcs-25/

[2] Hashgraph Online, “HCS-25 Feedback Scorecard Adapter (erc8004-feedback),”
https://hol.org/standards/ai-trust-score/adapters/erc8004-feedback

[3] J. R. Douceur, “The Sybil Attack,” in Peer-to-Peer Systems: First International Workshop, IPTPS 2002, pp. 251–260, 2002. DOI: 10.1007/3-540-45748-8_24.

[4] X. L. Dong, L. Berti-Équille, and D. Srivastava, “Integrating Conflicting Data: The Role of Source Dependence,” Proceedings of the VLDB Endowment, vol. 2, no. 1, pp. 550–561, 2009. DOI: 10.14778/1687627.1687690.

[5] E. Cantoni and X. de Luna, “Semiparametric inference with missing data: Robustness to outliers and model misspecification,” Econometrics and Statistics, vol. 16, pp. 108–120, 2020. DOI: 10.1016/j.ecosta.2020.01.003.
