# Ownership & Accountability

---

## Q1: Tell me about a time you took ownership of a critical decision that saved the team significant time or cost.

**Answer (STAR):**

- **Situation:** The team was planning a major database migration for a subsession data store. The migration was already in the roadmap and resources were being allocated.
- **Task:** As the lead, I was responsible for the design and execution plan. Before diving into implementation, I wanted to validate whether the migration was truly necessary at that point.
- **Action:** I conducted a thorough analysis — reviewed current query patterns, data volume growth projections, performance metrics, and the actual pain points the migration was supposed to solve. I built a cost-benefit comparison and presented my findings to stakeholders, recommending we defer the migration.
- **Result:** The team agreed to postpone. This saved thousands of dollars in infrastructure cost, weeks of engineering effort, and avoided unnecessary downtime risk. The system continued to perform well without the migration, validating the decision.

> **Key takeaway:** Ownership isn't just about executing — it's about questioning whether something should be done at all.

---

## Q2: Describe a situation where you owned a production issue end-to-end.

**Answer (STAR):**

- **Situation:** A critical API started failing intermittently in the staging environment. The subsession API was returning timeouts and connection errors that couldn't be reproduced consistently.
- **Task:** I took ownership of the root cause analysis even though the issue crossed multiple service boundaries (API gateway, backend service, downstream dependencies).
- **Action:** I systematically traced the request flow across services, analyzed network configurations, and reviewed VPC peering rules. I discovered that a VPC peering configuration was preventing the service from reaching a downstream dependency in certain network paths. I documented the RCA, proposed the fix, coordinated with the infrastructure team, and validated the fix in staging before production rollout.
- **Result:** The issue was resolved with zero production impact. The RCA document became a reference for the team for future networking issues.

> **Key takeaway:** Owning an issue means going beyond your service boundary to find the real root cause.

---

## Q3: How do you handle situations where a project's direction needs to change?

**Answer:**

I believe in data-driven decision-making. When I see signals that a project's direction may not be optimal — whether from performance data, user feedback, or technical analysis — I proactively raise it with stakeholders. I prepare evidence, propose alternatives, and facilitate a decision rather than just flagging the concern. The DB migration deferral is a strong example of this: I didn't just say "I'm not sure about this" — I did the analysis to back up the recommendation.

---

## Q4: Give an example of when you went above and beyond your defined responsibilities.

**Answer (STAR):**

- **Situation:** Production data had accumulated orphan records (events with no parent references) over time, causing data inconsistencies and confusing analytics.
- **Task:** This wasn't assigned to anyone — it was a known but deprioritized issue.
- **Action:** I took the initiative to analyze production data patterns, identified the orphan records, wrote safe cleanup scripts with rollback capability, and executed the cleanup in a controlled manner with proper approvals.
- **Result:** Cleaned up stale data, improved data integrity, and established a pattern for periodic data hygiene that the team adopted going forward.

---

## Q5: Tell me about a time you prevented a high-risk infrastructure change.

**Answer (STAR):**

- **Situation:** A migration of our 9.2 TiB PostgreSQL RDS database (db.r5.12xlarge) to a new Aurora cluster using DMS was being considered. The goal was to improve failover and resilience capability. The approach involved copying the entire database.
- **Task:** As Lead Engineer, I was responsible for evaluating and implementing resilience improvements while minimizing downtime and aligning with our future architecture roadmap.
- **Action:**
  1. **Verified existing resilience** : Found Multi-AZ was already enabled across all environments. Located the PR that had enabled it. Ran a controlled failover reboot test in QA and measured actual downtime: 30–60 seconds with automatic standby promotion.
  2. **Downtime risk assessment** : For a 9.2 TiB database, a full DMS copy would carry significant replication lag, multi-environment outage exposure, and rollback complexity. The migration itself posed higher downtime risk than steady-state.
  3. **Cost comparison** : Existing Multi-AZ: ~$2,099/10 months. Aurora + DMS: ~$8,130/10 months. Cost avoided: ~$6,031.
  4. **Roadmap alignment** : Identified that a Parquet-based data architecture migration was already planned for the following year. Performing Aurora migration now would create two major data transitions in 12 months : double the risk and redundant engineering effort.
  5. **Stakeholder alignment** : Presented risk, cost, and roadmap analysis. Reframed the goal from "replatforming" to "validated resilience."
