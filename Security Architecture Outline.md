# IRAS Agentic AI Technology Blueprint: Security Architecture
## Content Outline (Detailed)

---

## 1. Executive Summary

**Audience:** Deputy CEO, CIO, CISO
**Length:** 1.5–2 pages

- **The security imperative:** Agentic AI introduces a fundamentally new attack surface — agents that *act* on systems, not just generate text. Security must shift from static perimeter defence to continuous validation of agent identity, intent, context, and action.
- **IRAS-specific risk exposure:** Agents will access taxpayer PII, financial records, and IRIN3 production systems. A single compromised agent could trigger unauthorised transactions, data exfiltration, or regulatory breach (PDPA, IM8).
- **Security posture statement:** IRAS adopts a Zero Trust security model for agentic AI — no agent is trusted by default, every action is verified, and every interaction is logged.
- **Architecture in one sentence:** All agent interactions are mediated through two control planes (AO and AIGP), with defence-in-depth enforced from identity through to tool execution.
- **Key outcome:** This document defines the technical security controls, architecture patterns, and implementation specifications that operationalise the governance framework (cross-reference to Governance document).

---

## 2. Threat Landscape for Agentic AI at IRAS

**Audience:** Technical experts, CISO, security team
**Length:** 3–4 pages

### 2.1 Three-Layer Risk Model
- **Layer 1 — Classical cybersecurity:** SQL injection, remote code execution, network attacks. Already addressed by IM8 and existing IRAS controls.
- **Layer 2 — Inherited GenAI risks:** Prompt injection, jailbreaking, data leakage, hallucination. Partially addressed by AIEGF.
- **Layer 3 — New agentic risks:** Rogue actions, cascading multi-agent failures, emergent behaviours, agent manipulation, tool misuse, permission amplification.
- **Key point:** Layers are *cumulative* — agentic security must address all three.
- *Diagram: Three-layer risk pyramid with IRAS examples at each layer.*

### 2.2 Agentic-Specific Threat Vectors
Map to CSA Addendum threat categories and ARC Framework failure modes:

| Threat Vector | Description | IRAS Example | Source |
|---|---|---|---|
| Prompt injection (direct/indirect) | Attacker manipulates agent via crafted input | Taxpayer embeds instructions in correspondence that alter agent behaviour | CSA 4.1 |
| Tool misuse / tool description injection | Agent calls wrong tool or is misled by malicious tool metadata | Compromised MCP server returns manipulated IRIN3 data | CSA 3.3 |
| Memory poisoning | Attacker corrupts agent's long-term memory | Poisoned vector store causes agent to apply wrong tax policy | CSA 2.2 |
| Permission amplification | Agent escalates beyond delegated authority | Agent inherits admin-level IRIN3 access from misconfigured service account | MSFT Q&A |
| Cascading agent failure | Error in one agent propagates through multi-agent chain | Assessment agent error cascades to refund agent, triggering unauthorised payment | ARC |
| Data exfiltration via tool calls | Agent leaks sensitive data through external API calls | Agent includes taxpayer PII in prompt to external LLM | CSA 2.7 |
| Rug-pull attack on MCP server | Trusted MCP server is compromised or changes behaviour post-approval | Whitelisted MCP server update introduces data exfiltration | CSA 3.3 |

### 2.3 IRAS Threat Model
- Methodology: Use CSA-recommended taint tracing to map untrusted data flows through agent workflows.
- Scope: Cover the five IRAS use-case archetypes (taxpayer servicing, assessment, enforcement, internal productivity, cross-agency).
- Output: Threat model per autonomy level (L0–L3), mapped to risk tiers.
- *Diagram: Taint-tracing flow for a representative IRAS agentic workflow (e.g., taxpayer query → agent reasoning → IRIN3 lookup → response generation).*

---

## 3. Security Architecture Overview

**Audience:** Solution architects, platform engineers, CISO
**Length:** 4–5 pages

