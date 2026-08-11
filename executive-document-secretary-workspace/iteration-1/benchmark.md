# Skill Benchmark: executive-document-secretary

**Execution**: Local orchestrator fallback, non-independent
**Date**: 2026-08-11T15:55:09Z
**Evals**: 1, 2, 3 (1 run each per configuration)

## Summary

| Metric | With Skill | Without Skill | Delta |
|--------|------------|---------------|-------|
| Pass Rate | 100% ± 0% | 40% ± 30% | +0.60 |
| Time | 0.0s ± 0.0s | 0.0s ± 0.0s | +0.0s |
| Tokens | 1967 ± 958 | 1169 ± 136 | +797 |

## Interpretation

All configured sub-agent runs failed before producing tokens because the active OMO `writing` category referenced unavailable `opencode/hy3-free`. The outputs were produced locally as an explicit fallback. The 100% versus 40% result validates the assertions and illustrates the intended behavioral contrast; it is not independent evidence of model-level improvement.

The budget baseline passed three of four checks, with the cited `Format basis` as the differentiator. The tone assertion in the missing-intake test passed in both configurations and is weakly discriminating by itself. Time and token comparisons are not meaningful; the token column contains output-character proxies.