- **Result:** Migration was canceled. Saved ~$6K in infrastructure cost. Reduced downtime exposure across all environments. Prevented redundant architectural work before the Parquet migration. Received leadership recognition for saving significant time and effort.

> **Executive summary:** A 9.2 TiB PostgreSQL RDS migration to Aurora was being considered for resilience. I validated Multi-AZ was already functioning with 30–60 second failover. Given the database size and upcoming Parquet migration, I determined the change introduced redundant risk and cost. I recommended postponing, improving resilience through validation rather than architectural churn.

---

## Q6: Describe a time you had to adapt to a major infrastructure or process change.

**Answer (STAR):**

- **Situation:** Our team was asked to migrate our application infrastructure to a new AWS account as part of a larger organizational restructuring. This involved moving ECS services, RDS instances, S3 buckets, IAM roles, and CI/CD pipelines : all with zero downtime requirements.
- **Task:** As Lead Engineer, I was responsible for planning and executing the migration safely.
- **Action:** I documented all current dependencies (service-to-service calls, IAM permissions, secrets, environment variables). I created a migration runbook with rollback steps for each component. I automated the deployment pipeline configurations for the new account and ran parallel environments during cutover. I coordinated the cutover window with stakeholders and monitored traffic routing carefully.
- **Result:** Migration completed with zero downtime. CI/CD pipelines were operational in the new account from day one. The runbook became a reusable template for future account migrations.

> **Key takeaway:** Large infrastructure changes succeed through thorough dependency mapping and staged execution, not speed.


---

## Q5: Tell me about a time you prevented a high-risk infrastructure change.

**Answer (STAR):**

- **Situation:** A migration of our 9.2 TiB PostgreSQL RDS database (db.r5.12xlarge) to a new Aurora cluster using DMS was being considered. The goal was to improve failover and resilience capability. The approach involved copying the entire database.
- **Task:** As Lead Engineer, I was responsible for evaluating and implementing resilience improvements while minimizing downtime and aligning with our future architecture roadmap.
- **Action:**
  1. **Verified existing resilience** : Found Multi-AZ was already enabled across all environments. Located the PR that had enabled it. Ran a controlled failover reboot test in QA and measured actual downtime: 30–60 seconds with automatic standby promotion.
  2. **Downtime risk assessment** : For a 9.2 TiB database, a full DMS copy would carry significant replication lag, multi-environment outage exposure, and rollback complexity. The migration itself posed higher downtime risk than steady-state.
  3. **Cost comparison** : Existing Multi-AZ: ~$2,099/10 months. Aurora + DMS: ~$8,130/10 months. Cost avoided: ~$6,031.
  4. **Roadmap alignment** : Identified that a Parquet-based data architecture migration was already planned for the following year. Performing Aurora migration now would create two major data transitions in 12 months : double the risk and redundant engineering effort.
  5. **Stakeholder alignment** : Presented risk, cost, and roadmap analysis. Reframed the goal from "replatforming" to "validated resilience."
- **Result:** Migration was canceled. Saved ~$6K in infrastructure cost. Reduced downtime exposure across all environments. Prevented redundant architectural work before the Parquet migration. Received leadership recognition for saving significant time and effort.

> **Executive summary:** A 9.2 TiB PostgreSQL RDS migration to Aurora was being considered for resilience. I validated Multi-AZ was already functioning with 30–60 second failover. Given the database size and upcoming Parquet migration, I determined the change introduced redundant risk and cost. I recommended postponing, improving resilience through validation rather than architectural churn.

---

## Q6: Describe a time you had to adapt to a major infrastructure or process change.

**Answer (STAR):**

- **Situation:** Our team was asked to migrate our application infrastructure to a new AWS account as part of a larger organizational restructuring. This involved moving ECS services, RDS instances, S3 buckets, IAM roles, and CI/CD pipelines : all with zero downtime requirements.
- **Task:** As Lead Engineer, I was responsible for planning and executing the migration safely.
- **Action:** I documented all current dependencies (service-to-service calls, IAM permissions, secrets, environment variables). I created a migration runbook with rollback steps for each component. I automated the deployment pipeline configurations for the new account and ran parallel environments during cutover. I coordinated the cutover window with stakeholders and monitored traffic routing carefully.
- **Result:** Migration completed with zero downtime. CI/CD pipelines were operational in the new account from day one. The runbook became a reusable template for future account migrations.

> **Key takeaway:** Large infrastructure changes succeed through thorough dependency mapping and staged execution, not speed.

