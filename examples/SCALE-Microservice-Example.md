# SCALE Worked Example: Securing a User Profile Microservice

## Overview
This example applies the **S.C.A.L.E Security Reasoning Framework** to a common engineering scenario: a microservice that handles user profile data. The goal is to show how developers can use SCALE to reason about practical risks and guardrails in a straightforward, engineering-friendly way.

---

# 1. Scope  
Define the system or boundary.

Included components:
- REST/JSON HTTP microservice
- User profile CRUD operations
- PostgreSQL database
- API Gateway / Load Balancer
- JWT-based authentication
- Outbound calls to an email service
- CI/CD pipeline for deployments
- Containerized runtime (Docker/Kubernetes)

Out of scope: other services not directly interacting with profile data.

---

# 2. Context & Constraints  
Identify architecture, limits, and trust zones.

### Architecture & Environment
- Stateless microservice deployed in Kubernetes  
- Each instance uses a shared secrets store (Vault/SM)  
- Ingress via API Gateway with JWT validation  
- Terraform-managed infrastructure  
- Uses a shared base container image

### Constraints
- Updates must not break dependent services  
- Database downtime tolerance is low  
- Compliance requires access logging for personal data  
- The service must handle elevated traffic bursts  
- Dev teams require fast deployments (CI/CD driven)

### Trust Boundaries
- External clients → API Gateway → Microservice  
- Microservice ↔ Database  
- Microservice ↔ Email provider  
- CI/CD pipeline → Container registry → Cluster

---

# 3. Attack & Threat Scenarios  
Determine realistic failure modes.

### API-Level Threats
- Broken object-level authorization (e.g., user editing someone else’s profile)
- Missing or weak input validation → injection attacks
- JWT forgery or misuse
- Insecure direct object reference (IDOR)
- Unbounded data updates or query manipulation

### Database Threats
- SQL injection
- Excessive privilege on the DB user account
- Unauthorized data extraction
- Unencrypted data-at-rest or data-in-transit
- Abuse of shared connections

### Container / Runtime Threats
- Leaked environment variables (secrets)
- Dependency vulnerabilities in the base image
- Container escape risks
- Over-permissive network policies

### CI/CD & Supply Chain Threats
- Malicious dependency injection via package updates
- Compromise of CI pipeline secrets
- Pushing unscanned images to production
- Poisoned container base image

### Operational Threats
- Logging personal data in plaintext
- Overly verbose debug endpoints exposed
- Missing rate limits → denial of service

---

# 4. Leverage Patterns & Controls  
Apply proven mitigations and guardrails.

### API-Level Controls
- Enforce object-level authorization  
- Validate all input (schema + type + range)  
- Strong JWT validation with rotation  
- Use UUIDs instead of incrementing IDs  
- Central API Gateway policies (rate limit, authN, authZ)

### Data & Database Controls
- Parameterized queries or ORM protections  
- Principle of least privilege for DB user  
- Encrypt data-at-rest (KMS-managed)  
- Connection pooling with strict limits  
- Row-level access checks if necessary

### Container & Runtime Controls
- Use minimal base images (distroless, alpine, etc.)  
- Container image scanning in CI  
- Kubernetes NetworkPolicies blocking lateral movement  
- Secrets injected via Vault/SM, not ENV files  
- Resource limits to prevent noisy-neighbor attacks

### CI/CD & Supply Chain Controls
- Pin dependency versions  
- Use SCA scanning in PR and pipeline  
- Mandatory SAST for critical paths  
- Signed container images (Sigstore/Cosign)  
- Enforce policy-as-code (OPA/Conftest)

### Operational Controls
- Structured logging with sensitive field redaction  
- Health-check endpoints separate from admin endpoints  
- Observability: traces, metrics, error rate alerts  
- API Gateway rate limiting  
- WAF rules for common API abuse signatures

---

# 5. Enablement & Scale  
Ensure solutions are adoptable and automatable.

### Developer Enablement
- Provide OpenAPI schemas with built-in validation  
- Offer code templates for secure CRUD operations  
- Provide a shared Typescript/Go/Python SDK  
- Self-service secrets onboarding via UI/CLI  
- Local dev environment that mirrors prod guardrails

### Reusable Guardrails
- Standardized JWT middleware  
- Centralized schema validation library  
- Shared base image with security hardening baked in  
- Terraform modules that enforce least privilege  
- Standard CI pipeline template (scan → test → sign → deploy)

### Automation
- Automated SCA and SAST in PR checks  
- Auto-scan containers on build and push  
- Auto-provision RBAC roles for new services  
- Auto-inject service identity (SPIFFE/SPIRE)

### Scaling Across Teams
- Document secure API patterns  
- Provide internal examples of object-level auth  
- Maintain a catalog of approved dependencies  
- Offer a “microservice starter kit” with SCALE guardrails implemented

---

# Conclusion
This example shows how developers can apply the **S.C.A.L.E framework** to practical microservice design and deployment. By following the five steps, Scope, Context, Attack Scenarios, Leverage Controls, and Enablement teams can reason about risk, choose the correct guardrails, and scale security patterns across the engineering organization without adding unnecessary friction.

