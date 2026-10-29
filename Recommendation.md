# Part 4 — Utility Recommendation

**Proposal: Pilot opt-in, off-peak laundry scheduling insights.** Help customers identify convenient times to run washing machines outside locally defined peak-demand or higher-tariff periods. This may reduce grid peaks and, where time-of-use pricing exists, household bills. **Shifting a wash does not itself save energy.** Do not automate appliance control.

## Evidence and opportunity

Part 2 analysed four UK REFIT homes (January–March 2015) and found **307 candidate washing-machine cycles**. Pooled, exposure-adjusted starts were highest around **08:00 UK time** and on **Saturday**, but households differed. These are usage trends—not proof that those hours are grid peaks. Suggestions must depend on each customer's routine and **local** tariff or demand peaks.

## Measuring real-world value

Run a consent-based pilot against a randomly assigned control group. Offer an occasional personalised timing suggestion, not instant model-triggered alerts. Measure:

- **Primary:** change in peak-window household kWh and peak kW relative to baseline and controls.
- **Secondary:** proportion of *confirmed* washer cycles shifted, applicable bill savings, engagement and opt-outs.
- **Validation:** ask a sample of participants to confirm cycles or use temporary appliance-level monitoring; report missed events and false alerts.

A reduction in peak demand must be **measured**, not inferred from predicted washer power.

## Practical limitations

Other appliances can resemble washer loads. REFIT also contains missing and forward-filled measurements and `Issues` flags: **293 of 307 candidate cycles** overlapped at least one flagged minute. The Part 3 validation-selected Random Forest produced false positives and **40.9% / 75.7% daily absolute energy error** on evaluation Houses 7 and 10. Seq2Point improved some results but was not consistently better on validation. These estimates are unsuitable for **billing or guaranteed savings claims**.

REFIT homes are in the **UK**. Indian appliance types, tariffs, routines, climate and housing may differ; test locally before deployment and respect customer privacy and consent.

## If only 15- or 30-minute readings are available

Averaging obscures short heating peaks, ON/OFF transitions and overlapping appliance signatures, weakening cycle detection and washer-specific energy estimates. **Thirty-minute readings lose more detail than fifteen-minute readings.** Household-level peak shifting can still be assessed, but individual washer attribution needs finer or appliance-level validation data.

*Evidence: `RESULTS.md`, cleaned REFIT documentation and the Part 3 Seq2Point comparison. This is a proposed pilot, not a measured savings result.*
