---
step_id: MLOPS-S01
step_name: Model Validation
workflow_id: MLOPS001
sequence: 1
type: "validation"
agent: "Linh (MLOps)"
description: "Validate model quality, data distribution, A/B test plan"
estimated_time: "40-50 minutes"
---

# Step 01: Model Validation & Quality Assurance

## Objective

Validate that ML model is production-ready:
1. Confirm performance metrics meet business SLOs
2. Validate no data drift between training and production data
3. Confirm A/B testing and rollback strategy
4. Document model version and metadata
5. Gate entry to deployment phase

## Execution Flow

### Substep 1A: Model Overview

As Linh (MLOps Lead), gather model information:

```
"Hi! I'm Linh, your MLOps engineer for this deployment.

Before we deploy your model to production, we need to validate it's ready:
- Performance metrics meet expectations
- Training data matches production data distribution
- We have a safe way to test and rollback

Let me walk you through the validation process."
```

Collect:
- Model name and version
- Problem statement (classification, regression, ranking, etc.)
- Business objective and success metrics
- Model training date and last validation

### Substep 1B: Performance Metrics Validation

Evaluate model quality against baselines:

**Classification Models**:
- [ ] Accuracy (overall correctness)
- [ ] Precision (false positives)
- [ ] Recall (false negatives)
- [ ] F1-score (balance)
- [ ] ROC-AUC (discrimination)
- [ ] Confusion matrix documented

**Regression Models**:
- [ ] MAE (mean absolute error)
- [ ] RMSE (root mean square error)
- [ ] R² (variance explained)
- [ ] Residual analysis (no systematic bias)

**Ranking Models**:
- [ ] NDCG@K (normalized discounted cumulative gain)
- [ ] MRR (mean reciprocal rank)
- [ ] MAP (mean average precision)

**Prompt for Data Scientist**:
> "What are the main metrics for your model? What was the performance on hold-out test data? Are there per-segment metrics we should track?"

**Scoring Criteria**:
- **PASS**: Metrics meet or exceed baseline; no unexpected degradation per segment
- **CONDITIONAL**: Metrics slightly below baseline but acceptable with monitoring
- **FAIL**: Metrics significantly below baseline; block deployment until retrained

---

### Substep 1C: Data Drift Detection

Validate training data matches production distribution:

**Statistical Tests**:
- [ ] Kolmogorov-Smirnov test on numeric features (p > 0.05)
- [ ] Chi-square test on categorical features
- [ ] Compare feature distributions: training vs production sample
- [ ] Check for missing/new values not seen in training

**Feature Analysis**:
- [ ] Feature ranges match training data (no OOV values)
- [ ] Missing value percentage similar to training
- [ ] Feature correlations stable
- [ ] No new categories in categorical features

**Prompt for Data Engineer**:
> "What's the most recent production data sample? Have we checked if it's similar to training data? Are there any new values or edge cases we haven't seen?"

**Scoring Criteria**:
- **PASS**: No significant data drift detected; KS/chi-square p-values > 0.05
- **CONDITIONAL**: Minor drift detected but within tolerance; plan monitoring
- **FAIL**: Significant drift detected; block deployment; retrain with recent data

---

### Substep 1D: Latency & Throughput Validation

Validate serving performance requirements:

**Load Testing**:
- [ ] Model inference latency (p50, p95, p99)
- [ ] Throughput (requests/second)
- [ ] Memory usage per inference
- [ ] GPU/CPU utilization under load
- [ ] Batch vs single-instance performance

**Prompt for Team**:
> "What's the expected latency SLO for this model? (e.g., p99 < 100ms) How many requests/second do you expect? Do we need GPU or will CPU suffice?"

**Scoring Criteria**:
- **PASS**: Meets latency SLO; throughput sufficient for expected load
- **CONDITIONAL**: Slightly above target latency but acceptable with optimization
- **FAIL**: Cannot meet latency/throughput SLOs; requires infrastructure upgrade

---

### Substep 1E: A/B Testing & Rollback Plan

