---
step_id: MLOPS-S03
step_name: Monitor
workflow_id: MLOPS001
sequence: 3
type: "monitoring"
agent: "Linh (MLOps)"
description: "Setup monitoring, data drift detection, cost tracking, post-deployment"
estimated_time: "40-50 minutes"
is_final_step: true
---

# Step 03: Post-Deployment Monitoring & Drift Detection

## Objective

Establish comprehensive monitoring and drift detection:
1. Setup real-time performance monitoring (Langfuse, MLflow)
2. Configure data drift alerts
3. Setup cost tracking and optimization
4. Document incident response procedures
5. Plan model retraining triggers

## Execution Flow

### Substep 3A: Real-time Performance Monitoring

Setup metrics collection:

**Langfuse Integration** (for LLM/inference monitoring):
```python
from langfuse import Langfuse

langfuse = Langfuse(
    public_key="pk_...",
    secret_key="sk_..."
)

# Log inference
response = model.predict(input_text)

langfuse.trace(
    name="sentiment_classification",
    input={"text": input_text},
    output={"label": response["label"], "confidence": response["confidence"]},
    metadata={
        "model_version": "2.1.4",
        "latency_ms": 45,
        "user_id": user_id,
        "timestamp": datetime.now()
    }
)
```

**MLflow Model Registry**:
```python
# Log model performance metrics
mlflow.log_metric("accuracy", 0.925, step=0)
mlflow.log_metric("precision", 0.928, step=0)
mlflow.log_metric("recall", 0.922, step=0)

# Register model version
mlflow.register_model(
    model_uri="runs:/{{run_id}}/model",
    name="sentiment-classifier"
)
```

**Key Metrics Dashboard**:
```
Real-time Model Performance (Last 24 Hours)
├─ Model Accuracy: 92.5% (target: 90%)
├─ Latency p99: 52ms (target: 100ms)
├─ Request Rate: 2.5K req/min
├─ Error Rate: 0.01% (alerts if > 0.1%)
├─ Model Version Distribution:
│  ├─ v2.0.3 (baseline): 90% traffic, 91.8% accuracy
│  └─ v2.1.4 (canary): 10% traffic, 92.5% accuracy
└─ Inference Cost: $245/day (0.025¢ per inference)
```

### Substep 3B: Data Drift Detection

Monitor for input data changes:

**Statistical Drift Checks**:
```python
# Compare production data to training distribution
def check_data_drift(production_sample, training_stats):
    drift_detected = False
    drift_report = {}

    for feature in production_sample.columns:
        # Kolmogorov-Smirnov test
        ks_stat = ks_test(
            production_sample[feature],
            training_stats[feature]['distribution']
        )

        if ks_stat.pvalue < 0.05:  # Significant drift
            drift_detected = True
            drift_report[feature] = {
                "ks_statistic": ks_stat.statistic,
                "p_value": ks_stat.pvalue,
                "severity": "high"
            }

    return drift_detected, drift_report
```

**Drift Monitoring Strategy**:
- **Hourly check** — Compare last hour of data to training stats
- **Alert threshold** — If KS p-value < 0.05 for any feature
- **Action** — If drift detected, trigger model retraining workflow
- **Rollback** — If accuracy drops > 5%, auto-rollback to previous version

**Example Drift Alert**:
```
Data Drift Detected!

Drift Summary:
├─ Feature: review_text_length
│  ├─ Training mean: 156 chars
│  ├─ Production mean: 98 chars (-37%)
│  ├─ KS p-value: 0.002 (significant)
│  └─ Action: Alert team, plan retraining
│
├─ Feature: user_verified
│  ├─ Training dist: 60% True, 40% False
│  ├─ Production dist: 45% True, 55% False
│  ├─ KS p-value: 0.008 (significant)
│  └─ Action: Alert team, investigate cause
│
└─ Recommendation: Retrain model with recent data (last 2 weeks)
```

**Drift Detection Configuration**:
```yaml
# drift_config.yaml
checks:
  - name: kolmogorov_smirnov
    frequency: hourly
    threshold_pvalue: 0.05
    baseline: training_distribution

  - name: population_stability_index
    frequency: daily
    threshold_psi: 0.25

  - name: accuracy_degradation
    frequency: hourly
    threshold: 0.05  # If accuracy drops > 5%

actions:
  on_drift_high: [alert_team, scale_canary]
  on_accuracy_drop: [alert_team, prepare_rollback]
```

### Substep 3C: Cost Tracking & Optimization

Monitor inference costs:

