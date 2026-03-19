---
step_number: 3
step_name: "Apply Fix & Verify Recovery"
next_step_file: "steps/step-04-postmortem.md"
estimated_time: "5-20 minutes"
lead_agent: "SRE Minh"
supporting_agents: ["Architect Khang", "on-call engineer"]
skill_loads:
  - "references/5.07 - RELIABILITY - Resilience & Fault Tolerance/SKILL.md"
  - "references/5.09 - ERROR HANDLING - Error Recovery Patterns/SKILL.md"
  - "references/5.10 - PRODUCTION READINESS - Go-Live & Operations/SKILL.md"
---

# STEP 3: Apply Fix & Verify Recovery

## STEP GOAL

Execute the fix safely with rollback plan, verify that the system recovers to baseline, and confirm user impact is resolved. This is the **action phase** — move quickly but safely.

**Outputs**:
- Fix applied and logged
- Rollback plan documented
- Recovery verified via metrics
- User-facing impact eliminated
- Incident now "resolved" (postmortem phase next)

## Skill Files to Load

```
Load: references/5.07 - RELIABILITY - Resilience & Fault Tolerance/SKILL.md
      → Circuit breaker management, fallback patterns, graceful degradation

Load: references/5.09 - ERROR HANDLING - Error Recovery Patterns/SKILL.md
      → Recovery patterns, retry strategies, idempotency

Load: references/5.10 - PRODUCTION READINESS - Go-Live & Operations/SKILL.md
      → Deployment safety, rollback procedures, configuration changes
```

## MANDATORY EXECUTION RULES

**Fix Safety Protocol**:
1. **Have rollback plan FIRST** — Before applying fix, document how to undo
2. **Test if possible** — Apply in staging first, or use canary deployment
3. **Communicate change** — Notify team in #incidents before making change
4. **Apply atomically** — Single change, not multiple at once
5. **Monitor immediately** — Watch metrics for 2-5 minutes post-fix
6. **Be ready to rollback** — Keep fingers on trigger, rollback if metrics don't improve

**Verification Rules**:
1. **Metrics must improve** — p99 latency or error rate must move toward baseline
2. **Errors must decrease** — Error rate trending down (not just stable)
3. **User reports must improve** — Direct feedback from users/support
4. **No new errors** — Fix shouldn't introduce different failure mode

**Time-Box**: Fix application + verification within 20 minutes. If fix doesn't work → rollback immediately.

## EXECUTION PROTOCOL

### Phase 1: Fix Strategy Selection (2 min)

**Strategy Options by Root Cause**:

#### **Root Cause: Dependency Failure**
```
Options:
  A) Restore dependent service (if infrastructure issue)
  B) Enable circuit breaker + fallback (immediate, safe)
  C) Route traffic around failed dependency
  D) Scale dependent service

Priority: B (safest, fastest) > A (requires ops) > C (requires routing) > D (slow)
```

**Implementation**:
```
Fallback Pattern (immediate):
- Check: if (call_to_service_X failed) { use cached_result_or_null_response }
- Deploy: Update config flag to enable fallback
- Rollback: Revert config flag (< 30 seconds)

Example:
  Config: feature_flag.use_fallback_for_service_x = true
  Then: requests use cached data instead of live service-x
  Result: Errors disappear immediately, users get stale data (acceptable temporarily)
```

#### **Root Cause: Resource Exhaustion**
```
Options:
  A) Scale up (add more servers)
  B) Kill runaway processes
  C) Increase limits (file descriptors, memory)
  D) Restart service (clears memory)

Priority: B (fastest, most effective) > A (takes time) > D (causes brief outage) > C (if limits too restrictive)
```

**Implementation**:
```
Identify Runaway Process:
- ps aux | grep [service_name] | sort -k3 -rn
- Kill: kill -9 [PID_of_hog]

Result: Memory/CPU released immediately
Rollback: Restart service normally (takes 30-60s)
```

#### **Root Cause: Database Issue**
```
Options:
  A) Kill slow queries (immediate)
  B) Restart database connection pool
  C) Optimize index (requires restart, higher risk)
  D) Scale read replicas (for heavy reads)

Priority: A (immediate relief) > B (fast, temporary fix) > D (longer-term) > C (risky)
```

**Implementation**:
```
Kill Slow Query:
- SELECT * FROM performance_schema.processlist WHERE TIME > 30;
- KILL [query_id];

Result: Locks released, throughput returns
Rollback: Query restarts naturally or connection timeout

Connection Pool Restart:
- Restart application instance (load balancer removes gracefully)
- Fresh connection pool connections
- Requests rerouted to healthy instances
```

#### **Root Cause: Code Regression**
```
Options:
  A) Rollback deployment (safest, fastest)
  B) Hotfix + redeploy (more effort, keeps new features)

Priority: A (if new deploy < 1 hour old) > B (if rollback would break other things)
```

**Implementation**:
```
Blue-Green Rollback:
- Current (GREEN): version-v2-with-bug
- Previous (BLUE): version-v1-stable
- Action: Flip traffic back to BLUE
- Time: < 30 seconds
- Verification: Metrics should return to BLUE baseline immediately
```

#### **Root Cause: Configuration Change**
```
Options:
  A) Revert config change (immediate)
  B) Adjust config to correct value (if you know what's wrong)

Priority: A (always safest) > B (if confident in correct value)
```

**Implementation**:
```
Config Revert:
- Find: Last stable config commit/version
- Deploy: Revert to that version
- Time: < 60 seconds if automated config deploy
- Verification: Metrics should improve immediately
```