Verify testing strategy:

**A/B Testing Plan**:
- [ ] Control baseline defined (old model or heuristic)
- [ ] Experiment design documented (% traffic, duration, metrics)
- [ ] Ramp-up strategy planned (5% → 10% → 50% → 100%)
- [ ] Success criteria defined (metric threshold for expansion)
- [ ] Failure criteria defined (metric threshold for rollback)
- [ ] Monitoring/alerting for A/B metrics

**Rollback Strategy**:
- [ ] Instant rollback to previous version (< 1 min)
- [ ] Model versions versioned (semantic versioning or git hash)
- [ ] Previous model artifacts stored and validated
- [ ] Rollback procedure documented and drilled

**Prompt for Product/Engineering**:
> "How will we test this model with real users? What percentage of traffic should we send to the new model initially? If something goes wrong, how fast can we rollback?"

**Scoring Criteria**:
- **PASS**: A/B test plan clear and executable; rollback quick and tested
- **CONDITIONAL**: A/B plan partial; need more details on ramp-up strategy
- **FAIL**: No A/B test plan or rollback strategy; cannot deploy safely

---

### Substep 1F: Model Metadata & Versioning

Document model for reproducibility:

```markdown
## Model Metadata

**Model ID**: sentiment-classification-v2.1.4
**Training Date**: 2026-03-10
**Training Data**: reviews_2026-01-01_to_2026-03-10 (125K samples)
**Framework**: PyTorch 2.0
**Model Size**: 450 MB
**Input Shape**: Variable length text (max 512 tokens)
**Output**: 3-class logits + softmax probabilities

**Performance (Test Set)**:
- Accuracy: 92.5%
- Precision: 0.925
- Recall: 0.922
- F1: 0.924

**Data Drift**: No significant drift detected (KS p=0.23)
**Latency (p99)**: 45ms (CPU), 12ms (GPU)
**Throughput**: 2,000 req/sec (GPU), 200 req/sec (CPU)

**Known Issues**: N/A
**Approved By**: Data Lead (Jane), MLOps Lead (Linh)
**Approval Date**: 2026-03-17
```

---

### Substep 1G: Validation Scorecard & Gate Decision

Summarize validation results:

```markdown
## Model Validation Gate

| Dimension | Status | Notes |
|-----------|--------|-------|
| Performance Metrics | ✅ PASS | Accuracy 92.5% (target: 90%) |
| Data Drift | ✅ PASS | KS p=0.23 (threshold: 0.05) |
| Latency | ✅ PASS | p99=45ms (target: 100ms) |
| Throughput | ✅ PASS | 2K req/sec (need 500 req/sec) |
| A/B Test Plan | ✅ PASS | Ramp-up strategy defined |
| Rollback Plan | ✅ PASS | Can rollback < 1 min |

**Gate Decision**: ✅ **PASS — Model approved for deployment**

**Conditions**: None
**Approval**: Data Lead (Jane) + MLOps Lead (Linh)
```

### Substep 1H: Transition to Step 02

Confirm and proceed:

```
"Excellent! Model validation is complete. All gates passed.

In Step 02, I'll set up the serving infrastructure:
- Deploy model to serving platform (KServe, BentoML, or similar)
- Setup GPU optimization if needed
- Configure versioning and traffic routing
- Prepare for A/B test deployment

Ready to proceed with infrastructure setup?"
```

**Proceed to [Step 02](step-02-deploy.md)**

## Deliverables from This Step

1. **Model Validation Report**
   - Performance metrics summary
   - Data drift analysis
   - Latency/throughput baseline
   - A/B test plan
   - Rollback strategy

2. **Model Metadata Document**
   - Version, framework, size, architecture
   - Input/output specifications
   - Training data provenance
   - Known issues or limitations

3. **Validation Gate Decision**
   - PASS/FAIL decision
   - Approval signatures
   - Conditions (if any)

---

**Navigation**: [← Back to Workflow Overview](../workflow.md), [Next: Step 02 →](step-02-deploy.md)
