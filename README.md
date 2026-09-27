Open Distributed Edge Compute & Authorized Bandwidth Relaying Protocol (EWASTE-v1.0 / ODEC-v1.0)

Classification: Open Standard / Bare-Metal Edge Compute & Bandwidth Orchestration
Repository Target: ewaste-protocol
Canonical Reference ID: EWASTE-v1.0 (A.K.A. ODEC-v1.0)
Target Infrastructure: Decommissioned Consumer Silicon, Legacy Hardware, Edge Gateways, P2P Networks
License: Unlicense (Public Domain)
Abstract

Centralized machine learning (ML) infrastructure and commercial data-relaying networks face high capital expenditure (CapEx), middleman rent-seeking, and rigid deployment topologies. This specification defines an open, stateless, zero-rent protocol designed to orchestrate volunteer edge compute and authorized network relaying over heterogeneous legacy hardware.

By eliminating native protocol tokens, middleman platform fees, and centralized control layers, the system enables direct peer-to-peer (P2P) coordination. We outline a modular architecture featuring localized pre-loaded ML inference, probabilistic result verification, authorization-first network routing, and a concrete threat model for host safety and protocol integrity.
1. Introduction & Economic Architecture
1.1 Capital Expenditure vs. Operational Expenditure

While modern data center accelerators offer high compute density per watt, acquiring new hardware requires substantial capital expenditure (CapEx). Conversely, legacy and idle consumer hardware (e.g., decommissioned laptops, single-board computers, retired workstations) carries zero marginal CapEx, operating purely on local operational expenditure (OpEx→Watts/Electricity).

This protocol provides an open orchestration layer to aggregate idle, heterogeneous hardware into a functional edge compute network. Operators ("scavengers") monetize underutilized hardware and household bandwidth up to the point where local electricity costs match market compute yields.
1.2 Zero-Rent Protocol Mechanics

To minimize operational friction and eliminate speculative financial overhead:

    No Native Protocol Token: Value transfers settle directly between task submitters ("buyers") and node operators via existing digital payment rails (e.g., Layer-2 stablecoins, state channels, or direct micro-settlement APIs).
    Zero Protocol Fee: 100% of buyer payments route directly to the executing node operator (100).
    Minimal State Overlay: The network operates purely as an open discovery and verification protocol, maintaining no centralized state or custodial funds.

2. System Topology & Node Categorization

The network decouples compute tasks from WAN bandwidth capabilities to optimize hardware utilization and respect household network constraints.

                           [ Client / Buyer ]
                                   │
                 ┌─────────────────┴─────────────────┐
                 │ (Direct P2P Payment Settlement)   │
                 ▼                                   ▼
      [ Compute Request: ML ]         [ Relaying Request: Auth-Only ]
                 │                                   │
                 └─────────────────┬─────────────────┘
                                   │
  ┌────────────────────────────────┴────────────────────────────────┐
  │ Local Area Network (LAN) Operator Cluster                       │
  │                                                                 │
  │   ┌──────────────────────────────────────────────────────────┐  │
  │   │ WAN Gateway Node (Node 0)                                │  │
  │   │ - Handles Ingress/Egress & Rate Limits                   │  │
  │   │ - Enforces Fail-Closed Behavior on Access Challenges     │  │
  │   └────────────────────────────┬─────────────────────────────┘  │
  │                                │                                │
  │              ┌─────────────────┴─────────────────┐              │
  │              │ Local Area Network (LAN)          │              │
  │              ▼                                   ▼              │
  │   ┌─────────────────────┐             ┌─────────────────────┐   │
  │   │ Worker Node 1       │             │ Worker Node N       │   │
  │   │ (Pre-loaded Model)  │ . . . . . . │ (Pre-loaded Model)  │   │
  │   │ - Local ML Output   │             │ - Local ML Output   │   │
  │   └─────────────────────┘             └─────────────────────┘   │
  └─────────────────────────────────────────────────────────────────┘

2.1 The LAN Node Cluster

Multiple devices operating behind a single public WAN gateway automatically form a localized cluster:

    Primary Gateway Node (Node 0): Manages public network interactions, enforcing rate limits, local bandwidth caps, and network isolation policies. Outbound transport interfaces optionally wrap egress packets in LMTI-v1.0 uniform block padding and latency jitter to shield node metadata.
    Internal Compute Workers (Nodes 1–N): LAN-connected devices processing offloaded compute tasks. Compute nodes execute bare-metal quantized inference using local runtimes (LMCI-v1.0) and do not route raw external proxy traffic, preventing internal network saturation and IP address degradation.

3. Workload Specification & Output Verification

Large-scale foundation models require high-bandwidth interconnects unsuited for legacy consumer hardware. Therefore, this protocol specifically targets Localized, Quantized Edge Inference (e.g., 1B–3B parameter models, text/image classification, feature extraction, AST transformations).
3.1 Pre-Loaded Model Storage (LMCI-v1.0 Integration)

