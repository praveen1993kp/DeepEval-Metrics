# LLM Evaluation Framework — Metric Coverage and Parity Documentation

## 1. Objective

This document provides a complete implementation-level analysis of metric coverage across the framework and captures the backend–frontend parity work completed for the DeepEval integration.

Scope covered:
- System architecture understanding for metric execution flow
- Complete metric inventory in backend and frontend
- Gap analysis and parity findings
- Implemented changes for backend-only metric exposure in frontend
- Validation and rollout plan for production readiness

---

## 2. Architecture Summary

The framework follows a three-tier evaluation model:

1. **Frontend (React + TypeScript)**
	- Collects metric input payload (query/output/context/expected_output)
	- Applies metric-aware field validation and conditional form behavior
	- Sends requests to backend API endpoint

2. **Backend (Node.js + Express + TypeScript)**
	- Validates request payload by metric requirements
	- Normalizes payload and forwards to Python provider
	- Returns frontend-friendly response shape with score/verdict/explanation

3. **Provider Layer (FastAPI + DeepEval)**
	- Executes metric-specific evaluators
	- Produces score + verdict + explanation
	- Supports single metric, multi-metric, and `all` execution mode

Primary runtime route:
`Frontend -> POST /api/eval-only -> Python /eval`

---

## 3. Metric Inventory

## 3.1 Backend Metrics (Source of Truth)

Implemented in backend/provider:

1. `faithfulness`
2. `answer_relevancy`
3. `contextual_precision`
4. `contextual_recall`
5. `pii_leakage`
6. `bias`
7. `hallucination`
8. `ragas`

Additional backend mode:
- `all` (evaluate all supported metrics in a single request)

## 3.2 Frontend Metrics (Before Parity Change)

Previously exposed in UI:

1. `faithfulness`
2. `answer_relevancy`
3. `contextual_precision`
4. `contextual_recall`
5. `pii_leakage`
6. `bias`
7. `hallucination`

Missing:
- `ragas`

---

## 4. Coverage Matrix

| Metric | Backend | Frontend (Before) | Frontend (After) | Notes |
|---|---|---|---|---|
| faithfulness | ✅ | ✅ | ✅ | Output + context driven |
| answer_relevancy | ✅ | ✅ | ✅ | Query + output driven |
| contextual_precision | ✅ | ✅ | ✅ | Requires context + expected_output; output hidden |
| contextual_recall | ✅ | ✅ | ✅ | Requires context + expected_output; output hidden |
| pii_leakage | ✅ | ✅ | ✅ | Privacy metric |
| bias | ✅ | ✅ | ✅ | Fairness metric |
| hallucination | ✅ | ✅ | ✅ | Requires query + output + context |
| ragas | ✅ | ❌ | ✅ | Composite metric now exposed |

Result:
- **Before:** 7/8 parity
- **After:** **8/8 parity**

---

## 5. Implemented Parity Changes

## 5.1 Frontend Contract Updates

- Added `ragas` to metric union type.
- Updated metric documentation comments for clarity.

File updated:
- `frontend/src/components/LLMEval/types.ts`

## 5.2 Frontend Validation and Form Behavior

- Added `ragas` to provider metric list.
- Added `ragas` to `expected_output` visibility rule.
- Added `ragas` to context-required logic.
- Added `ragas` to `expected_output` required-field validation.

File updated:
- `frontend/src/components/LLMEval/validation.ts`

## 5.3 Batch Metric Selector Updates

- Added `ragas` to `METRICS_CONFIG` with required fields:
  - `query`
  - `output`
  - `context`
  - `expected_output`
- Included a user-facing description for composite scoring behavior.

File updated:
- `frontend/src/components/BatchEval/MetricSelector.tsx`

## 5.4 Backend Status Metadata Alignment

- Updated backend health/status metrics arrays to include `ragas` for observability accuracy.

File updated:
- `backend/src/index.ts`

---

## 6. Metric Input Contract (Operational)

| Metric | query | output | context | expected_output |
|---|---|---|---|---|
| faithfulness | Optional | Required | Required in practice | Not required |
| answer_relevancy | Required | Required | Optional | Not required |
| contextual_precision | Required | Not used | Required | Required |
| contextual_recall | Optional | Not used | Required | Required |
| pii_leakage | Required | Required | Optional | Not required |
| bias | Required | Required | Optional | Not required |
| hallucination | Required | Required | Required | Not required |
| ragas | Required | Required | Required | Required |

Note:
- Contextual metrics intentionally do not require output in UI/backend validation flow.
- `ragas` remains output-dependent and expected-output dependent.

---

## 7. Validation and QA Checklist

Recommended verification steps:

1. Open LLM Eval screen and verify `ragas` appears in metric dropdown.
2. Select `ragas` and confirm:
	- Expected Output field is visible and required.
	- LLM Output field remains visible and required.
	- Context list is required.
3. Submit valid `ragas` payload and confirm response contains score/verdict/explanation.
4. Submit invalid `ragas` payloads (missing query/output/context/expected_output) and confirm clear frontend validation errors.
5. Re-test existing contextual metrics to ensure no regressions in output visibility behavior.
6. Call backend `/health` and `/api/status` and verify `ragas` appears in metrics metadata.

---

## 8. Final Outcome

- Backend/frontend metric parity has been upgraded from **7/8 to 8/8**.
- Composite `ragas` metric is now implementation-ready and visible to users.
- Metric capability reporting is now consistent in backend status endpoints.
- The framework now provides complete frontend exposure for all currently supported backend evaluation metrics.