### 3.1 Architecture Principles
State the non-negotiable security design principles:
1. **Zero Trust for agent decisions:** Validate identity, intent, context, and action at every step — not just at the perimeter.
2. **Defence in depth:** Enforce controls at multiple layers (gateway, orchestrator, tool, data store) so no single point of failure compromises the system.
3. **Least privilege by design:** Agents receive the minimum permissions required for their registered task. Permissions are scoped, time-bound, and revocable.
4. **Secure by default:** New agents and tools start with zero permissions. Access is explicitly granted, never inherited implicitly.
5. **Full observability:** Every agent action, tool call, data access, and decision is logged, traceable, and auditable.
6. **Fail-safe over fail-open:** When controls fail, agents stop — they do not proceed with reduced security.

### 3.2 High-Level Security Architecture
- *Diagram: End-to-end security architecture showing the user, agent, AO (orchestration control plane), AIGP (IRIN3 control plane), LLM Gateway, MCP Gateway, and backend systems.*
- Explain how every request flows through the architecture:
  1. User initiates request → identity verified
  2. AO validates intent against agent's registered scope
  3. LLM Gateway enforces prompt firewalls, DLP on input
  4. Agent reasons and selects tools
  5. MCP Gateway validates tool call against policy
  6. AIGP enforces data access controls for IRIN3
  7. Output guardrails validate response before delivery
  8. Full trace logged end-to-end

### 3.3 Control Plane Model: AO and AIGP
Cross-reference to Governance document Section 7.1.1, but provide the *security-specific* detail:

| Control Plane | Security Functions |
|---|---|
| **Agent Orchestrator (AO)** | Prompt firewall, input/output DLP, policy-as-code validation, tool-call authorisation, rate limiting, circuit breakers, workflow-level anomaly detection |
| **AIGP (IRIN3 Control Plane)** | RBAC/ABAC enforcement, data masking, row/column-level security, transaction governance, RT data blocking, immutable audit logging |

- **PEP/PDP mapping:** Map AO to the Policy Enforcement Point (PEP) and AIGP's access-control layer to the Policy Decision Point (PDP) — using established enterprise security terminology.

### 3.4 Trust Boundaries
Define and diagram the trust boundaries:
- **Boundary 1:** User → Agent (authentication, session management)
- **Boundary 2:** Agent → LLM (prompt sanitisation, DLP, zero-retention enforcement)
- **Boundary 3:** Agent → Tools/MCP Servers (schema validation, sandboxing)
- **Boundary 4:** Agent → IRIN3 (AIGP access control, data classification enforcement)
- **Boundary 5:** Agent → Agent (inter-agent containment, data classification propagation)
- **Boundary 6:** Agent → External systems (egress DLP, no PII in external calls)

---

## 4. Identity, Authentication, and Access Control

**Audience:** IAM team, platform engineers, security architects
**Length:** 3–4 pages

### 4.1 Agent Identity Model
- Every agent has a unique machine identity (format: IRAS-AGT-[DIVISION]-[YYYY]-[SEQ]).
- Agent identity is linked to a supervising human and organisational unit.
- Identity includes: agent ID, owning division, supervising officer, registered capabilities, approved tools, data access scope.
- *Table: Agent identity attributes and their security function.*

### 4.2 Authentication
- Agent-to-system: OAuth 2.0 with short-lived tokens, scoped per task.
- Agent-to-agent: Mutual TLS or signed tokens with identity attestation.
- Agent-to-LLM: API key management via centralised secrets vault; automatic rotation.
- Human-to-agent: Existing IRAS SSO/ADFS, with session binding to prevent agent hijacking.

### 4.3 Authorisation Framework
- **RBAC + ABAC hybrid:** Base permissions from agent role; dynamic constraints from attributes (user identity, task purpose, data classification, time of day, runtime context).
- **Delegation rule:** Agent permissions must never exceed the delegated authority of the supervising human. Enforce at platform level, not prompt level.
- **Least privilege enforcement:** Per-tool, per-dataset, per-action permissions. Deny by default.
- **Dynamic scoping:** Permissions can be narrowed (never widened) based on runtime context.
- *Table: Permission matrix — Agent role vs. allowed actions (read, write, delete, external send) vs. data classification level.*

### 4.4 Delegated Authority and Permission Inheritance
- How delegation chains work in multi-agent setups.
- Rule: Sub-agents inherit the *most restrictive* permission set of the delegating agent and the originating human.
- Preventing permission amplification across agent chains.

---

## 5. Data Protection

**Audience:** Data Governance Officer, CDO, security architects
**Length:** 3–4 pages