**Cost Metrics**:
```
Model Inference Cost (Monthly)
├─ GPU hours: 720h @ $1.50/h = $1,080
├─ Inference count: 2.5M inferences
├─ Cost per inference: $0.00043
├─ Monthly spend: $1,080
├─ Forecasted annual: $12,960

Cost Drivers:
├─ Canary traffic (10%): $108/month
├─ Baseline model (90%): $972/month
└─ Overhead (monitoring): $0

Optimization Opportunities:
├─ Compress model (reduce GPU memory): Save ~30%
├─ Batch processing: Save ~40%
├─ Use CPU for inference: Save ~80% (but slower)
└─ Route low-confidence to heuristic: Save ~15%
```

**Cost Alerts**:
- Alert if daily spend > 120% of baseline ($3)
- Alert if cost per inference > target threshold
- Weekly cost report with YTD trends

### Substep 3D: Incident Response Procedures

Document how to handle issues:

**Issue: High Latency (p99 > 150ms)**
```
Symptom: p99 latency spiked from 52ms to 180ms
Triage:
  1. Check model version distribution (is canary overloaded?)
  2. Check GPU utilization (is GPU saturated?)
  3. Check request rate (sudden traffic spike?)
  4. Check data drift (new features causing slowdown?)

Response:
  IF canary is bottleneck:
    - Reduce canary traffic from 10% → 5%
    - If still high, rollback to v2.0.3
  IF GPU saturated:
    - Add more GPU replicas (auto-scale)
    - Consider model quantization to reduce memory
  IF traffic spike:
    - Expected? If yes, scale up as planned
    - If unexpected, investigate cause
```

**Issue: Accuracy Drop**
```
Symptom: Model accuracy dropped from 92.5% to 87.3%
Triage:
  1. Check if it's canary or baseline
  2. Check data drift (new data distribution?)
  3. Verify metric calculation (bugs in logging?)
  4. Check for systemic issues (feature missing, schema change?)

Response:
  IF it's canary with > 5% drop:
    - Immediate auto-rollback to v2.0.3
    - Notify Data Science team
    - Retrain model with recent data
  IF it's baseline (should never happen):
    - Investigate immediately (data corruption?)
    - Possible production data issue
```

**Issue: Data Drift Detected**
```
Symptom: KS p-value < 0.05 for multiple features
Trigger: Automatic alert + notification to team

Response:
  SHORT-TERM:
    1. Acknowledge alert, gather context
    2. Investigate cause (new user segment? seasonality?)
    3. Monitor accuracy closely (may still be OK)

  MID-TERM:
    1. Schedule model retraining with recent data
    2. Validate new model on recent data sample
    3. Deploy if performance improves

  LONG-TERM:
    1. Setup automated retraining pipeline
    2. Plan for periodic model updates (weekly? monthly?)
```

### Substep 3E: Monitoring Dashboard Setup

Create comprehensive dashboards:

**Dashboard 1: Model Health**
```
Real-time Model Performance
├─ Requests: 2,345 req/min (↑5% vs yesterday)
├─ Latency: p99 = 52ms (target: 100ms) ✅
├─ Error Rate: 0.01% (target: < 0.1%) ✅
├─ Model Accuracy: 92.5% (target: 90%) ✅
├─ Data Drift: ✅ No drift detected
├─ Active Versions:
│  ├─ v2.0.3: 90% traffic (accuracy: 91.8%)
│  └─ v2.1.4: 10% traffic (accuracy: 92.5%)
└─ Status: 🟢 Healthy
```

**Dashboard 2: Data Quality**
```
Data Drift & Quality Metrics
├─ Feature Distributions (vs Training):
│  ├─ review_text_length: ✅ Stable (KS p=0.34)
│  ├─ user_verified: ✅ Stable (KS p=0.18)
│  ├─ review_rating: ⚠️ Minor drift (KS p=0.08)
│  └─ timestamp: N/A (expected to vary)
├─ Missing Values:
│  ├─ review_text: 0.02% (expected: < 0.1%)
│  ├─ user_verified: 0.00%
│  └─ review_rating: 0.05%
└─ Last Check: 2026-03-17 14:32 UTC
```

**Dashboard 3: Cost Tracking**
```
Model Inference Costs
├─ Today: $3.45 (avg: $2.98)
├─ This Week: $19.23 (avg: $2.75/day)
├─ This Month: $87.45 (pace: $2,624/month)
├─ Cost per Inference: $0.00043
├─ GPU Utilization: 65% (target: 70-80%)
└─ Efficiency Score: 8.5/10 (good)
```

