# Debugging & Root Cause Analysis

---

## Q1: Walk me through your approach to debugging a production issue.

**Answer:**

My systematic approach:

1. **Understand the symptom** — What exactly is failing? Error rate spike? Latency increase? Specific error codes? Don't start debugging until you can clearly state the symptom.
2. **Check recent changes** — What was deployed recently? Config changes? Infrastructure changes? Most production issues correlate with a recent change.
3. **Triage scope** — Is it affecting all users or specific tenants? All endpoints or specific ones? All AZs or one? This narrows the blast radius.
4. **Logs + Metrics + Traces** — Start with centralized logs (CloudWatch, ELK). Correlate with metrics (latency, error rate, CPU/memory). Use distributed tracing (X-Ray) to trace the request across services.
5. **Form hypothesis → Test** — Based on evidence, form a hypothesis. Test it. If wrong, widen scope.
6. **Go deeper if needed** — If application-level investigation is inconclusive, go to infrastructure: VPC, security groups, DNS, route tables, load balancer config.
7. **Document** — Write the RCA with timeline, root cause, fix, and prevention measures.

---

## Q2: Tell me about a complex debugging scenario you handled.

**Answer (STAR):**

- **Situation:** API calls between two services were intermittently timing out. The pattern was inconsistent — sometimes it worked, sometimes it didn't. Standard application logs showed timeout errors but no clear cause.
- **Task:** Identify root cause and fix before production deployment.
- **Action:** 
  - Verified application config (timeouts, connection pools) — all correct
  - Checked ECS task health and scaling — healthy
  - Traced network path: noticed requests from certain AZs consistently failed
  - Investigated VPC peering configuration — found that route table entries were missing for one AZ's subnet
  - The VPC peering connection existed, but the routing was incomplete
  - Worked with infra team to add missing routes and validate connectivity
- **Result:** All intermittent timeouts resolved. Documented the debugging methodology and added network connectivity checks to the deployment checklist.

> **Key insight:** When application-level debugging hits a wall, go to the network layer. Intermittent failures often point to infrastructure issues.

---

## Q3: How do you ensure issues don't recur after an RCA?

**Answer:**

1. **Immediate fix** — Resolve the symptom first (restore service)
2. **Root cause fix** — Address the underlying cause (not just the symptom)
3. **Prevention measures:**
   - Add monitoring/alerting for the specific failure mode
   - Add automated checks (e.g., connectivity validation in CI/CD)
   - Update runbooks with the debugging steps
   - Add integration tests that cover the failure scenario
4. **Knowledge sharing** — Share the RCA document with the broader team. Use it as a learning opportunity.
5. **Process improvement** — If the issue was caused by a process gap (e.g., missing deployment checklist item), update the process.

---

## Q4: How do you handle debugging when the issue spans multiple microservices?

**Answer:**

- **Distributed tracing** — Use trace IDs (correlation IDs) that propagate across service boundaries. Follow the trace to find where latency spikes or errors originate.
- **Service dependency map** — Know the call graph. When Service A fails, trace through to Service B, C, D.
- **Isolate the failing service** — Use health checks, direct API calls, and log analysis to identify which service in the chain is the actual source of failure.
- **Reproduce in lower environments** — If possible, reproduce with similar data/traffic patterns in staging.
- **Blame the network last, but don't forget it** — Most issues are application-level. But when app-level investigation is exhausted, infrastructure (DNS, routing, security groups, peering) is the next layer.

**Anti-pattern to avoid:** Don't just restart containers and hope it fixes itself. Understand *why* before fixing.