To eliminate the overhead of transferring multi-gigabyte model weights over consumer connections, nodes maintain a local library of standard, open-weights models stored in quantized formats (GGUF, EXL2, AWQ) via LMCI-v1.0 local weight isolation standards. Tasks specify a deterministic model content hash; if a node lacks the required model locally, it rejects the job allocation.
3.2 Probabilistic Verification

Because output hashes only confirm data receipt—not computational correctness—the client software employs a Probabilistic Sampling Verification model:

    Spot Checks: The buyer client assigns $X%$ of tasks redundantly to two independent worker nodes.
    Deterministic Comparison: For deterministic workloads (e.g., low-temperature generation, feature extraction, or AST parsing), outputs and intermediate tensor state hashes must match within an acceptable floating-point tolerance (ϵ).
    Reputation Penalties: Nodes returning inconsistent results are flagged locally by the buyer client, dropping their allocation priority for future high-value tasks.

4. Authorization-First Network Relaying

To ensure strict legal compliance, operational transparency, and physical host protection for node operators, network routing functionality operates under a strict Authorization-First Framework.
4.1 Explicit Destination Permissions

Node operators must explicitly configure allowed routing targets (e.g., specific domain allowlists, internal API endpoints, or partner networks). The protocol strictly prohibits arbitrary, unconstrained open-proxy behavior.
4.2 Fail-Closed Rate Limiting & Challenge Handling

Rather than attempting to bypass anti-bot measures, evade access controls, or alter traffic signatures:

    Fail-Closed Principle: If an outbound request encounters a rate limit (HTTP 429), access challenge, or authorization failure (HTTP 403), the gateway node instantly halts requests to that target destination.
    No Evasion Mechanics: The protocol does not inject artificial human jitter to bypass scrapers, alter headers to misrepresent user agents, or attempt to mask network origins. Rate limits are treated as explicit instructions to back off or seek formal API access.

5. Threat Model & Security Framework
Threat Vector	Attack Mechanics	Protocol Mitigation
Malicious Worker Node	Submits dummy or hallucinated ML results to claim payment without compute.	Probabilistic duplicate sampling ($X%$) + client-side target state hash checks.
Malicious Buyer Node	Attempts to run unsafe arbitrary code, privilege-escalate, or exploit network endpoints.	Containerized sandboxing (WASM/OCI) + host permission controls (isolated_sandbox_required).
Sybil Subnet Claim	Datacenter nodes impersonating residential edge endpoints to corner task routing.	Hardware attestation + baseline RTT and hop-distance profiling checks.
ISP ToS Violation	High-volume raw relaying triggering residential account suspension or IP degradation.	Host-configured bandwidth caps + fail-closed non-residential routing + local LAN isolation.
5.1 Host Sandboxing & Resource Controls

All workload execution occurs within isolated runtime environments (WebAssembly micro-runtimes or unprivileged OCI containers). Executing tasks are restricted from accessing local host storage, adjacent LAN devices, or unmapped network interfaces.
5.2 Sybil & Topology Classification

While Autonomous System Number (ASN) lookups distinguish data center subnets from residential ISPs, they do not prove physical location or hardware identity. The protocol combines ASN mapping with Round-Trip Time (RTT) Latency Profiling across multiple peer nodes to establish relative network distance and verify topology claims without requiring centralized identity providers.
6. Economic Realities & Operator Viability
6.1 Cost-Benefit Equilibrium

An operator's net income is defined by:

Profit=RevenueTasks​−(Power Consumption (kW)×Electricity Rate (kWh$​))

    Compute Selection: Nodes automatically pause task execution if market task yields fall below local electricity costs (Yield<Power Cost).
    Hardware Lifecycle Extension: The protocol provides an operational runway for legacy hardware that would otherwise be landfilled, shifting the economic barrier from CapEx hardware acquisition to pure energy efficiency (OpEx→Watts).

7. Strategic System Alignment

By coupling EWASTE-v1.0 (Hardware Repurposing) with LMCI-v1.0 (Bare-Metal Local Inference), LMTI-v1.0 (Transport Shielding), and ATN-v1.0 (Task Graph Execution), the operational stack forms a closed, self-sustaining physical loop:

Discarded Silicon (CapEx = $0)⟶Bare-Metal Local Compute⟶Zero-Rent P2P Execution⟶Sovereign Utility
8. Conclusion

This protocol provides a practical, legally defensible framework for distributed edge compute and authorized network relaying. By abandoning rent-seeking crypto-tokens, rejecting anti-detection evasion techniques, and focusing on localized, verifiable ML workloads, the system establishes a clean, open utility layer for volunteer and scavenged compute infrastructure.