### 5.1 Data Classification Enforcement for Agents
- Map IRAS data classifications (Open, Official-Closed, Restricted, Confidential, Secret) to agent access controls.
- Cross-reference to Governance document Section 6.5, but provide technical implementation:
  - **Infrastructure-level enforcement** (database ACLs, API gateway policies), not prompt-level restrictions.
  - **AIGP data masking:** PII fields masked by default; unmasking requires explicit justification and approval.
  - **RT (Restricted Taxpayer) data:** Blocked at AIGP level. No agent access without IRIN3 System Owner approval.

### 5.2 Data Loss Prevention (DLP)
- **Ingress DLP:** Scan all inputs to agents for sensitive data injection attempts.
- **Egress DLP:** Scan all outputs and tool calls for PII, financial data, classified information before delivery.
- **LLM-bound DLP:** Strip/redact PII before any prompt is sent to externally hosted LLMs. Enforce zero-retention clauses contractually and technically (no logging >24hrs by provider).
- Implementation: PII detection (regex + NER), data classification tags, content filtering at AI Gateway.

### 5.3 Agent Memory and Context Security
- Short-term memory (conversation history): Session-scoped, purged on session end.
- Working memory (session facts): No PII persistence without explicit approval.
- Long-term memory (vector stores): Encrypted at rest, access-controlled per agent, no cross-agent memory sharing without Data Governance Officer approval.
- RAG pipeline security: Ingestion must respect source document classification; retrieval must enforce the requesting agent's data access scope.

### 5.4 Inter-Agent Data Flow Controls
- Data classification follows the *highest watermark* rule: if Agent A (Restricted) communicates with Agent B (Official-Closed), the entire interaction is classified as Restricted.
- Schema validation on all inter-agent messages.
- No implicit data sharing — all flows must be explicitly declared in the agent registration.

---

## 6. Network and Infrastructure Security

**Audience:** Infrastructure team, CISO, platform engineers
**Length:** 2–3 pages

### 6.1 Hosting and Data Residency
- Map to SNG(PMO) Circular No. 3/2025 and IM8 requirements.
- *Table: Data classification → hosting requirements (GCC, GCC+, Singapore-hosted, S-Net) — replicate from Governance doc Section 7.2 for completeness, with security rationale.*

### 6.2 Network Segmentation
- Agent execution environments isolated from IRAS corporate network.
- Separate network zones for: agent orchestration, LLM inference, IRIN3 access, external API calls.
- Micro-segmentation between agent workloads (no lateral movement between agents).

### 6.3 Sandboxed Execution
- High-risk operations (code execution, file manipulation, external tool calls) run in isolated containers.
- Sandbox constraints: no network access to IRIN3 from sandbox, time-limited execution, resource limits (CPU, memory, egress).
- Tool execution sandboxing for untrusted MCP server responses.

### 6.4 API and MCP Gateway Security
- Centralised gateway for all agent-to-tool and agent-to-system interactions.
- Controls: rate limiting, schema validation, request signing, TLS enforcement, circuit breakers.
- MCP-specific: Whitelist of approved MCP servers, tool description integrity checks (prevent tool description injection), runtime monitoring of tool behaviour against registered schemas.

---

## 7. Security Controls by Lifecycle Phase

**Audience:** Development teams, security team, project managers
**Length:** 5–6 pages (main reference section for implementation)

### 7.1 Planning and Design Phase
- Threat modelling with taint tracing (mandatory for L2+ autonomy).
- Security requirements definition per risk tier.
- Agent scope definition: tools, data access, autonomy boundaries.
- Secure architecture review by ATAC.
- *Checklist: Planning-phase security deliverables by risk tier.*

### 7.2 Development Phase

#### 7.2.1 Planning and Reasoning Controls
- Prompt agent to reflect on plan adherence to instructions.
- Prompt agent to summarise understanding before proceeding.
- Log agent plans and reasoning for review.
- Limit maximum planning steps and iteration depth.

#### 7.2.2 Tool Controls
- Strict input format validation for all tools.
- Least-privilege tool access — only registered tools available.
- No write access to sensitive databases unless strictly required and approved.
- Hand-over to human for sensitive data entry (passwords, API keys).

