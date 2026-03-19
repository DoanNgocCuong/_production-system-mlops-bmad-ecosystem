---
step_number: 2
step_name: "API Design & Database Strategy"
next_step_file: "steps/step-03-build-deploy.md"
estimated_time: "2-3 hours"
lead_agent: "Architect Khang"
supporting_agents: ["SRE Minh"]
skill_loads:
  - "references/5.02 - DESIGN - API Design/SKILL.md"
  - "references/5.03 - DESIGN - Database & Data Management/SKILL.md"
---

# STEP 2: API Design & Database Strategy

## STEP GOAL

Finalize API contracts and select appropriate database technology:

- **API protocol selection**: REST vs. GraphQL vs. gRPC decision
- **API contract specification**: Endpoints, request/response schemas, error codes
- **Database technology selection**: PostgreSQL, MongoDB, Redis, etc.
- **Schema design**: Core entities, relationships, normalization
- **Indexing strategy**: Query patterns and optimization

**Output**: `api-design-document.md` + `database-schema.md` ready for step-03

## Skill Files to Load

```
Load: references/5.02 - DESIGN - API Design/SKILL.md
      → REST, GraphQL, gRPC comparisons, best practices

Load: references/5.03 - DESIGN - Database & Data Management/SKILL.md
      → Database selection, schema design, indexing strategies
```

## MANDATORY RULES

1. **API first** — Design API before database schema
2. **Schema normalization** — Balance between normalization and query efficiency
3. **Index planning** — Plan indexes based on expected query patterns
4. **Error handling** — Define standard error response format
5. **Versioning** — Plan for API versioning (v1, v2, etc.)

## EXECUTION PROTOCOL

### Phase 1: API Protocol Selection (20 min)

**Protocol Options**:

**Option A: REST**
- Pros: Simple, cacheable, widely understood, HTTP native
- Cons: Over/under fetching, versioning complexity
- Best for: CRUD operations, standard web services

**Option B: GraphQL**
- Pros: Exact data fetching, single endpoint, schema-driven
- Cons: Caching complexity, query DoS, learning curve
- Best for: Mobile apps, complex queries, multiple clients

**Option C: gRPC**
- Pros: High performance, binary protocol, streaming support
- Cons: Not browser-native, complexity, tooling gap
- Best for: Internal services, high-frequency calls, real-time

**Selection Matrix**:

| Criteria | REST | GraphQL | gRPC |
|----------|------|---------|------|
| Ease of learning | ✓✓✓ | ✓ | ✓ |
| Performance | ✓✓ | ✓✓ | ✓✓✓ |
| Caching | ✓✓✓ | ✓ | ✓ |
| Monitoring | ✓✓✓ | ✓✓ | ✓✓ |

**Decision**: SELECTED = [REST | GraphQL | gRPC]

**Rationale**:
- [Reason 1]
- [Reason 2]

### Phase 2: REST API Contract Specification (30 min)

**If REST Selected**:

**Resource Model**:

```
Core Resources:
- /v1/users → User management
- /v1/items → Item catalog
- /v1/orders → Order processing
- /v1/payments → Payment handling
```

**Endpoint Specification Template**:

```
Endpoint: POST /v1/items

Request:
{
  "name": "string",
  "price": "decimal",
  "category": "string",
  "quantity": "integer"
}

Response (201):
{
  "id": "uuid",
  "name": "string",
  "price": "decimal",
  "created_at": "timestamp"
}

Errors:
- 400: Invalid input (schema validation)
- 401: Unauthorized
- 409: Conflict (duplicate)
- 500: Server error
```

**Standard Error Response**:

```json
{
  "error_code": "VALIDATION_ERROR",
  "message": "Invalid input format",
  "details": {
    "field": "price",
    "issue": "must be positive number"
  }
}
```

**Rate Limiting**: Define limits (e.g., 1000 req/min per user)

**Document**: Create `api-contract.md` with all endpoints

### Phase 3: Database Technology Selection (25 min)

**Database Options**:

**Option A: PostgreSQL (Relational)**
- Pros: ACID, powerful queries, ecosystem, open-source
- Cons: Scaling writes is hard, schema migrations
- Best for: Strong consistency, structured data, complex queries

**Option B: MongoDB (Document)**
- Pros: Flexible schema, horizontal scaling, good for JSON
- Cons: Eventual consistency, higher memory, weaker queries
- Best for: Flexible data, rapid iteration, unstructured

**Option C: Redis (Key-Value)**
- Pros: Ultra-fast, in-memory, pub/sub, data structures
- Cons: Not durable by default, limited to RAM, no complex queries
- Best for: Caching, sessions, real-time leaderboards

**Option D: DynamoDB / Firestore (Serverless)**
- Pros: Managed, auto-scaling, pay-per-request
- Cons: Vendor lock-in, limited query patterns, cost unpredictable
- Best for: Variable load, serverless architecture

**Selection Framework**:

```
1. Query patterns?
   → Complex joins → PostgreSQL
   → Simple key-value → Redis / DynamoDB
   → Flexible, nested → MongoDB

2. Scale?
   → Strong consistency critical → PostgreSQL
   → Eventually consistent OK → MongoDB or DynamoDB

3. Data volume?
   → < 100GB → PostgreSQL or MongoDB
   → > 1TB and growing → Consider partitioning strategy

4. Operational maturity?
   → Managed service preferred → DynamoDB / RDS / Cloud SQL
   → Self-hosted → PostgreSQL / MongoDB
```

**Decision**: SELECTED = [PostgreSQL | MongoDB | Redis | DynamoDB]

**Rationale**:
- [Reason 1]
- [Reason 2]

### Phase 4: Schema Design & Indexing (25 min)

**For PostgreSQL**:

```sql
-- Core tables
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  name VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);

CREATE TABLE items (
  id UUID PRIMARY KEY,
  creator_id UUID REFERENCES users(id),
  title VARCHAR NOT NULL,
  category VARCHAR,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_items_creator ON items(creator_id);
CREATE INDEX idx_items_category ON items(category);
```

**For MongoDB**:

```javascript
db.users.insertOne({
  _id: ObjectId(),
  email: "user@example.com",
  name: "John",
  created_at: new Date()
});

db.users.createIndex({ email: 1 });

db.items.createIndex({ creator_id: 1 });
db.items.createIndex({ category: 1 });
```

**Indexing Strategy**:

```
Query Pattern Analysis:
1. Find item by ID → Create primary index (automatic)
2. List items by creator → Create (creator_id) index
3. Filter by category → Create (category) index
4. Search by title → Consider full-text index if needed
```

**Document**: Create `database-schema.md` with DDL/document structure

## MENU - Choose Your Path

### [A] Accept API and database design
- Confirms all selections
- Generates final documents
- Proceeds to step-03

### [C] Continue refining
- Asks more design questions
- Revisits protocol or database selection
- Loops back to evaluation

---

**Bấm [A] để chấp nhận hoặc [C] để refine thêm**
