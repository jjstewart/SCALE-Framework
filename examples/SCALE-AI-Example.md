# SCALE Worked Example: Securing an LLM-Powered Internal Assistant

## Overview
This document demonstrates how to apply the **S.C.A.L.E Security Reasoning Framework** to a realistic AI system: an internal LLM-powered assistant used by engineering teams for log analysis, code drafting, alert triage, and document retrieval.

---

# 1. Scope  
Define the system or boundary.

Systems and components included:
- LLM API calls (OpenAI, Anthropic, or internal models)
- API gateway and service wrapper
- Retrieval-Augmented Generation (RAG) service
- Vector database / embeddings store
- Internal document repositories
- Authentication and authorization layer
- Logging and monitoring paths

---

# 2. Context & Constraints  
Identify architecture, limits, and trust zones.

### Context
- LLM may be external → data egress risk  
- RAG index contains internal or confidential documents  
- Authentication via corporate SSO → trust boundary at identity provider  
- Caching prompts/responses for latency → storage risk  
- System must support engineers without slowing productivity  
- The LLM does not understand corporate boundaries → risk of hallucinated permission bypass

### Constraints
- Some documents cannot leave internal networks  
- Compliance requires auditability of all queries  
- Model usage costs must be controlled  
- Engineering will not adopt high-friction guardrails

---

# 3. Attack & Threat Scenarios  
Determine realistic failure modes.

### Prompt-Level Threats
- Direct prompt injection
- Indirect prompt injection (via logs, docs, tickets)
- Model steering
- Context leakage
- Jailbreaks

### RAG & Data Threats
- Sensitive document exfiltration through model outputs
- Retrieval poisoning
- Embedding inversion attacks
- Data injection into RAG sources

### System-Level Threats
- Bypassing RBAC using natural language prompts
- “Act as admin” privilege escalation
- Unauthorized cross-dataset enrichment
- Hallucinated instructions or policies

### Operational Threats
- Logging sensitive prompts or responses
- Excessive model consumption degrading service
- Compromised model weights or supply chain attacks

---

# 4. Leverage Patterns & Controls  
Apply proven mitigations and guardrails.

### Identity & Authorization
- RBAC enforcement before LLM invocation
- Policy Enforcement Point (PEP) between user and model
- Immutable system instructions to anchor model behavior

### Data Controls
- Redaction layer before API calls
- Dataset tiering (public → internal → restricted)
- Per-user document filtering for RAG queries
- Encryption of embeddings and metadata

### Model Safety Guardrails
- Prompt hardening
- Safety wrapper validating request patterns
- Output filtration and validation
- Split-execution: model proposes; deterministic logic approves

### Monitoring & Observability
- Prompt anomaly detection (injection, jailbreak patterns)
- Logging on RAG recall paths
- Rate limiting
- Detection of unusual query patterns

### Operational Controls
- Canary rollouts for new model versions
- Separation of business logic from model-generated output
- Review workflow for high-risk or admin-level requests

---

# 5. Enablement & Scale  
Ensure solutions are adoptable and automatable.

### Developer Enablement
- Standardized prompt templates
- SDK wrappers enforcing redaction, auth, and retry logic
- Self-service onboarding with least-privilege scopes
- Safe sandbox for injection testing

### Reusable Guardrails
- Shared embedding pipelines
- Approved RAG connectors
- Centralized redaction library
- YAML-based policy definitions reusable across services

### Automation
- Auto-scanning new documents before RAG ingestion
- Auto-detection of anomalous or high-risk prompts
- CI checks to detect unsafe LLM usage patterns in code

### Scaling Across Teams
- Documentation of approved AI patterns
- Versioned guardrails
- AI Security Review powered by SCALE

---

# Conclusion
Applying **S.C.A.L.E** to AI systems provides a structured, engineering-focused approach to identifying risks, selecting controls, and designing guardrails that scale. This example demonstrates how SCALE can be used to secure modern LLM-integrated applications without obstructing engineering workflows.