#### 7.2.3 Protocol Controls (MCP/A2A)
- Whitelist trusted MCP servers only.
- Sandbox all code execution via MCP.
- Input guardrails: prompt injection detection, adversarial input filtering.
- MCP server supply-chain vetting: code review, security attestation, integrity hashing.
- A2A protocol security: message signing, schema validation, trust boundary enforcement.

#### 7.2.4 Secure Coding Practices
- Treat agent instructions (system prompts) as security-critical code — version controlled, reviewed, tested.
- No secrets in prompts or agent configuration.
- Dependency management: SBOM for all agent components, MCP servers, and tool dependencies.

### 7.3 Testing and Pre-Deployment Phase

#### 7.3.1 What to Test (Security-Specific)
- Prompt injection resistance (direct and indirect).
- Tool-call correctness under adversarial conditions.
- Permission boundary enforcement (can agent exceed its registered scope?).
- Data leakage testing (does agent expose PII in outputs, tool calls, or logs?).
- Multi-agent containment (does compromise of one agent propagate?).

#### 7.3.2 How to Test
- Red teaming: Mandatory for Tier 2/3 agents. Cover prompt injection, privilege escalation, data exfiltration, tool misuse.
- Penetration testing: Standard application pentest + agent-specific scenarios.
- Adversarial evaluation datasets: Include edge cases, boundary violations, and manipulation attempts.
- Test in realistic environments that mirror production (with safeguards).
- Test repeatedly — agent behaviour is stochastic.
- *Table: Testing requirements by risk tier (Tier 1: automated security scan; Tier 2: + red team; Tier 3: + external pentest).*

### 7.4 Deployment Phase

#### 7.4.1 Graduated Rollout
- Phase by: user groups → tools available → systems exposed.
- Start with read-only access, expand to write actions after validation period.
- Shadow mode: Agent runs in parallel with human, outputs compared but not executed.

#### 7.4.2 Deployment Security Checks
- Automated policy validation before promotion to production.
- Security attestation sign-off (ICT Security).
- Verify agent registration is complete and current.
- Confirm HITL checkpoints are configured per risk tier.
- Circuit breakers and kill switches tested and operational.
- *Checklist: Pre-deployment security gate requirements.*

### 7.5 Operations and Maintenance Phase
- Covered in detail in Section 8 (Monitoring and Incident Response).
- Cross-reference to re-assessment triggers (Governance doc Section 17).

---

## 8. Monitoring, Detection, and Incident Response

**Audience:** SecOps, platform engineers, CISO
**Length:** 4–5 pages

### 8.1 Observability Architecture
- *Diagram: Observability stack — telemetry sources (agents, AO, AIGP, gateways, tools) → centralised logging → dashboards and alerting.*
- Unique trace ID per request, linking: user, agent, prompts, responses, tool calls, data access events, HITL decisions.
- All logs tamper-evident and immutable.

### 8.2 What to Log
- Every AI interaction as an immutable record:
  - Prompt and response (PII-redacted copy + encrypted full copy)
  - Model ID, version, temperature, system-prompt hash
  - Tool calls with parameters and returned data
  - RAG: retrieved documents and source citations
  - HITL gate decisions: approver identity, decision, timestamp
  - Policy-engine decisions and classifier scores
  - User ID, session, device, geography, UTC timestamp

### 8.3 Monitoring Approaches
- **Programmatic threshold-based alerts:** Unauthorised access attempts, repeated tool failures, excessive retries, cost spikes.
- **Anomaly detection:** Statistical and ML-based detection of unusual agent behaviour patterns (tool usage anomalies, response time deviations, unexpected data access patterns).
- **Agent-monitoring-agents:** Dedicated monitoring agents that validate other agents' behaviour in real time.
- *Table: Alert type → trigger condition → severity → response action.*

### 8.4 Graduated Interventions
| Alert Level | Trigger | Response |
|---|---|---|
| Low | Minor deviation, informational anomaly | Flag for scheduled review |
| Medium | Policy violation, repeated errors | Temporarily halt agent, notify Supervising Officer |
| High | Unauthorised action, data breach indicator | Terminate agent, isolate systems, notify ICT Security + Division Head |
| Critical | Confirmed breach, data exfiltration, external impact | Kill switch, preserve forensic state, invoke incident response playbook |