#### **Root Cause: External Service Issue**
```
Options:
  A) Enable fallback/cache (serve stale data)
  B) Switch to backup provider
  C) Degrade gracefully (disable non-essential feature)

Priority: A (immediate, maintains availability) > B (if backup available) > C (user impact)
```

**Implementation**:
```
Enable Fallback:
- Config: external_service.use_cache_when_down = true
- Result: Requests use cached data (last known good state)
- Latency: Slightly better (no network call to external service)
- Rollback: Flip config back when external service recovers
```

### Phase 2: Prepare Rollback Plan (2 min)

**Rollback Template**:

```
IF FIX DOESN'T WORK:

Rollback Action: [Specific action to revert]
  What: [Exactly what to undo]
  How: [Step-by-step instructions]
  Time to Execute: [X minutes]
  Who: [Engineer performing rollback]

Example - Config Change Rollback:
  What: Revert feature_flag.use_fallback = true to false
  How:
    1. kubectl set env deployment/service-name FEATURE_FLAG_FALLBACK=false
    2. Verify: kubectl describe pod [pod_name] | grep FEATURE_FLAG
    3. Check: metrics should start returning to ERROR state (showing it's broken)
  Time: < 2 minutes
  Who: SRE on-call

Example - Deployment Rollback:
  What: Switch traffic from v2-broken back to v1-stable
  How:
    1. kubectl patch service [name] -p '{"spec":{"selector":{"version":"v1"}}}'
    2. Verify: Traffic shifting visible in metrics (latency drops)
    3. Confirm: Error rate approaching 0 within 2 minutes
  Time: < 30 seconds
  Who: SRE on-call
```

### Phase 3: Apply Fix (5-10 min)

**Before Applying**:

1. **Announce in Slack**: "@channel Applying fix: [Brief description]. Rollback ready if needed."
2. **Get confirmation**: "Proceeding with fix at [time]"
3. **Take screenshot**: Before metrics for comparison
4. **Start stopwatch**: Time the fix application

**During Fix Application**:

```
Step 1: Apply change (deploy, config, restart, kill process)
  - Execute fix command/deployment
  - Confirm: "Applied at [timestamp]"
  - Document: What exactly was done

Step 2: Monitor immediately
  - Watch error rate (should drop)
  - Watch latency (should improve)
  - Watch logs (should show fewer errors)

Step 3: Wait 2-5 minutes minimum
  - Don't declare success too early
  - Error rate must show downward trend
  - No new error messages appearing

Step 4: Check user reports
  - If users report in #incidents: "Issue still happening"
  - Then: Prepare rollback, try different fix
```

**After Applying Fix**:

Document in incident log:
```
FIX APPLIED:
  Timestamp: [When]
  Action: [What was changed]
  Expected Outcome: [Should see X improvement]
  Applied By: [Engineer]
```

### Phase 4: Verify Recovery (5 min)

**Recovery Verification Checklist**:

```
METRICS VERIFICATION:
  ✓ Error Rate: [Baseline X%] → Current [Y%] (should be < 2% above baseline)
  ✓ Latency p99: [Baseline X]ms → Current [Y]ms (should be within 20% of baseline)
  ✓ Throughput RPS: [Baseline X] → Current [Y] (should be normal)
  ✓ CPU/Memory: [Baseline X%] → Current [Y%] (should be normal)

ERROR LOG VERIFICATION:
  ✓ Error count per second: Trending down (not stable)
  ✓ No new error types: Old errors decreasing, no new ones appearing
  ✓ Critical errors gone: SEV1 errors (timeouts, OOM) not appearing

USER IMPACT VERIFICATION:
  ✓ Support report: No new tickets in last 5 minutes
  ✓ Slack reports: No new user complaints in #incidents
  ✓ Smoke test: [Operation] working again? YES / NO
```

**Decision Gate**:

```
IF (error_rate < baseline + 2%) AND (latency approaching baseline):
  → Fix successful ✓
  → Continue to step-04 postmortem

IF (error_rate not changing) OR (new errors appearing):
  → Fix failed ✗
  → Rollback immediately
  → Try alternative fix (back to phase 1)
```

**Recovery Report**:

```
RECOVERY VERIFIED:
  Status: RESOLVED
  Fix Applied: [What was done]
  Recovery Time: [T_start to T_resolved in minutes]
  Metrics Status:
    - Error rate: Baseline ↓
    - Latency: Baseline ↓
    - Throughput: Normal ✓
  User Impact: Resolved

  Next Step: Postmortem documentation
```

## DECISION POINTS

### [A] Advanced Verification (Additional Testing)

Use if:
- Metrics improving but not returned to baseline
- Want to perform additional sanity checks
- Need to verify fix in staging before declaring fully recovered

Actions:
- Run smoke test suite
- Test specific user journeys (checkout, payment, etc.)
- Verify across all regions if geo-distributed

### [C] Continue to Postmortem

- Accept recovery as verified
- Declare incident RESOLVED
- Move to step-04-postmortem.md

---

## SUCCESS / FAILURE METRICS

**Fix Success**:
- ✓ Fix applied without introducing new errors
- ✓ Metrics returned to baseline within 5 minutes
- ✓ No rollback needed
- ✓ User-facing impact eliminated
- ✓ MTTR (Mean Time To Recovery) < 30 minutes

**Fix Failures** (refinement for postmortem):
- ✗ Fix applied but metrics didn't improve
- ✗ Fix introduced different failure mode
- ✗ Rollback required, tried second fix
- ✗ MTTR > 1 hour despite clear root cause

---

**Bây giờ hãy áp dụng fix: Chọn strategy, chuẩn bị rollback.**

**[A] Kiểm tra thêm | [C] Quá trình fix hoàn tất**
