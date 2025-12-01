# SCALE Worked Example: STRIDE Threat Modeling in Under 45 Minutes
(For teams who are experiencing STRIDE in the duration of days)

## Overview
Most teams that complain “STRIDE takes forever” are doing it wrong:  
- Full-system DFDs  
- 4-hour workshops per service  
- 40-page reports nobody ever reads  

This example proves you can do **useful, actionable STRIDE threat modeling in < 45 minutes** on a real feature and ship the fixes the same week.

Feature: **Bulk CSV upload to invite team members**  
`POST /api/v1/organizations/:org_id/bulk-invite` (multipart CSV → background job → send emails)

---
# 1. Scope (3 min)
Only model what actually changes.

**In scope**
- Single endpoint + background job
- File lands in pod `/tmp`, parsed, then deleted
- CSV → list of emails + optional names/roles

**Out of scope**
- Authentication (existing JWT middleware)
- Email delivery service
- The rest of the platform

---
# 2. Context & Constraints (5 min)
Reality check – no architecture diagram required.

- 40-person startup, shipping weekly, SOC 2 Type 1 in 3 months
- Rails/Node API on Kubernetes (AWS/GCP)
- No malware scanning service yet
- File is written to ephemeral `/tmp`
- Hard constraint: ship in 7 days, max 2 new dependencies this quarter

Trust boundary (one line):  
Internet → API Gateway → Pod → /tmp → Background worker → SendGrid

---
# 3. Attack & Threat Scenarios – STRIDE (25 min)
One data flow. Six STRIDE categories. Only credible threats make the list.

| STRIDE       | Threat                                      | How it happens                                   | Likelihood | Impact |
|--------------|---------------------------------------------|--------------------------------------------------|------------|--------|
| **S**poofing | Attacker impersonates another org          | Guess/brute-force `:org_id` (sequential IDs)     | Medium     | High   |
| **T**ampering | Formula injection in CSV cells              | Upload `=HYPERLINK("evil.com", "Click")`         | High       | High   |
| **R**epudiation | No proof who uploaded the CSV              | Logs deleted or never created                    | Medium     | Medium |
| **I**nformation Disclosure | PII from CSV leaks via logs or /tmp       | Row content logged, pod compromised              | High       | High   |
| **D**enial of Service | Zip bomb / billion laughs CSV            | 5 MB file expands to 50 GB → OOM all pods        | High       | High   |
| **E**levation of Privilege | Unlimited rows → queue/resource exhaustion | 10 M row CSV → worker runs forever               | High       | High   |

Only 6 threats. Done in 25 minutes.

---
# 4. Leverage Patterns & Controls (10 min)
One control per threat. All implementable this week.

| STRIDE | Mitigation (code-ready)                                                                 | Effort    |
|-------|------------------------------------------------------------------------------------------|-----------|
| S     | Use UUIDv4 for `org_id` + explicit `can?(:invite, org)` check                            | 30 min    |
| T     | Strip leading `= + - @` from every cell before using in email                           | 15 min    |
| R     | Immutable audit event: `actor_id`, `org_id`, `row_count`, `upload_ip`                   | 20 min    |
| I     | Never log any CSV content; redact entire payload; `File.delete` immediately after parse | 15 min    |
| D     | Hard limits: 5 MB file, 1 000 rows max; reject earlier                                  | 20 min    |
| E     | Job timeout 60 s + max retries = 1; circuit-breaker on worker queue                     | 30 min    |

Total implementation time: **~2.5 hours**

---
# 5. Enablement & Scale (5 min)
Make this the new normal instead of a one-off.

| Action                                    | How                                                                 |
|-------------------------------------------|---------------------------------------------------------------------|
| PR template                               | Add mandatory STRIDE checkbox list (S-T-R-I-D-E)                    |
| Reusable module                           | `BulkUploadSafety` concern with limits, stripping, audit, cleanup  |
| Calendar event                            | Recurring 30-min “Quick STRIDE” slot any engineer can book          |
| One-pager template                        | Notion/Google-Doc with exactly the tables above                     |
| Company rule                              | Any file upload or bulk endpoint → 30-min STRIDE required           |
| Gallery                                   | Internal wiki of completed STRIDE one-pagers (learning library)     |

Next time someone adds file upload, they copy the template, spend 30 minutes, and ship safely — forever.

---
# Time Breakdown

| Step                  | Time   | Deliverable                     |
|-----------------------|--------|---------------------------------|
| Scope                 | 3 min  | One-sentence boundary           |
| Context               | 5 min  | Constraints & trust boundary    |
| STRIDE analysis       | 25 min | 6 high-impact threats           |
| Controls              | 10 min | 6 concrete mitigations          |
| Enablement            | 5 min  | Repeatable process & templates  |
| **Total**             | **48 min** | **Shippable fixes + scalable process** |

---
# Conclusion
You just did real STRIDE threat modeling not the academic 3-day version in under 45 minutes and shipped every meaningful fix the same sprint. 
Now you "scale" this (pun intended) across all of the data flows and if you can, you run the sessions in parallel
No full-system DFD. No per-endpoint sprawl. Just an actionable 2-page model with prioritized mitigations that ship in one sprint.

Stop doing marathon STRIDE workshops that produce PDFs.  
Start doing 45-minute SCALE+STRIDE sessions that produce pull requests.

Your engineers will thank you. Your auditors will thank you. And you’ll actually have threat modeling that scales.
