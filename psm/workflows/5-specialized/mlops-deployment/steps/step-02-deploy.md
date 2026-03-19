---
step_id: MLOPS-S02
step_name: Deploy Model
workflow_id: MLOPS001
sequence: 2
type: "deployment"
agent: "Linh (MLOps)"
description: "Deploy model to serving infrastructure, setup versioning, GPU optimization"
estimated_time: "60-80 minutes"
---

# Step 02: Model Deployment & Infrastructure Setup

## Objective

Deploy validated model to production serving infrastructure:
1. Setup model serving platform (KServe, BentoML, Seldon Core)
2. Configure versioning and traffic routing
3. Optimize GPU/CPU utilization
4. Setup pre/post-processing pipelines
5. Prepare for A/B test traffic routing

## Execution Flow

### Substep 2A: Serving Infrastructure Setup

Select and configure model serving platform:

**Platform Options**:

**Option A: KServe (Kubernetes-native)**
- Advantages: Native K8s integration, auto-scaling, versioning
- Use case: High-volume, latency-sensitive models
- Setup: 30 mins, requires Kubernetes cluster
- Cost: Medium (shares cluster resources)

**Option B: BentoML**
- Advantages: Python-first, flexible serving, easy containerization
- Use case: Quick deployments, custom preprocessing
- Setup: 1 hour, simpler packaging
- Cost: Medium (dedicated containers)

**Option C: Serverless (AWS Lambda, Google Cloud Functions)**
- Advantages: No ops, pay-per-invocation, auto-scaling
- Use case: Bursty traffic, small models
- Setup: 30 mins, but cold start latency
- Cost: Low (if infrequent), high (if frequent)

**Option D: Cloud ML Services (SageMaker, Vertex AI)**
- Advantages: Fully managed, auto-scaling, monitoring included
- Use case: Enterprise deployments, low ops overhead
- Setup: 1-2 hours, vendor lock-in
- Cost: High (premium for managed service)

**Prompt for DevOps**:
> "What infrastructure do we have available? What's our latency requirement? Do we expect bursty or steady traffic?"

### Substep 2B: Model Packaging & Containerization

Prepare model for deployment:

**Artifact Preparation**:
```bash
# Directory structure
model-artifact/
├── model.pth              # Model weights
├── config.yaml            # Model config (architecture, hyperparams)
├── tokenizer.pkl          # Preprocessing artifacts
├── metadata.json          # Version, schema, feature names
└── requirements.txt       # Dependencies
```

**Docker Image Creation**:
```dockerfile
FROM python:3.10-slim

COPY model-artifact/ /app/model/
COPY serving.py /app/
COPY requirements.txt /app/

RUN pip install -r /app/requirements.txt

CMD ["python", "/app/serving.py"]
```

**Versioning Strategy**:
- Semantic versioning: v2.1.4 (major.minor.patch)
- Docker tag: `sentiment-classifier:v2.1.4`, `sentiment-classifier:latest`
- Git tag: `model/v2.1.4` (link to training code)
- Model registry: Document in MLflow/Weights&Biases with metadata

### Substep 2C: GPU/CPU Optimization

Optimize resource utilization:

**GPU Optimization** (if applicable):
- [ ] Model quantization (FP32 → FP16 or INT8)?
- [ ] Model distillation (reduce size while maintaining accuracy)?
- [ ] Batch processing for higher throughput?
- [ ] GPU memory allocation optimized?
- [ ] Mixed precision training for faster inference?

**CPU Optimization**:
- [ ] ONNX export (framework-agnostic format)?
- [ ] Pruning unnecessary weights?
- [ ] Compiler optimizations (TensorRT for NVIDIA)?
- [ ] Multi-threaded inference?

**Example Configuration**:
```yaml
# KServe InferenceService
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: sentiment-classifier
spec:
  predictor:
    pytorch:
      image: myregistry.azurecr.io/sentiment-classifier:v2.1.4
      resources:
        requests:
          cpu: "2"
          memory: "4Gi"
          nvidia.com/gpu: "1"
        limits:
          cpu: "4"
          memory: "8Gi"
          nvidia.com/gpu: "1"
  canary:
    pytorch:
      image: myregistry.azurecr.io/sentiment-classifier:v2.0.3
    trafficPercent: 10
```

**Prompt for Platform Eng**:
> "Do we have spare GPU capacity? What's the cost per GPU per hour? Can we batch requests for better throughput?"

### Substep 2D: Pre/Post-processing Pipeline

Setup data transformation:

**Preprocessing** (Input):
```python
# Tokenization, normalization, padding
def preprocess(text: str) -> torch.Tensor:
    tokens = tokenizer.encode(text, return_tensors='pt')
    return tokens[:512]  # Max 512 tokens

# Health check
assert preprocess("hello") is not None
```

**Postprocessing** (Output):
```python
# Convert logits to user-friendly format
def postprocess(logits: torch.Tensor) -> dict:
    probs = torch.softmax(logits, dim=1)
    label = ["negative", "neutral", "positive"][probs.argmax()]
    confidence = probs.max().item()
    return {
        "label": label,
        "confidence": confidence,
        "probabilities": {
            "negative": probs[0, 0].item(),
            "neutral": probs[0, 1].item(),
            "positive": probs[0, 2].item(),
        }
    }
```

### Substep 2E: Traffic Routing & Versioning

Setup A/B test configuration:

**Blue-Green Deployment**:
```
Before Deployment:
  sentiment-classifier -> [v2.0.3] (100% traffic)

Deployment Step:
  sentiment-classifier -> [v2.0.3: 100%, v2.1.4: 0%]
                         (new version ready, no traffic)

Gradual Rollout:
  sentiment-classifier -> [v2.0.3: 95%, v2.1.4: 5%]
  (after 1 hour)         [v2.0.3: 80%, v2.1.4: 20%]
  (after 6 hours)        [v2.0.3: 0%, v2.1.4: 100%]
  (after 24 hours)
```

**Traffic Split Configuration** (KServe):
```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: sentiment-classifier
spec:
  predictor:
    canary:
      trafficPercent: 10  # 10% to new version
    canaryTrafficPercent: 10
    standardModelSpec:
      image: sentiment-classifier:v2.0.3  # Control (90%)
    canaryModelSpec:
      image: sentiment-classifier:v2.1.4  # Experiment (10%)
```

### Substep 2F: Health Checks & Smoke Tests

Validate deployment:

**Health Check Endpoint**:
```python
@app.get("/health")
def health():
    return {
        "status": "healthy",
        "version": "2.1.4",
        "ready": True
    }
```

**Smoke Tests** (Run after deployment):
```python
def smoke_test():
    # Test 1: Model loads and runs
    result = predict("This movie was amazing!")
    assert result["label"] in ["negative", "neutral", "positive"]
    assert 0 <= result["confidence"] <= 1

    # Test 2: Latency acceptable
    import time
    start = time.time()
    predict("Test text")
    latency = (time.time() - start) * 1000
    assert latency < 100, f"Latency {latency}ms exceeds 100ms threshold"

    # Test 3: Batch processing
    results = batch_predict(["Text 1", "Text 2", "Text 3"])
    assert len(results) == 3
```

### Substep 2G: Monitoring Dashboard Setup

Create observability:

**Metrics to Track**:
- Request rate (req/sec)
- Latency (p50, p95, p99)
- Model accuracy (per-segment)
- Data drift indicators
- Error rate
- Model version distribution (blue-green split)

**Dashboard Example**:
```
Model Deployment Dashboard
├─ Request Metrics
│  ├─ Total requests: 2.3M/day
│  ├─ QPS: 26.6 req/sec
│  └─ Error rate: 0.02%
│
├─ Latency
│  ├─ p50: 45ms
│  ├─ p95: 78ms
│  └─ p99: 120ms
│
├─ Model Performance
│  ├─ v2.0.3 accuracy: 91.8%
│  ├─ v2.1.4 accuracy: 92.5% (+0.7%)
│  ├─ v2.0.3 traffic: 90%
│  └─ v2.1.4 traffic: 10%
│
└─ Data Quality
   ├─ Data drift: OK
   ├─ Feature distributions: Stable
   └─ Missing values: < 0.1%
```

### Substep 2H: Transition to Step 03

Confirm deployment success:

```
"Model deployment complete!

Summary:
- Model v2.1.4 deployed to sentiment-classifier service
- Running in canary mode: 10% traffic to new version
- Monitoring active on latency, accuracy, data drift
- Smoke tests passed; health check OK

Next, I'll setup detailed monitoring and data drift detection.
This ensures we catch any issues in the first 24 hours."
```

**Proceed to [Step 03](step-03-monitor.md)**

## Deliverables from This Step

1. **Deployment Configuration**
   - Model serving platform setup
   - Docker image tagged and pushed to registry
   - Infrastructure as Code (Terraform, Helm charts)

2. **Versioning Metadata**
   - Model registry entry (MLflow, Weights&Biases)
   - Semantic version, training date, performance
   - Docker image tags and SHA

3. **Traffic Routing Configuration**
   - A/B test split (10% canary, 90% baseline)
   - Auto-rollback thresholds
   - Ramp-up schedule for first 24 hours

4. **Monitoring Setup**
   - Dashboards created and linked
   - Key metrics instrumented
   - Alerts configured for latency/errors

---

**Navigation**: [← Back to Step 01](step-01-model-validation.md), [Next: Step 03 →](step-03-monitor.md)
