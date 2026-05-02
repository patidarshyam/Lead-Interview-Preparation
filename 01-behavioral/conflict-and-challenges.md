# Conflict & Challenges

---

## Q1: Tell me about a time you disagreed with a technical decision. How did you handle it?

**Answer (STAR):**

- **Situation:** The team had committed to a database migration on the roadmap. Based on my analysis, I believed it was premature and would consume significant resources without proportional benefit.
- **Task:** I needed to respectfully challenge the existing plan while providing a strong alternative recommendation.
- **Action:** Rather than simply objecting, I prepared a detailed analysis: current system performance metrics, projected growth, cost comparison (migration cost vs. staying on current system), and risk assessment. I presented this in a design review meeting, framing it as "let's validate our assumptions before committing resources" rather than "this is wrong."
- **Result:** The team and stakeholders agreed with the deferral. The key was coming with data, not just opinions, and framing the discussion constructively.

> **Key takeaway:** Disagree with data, not emotion. Frame it as collective decision-making.

---

## Q2: Describe a time when you faced a significant technical blocker. How did you work through it?

**Answer (STAR):**

- **Situation:** An API was failing intermittently with connection timeouts. Standard debugging (logs, metrics, application config) wasn't revealing the cause. The issue crossed service boundaries.
- **Task:** Find the root cause and resolve it before it impacted production.
- **Action:** I went beyond application-level debugging. I traced the network path, reviewed VPC configurations, subnet routing tables, and security group rules. I discovered that VPC peering between two services was misconfigured for certain availability zones. I worked with the infrastructure team to fix the peering configuration and validated end-to-end connectivity.
- **Result:** Issue resolved. I documented the debugging methodology so the team could apply similar network-level debugging in the future.

---

## Q3: How do you handle pressure when multiple critical issues arise simultaneously?

**Answer:**

I use a triage approach:

1. **Assess impact** — Which issue affects production/customers? That goes first
2. **Delegate where possible** — If I have team members who can own one issue, I delegate with clear context
3. **Communicate early** — I inform stakeholders about timelines and trade-offs immediately rather than trying to silently handle everything
4. **Document as I go** — Even under pressure, I take notes. This prevents re-investigation and helps the team learn from the incident

---

## Q4: Tell me about a time when your initial approach to a problem didn't work.

**Answer (STAR):**

- **Situation:** When investigating the authentication issue in an internal support tool, my initial hypothesis was that the token refresh mechanism was broken.
- **Task:** Fix authentication failures that were blocking the support team.
- **Action:** I spent time debugging the token flow, but the issue persisted. I stepped back, widened the investigation scope, and discovered the root cause was actually in how the tool was integrated with the identity provider (Okta) — the configuration had drifted after an Okta upgrade. I also identified that adding a WAF (Web Application Firewall) would prevent similar issues and add a security layer.
- **Result:** Fixed the authentication issue and implemented WAF as a preventive measure. The lesson was: when your first hypothesis fails, widen the scope rather than digging deeper into the same hole.

---

## Q5: Tell me about a time you challenged a stakeholder decision backed by data.

**Answer (STAR):**

- **Situation:** A database migration from RDS PostgreSQL to Aurora was on the roadmap with resources already being allocated. The team assumed the current setup lacked sufficient resilience and failover capability.
- **Task:** I was assigned to execute the migration but wanted to validate the assumption first.
- **Action:** I reviewed the existing RDS configuration and discovered Multi-AZ was already enabled. I ran a controlled failover test in QA : automatic standby promotion occurred in 30–60 seconds. I also prepared a cost comparison (existing: ~$2K vs. new Aurora: ~$8K over 10 months) and mapped the timeline against the upcoming Parquet migration. I brought this analysis to the stakeholder review with clear trade-off framing: "Here's what we'd gain vs. what we'd risk and spend."
- **Result:** Stakeholders agreed to cancel the migration. The data-backed recommendation was accepted without conflict. Saved ~$6K and prevented a high-risk operation on 9.2 TiB of production data.

> **Key takeaway:** Challenging decisions works best when you replace the objection with a quantified alternative, not just a counter-opinion.

---

## Q6: How do you handle a situation where you're under pressure to deliver but the technical approach is wrong?

**Answer:**

This happens regularly, and my approach is:

1. **Isolate the risk clearly** : I document what will break or cost more if we proceed with the flawed approach. Vague concerns don't move stakeholders; specific cost/risk estimates do.
2. **Propose a time-boxed validation** : "Give me one day to validate this assumption before we commit two weeks to building it wrong" is a hard argument to reject.
3. **Escalate with options, not problems** : I bring three choices (proceed as-is with known risks, pivot approach, defer), let stakeholders decide with full information.
4. **Accept the decision and execute** : If the decision is to proceed despite the risk, I document the decision, flag it clearly, and execute with guardrails. I don't undermine the decision, but I make the risk visible.

> The DB migration challenge is the clearest example : I validated the concern with a failover test and cost model before escalating, which made the recommendation impossible to dismiss.


---

## Q5: Tell me about a time you challenged a stakeholder decision backed by data.

**Answer (STAR):**

- **Situation:** A database migration from RDS PostgreSQL to Aurora was on the roadmap with resources already being allocated. The team assumed the current setup lacked sufficient resilience and failover capability.
- **Task:** I was assigned to execute the migration but wanted to validate the assumption first.
- **Action:** I reviewed the existing RDS configuration and discovered Multi-AZ was already enabled. I ran a controlled failover test in QA : automatic standby promotion occurred in 30–60 seconds. I also prepared a cost comparison (existing: ~$2K vs. new Aurora: ~$8K over 10 months) and mapped the timeline against the upcoming Parquet migration. I brought this analysis to the stakeholder review with clear trade-off framing: "Here's what we'd gain vs. what we'd risk and spend."
- **Result:** Stakeholders agreed to cancel the migration. The data-backed recommendation was accepted without conflict. Saved ~$6K and prevented a high-risk operation on 9.2 TiB of production data.

> **Key takeaway:** Challenging decisions works best when you replace the objection with a quantified alternative, not just a counter-opinion.

---

## Q6: How do you handle a situation where you're under pressure to deliver but the technical approach is wrong?

**Answer:**

This happens regularly, and my approach is:

1. **Isolate the risk clearly** : I document what will break or cost more if we proceed with the flawed approach. Vague concerns don't move stakeholders; specific cost/risk estimates do.
2. **Propose a time-boxed validation** : "Give me one day to validate this assumption before we commit two weeks to building it wrong" is a hard argument to reject.
3. **Escalate with options, not problems** : I bring three choices (proceed as-is with known risks, pivot approach, defer), let stakeholders decide with full information.
4. **Accept the decision and execute** : If the decision is to proceed despite the risk, I document the decision, flag it clearly, and execute with guardrails. I don't undermine the decision, but I make the risk visible.

> The DB migration challenge is the clearest example : I validated the concern with a failover test and cost model before escalating, which made the recommendation impossible to dismiss.

