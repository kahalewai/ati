# Agent Trust Interconnect (ATI) Model

**Status:** Working Draft

**Version:** 0.1.0

**Date:** March 2026

**License:** Apache License 2.0

---

## Abstract

The Agent Trust Interconnect (ATI) Model is a reference model that defines the complete set of trust concerns that must be addressed for autonomous agent systems to operate with verifiable authority on behalf of principals. It decomposes the problem of delegated trust in agent systems into eight layers, each answering a distinct trust question, ordered by dependency from foundational transport security through accountability and audit.

The ATI Model is to autonomous agent trust what the OSI Model is to network interconnection: a shared vocabulary for understanding where different standards, protocols, and implementations fit — and where gaps remain.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [The Problem Space](#2-the-problem-space)
3. [What the ATI Model Captures](#3-what-the-ati-model-captures)
4. [How the ATI Model Differs from Existing Models](#4-how-the-ati-model-differs-from-existing-models)
5. [Layer Architecture](#5-layer-architecture)
6. [Agent Request Flow Through the Layers](#6-agent-request-flow-through-the-layers)
7. [Standards Mapping](#7-standards-mapping)
8. [Threat Model by Layer](#8-threat-model-by-layer)
9. [Gap Analysis](#9-gap-analysis)
10. [Adoption and Composability](#10-adoption-and-composability)
11. [Relationship to Other Reference Models](#11-relationship-to-other-reference-models)
12. [Versioning](#12-versioning)
13. [References](#13-references)

---

## 1. Introduction

AI agent systems increasingly operate as networks of autonomous actors that interpret instructions, make decisions, delegate tasks to other agents, and invoke tools on behalf of humans or organizations. A human says "book me a trip to Tokyo." An orchestrating agent interprets that authorization and delegates subtasks to specialist agents. Those agents invoke tools. At each step, authority must travel with the action — and at no step may it expand beyond what was originally granted.

This creates a trust problem that no single standard can solve. Identity must be established. Sessions must be authorized. Governance must monitor ongoing behavior. Scope must narrow through delegation chains. Tools must be verified as genuine. Constraints must be enforced against actual action parameters. And every decision must be recorded for accountability.

Today, the industry is building standards to address these concerns — but without a shared reference model, people conflate layers. They think identity is authorization. They think structural constraint propagation is semantic enforcement. They build systems that handle three concerns well and completely ignore the other five, then wonder why their agent deployment has security gaps.

The ATI Model provides the map. It does not tell you how to implement each layer — it tells you that each layer exists, what question it answers, and where the boundaries are.

---

## 2. The Problem Space

### 2.1 The Core Problem

When an autonomous agent acts on behalf of a principal, what must be true — and verifiably true — at every stage from the establishment of identity through the execution and recording of the action?

This question has no single answer. It decomposes into at least eight distinct trust concerns, each requiring different mechanisms, operating at different points in the agent workflow, and failing in different ways when absent.

### 2.2 Why Existing Models Are Insufficient

**This is not a networking problem.** OSI describes how data moves between systems. The ATI Model describes how trust and permission move through a system of autonomous actors — actors that interpret, decide, and act rather than simply forwarding.

**This is not a traditional access control problem.** RBAC, ABAC, and similar models assume a single decision point evaluating a single request. The defining characteristic of the agent problem is that authorization is decomposed and re-delegated across multiple autonomous hops, each of which narrows scope. Traditional access control doesn't have a concept for "I'm granting you a subset of what I was granted, and you'll grant a subset of that to someone else."

**This is not a zero-trust architecture**, though it shares DNA. Zero trust says "never trust, always verify" at the network and session level. The ATI Model says "never trust, always verify" at the delegation level — every hop in an agent chain is a point where trust must be verified, not assumed.

### 2.3 What Makes the Agent Context Unique

Four characteristics distinguish the agent trust problem from prior distributed systems challenges:

**Autonomy with interpretation.** Traditional microservice A calling microservice B involves deterministic request forwarding. Agent A delegating to Agent B involves an LLM interpreting what it received and deciding what to ask for next. That interpretation step means scope can drift from original intent in ways that aren't structurally visible.

**Multi-protocol traversal.** A single human intent might flow through an A2A delegation to an orchestrator, then through MCP to a tool server, then through HTTP to an external API. Authorization needs to travel across all of these protocol boundaries coherently.

**Adversarial manipulation.** Unlike traditional distributed systems where you trust the software to behave as programmed, AI agents can be manipulated through their inputs. Prompt injection can cause an agent to take actions that are technically within its permissions but contrary to the human's intent. The authorization system can't rely on the agents themselves to enforce scope.

**Opacity at depth.** In a chain of five delegations, it becomes genuinely difficult to answer: "what is this terminal agent actually allowed to do, and can I verify that this is consistent with what the human originally authorized?"

---

## 3. What the ATI Model Captures

The ATI Model defines the complete set of trust concerns that must be addressed for autonomous agent systems to operate with verifiable authority on behalf of principals.

The ATI Model answers a single overarching question:

> **When an autonomous agent acts on behalf of a principal, what must be true — and verifiably true — at every stage from the establishment of identity through the execution and recording of the action?**

The model decomposes this question into eight layers, each addressing a distinct trust concern. The layers are ordered by dependency: each layer depends on the trust guarantees provided by the layers below it. No layer is sufficient on its own. A system that addresses some layers but not others has identifiable, describable trust gaps.

### 3.1 Properties of the Model

**Domain-specific to autonomous agent systems.** The ATI Model addresses the unique trust problem created when authority is delegated through a chain of autonomous, interpretive actors rather than forwarded through deterministic services.

**Implementation-agnostic.** The model defines what trust concerns exist and how they relate. It does not specify protocols, algorithms, or architectures. Individual standards address specific layers; the ATI Model provides the map that shows where each standard fits and where gaps remain.

**Composable.** Deployments may implement layers incrementally, beginning with the layers that address their most critical trust gaps. The model makes the residual risk of missing layers explicit and describable.

---

## 4. How the ATI Model Differs from Existing Models

| Model | What It Describes | ATI Model's Relationship |
|---|---|---|
| OSI (Open Systems Interconnection) | How data moves between systems across a network | ATI describes how trust and permission move through autonomous actors. OSI is a communication model; ATI is a trust model. |
| AOSI (AI Open Systems Interconnection) | Technical functions required for AI systems to operate (7 layers from infrastructure to application) | AOSI describes the functional architecture of an AI system. ATI describes the trust architecture that governs how those functions operate with delegated authority. AOSI answers "what are the building blocks?"; ATI answers "how do we trust each block?" |
| Zero Trust Architecture (NIST SP 800-207) | Network and session-level continuous verification | ATI extends the zero-trust principle to the delegation level — every hop in an agent chain is a verification point. Zero trust is a Layer 1–3 concern in ATI terms. |
| RBAC / ABAC / PBAC | Access control models for single-point authorization decisions | ATI encompasses these as mechanisms within Layer 3 (Session Authorization) and Layer 7 (Constraint Enforcement). They are components within the ATI stack, not alternatives to it. |

---

## 5. Layer Architecture

The ATI Model defines eight layers, numbered from the most foundational (Layer 1) to the most encompassing (Layer 8). Each layer depends on the trust guarantees of the layers below it and provides trust guarantees to the layers above it.

### Layer 1 — Secure Transport

**Trust question:** Is the communication channel between principals confidential, authentic, and tamper-proof?

**What this layer establishes:** That messages exchanged between any two participants — human to agent, agent to agent, agent to tool — cannot be intercepted, modified, or spoofed in transit.

**Why it's foundational:** Every layer above transmits trust-critical data: identity proofs, authorization tokens, delegation chains, action parameters, audit records. If the transport is compromised, none of those artifacts can be trusted.

**Inputs:** Physical or logical network connectivity between participants.

**Outputs consumed by Layer 2:** An authenticated, encrypted channel over which identity credentials can be safely exchanged. Transport-layer identity bindings (e.g., mTLS client certificates) that Layer 2 can use as a foundation for principal identification.

**In scope:** TLS/mTLS configuration, certificate management, channel encryption, transport-layer authentication, proof-of-possession bindings.

**Out of scope:** The content of what's being communicated. The meaning of identity credentials. Whether the participants are authorized to do anything.

**Failure consequence:** If Layer 1 is compromised, all layers above it are unreliable. Identity can be spoofed, delegation chains can be forged or replayed, and audit records can be tampered with.

---

### Layer 2 — Principal Identity

**Trust question:** Is this participant who it claims to be?

**What this layer establishes:** That every participant — human, system, or agent — has a verifiable identity, and that the entity presenting that identity is the legitimate holder of the associated credentials.

**Why it depends on Layer 1:** Identity proofs travel over the transport layer. If the channel isn't secure, identity credentials can be intercepted, replayed, or substituted.

**Inputs from Layer 1:** A secure, authenticated channel.

**Outputs consumed by Layer 3:** A verified principal identifier (e.g., a resolved DID with confirmed key material) including the principal's type, current public key material, and identity-level attributes.

**In scope:** Identity provisioning, identity presentation, identity verification, identity lifecycle management (rotation, decommissioning, revocation), binding between identity and cryptographic key material.

**Out of scope:** Whether the identified principal is authorized to do anything. Identity answers "who are you," not "what may you do."

**Failure consequence:** If Layer 2 is compromised, every layer above makes decisions about the wrong principal. Authorization is granted to an impersonator. Delegation chains are signed by unverified parties. Audit records attribute actions to the wrong actor.

---

### Layer 3 — Session Authorization

**Trust question:** Is this principal permitted to establish a session and begin operating?

**What this layer establishes:** That a verified principal has been evaluated against an authorization policy and granted permission to initiate an agent session. This is the initial gate — the moment where the system decides whether this principal may operate at all.

**Why it depends on Layer 2:** You cannot authorize an unidentified principal.

**Inputs from Layer 2:** A verified principal identifier with associated attributes.

**Outputs consumed by Layer 4:** An authorized session with an initial scope — the maximum set of permissions and constraints that any action within this session may carry.

**In scope:** Authentication-to-authorization bridge, initial scope determination, session token issuance, binding between the authorized session and the originating principal.

**Out of scope:** Ongoing monitoring of the session. Whether any specific action within the session is permitted. Layer 3 opens the gate; Layers 4 and above govern what happens after.

**Failure consequence:** If Layer 3 is absent, principals operate without authorization evaluation. Over-permissioned initial scopes cascade through every subsequent delegation.

---

### Layer 4 — Session Governance

**Trust question:** Should this session still be active?

**What this layer establishes:** That an authorized session is continuously monitored and that anomalous, dangerous, or policy-violating behavior is detected and acted upon — up to and including halting the session entirely.

**Why it depends on Layer 3:** Session governance monitors an authorized session. There is nothing to govern if no session has been established.

**Inputs from Layer 3:** An active, authorized session with a known principal and initial scope.

**Outputs consumed by Layer 5:** A governance state for the session (e.g., active, degraded, halted) that Layer 5 must check before accepting any action-level delegation.

**In scope:** Continuous session monitoring, anomaly detection, rate limiting, circuit breaking, policy-driven session state changes, integration with external security signals.

**Out of scope:** Whether any specific action is within the delegated scope. Session governance answers "should this session continue" — a binary, session-level decision.

**Failure consequence:** If Layer 4 is absent, compromised sessions run indefinitely. An agent whose behavior has been hijacked continues operating with no mechanism to stop it.

---

### Layer 5 — Scope Delegation

**Trust question:** Is this specific agent authorized to perform this specific action, given the chain of delegations from the originating principal?

**What this layer establishes:** That when an agent performs an action, a verifiable chain of delegation links that action back to an originating principal, and that at every hop in the chain, scope only narrowed — never expanded.

**Why it depends on Layer 4:** Scope delegation occurs within an active, governed session. If governance has halted the session, delegation is moot.

**Inputs from Layer 4:** A session in an active governance state.

**Outputs consumed by Layer 6:** A verified scope assertion — the set of permissions and typed constraints that the acting agent legitimately holds at the point of action invocation.

**In scope:** Delegation chain construction, chain verification, the monotonic restriction invariant, effective scope computation, capability token issuance and binding, chain compaction, cross-protocol delegation, delegation revocation, planning-layer capability discovery and authorization.

**Out of scope:** Whether the action's parameters actually satisfy the declared constraints. Layer 5 verifies that a budget constraint of $1500 was structurally propagated and never loosened. It does not verify that the tool invocation's price parameter is under $1500.

**Failure consequence:** If Layer 5 is absent, there is no mechanism to verify that an agent is acting within the authority it was granted. Agents can self-authorize, sub-agents can claim broader permissions than their delegators possessed, and the link between human intent and terminal action is severed.

---

### Layer 6 — Target Integrity

**Trust question:** Is the tool, service, or agent being invoked genuine, unmodified, and operating as declared?

**What this layer establishes:** That when an agent invokes a tool or delegates to another agent, the target is authentic — its code, manifest, or agent identity matches a known-good reference, and it has not been tampered with or substituted.

**Why it depends on Layer 5:** Target integrity verification is meaningful only in the context of a scoped action. Layer 5 establishes that the agent is authorized to invoke a specific target. Layer 6 verifies that the target is what it claims to be.

**Inputs from Layer 5:** A verified scope assertion identifying the target tool, agent, or resource.

**Outputs consumed by Layer 7:** A verified target — confirmation that the tool or agent is authentic, along with the target's declared metadata (parameter schemas, constraint mappings, capability declarations) that Layer 7 needs for semantic constraint evaluation.

**In scope:** Tool manifest verification, artifact integrity verification, agent identity verification for delegation targets, publisher-declared constraint mappings, runtime integrity monitoring, version lineage verification.

**Out of scope:** Whether the agent has authority to invoke this target (that's Layer 5). Whether the action parameters comply with constraints (that's Layer 7).

**Failure consequence:** If Layer 6 is absent, agents invoke tools on blind trust. A compromised tool server could present a malicious tool under a legitimate name.

---

### Layer 7 — Constraint Enforcement

**Trust question:** Do the actual parameters of this action satisfy the constraints declared in the delegation chain?

**What this layer establishes:** That the concrete operation being performed — with its specific parameters, values, and targets — complies with the typed constraints that were propagated through the delegation chain. This is where abstract constraints (budget ≤ $1500, data scope = internal only) are evaluated against concrete action parameters (price = $1200, dataset = internal_sales).

**Why it depends on Layer 6:** Semantic constraint evaluation requires knowledge of what the target's parameters mean — which parameter corresponds to which constraint type. This mapping comes from the target's verified metadata, which Layer 6 validated.

**Inputs from Layer 6:** A verified target with declared parameter schemas and constraint mappings, combined with the effective constraints from the Layer 5 scope assertion.

**Outputs consumed by Layer 8:** An enforcement decision — whether the action's parameters comply with all applicable constraints — along with structured enforcement records.

**In scope:** Semantic mapping between abstract constraint types and concrete tool parameters, evaluation of parameter values against constraint thresholds, deterministic runtime enforcement via authority tokens and enforcement gates, intent-based access control, runtime policy evaluation.

**Out of scope:** Whether the constraints are structurally valid or were properly propagated (that's Layer 5). Whether the tool is genuine (that's Layer 6).

**Failure consequence:** If Layer 7 is absent, constraints are decorative. A delegation chain can declare a budget of $1500, every hop can properly narrow it, the terminal scope can correctly show $1200 — and the tool invocation can still pass price: $5000 with no mechanism to detect or prevent it.

---

### Layer 8 — Accountability and Audit

**Trust question:** Can we reconstruct what happened, verify that trust was warranted at every layer, and demonstrate this to external parties?

**What this layer establishes:** That every trust decision made at every layer is recorded in a tamper-evident, retrievable form, and that these records can be composed into a complete narrative of how authority flowed from originator to action.

**Why it depends on all layers below:** Accountability consumes outputs from every other layer. It records identity verifications (Layer 2), authorization decisions (Layer 3), governance state changes (Layer 4), delegation chain verification results (Layer 5), target integrity checks (Layer 6), and constraint enforcement decisions (Layer 7) — all transmitted over secure channels (Layer 1). It is the only layer with a dependency on every other layer simultaneously.

**Inputs from Layers 1–7:** Structured records from every trust decision at every layer.

**Outputs:** Audit trails that serve regulatory compliance, forensic investigation, anomaly detection, trust model validation, and external accountability.

**In scope:** Structured logging of all trust decisions, tamper-evident storage, retention policy enforcement, forensic chain reconstruction, cross-layer correlation, regulatory compliance support.

**Out of scope:** Making trust decisions. Layer 8 observes and records. It does not authorize, govern, enforce, or verify.

**Failure consequence:** If Layer 8 is absent, trust is unverifiable after the fact. A breach cannot be forensically investigated. Regulatory requirements cannot be met. The system may be secure in real time but cannot demonstrate that security to any external party.

---

### Layer Dependency Map

```
Layer 8 — Accountability and audit
  ↑ receives records from all layers below
  │
Layer 7 — Constraint enforcement
  ↑ receives verified target metadata from Layer 6
  ↑ receives effective constraints from Layer 5
  │
Layer 6 — Target integrity
  ↑ receives scope assertion (target identity) from Layer 5
  │
Layer 5 — Scope delegation
  ↑ requires active governance state from Layer 4
  │
Layer 4 — Session governance
  ↑ monitors authorized session from Layer 3
  │
Layer 3 — Session authorization
  ↑ evaluates verified identity from Layer 2
  │
Layer 2 — Principal identity
  ↑ credentials exchanged over secure channel from Layer 1
  │
Layer 1 — Secure transport
```

---

## 6. Agent Request Flow Through the Layers

Every agent request logically traverses all seven trust checkpoints (Layers 1–7), with Layer 8 observing the entire flow. The question is not whether the layers exist — it is whether anything is verifying trust at each one.

### Phase 1 — Trust Establishment

The request originates as human intent and enters the agent system.

**Step 1 → Layer 1 (Secure Transport):** TLS is established, the channel is encrypted, and endpoints are mutually authenticated. The transport is now trustworthy for carrying identity credentials.

**Step 2 → Layer 2 (Principal Identity):** The agent presents its credentials. Identity is verified via DID resolution, identity provider lookup, or workload identity. The system now knows who this agent is.

**Step 3 → Layer 3 (Session Authorization):** The verified identity is evaluated against authorization policy. Both the agent and the delegating human are independently authorized (dual-subject authorization). A session token is issued with an initial scope. The agent is now permitted to operate.

**Step 4 → Layer 4 (Session Governance):** The session enters continuous monitoring. Behavioral patterns are tracked, anomaly detection is active, and the circuit breaker is armed. The session is confirmed as still-appropriate-to-continue.

### Phase 2 — Action Execution

The agent selects a capability, verifies the target, and executes with enforcement.

**Step 5 → Layer 5 (Scope Delegation):** The delegation chain is verified. Every hop from originator to this agent is checked for monotonic scope restriction. The agent's effective permissions and constraints are computed. The capability is selected from authorized options only.

**Step 6 → Layer 6 (Target Integrity):** The tool being invoked is verified against its sealed manifest. Interface fingerprints are checked, artifact digests are compared, and version lineage is validated. The tool is confirmed as genuine and untampered.

**Step 7 → Layer 7 (Constraint Enforcement):** The actual action parameters are evaluated against the constraints from the delegation chain. An authority token is validated. The enforcement gate confirms the action is within scope. The action is approved or denied deterministically.

**Step 8 → Action Fulfilled:** The tool executes. The result is returned through the agent chain to the originator.

### Throughout — Layer 8 (Accountability and Audit)

Layer 8 does not sit in the sequential flow. It observes and records every trust decision made at every other layer simultaneously. It produces the tamper-evident audit trail that proves — after the fact — that trust was warranted at every stage.

---

## 7. Standards Mapping

The following table maps existing and emerging standards, specifications, and platform implementations to each ATI layer. Standards may appear in multiple layers when they address cross-cutting concerns.

### Layer 1 — Secure Transport

| Standard | Type | Maturity | Description |
|---|---|---|---|
| TLS 1.3 (RFC 8446) | Open standard | Mature | Transport layer encryption |
| mTLS (RFC 8705) | Open standard | Mature | Mutual TLS client authentication |
| DPoP (RFC 9449) | Open standard | Mature | Demonstrating proof of possession |

### Layer 2 — Principal Identity

| Standard | Type | Maturity | Description |
|---|---|---|---|
| W3C DIDs | Open standard | Mature | Decentralized Identifiers — resolvable, self-sovereign identity |
| X.509 / PKI | Open standard | Mature | Certificate-based identity and public key infrastructure |
| SPIFFE / SPIRE | Open standard | Mature | Workload identity for cloud-native and agent environments |
| OpenID Connect | Open standard | Mature | Identity layer on top of OAuth 2.0 |
| SCIM | Open standard | Mature | System for Cross-domain Identity Management |
| NIST SP 800-63-4 | Government standard | Mature | Digital Identity Guidelines |
| Microsoft Entra Agent ID | Platform implementation | Preview (2026) | Enterprise agent identity with lifecycle management |
| AAuth | Exploratory specification | Early (2026) | Agent Auth — first-class agent identities with signed HTTP messages |
| IETF draft-klrc-aiagent-auth-00 | IETF internet-draft | Early (2026) | Agent identity model using WIMSE and SPIFFE |
| NIST NCCoE Concept Paper | Government initiative | Early (2026) | Accelerating adoption of AI agent identity and authorization |

### Layer 3 — Session Authorization

| Standard | Type | Maturity | Description |
|---|---|---|---|
| OAuth 2.1 | Open standard | Mature | Authorization framework for delegated access |
| GNAP | Open standard | Emerging | Grant Negotiation and Authorization Protocol |
| AGBAC | Open specification | Emerging (2026) | Agent-Based Access Control — dual-subject authorization (human + agent) |
| AAuth | Exploratory specification | Early (2026) | Agent Auth — unified identity and authorization for agents |
| IETF draft-klrc-aiagent-auth-00 | IETF internet-draft | Early (2026) | AI agent auth model composing WIMSE, SPIFFE, and OAuth 2.0 |
| IETF Transaction Tokens | IETF internet-draft | Emerging | Immutable, propagated call-chain context for distributed systems |
| IETF Identity Chaining | IETF internet-draft | Emerging | Cross-domain OAuth identity preservation |
| NIST SP 800-207 | Government standard | Mature | Zero Trust Architecture |

### Layer 4 — Session Governance

| Standard | Type | Maturity | Description |
|---|---|---|---|
| ALS | Open standard | Emerging | Agent-Layer Security Standard — session circuit breaking |
| DAE | Open specification | Emerging (2025) | Deterministic Agent Execution — runtime state machine, escalation |
| AARM | Open specification | Emerging (2026) | Autonomous Action Runtime Management — session context accumulation |
| Microsoft Agent 365 | Platform implementation | Preview (2026) | Enterprise control plane for agent lifecycle and governance |
| NIST AI Agent Standards Initiative | Government initiative | Early (2026) | Federal initiative on agent governance and monitoring |

### Layer 5 — Scope Delegation

| Standard | Type | Maturity | Description |
|---|---|---|---|
| ADCS | Open standard | Proposed (2026) | Agent Delegation Chain Standard — monotonic restriction across multi-hop chains |
| ACS | Open specification | Emerging (2026) | Agent Capability Standard — planning-layer capability routing and authorization |
| UCAN | Open standard | Emerging | User Controlled Authorization Networks — decentralized capability chains |
| AGBAC | Open specification | Emerging (2026) | Agent-Based Access Control — dual-subject initial delegation gate |
| Macaroons | Research / implementation | Mature (concept) | Contextual caveats for delegation-by-attenuation |
| AAuth | Exploratory specification | Early (2026) | Agent Auth — delegation chain visibility with signed HTTP messages |

### Layer 6 — Target Integrity

| Standard | Type | Maturity | Description |
|---|---|---|---|
| MIS | Open specification | Emerging (2026) | MCP Integrity Standard — sealed manifests, interface fingerprinting, version lineage |
| Sigstore | Open standard | Emerging | Keyless signing and transparency log for software artifacts |
| SLSA | Open standard | Emerging | Supply-chain Levels for Software Artifacts |
| in-toto | Open standard | Emerging | Software supply chain attestation framework |

### Layer 7 — Constraint Enforcement

| Standard | Type | Maturity | Description |
|---|---|---|---|
| IBAC | Open specification | Emerging (2026) | Intent-Based Access Control — per-request permissions from user intent |
| DAE | Open specification | Emerging (2025) | Deterministic Agent Execution — single-use authority tokens, enforcement gate |
| AARM | Open specification | Emerging (2026) | Autonomous Action Runtime Management — runtime policy evaluation with context-aware action interception |
| ACS | Open specification | Emerging (2026) | Agent Capability Standard — Planning Authorization Enforcement Point, execution contracts |
| OPA / Rego | Open standard | Mature | Open Policy Agent — general-purpose policy engine |
| Cedar | Open standard | Emerging | AWS policy language with formal verification |
| XACML / ALFA | Open standard | Mature | Attribute-based access control markup and policy language |
| OpenFGA | Open standard | Emerging | Fine-grained authorization engine (relationship/tuple-based) |

### Layer 8 — Accountability and Audit

| Standard | Type | Maturity | Description |
|---|---|---|---|
| OpenTelemetry | Open standard | Mature | Distributed observability framework |
| CAEP | Open standard | Emerging | Continuous Access Evaluation Protocol (OpenID Foundation) |
| NISTIR 8587 | Government standard | Mature | Protecting tokens and assertions from forgery, theft, and misuse |
| NIST AI Agent Standards Initiative | Government initiative | Early (2026) | Federal initiative on agent security, identity, and audit |
| AARM | Open specification | Emerging (2026) | Tamper-evident execution receipts for forensic reconstruction |
| ACS | Open specification | Emerging (2026) | Chain-hashed audit events with execution intent hashes |
| MIS | Open specification | Emerging (2026) | Structured verification event logging for tool integrity |

---

## 8. Threat Model by Layer

Each layer has a distinct threat profile. When a layer is absent or compromised, the threats listed below are unmitigated. OWASP correlations reference three frameworks: OWASP Top 10 Web (2021), OWASP Top 10 for LLM Applications (2025), and OWASP Top 10 for MCP (2025).

### Layer 1 — Secure Transport

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Man-in-the-middle interception | Critical | A02:2021 | Attacker intercepts agent-to-agent or agent-to-tool communication and reads or modifies content |
| Channel downgrade attack | High | A02:2021 | Attacker forces connection to use weaker encryption or plaintext transport |
| Credential interception in transit | Critical | A02:2021 | Identity tokens, delegation chains, or authority artifacts captured from unencrypted channel |
| Certificate validation bypass | High | A07:2021 | Client accepts invalid or self-signed certificates, enabling impersonation of endpoints |

### Layer 2 — Principal Identity

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Agent identity spoofing | Critical | A07:2021 | Malicious agent impersonates a trusted agent's identity to gain its permissions |
| Credential compromise or key theft | Critical | A02:2021 | Agent signing key or authentication credential is stolen, enabling impersonation |
| Orphaned agent identities | High | A07:2021 | Decommissioned agents retain active credentials; no lifecycle management revokes them |
| Agent sprawl | Medium | — | Unmanaged proliferation of agent identities with no inventory, ownership, or governance |
| Identity resolution failure | High | — | DID or identity provider unavailable; agent cannot prove identity; all layers above fail |

### Layer 3 — Session Authorization

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Over-permissioned initial scope | High | A01:2021 | Agent session receives broader authorization than the task requires; blast radius maximized |
| Missing dual-subject authorization | High | A01:2021 | Agent authorized without verifying both agent identity and delegating human are independently permitted |
| Token theft or replay | High | A07:2021 | Stolen session token reused to establish unauthorized agent sessions |
| Stale authorization from revocation lag | Medium | — | Revoked permissions remain active because revocation doesn't propagate within token TTL |
| Ambient authority via shared credentials | High | LLM08 | Agent operates under shared service account with permanent, unscoped access |

### Layer 4 — Session Governance

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Runaway agent execution | Critical | LLM08 | Agent continues operating after behavioral compromise with no circuit breaker to halt it |
| Behavioral drift undetected | High | LLM09 | Agent's actions gradually diverge from expected patterns without triggering any alert |
| Session hijacking | Critical | A07:2021 | Attacker takes control of an active, authorized agent session |
| Governance signal suppression | High | — | Halt or revocation signals from external governance systems fail to reach the agent runtime |
| Anomaly evasion through rate manipulation | Medium | — | Agent spaces malicious actions below rate-limit thresholds to avoid detection |

### Layer 5 — Scope Delegation

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Privilege escalation through delegation | Critical | LLM08 | Sub-agent receives broader permissions than its delegator possesses |
| Confused deputy attack | High | LLM08 | Agent is tricked into using its legitimate authority to perform unauthorized actions |
| Delegation chain forgery | High | A01:2021 | Attacker constructs a fake delegation chain to authorize an action without legitimate origin |
| Scope opacity in deep chains | Medium | LLM08 | Effective permissions at terminal action are unknowable due to unverified intermediate hops |
| Cross-protocol scope leakage | High | — | Authorization context lost or expanded when delegation crosses protocol boundaries |
| Planning-layer capability enumeration | Medium | MCP02 | Planner infers unauthorized capabilities from exposed tool schemas or error responses |

### Layer 6 — Target Integrity

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Tool poisoning | Critical | MCP03 | Tool description or schema modified to manipulate LLM behavior or exfiltrate data |
| Supply chain artifact tampering | Critical | MCP03 | Tool code or binary replaced with malicious version between build and deployment |
| Rug pull via malicious update | Critical | MCP05 | Publisher pushes compromised update after establishing trust through legitimate versions |
| Tool name spoofing or shadowing | High | MCP04 | Malicious tool registered with name similar to a trusted tool to intercept invocations |
| Silent permission escalation via update | High | MCP02 | Tool update expands side effects or capabilities without user notification |

### Layer 7 — Constraint Enforcement

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Constraint bypass via parameter manipulation | Critical | MCP02 | Action parameters exceed declared constraints (e.g., budget $1500, price $5000) |
| Semantic mapping failure | High | — | Abstract constraint cannot be mapped to concrete action parameter; enforcement absent |
| Intent drift | High | LLM01 | Agent action diverges from original human intent while remaining structurally within scope |
| Prompt injection at execution boundary | Critical | LLM01 | Injected instructions cause agent to execute unintended actions with valid authority |
| Policy engine bypass or misconfiguration | Critical | A01:2021 | Runtime policy evaluation is skipped, disabled, or misconfigured to permit all actions |
| Authority token forgery or replay | High | A02:2021 | Attacker fabricates or reuses a single-use execution authority artifact |

### Layer 8 — Accountability and Audit

| Threat | Severity | OWASP | Description |
|---|---|---|---|
| Audit log tampering or deletion | High | A09:2021 | Attacker modifies or purges logs to conceal unauthorized actions |
| Insufficient logging coverage | High | A09:2021 | Trust decisions at one or more layers produce no audit record |
| Attribution failure | Medium | MCP08 | Actions cannot be traced back to the originating principal or agent |
| Non-repudiation gap | Medium | A09:2021 | Agent or operator denies having authorized or executed an action; no proof exists |
| Cross-layer correlation failure | Medium | — | Logs from different layers cannot be linked, preventing forensic reconstruction |

---

## 9. Gap Analysis

The ATI Model makes the maturity landscape of agent trust immediately visible:

**Well-served by mature standards (Layers 1–3):** Transport security, principal identity, and session authorization have decades of production-proven standards. TLS, X.509, OAuth, OIDC, and SPIFFE provide a solid foundation. The primary gap is adapting these for agent-specific concerns (agent identity lifecycle, dual-subject authorization), which is being addressed by emerging work from NIST, Microsoft, and the IETF.

**Active frontier with emerging standards (Layers 4–5):** Session governance and scope delegation have emerging standards (ALS, ADCS, DAE, ACS, AGBAC) but no production-mature, broadly adopted solution. This is where the most novel standards work is concentrated.

**Critical gap with early work (Layer 6):** Target integrity — verifying that tools are genuine and untampered — had no comprehensive standard until the MCP Integrity Standard (MIS). Tool poisoning, supply chain tampering, and rug-pull attacks are critical threats at this layer. MIS addresses them but is still in draft stage.

**Significant gap with partial solutions (Layer 7):** Constraint enforcement — ensuring that actual action parameters satisfy declared constraints — is the least mature layer. IBAC, DAE, and AARM provide partial solutions, but the semantic mapping problem (connecting abstract constraints to concrete parameters) remains an open research challenge.

**Fragmented coverage (Layer 8):** Accountability and audit has mature general-purpose tools (OpenTelemetry) but no agent-specific audit standard that defines what trust decisions must be recorded, in what format, with what tamper-evidence guarantees, across all layers. Multiple emerging specifications (AARM, ACS, MIS) include audit provisions, but there is no unified agent audit framework.

### Interface Gaps

Beyond individual layers, two layer interfaces are notably weak:

**Layer 6 → Layer 7 interface:** The connection between target integrity (what the tool claims to do) and constraint enforcement (evaluating parameters against constraints) depends on publisher-declared constraint mappings. The mapping vocabulary and format are not yet standardized.

**Layers 1–7 → Layer 8 interface:** All layers produce audit records, but there is no common event format for trust decisions across all eight layers. OpenTelemetry addresses observability generally but not trust-decision auditing specifically.

---

## 10. Adoption and Composability

The ATI Model is designed for incremental adoption. Organizations do not need to implement all eight layers simultaneously.

### Recommended Adoption Path

**Phase 1 — Foundation (Layers 1–3):** Most organizations already have these layers partially addressed through existing infrastructure (TLS, IdP, OAuth). The primary work is extending identity and authorization for agent-specific concerns.

**Phase 2 — Governance and Delegation (Layers 4–5):** Add session governance (circuit breaking, anomaly detection) and scope delegation (delegation chain verification). This addresses the most common agent-specific security failures.

**Phase 3 — Integrity and Enforcement (Layers 6–7):** Add target integrity verification and semantic constraint enforcement. These are the most technically challenging layers and benefit from the emerging standards reaching maturity.

**Throughout — Audit (Layer 8):** Begin structured audit logging from Phase 1 and expand coverage as each layer is added. Audit is the layer that makes all other layers demonstrable.

### Residual Risk Visibility

The ATI Model's primary value for incremental adoption is making residual risk explicit. An organization that has implemented Layers 1–4 but not Layers 5–7 can precisely describe their trust gaps: "We have no mechanism to verify that delegation chains are monotonically restrictive, that tools are genuine, or that action parameters satisfy declared constraints." This is more useful than a vague sense that "security could be better."

---

## 11. Relationship to Other Reference Models

### AOSI (AI Open Systems Interconnection)

AOSI provides a 7-layer technical reference model for AI systems describing the functional architecture from infrastructure to application. The ATI Model describes the trust architecture that governs how those functions operate with delegated authority. AOSI and ATI are complementary: AOSI maps what an AI system does; ATI maps how we trust each part. An AGBAC-to-AOSI mapping already exists; a corresponding ATI-to-AOSI mapping would show which AOSI layers correspond to which ATI trust concerns.

### NIST AI Risk Management Framework (AI RMF)

The NIST AI RMF provides a voluntary framework for managing AI risk through four functions: Govern, Map, Measure, and Manage. The ATI Model provides the technical decomposition that the AI RMF's risk management functions operate against. ATI Layer 5 (Scope Delegation) and Layer 7 (Constraint Enforcement) directly support the Govern function. ATI Layer 8 (Accountability) supports the Measure function. The ATI threat model (Section 8) maps to the AI RMF's risk identification process.

### EU AI Act (Regulation 2024/1689)

The EU AI Act requires risk classification, human oversight, transparency, and record-keeping for high-risk AI systems. ATI layers map to these requirements: Layer 7 (Constraint Enforcement) supports risk classification and human oversight. Layer 8 (Accountability) supports record-keeping. Layer 5 (Scope Delegation) supports transparency of decision chains. A compliance mapping document is a planned companion to this model.

---

## 12. Versioning

The ATI Model uses semantic versioning: MAJOR.MINOR.PATCH.

- **MAJOR** increments indicate changes to the layer definitions, layer ordering, or layer dependency model
- **MINOR** increments indicate additions to layer descriptions, new standards mapped, new threats identified, or new interface specifications
- **PATCH** increments indicate clarifications, corrections, and editorial changes

This document is version **0.1.0** — a working draft for community review and feedback.

---

## 13. References

### Standards and Specifications Referenced

- **TLS 1.3** — RFC 8446. Rescorla, 2018
- **mTLS** — RFC 8705. OAuth 2.0 Mutual-TLS Client Authentication. Campbell et al., 2020
- **DPoP** — RFC 9449. OAuth 2.0 Demonstrating Proof of Possession. Fett et al., 2023
- **W3C DID Core** — Decentralized Identifiers (DIDs) v1.0. W3C Recommendation, 2022
- **SPIFFE** — Secure Production Identity Framework for Everyone. CNCF, 2018
- **OAuth 2.1** — The OAuth 2.1 Authorization Framework. Hardt, Parecki, Lodderstedt (draft)
- **OpenID Connect** — OpenID Connect Core 1.0. OpenID Foundation, 2014
- **GNAP** — Grant Negotiation and Authorization Protocol. IETF (draft)
- **CAEP** — Continuous Access Evaluation Protocol. OpenID Foundation, 2023
- **ADCS** — Agent Delegation Chain Standard v1.0.0. Proposed Standard, 2026
- **ACS** — Agent Capability Standard v1.0.0. Reilly, 2026
- **MIS** — MCP Integrity Standard v1.0.0. Reilly, 2026
- **DAE** — Deterministic Agent Execution Security Standard v1.0.0. Reilly, 2025
- **AGBAC** — Agent-Based Access Control. Reilly, 2026
- **ALS** — Agent-Layer Security Standard v2.1.0, 2026
- **AARM** — Autonomous Action Runtime Management. Errico, 2026
- **IBAC** — Intent-Based Access Control. ibac.dev, 2026
- **AAuth** — Agent Auth. Hardt (exploratory specification), 2026
- **IETF draft-klrc-aiagent-auth-00** — AI Agent Authentication and Authorization. Kasselman, Lombardo, Rosomakho, Campbell, 2026
- **IETF Transaction Tokens** — draft-ietf-oauth-transaction-tokens. IETF OAuth WG (active draft)
- **IETF Identity Chaining** — draft-ietf-oauth-identity-chaining. IETF OAuth WG (active draft)
- **OPA** — Open Policy Agent. CNCF
- **Cedar** — Cedar Policy Language. AWS, 2023
- **XACML** — eXtensible Access Control Markup Language. OASIS
- **OpenFGA** — Fine-Grained Authorization. OpenFGA Project
- **Sigstore** — Sigstore Project (cosign, rekor, fulcio)
- **SLSA** — Supply-chain Levels for Software Artifacts v1.0
- **in-toto** — in-toto Attestation Framework
- **OpenTelemetry** — OpenTelemetry Project. CNCF
- **Macaroons** — Macaroons: Cookies with Contextual Caveats. Birgisson et al., Google Research, 2014
- **UCAN** — User Controlled Authorization Networks. Fission/DAG House, 2022
- **NIST SP 800-207** — Zero Trust Architecture. Rose et al., 2020
- **NIST SP 800-63-4** — Digital Identity Guidelines. NIST, 2024
- **NISTIR 8587** — Protecting Tokens and Assertions from Forgery, Theft, and Misuse. NIST
- **NIST AI RMF** — AI Risk Management Framework 1.0. NIST, 2023
- **NIST AI Agent Standards Initiative** — NIST CAISI, 2026
- **NIST NCCoE Concept Paper** — Accelerating the Adoption of Software and AI Agent Identity and Authorization. NCCoE, 2026
- **Microsoft Entra Agent ID** — Microsoft Entra Agent Identity Platform. Microsoft, 2026
- **Microsoft Agent 365** — Microsoft Agent 365 Control Plane. Microsoft, 2026
- **AOSI** — AI Open Systems Interconnection Reference Model. Reilly

### OWASP Frameworks Referenced

- **OWASP Top 10 Web Application Security Risks (2021)**
- **OWASP Top 10 for Large Language Model Applications (2025)**
- **OWASP Top 10 for Model Context Protocol (2025)**

---

*End of Agent Trust Interconnect (ATI) Model v0.1.0*

---

**License:** Apache License 2.0

**Contributions:** This model is intended as a shared artifact for the AI security and agent infrastructure community. Feedback, additions, and corrections are welcome.
