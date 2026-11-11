# Query Tuning Runbook Template

## Summary

- Query / endpoint:
- Owner:
- Date:
- Database/version:
- Environment:
- User or business impact:

## Workload context

- Invocation rate and peak rate:
- Expected result size:
- Table sizes and approximate row counts:
- Representative bind parameters:
- Latency target:
- Concurrent workload:
- Recent schema/index/statistics changes:

## Baseline

- SQL statement:
- EXPLAIN output:
- EXPLAIN (ANALYZE, BUFFERS) output, if safe:
- Planning time:
- Execution time:
- Estimated versus actual rows:
- Loops:
- Buffers and temporary I/O:
- Existing relevant indexes:
- Lock waits or pool-acquisition waits:

## Hypothesis

State one measurable claim, e.g. “The query sorts a broad set because its index does not match the filter/order pattern.”

## Change

- Query/index/statistics/configuration changed:
- Why this change addresses the hypothesis:
- Rollback plan:
- Write/storage or other-query trade-off:

## Verification

- Test dataset and parameters:
- Before/after plan:
- Before/after latency (include percentiles where possible):
- Before/after buffers/rows/loops:
- Write-path impact:
- Other query regressions:
- Repeated runs and environmental controls:

## Decision

- Accepted/reverted:
- Reason:
- Follow-up monitoring:
- Date to re-evaluate:
