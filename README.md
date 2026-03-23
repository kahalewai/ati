# Agent Trust Interconnect (ATI) Model

<br>

The ATI Model is an open reference model that defines the complete set of trust concerns for autonomous AI agent systems. It provides a shared vocabulary for understanding where standards, protocols, and implementations fit — and where gaps remain. AI agents delegate tasks to other agents, which invoke tools, which access data — all on behalf of a human who said something like "book me a trip to Tokyo." At every hop in that chain, trust must be verified. Identity must be established. Authorization must be checked. Scope must narrow. Tools must be genuine. Constraints must be enforced. And every decision must be recorded.

<br>

No single standard solves this. The industry is building standards for individual pieces — but without a shared reference model, people conflate layers. They think identity is authorization. They think structural constraint propagation is semantic enforcement. They build systems that handle three concerns well and ignore the other five. The ATI Model provides the missing map.

<br>

## The Eight Layers

Every agent request logically traverses all eight trust layers. The question is not whether the layers exist — it is whether anything is verifying trust at each one.

| Layer | Name | Trust Question |
|---|---|---|
| **8** | Accountability & audit | Can we prove what happened? |
| **7** | Constraint enforcement | Does the action satisfy its constraints? |
| **6** | Target integrity | Is the tool or agent genuine and unmodified? |
| **5** | Scope delegation | Is this agent authorized for this specific action? |
| **4** | Session governance | Should this session still be active? |
| **3** | Session authorization | Is this principal permitted to operate? |
| **2** | Principal identity | Is this participant who it claims to be? |
| **1** | Secure transport | Is the channel confidential and authentic? |

Each layer depends on the trust guarantees provided by the layers below it. No layer is sufficient on its own.

<br>

## Standards Landscape at a Glance

The ATI Model maps 55+ standards, specifications, and platform implementations across all eight layers. Here is where things stand:

**Layers 1–3 (Secure Transport, Identity, Session Auth)** — Well-served by mature standards. TLS, X.509, OAuth, OIDC, SPIFFE, and DIDs provide a solid foundation. Emerging work adapts these for agent-specific concerns.

**Layers 4–5 (Session Governance, Scope Delegation)** — Active frontier. ALS, ADCS, DAE, ACS, AGBAC, and AARM are building the standards that don't yet exist. This is where the most novel work is concentrated.

**Layer 6 (Target Integrity)** — Critical gap being addressed. The MCP Integrity Standard (MIS) provides sealed manifests, interface fingerprinting, and version lineage for tool verification. Sigstore, SLSA, and in-toto provide supply-chain foundations.

**Layer 7 (Constraint Enforcement)** — Least mature layer. IBAC, DAE, and AARM provide partial solutions. The semantic mapping problem — connecting abstract constraints like "budget ≤ $1500" to concrete tool parameters like "price" — remains an open challenge.

**Layer 8 (Accountability & Audit)** — Fragmented coverage. OpenTelemetry is mature for general observability. No unified agent-specific audit framework exists yet. Multiple emerging specs include audit provisions.

<br>

## Threat Model

Each ATI layer has a distinct threat profile. When a layer is absent or compromised, its threats are unmitigated. The threat model correlates to three OWASP frameworks:

- **OWASP Top 10 Web Application Security Risks (2021)**
- **OWASP Top 10 for LLM Applications (2025)**
- **OWASP Top 10 for MCP (2025)**

Key threats include:

| Layer | Example Threat | Severity |
|---|---|---|
| 7 | Prompt injection at execution boundary | Critical |
| 7 | Constraint bypass via parameter manipulation | Critical |
| 6 | Tool poisoning — modified descriptions manipulate LLM | Critical |
| 6 | Rug pull via malicious tool update | Critical |
| 5 | Privilege escalation through delegation | Critical |
| 4 | Runaway agent execution — no circuit breaker | Critical |
| 2 | Agent identity spoofing | Critical |
| 1 | Man-in-the-middle interception | Critical |

The full threat model with all threats, severity ratings, and OWASP correlations is in the [specification](ati-model-v0.1.0.md#8-threat-model-by-layer) and the [visual reference](ati-model-threat-model.html).

<br>

## How to Use the ATI Model

### For Security Architects

Use the layer model to audit your agent deployment. Walk through each layer and ask: is anything verifying trust here? The gap analysis in the specification tells you what residual risk looks like when a layer is missing.

### For Standards Authors

Use the model to position your standard. Which layer(s) does it address? What are the inputs it consumes from the layer below? What outputs does it provide to the layer above? The standards mapping shows where your work fits alongside others.

### For Engineering Teams

Use the model to plan incremental adoption:

1. **Phase 1 — Foundation (Layers 1–3):** Extend your existing identity and authorization infrastructure for agent-specific concerns.
2. **Phase 2 — Governance and Delegation (Layers 4–5):** Add session circuit breaking and delegation chain verification.
3. **Phase 3 — Integrity and Enforcement (Layers 6–7):** Add tool integrity verification and runtime constraint enforcement.
4. **Throughout — Audit (Layer 8):** Begin structured audit logging from Phase 1 and expand as each layer is added.

### For Compliance and Risk Teams

Use the threat model to map agent-specific risks to your existing risk framework. The OWASP correlations connect ATI threats to established risk taxonomies. The layer model makes residual risk from missing layers explicit and describable for audit purposes.

<br>

## Key Design Decisions

**Why "Agent Trust Interconnect"?** Every layer is answering a trust question. The unifying concept across all eight layers is trust — established, constrained, verified, and audited. "Interconnect" communicates that the layers have interfaces and dependencies, not just that they exist as a list.

**Why eight layers?** Each layer answers a question that no other layer answers. We tested every interface between layers for gaps and found the decomposition to be complete — no trust concern falls between layers, and no two layers answer the same question.

**Why not a networking model?** This isn't about how packets move. It's about how trust and permission move through a system of autonomous actors that interpret, decide, and act rather than simply forwarding.

**Why implementation-agnostic?** The model defines what trust concerns exist and how they relate. Individual standards (ADCS, ALS, DAE, ACS, MIS, AGBAC, IBAC, AARM, etc.) address specific layers. The ATI Model provides the map; the standards provide the implementations.

<br>

## Related Work

| Model | Relationship to ATI |
|---|---|
| **OSI** (Open Systems Interconnection) | OSI describes network communication layers. ATI describes agent trust layers. Same structural principle, different domain. |
| **AOSI** (AI Open Systems Interconnection) | AOSI describes the functional architecture of AI systems. ATI describes the trust architecture. AOSI maps what an AI system does; ATI maps how we trust each part. |
| **NIST SP 800-207** (Zero Trust Architecture) | Zero trust is a Layer 1–3 concern in ATI terms. ATI extends the principle to delegation-level verification across all layers. |
| **NIST AI RMF** | AI RMF provides risk management functions (Govern, Map, Measure, Manage). ATI provides the technical decomposition those functions operate against. |
| **EU AI Act** | EU AI Act requires risk classification, human oversight, and record-keeping. ATI layers map directly to these requirements. |

<br>

## Current Status

**Version:** 0.1.0 — Working Draft

The ATI Model is a working draft released for community review and feedback. The layer definitions, standards mapping, and threat model reflect the state of the agent security landscape as of March 2026. As standards mature and new ones emerge, the mapping will be updated.

---

## Contributing

The ATI Model is intended as a shared artifact for the AI security and agent infrastructure community. Contributions are welcome:

- **Feedback on layer definitions** — Are the boundaries between layers correct? Does any trust concern fall between layers?
- **Standards mapping additions** — Is there a standard, specification, or implementation that should be mapped to a layer?
- **Threat model refinement** — Are there agent-specific threats missing from a layer's threat profile?
- **OWASP correlation updates** — As OWASP frameworks evolve, correlations should be updated.
- **Implementation experience** — Reports from teams implementing trust at specific layers validate or challenge the model.

---

## License

Apache License 2.0

---

## Acknowledgments

The ATI Model was developed through analysis of the emerging agent security standards landscape, including work by authors and organizations contributing to ADCS, ACS, MIS, DAE, AGBAC, ALS, AARM, IBAC, AAuth, and the NIST AI Agent Standards Initiative. The model builds on decades of prior work in distributed systems trust, capability-based security, and layered reference architectures.
