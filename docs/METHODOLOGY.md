# HKSPS Hour Methodology (v1)

How a rendered hour is counted. Published on the public methodology page and stored in `hour_policies`.

1. **Source.** The ledger (`duty_hours`) is the only source of rendered hours. Credits add; reversals subtract. Entries exist only for approved applications.
2. **Raw time.** End time minus start time, in seconds.
3. **Minimum.** An entry shorter than `min_duty_minutes` (default 30) is rejected with a message to the teacher.
4. **Rounding.** Raw time is rounded to the nearest `rounding_minutes` (default 15) when the entry is written.
5. **Cap.** Credited time on one application cannot exceed `cap_per_application_hours` (default 16). An entry that would pass the cap is trimmed to the remaining allowance, and rejected when none remains.
6. **Reversal.** A reversal carries the negative of the credit it reverses and a required reason. A credit can be reversed once.
7. **Term.** An entry counts toward the scholarship record whose term contains `duty_date`.
8. **Completion.** Completion percent is rendered hours divided by required hours, to two decimals. A record is complete when rendered hours are at least the required hours.
9. **Versioning.** Each entry stores the policy version applied. A new version takes effect at its `effective_from` and applies only to entries written afterward.

## Values

| Setting | Default |
|---|---|
| `min_duty_minutes` | 30 |
| `rounding_minutes` | 15 |
| `cap_per_application_hours` | 16 |
