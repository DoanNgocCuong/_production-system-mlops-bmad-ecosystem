---
step_number: 5
step_name: "Security, Infrastructure & Deployment"
next_step_file: "steps/step-06-readiness.md"
estimated_time: "2.5-3 hours"
lead_agent: "SRE Minh"
supporting_agents: ["Architect Khang"]
skill_loads:
  - "references/5.11 - SECURITY & COMPLIANCE/SKILL.md"
  - "references/5.12 - CONFIGURATION MANAGEMENT/SKILL.md"
  - "references/5.13 - INFRASTRUCTURE & DEPLOYMENT/SKILL.md"
---

# STEP 5: Security, Infrastructure & Deployment

## STEP GOAL

Secure service and prepare infrastructure for production deployment:

- **Authentication/Authorization**: JWT, OAuth2, RBAC implementation
- **Secret management**: Vault setup, credential rotation
- **Docker & Kubernetes**: Deployment manifests, resource limits
- **Network security**: Ingress rules, TLS/mTLS configuration
- **Data protection**: Encryption at rest, in transit

**Output**: `security-plan.md` + `kubernetes-deployment.yaml` + `infrastructure-config.md` ready for step-06

## Skill Files to Load

```
Load: references/5.11 - SECURITY & COMPLIANCE/SKILL.md
      → Authentication, authorization, OWASP top 10

Load: references/5.12 - CONFIGURATION MANAGEMENT/SKILL.md
      → Environment variables, secrets, ConfigMaps, SealedSecrets

Load: references/5.13 - INFRASTRUCTURE & DEPLOYMENT/SKILL.md
      → Kubernetes, Helm, IaC, networking, compute resources
```

## MANDATORY RULES

1. **Secrets never in code** — Always use secret management
2. **Least privilege** — RBAC with minimal required permissions
3. **Encryption everywhere** — TLS for all network traffic, encryption at rest
4. **Immutable infrastructure** — Infrastructure as Code, no manual changes
5. **Defense in depth** — Multiple security layers

## EXECUTION PROTOCOL

### Phase 1: Authentication & Authorization (25 min)

**Authentication Strategy Selection**:

**Option A: JWT (JSON Web Token)**
- Pros: Stateless, scalable, no session storage
- Cons: Token revocation challenging, payload size
- Best for: Microservices, mobile apps, SPA frontend

**Option B: OAuth2**
- Pros: Delegate authentication, third-party integration
- Cons: More complex, additional redirect
- Best for: Third-party integrations, single sign-on

**Option C: API Key**
- Pros: Simple for service-to-service
- Cons: Limited scoping, harder to rotate
- Best for: Internal services, webhooks

**JWT Implementation**:

```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthCredentials
import jwt
from datetime import datetime, timedelta

security = HTTPBearer()
SECRET_KEY = os.getenv('JWT_SECRET_KEY')

def create_token(user_id: str, expires_delta: timedelta = None):
    if not expires_delta:
        expires_delta = timedelta(hours=1)

    expire = datetime.utcnow() + expires_delta
    payload = {
        'user_id': user_id,
        'exp': expire,
        'iat': datetime.utcnow()
    }

    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')

async def verify_token(credentials: HTTPAuthCredentials = Depends(security)):
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=['HS256'])
        user_id = payload.get('user_id')
        if not user_id:
            raise HTTPException(status_code=401, detail="Invalid token")
        return user_id
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

@app.get("/api/protected")
async def protected_route(user_id: str = Depends(verify_token)):
    return {"message": f"Hello {user_id}"}
```

**Authorization (RBAC)**:

```python
from enum import Enum

class Role(str, Enum):
    ADMIN = "admin"
    USER = "user"
    SERVICE = "service"

class Permission(str, Enum):
    READ = "read"
    WRITE = "write"
    DELETE = "delete"

ROLE_PERMISSIONS = {
    Role.ADMIN: [Permission.READ, Permission.WRITE, Permission.DELETE],
    Role.USER: [Permission.READ, Permission.WRITE],
    Role.SERVICE: [Permission.READ]
}

async def check_permission(
    required_permission: Permission,
    token: str = Depends(verify_token),
    db: Database = Depends(get_db)
):
    user = await db.get_user(token.user_id)
    permissions = ROLE_PERMISSIONS.get(user.role, [])

    if required_permission not in permissions:
        raise HTTPException(status_code=403, detail="Insufficient permissions")

    return user
```

