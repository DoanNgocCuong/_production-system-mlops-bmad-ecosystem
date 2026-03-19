# New Service Setup — Definition of Done Checklist

**Validation Gate**: This checklist confirms the service is production-ready across all six dimensions before deployment.

## 📋 Architecture

- [ ] Architecture pattern selected and documented (monolith, microservice, serverless)
- [ ] Architecture Decision Record (ADR) created explaining pattern choice
- [ ] Technology stack finalized (language, framework, databases, message queues)
- [ ] Service boundaries and dependencies mapped
- [ ] Scaling strategy defined (horizontal, vertical, auto-scaling approach)

## 🔧 API & Database

- [ ] API endpoints designed and documented (OpenAPI/Swagger spec)
- [ ] API contracts validated (request/response schemas, error codes)
- [ ] Database schema designed and indexed for performance
- [ ] Data migration strategy documented (initial and future upgrades)
- [ ] Backwards compatibility plan defined for API evolution

## 🏗️ Build & Deploy

- [ ] CI/CD pipeline configured and tested (build, test, deploy stages)
- [ ] Docker image created and optimized (size, layers, security scanning)
- [ ] Kubernetes manifests prepared (deployment, service, configmap)
- [ ] Deployment tested in staging environment
- [ ] Rollback procedure documented and tested

## 🛡️ Reliability & Observability

- [ ] Circuit breakers configured for external service calls
- [ ] Logging structured and centralized (JSON format, key fields)
- [ ] Distributed tracing enabled (OpenTelemetry or similar)
- [ ] Health check endpoints implemented (/health, /ready)
- [ ] Alerts configured for key metrics (error rate, latency, throughput)
- [ ] Incident runbooks drafted (common failure scenarios)

## 🔒 Security & Infrastructure

- [ ] Authentication implemented (OAuth2, JWT, service mesh identity)
- [ ] Authorization enforced (RBAC, policy-based access control)
- [ ] Secrets management configured (Vault, cloud provider, rotation)
- [ ] Infrastructure provisioned (networking, load balancer, DNS)
- [ ] TLS/SSL configured for all external communication
- [ ] Security scanning integrated into CI/CD (SAST, dependency check)

## ✅ Production Readiness

- [ ] Production Readiness Review (PRR) completed for all 9 dimensions
- [ ] Runbooks written for common operational tasks
- [ ] Rollback tested from production-like environment
- [ ] Load testing performed and results documented
- [ ] Disaster recovery plan defined and tested
- [ ] On-call rotation and escalation policy established