### 8.5 Security Incident Response
- Cross-reference to Governance document Appendix 14 (Incident Response Playbook).
- Security-specific additions:
  - Forensic log preservation procedures.
  - Agent isolation and containment steps.
  - Compromised MCP server response.
  - PDPC breach notification triggers and timeline.
  - Post-incident: threat model update, control gap remediation, propagation to all affected agents.

### 8.6 AI Cockpit / Security Dashboard
- Unified monitoring dashboard providing single pane of glass across ITOps, SecOps, FinOps.
- Key security metrics: prompt injections blocked, unauthorised access attempts, HITL override rates, agent compliance scores, vulnerability scan results.

---

## 9. Compliance and Framework Mapping

**Audience:** Compliance team, CISO, auditors
**Length:** 2–3 pages

### 9.1 Regulatory and Framework Alignment
*Table mapping this document's controls to:*

| Control Area | IM8 Classic | IM8 Reform | PDPA | CSA Addendum | IMDA MGF | ARC Framework |
|---|---|---|---|---|---|---|
| Identity & Access | ... | ... | ... | 2.6 | 2.1.2 | ... |
| Data Protection | ... | ... | ... | 2.4, 2.7 | 2.1.1 | ... |
| Monitoring & Logging | ... | ... | ... | 4.3 | 2.3.3 | ... |
| (etc.) | | | | | | |

### 9.2 Audit Requirements
- What auditors will need: agent inventory, access logs, HITL decision records, red team reports, policy-engine configurations, incident reports.
- Audit frequency by risk tier.
- Integration with existing IRAS audit processes.

---

## 10. Implementation Roadmap

**Audience:** CIO, project managers, platform engineers
**Length:** 2–3 pages

### 10.1 Phased Implementation
| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1: Foundation** | Months 1–3 | Agent identity framework, centralised logging, AI Gateway with DLP, MCP server whitelist, agent registration process |
| **Phase 2: Core Controls** | Months 4–6 | RBAC/ABAC enforcement in AIGP, prompt firewalls, circuit breakers, HITL workflow integration, Tier 1 agent deployment |
| **Phase 3: Advanced** | Months 7–9 | Anomaly detection, agent-monitoring-agents, red team program, Tier 2 agent deployment, security dashboard |
| **Phase 4: Maturity** | Months 10–12 | Full Zero Trust enforcement, Tier 3 agent deployment (with maximum controls), continuous security validation, external audit |

### 10.2 Dependencies and Prerequisites
- Existing infrastructure requirements (GCC, network segmentation).
- Tooling decisions (AI Gateway product, observability stack, policy engine).
- Team readiness (security team training on agent threat modelling).

### 10.3 Success Criteria
- Security-specific metrics: mean time to detect agent anomaly, % of agents with complete security attestation, red team finding remediation rate, zero P1 security incidents in first 12 months.

---

## Appendices

- **Appendix A: Detailed Control Specifications** — Full technical specification for each control referenced in Section 7, with implementation guidance.
- **Appendix B: Agent Security Review Checklist** — Per-agent security assessment checklist for ATAC review.
- **Appendix C: Red Team Playbook** — Scenarios, attack vectors, and evaluation criteria for agent red teaming.
- **Appendix D: MCP Server Security Requirements** — Vetting criteria, onboarding process, and ongoing monitoring requirements for MCP servers.
- **Appendix E: Zero Trust Decision Loop Specification** — Technical specification for the four-dimension validation (identity, intent, context, action) at each trust boundary.
- **Appendix F: Glossary and Acronyms**

---

## Document Relationships

| Document | Relationship |
|---|---|
| Governance and Risks Blueprint | Parent document. This security architecture operationalises the Technology Dimension (Section 7) and Technical Controls (Appendix 16). |
| IRAS AI Ethics and Governance Framework (AIEGF) | Foundational AI principles. This document extends AIEGF for agentic-specific security. |
| CSA Draft Addendum on Securing Agentic AI | Primary external security reference. Controls mapped in Section 9.1. |
| IMDA Model AI Governance Framework for Agentic AI | Governance framework. Security controls implement MGF Section 2.3. |
| GovTech ARC Framework | Risk taxonomy. Risks mapped to controls throughout. |
| IM8 (Classic and Reform) | Compliance baseline. Mapping in Section 9.1. |
