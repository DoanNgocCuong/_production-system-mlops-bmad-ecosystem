# ML Deployment — Readiness Checklist

**Definition of Done**: This checklist confirms the ML model is ready for production deployment and monitoring.

## 📋 Model Validation

- [ ] Model metrics verified against SLO (accuracy, precision, recall, F1 score)
- [ ] Baseline performance established (training vs production performance gap)
- [ ] Data drift check passed (training/production data distribution aligned)
- [ ] A/B test plan defined (control vs treatment groups, sample size, duration)
- [ ] Bias evaluation completed (demographic parity, equalized odds assessment)
- [ ] Model versioning documented (model ID, version, checkpoint path)
- [ ] Inference latency tested and validated against targets

## 🏗️ Deployment

- [ ] Serving infrastructure ready (container image built, registry pushed)
- [ ] GPU/resource allocation confirmed (compute type, quantity, cost estimate)
- [ ] Model registry updated (model artifact, metadata, lineage)
- [ ] Feature store ready (feature definitions, data pipelines operational)
- [ ] Rollback mechanism tested (previous version can be deployed immediately)
- [ ] Canary deployment strategy documented (traffic split percentages, metrics)
- [ ] Load testing completed and passed (expected QPS, P99 latency validated)

## 📊 Monitoring

- [ ] Langfuse or MLflow tracing configured (inference logging, feature logging)
- [ ] Drift detection enabled (monitor input/output distribution shifts)
- [ ] Performance baselines established (accuracy, latency, cost per prediction)
- [ ] Cost tracking active (per-prediction cost, total monthly cost model)
- [ ] Alerts configured (accuracy degradation, latency spikes, cost anomalies)
- [ ] Monitoring dashboard created (live model performance view)
- [ ] Data quality checks automated (missing values, outliers, distributions)

## 🔚 Final Status

- [ ] Deployment verified in production (health checks passing)
- [ ] Monitoring dashboard reviewed (24 hours post-deploy minimum)
- [ ] Model performance validated live (no accuracy drop observed)
- [ ] Stakeholders notified (data science team, product team, business)
- [ ] Retraining schedule established (when to retrain, trigger conditions)
- [ ] Incident response plan documented (model degradation response steps)