### Phase 2: Secret Management (20 min)

**HashiCorp Vault Setup**:

```bash
# Initialize Vault
vault operator init -key-shares=5 -key-threshold=3

# Unseal Vault (requires 3 of 5 keys)
vault operator unseal [key1]
vault operator unseal [key2]
vault operator unseal [key3]

# Enable secret engine
vault secrets enable -path=secret kv-v2

# Create secret
vault kv put secret/api-service \
  DATABASE_PASSWORD=securepassword123 \
  JWT_SECRET=jwt-secret-key \
  API_KEY=service-api-key
```

**Application Configuration**:

```python
import hvac

def get_secrets():
    client = hvac.Client(
        url=os.getenv('VAULT_ADDR'),
        token=os.getenv('VAULT_TOKEN')
    )

    secret = client.secrets.kv.v2.read_secret_version(
        path='api-service'
    )

    return {
        'db_password': secret['data']['data']['DATABASE_PASSWORD'],
        'jwt_secret': secret['data']['data']['JWT_SECRET'],
        'api_key': secret['data']['data']['API_KEY']
    }

# Use in environment
os.environ.update(get_secrets())
```

**Kubernetes Secret Management**:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-service-secrets
type: Opaque
stringData:
  DATABASE_PASSWORD: ${DB_PASSWORD}
  JWT_SECRET: ${JWT_SECRET}
  API_KEY: ${API_KEY}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  template:
    spec:
      containers:
      - name: api
        envFrom:
        - secretRef:
            name: api-service-secrets
```

### Phase 3: Kubernetes Deployment (30 min)

**Complete K8s Deployment Manifest**:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: api-service
  template:
    metadata:
      labels:
        app: api-service
        version: v1
    spec:
      serviceAccountName: api-service
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsReadOnlyRootFilesystem: true
      containers:
      - name: api
        image: registry.example.com/api-service:v1.2.3
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
              - ALL
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: cache
          mountPath: /app/cache
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: LOG_LEVEL
          value: "info"
        envFrom:
        - configMapRef:
            name: api-config
        - secretRef:
            name: api-secrets
      volumes:
      - name: tmp
        emptyDir: {}
      - name: cache
        emptyDir: {}
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - api-service
              topologyKey: kubernetes.io/hostname

---
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: production
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: api-service

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-service-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api-service
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443  # HTTPS only

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**TLS Configuration**:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: api-service-cert
  namespace: production
spec:
  secretName: api-service-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - api.example.com

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-service-ingress
  namespace: production
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.example.com
    secretName: api-service-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### Phase 4: Data Protection (15 min)

**Encryption at Rest**:

```yaml
# PostgreSQL with encryption
apiVersion: v1
kind: Secret
metadata:
  name: db-encryption-key
  namespace: production
type: Opaque
stringData:
  key: "$(openssl rand -base64 32)"

---
# Enable encryption in application
DATABASE_URL: "postgresql://user:pass@db.example.com:5432/app?sslmode=require"
ENCRYPTION_KEY: ${DB_ENCRYPTION_KEY}
```

**TLS for All Communications**:

```
- Database: PostgreSQL sslmode=require
- Cache: Redis with TLS
- External APIs: HTTPS only
- Internal services: mTLS with cert-manager
```

**Document**: Create `security-infrastructure.md`

## MENU - Choose Your Path

### [A] Accept security and infrastructure design
- Confirms K8s deployment config
- Validates security controls
- Proceeds to step-06

### [C] Continue refining
- Adjusts RBAC rules
- Refines secret management
- Revisits deployment strategy

---

**Bấm [A] để chấp nhận hoặc [C] để refine thêm**