### Substep 3F: Model Retraining Trigger

Define when to retrain:

**Automatic Retraining Triggers**:
1. **Schedule-based** — Weekly: Every Monday at 9 AM
2. **Performance-based** — If accuracy drops > 5%
3. **Data drift-based** — If KS p-value < 0.05 for 3+ features
4. **Volume-based** — If >= 100K new inferences since last training
5. **Manual** — Anytime team requests retraining

**Retraining Pipeline**:
```
Trigger Condition Met
  ↓
Fetch Recent Training Data (last 2 weeks)
  ↓
Validate Data Quality
  ↓
Train New Model
  ↓
Validate Performance (Must improve or match baseline)
  ↓
IF improved:
  Deploy as Canary (5-10% traffic)
  Monitor for 24 hours
  If stable, ramp up to 100%
ELSE:
  Archive model
  Alert team (debugging needed)
```

**Example Retraining Config**:
```yaml
# retraining_config.yaml
schedule:
  - cron: "0 9 * * MON"  # Every Monday 9 AM
    name: "weekly_retrain"

triggers:
  accuracy_drop:
    threshold: 0.05  # 5% drop
    window: "7d"     # Last 7 days

  data_drift:
    metric: "kolmogorov_smirnov"
    threshold: 0.05  # p-value

  volume:
    count: 100000    # New inferences

validation:
  must_improve: false  # Can match baseline
  min_accuracy: 0.90
  max_latency_ms: 100

deployment:
  strategy: "canary"
  initial_traffic: 0.05  # 5%
  ramp_schedule:
    - "2h:  10%"
    - "6h:  25%"
    - "12h: 50%"
    - "24h: 100%"
```

### Substep 3G: Team Communication & Runbooks

Document operational procedures:

**On-call Runbook**:
```markdown
# Model Deployment On-Call Runbook

## 1. Morning Standup (Daily)
- Check dashboards for alerts
- Review overnight logs for any anomalies
- Verify both model versions running

## 2. Canary Monitoring (First 24 hours)
- Monitor accuracy hourly (should be ≥92%)
- Check latency (p99 should stay < 100ms)
- Watch error rates (should stay < 0.1%)
- If all metrics good, expand to 25% traffic after 6h

## 3. Incident Response
- High Latency → Check GPU utilization, reduce canary traffic if needed
- Accuracy Drop → Auto-rollback if > 5% degradation
- Data Drift → Alert team, prepare retraining

## 4. Escalation Path
- Immediate: Post in #ml-alerts Slack channel
- If critical: Page on-call SRE (Minh)
- If business impact: Notify VP Product
```

### Substep 3H: Workflow Complete

Summarize deployment:

```
"Model deployment complete and monitoring active!

Summary:
- Model v2.1.4 deployed to production (10% canary, 90% baseline)
- Accuracy: 92.5% (0.7% improvement over v2.0.3)
- Latency: p99 = 52ms (well below target)
- No data drift detected
- Monitoring active: Langfuse, MLflow, dashboards
- Cost: ~$245/day

Next 24 Hours:
- Monitor canary closely (accuracy, latency, errors)
- If stable, expand to 25% traffic after 6h
- If any issues, auto-rollback to v2.0.3

Long-term:
- Weekly model retraining scheduled
- Data drift monitoring continuous
- Cost optimization (model compression) planned for next sprint
- Team trained on incident response"
```

---

## Deliverables from This Step

1. **Monitoring Configuration**
   - Langfuse integration running
   - MLflow model registry updated
   - Dashboards created and linked

2. **Data Drift Detection Setup**
   - Statistical tests configured
   - Alert thresholds defined
   - Automatic retraining triggers active

3. **Cost Tracking Report**
   - Daily/weekly/monthly cost breakdown
   - Cost per inference
   - Optimization opportunities identified

4. **Incident Response Runbook**
   - Common issues and responses documented
   - Escalation procedures defined
   - On-call procedures established

5. **Retraining Pipeline Config**
   - Schedule-based and event-based triggers
   - Validation criteria for new models
   - Canary deployment strategy

## Workflow Complete

This is the **FINAL STEP** of the MLOps Deployment workflow.

**Next Activities**:
- Day 1: Hourly monitoring, canary expansion checkpoints
- Day 2-7: Daily summary reviews, watch for data drift
- Week 2+: Weekly model review, plan next retraining cycle
- Month 1+: Retrospective on deployment, lessons learned

---

**Navigation**: [← Back to Step 02](step-02-deploy.md), [Back to Workflow Overview](../workflow.md)
