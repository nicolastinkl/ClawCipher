```markdown
# ClawCipher Protocol Specification v0.1.0 (Alpha)

**Protocol Type**: Privacy-First Autonomous Agent Infrastructure Layer  
**Based on**: OpenClaw MIT Core (Fork)  
**Architectural Philosophy**: Durovian Sovereignty (Global-first, Anti-censorship, Protocol-layer)  
**Date**: 2026-03-20  
**Classification**: Confidential / Pre-release

---

## 1. Executive Summary

### 1.1 Problem Statement
OpenClaw enables autonomous AI agents but operates under **compromised trust assumptions**:
- **Metadata Exposure**: Agent-to-LLM communications reveal behavioral patterns, identity, and intent to cloud providers (AWS, OpenAI, Anthropic)
- **Centralized Skills Governance**: The 20% malicious risk in Skills marketplace relies on centralized human audit, vulnerable to state coercion and developer malice
- **Forensic Vulnerability**: Local execution leaves persistent traces on host OS, susceptible to physical device seizure and cold-boot attacks
- **Financial Surveillance**: Crypto-native agents require KYC/AML compliant rails, breaking the autonomy principle

### 1.2 Solution Thesis
**ClawCipher** is a **transport-layer privacy protocol** for AI Agents, analogous to TLS for HTTP but extending to:
- **Metadata-immune routing** (Onion-layer traffic shaping)
- **Zero-knowledge computation verification** (ZK-SNARKs for Skills attestation)
- **Anti-forensic execution environments** (RAM-only volatile states)
- **Crypto-native economic layer** (Non-custodial, permissionless agent economies)

### 1.3 Target Sovereign Users
- **High-net-worth individuals** (HNWI) requiring unobservable portfolio management automation
- **Journalist/NGO networks** operating in hostile jurisdictions (Iran, Russia, Belarus, China)
- **Decentralized Autonomous Organizations** (DAOs) requiring autonomous treasury management without single-signature trust
- **Cypherpunk developers** building censorship-resistant automation infrastructure

---

## 1.4 v0.1 Scope & Acceptance Charter (Normative)

### 1.4.1 What v0.1 *is*
v0.1 is an **engineerable** subset of the full ClawCipher thesis, optimized for:
- A drop-in compatibility surface for OpenClaw-style agent runtimes
- A working privacy transport that can be measured and regression-tested
- A *policy-enforced* Skills execution model that is safe-by-default
- ZK attestation only where it is tractable and testable (canonical Skills)

### 1.4.2 What v0.1 is *not*
The following are explicitly out-of-scope for v0.1 and moved to v1.0+:
- Full multi-hop onion routing with global passive adversary guarantees
- A decentralized marketplace (IPFS registry + L2 contracts + staking/slashing)
- ERC-4337 agent wallets, ZK payment channels, and cross-agent hiring
- A full ClawOS distribution (Live USB / hardened OS supply chain)
- "Hidden volumes" plausible deniability storage and TPM/secure-boot hard requirements

### 1.4.3 Threat model (v0.1)
v0.1 is designed to resist:
- **Passive local network adversary** (ISP, corporate proxy, hostile WiFi): observes client egress but does not control endpoint devices
- **Active local network adversary** (MITM attempts): can inject/drop packets but cannot break modern AEAD or signatures
- **Malicious Skill authors**: attempt exfiltration via filesystem/network/process APIs

v0.1 does **not** claim resistance to:
- **Global passive adversary** with ubiquitous visibility and long-term correlation
- **Endpoint compromise** (kernel-level malware, physical probing attacks beyond reasonable consumer defenses)

### 1.4.4 Privacy modes (v0.1)
All agents MUST support the following modes:
- **off**: direct outbound networking allowed; Skills policy still enforced
- **standard** (default): single-hop privacy tunnel, size padding (bucketed), timing jitter (bounded); no cover traffic
- **high**: standard + cover traffic enabled + stricter padding + reduced metadata surface (more aggressive normalization)

### 1.4.5 Canonical Skills set (v0.1)
v0.1 defines exactly five canonical Skills that MUST be supported:
- `email`
- `calendar`
- `file_ops`
- `web_search`
- `api_call`

Only canonical Skills are REQUIRED to provide ZK attestation in v0.1.

### 1.4.6 Compatibility definition (v0.1)
"Drop-in replacement for `openclaw-core`" means:
- Agent initialization APIs remain compatible at the type and behavioral levels
- LLM call semantics (request/response shape, streaming behavior if present) remain compatible
- Skills invocation semantics remain compatible

### 1.4.7 Acceptance tests (v0.1)
An implementation is considered v0.1-compliant only if it passes:
- **A1 (Tunnel enforcement)**: with direct egress blocked, the agent can still complete an LLM request via the tunnel
- **A2 (Destination concealment)**: a passive local observer cannot directly observe the true destination domain of LLM calls
- **A3 (Latency budget)**: in `standard`, p95 end-to-end latency ≤ 1.5× baseline direct call for a fixed prompt; in `high`, p95 ≤ 3× baseline
- **A4 (Policy hard-fail)**: Skills attempting undeclared filesystem or network access MUST fail with a deterministic policy error
- **A5 (No spawn by default)**: Skills attempting process spawning/system calls MUST fail
- **A6 (Canonical ZK gate)**: canonical Skills without valid proof MUST not execute (cryptographic failure, not warning)
- **A7 (ZK verify budget)**: canonical proof verification p95 < 5 seconds on consumer hardware (M1-class)
- **A8 (Ephemeral mode)**: no sensitive plaintext is written to disk during a standard task run in ephemeral mode

## 2. Requirements Specification

### 2.1 Functional Requirements (FR)

#### FR-1: Zero-Knowledge Skills Verification (ZK-Skill)
**Priority**: P0 (Critical)  
**Description**: v0.1 restricts mandatory ZK attestation to canonical Skills. For all other Skills, hard policy enforcement is required regardless of proof presence.
- **FR-1.1**: Skills developers must generate zk-SNARK proofs attesting that code:
  - Does not access filesystem paths outside declared manifest
  - Does not initiate network connections to non-whitelisted domains
  - Does not execute shell commands or system calls
- **FR-1.2 (v0.1)**: For canonical Skills, proof verification must complete in <5 seconds on consumer hardware (M1 MacBook Pro or equivalent)
- **FR-1.3 (v0.1)**: Canonical Skills rejected by verification must fail with a cryptographic error, not heuristic warning
- **FR-1.4 (v0.1)**: Non-canonical Skills MUST be sandboxed by policy enforcement even if proofs are not required

#### FR-2: Onion-Routed Agent Communication (ClawTunnel)
**Priority**: P0  
**Description**: v0.1 mandates tunnel enforcement with measurable privacy properties; multi-hop onion routing is a v1.0+ goal.
- **FR-2.1 (v0.1)**: Implement single-hop privacy tunnel mode (Client → Entry) for all LLM API calls
- **FR-2.2 (v0.1)**: Support bucketed packet/request size normalization and bounded timing jitter
- **FR-2.3 (v0.1)**: Provide `privacyMode` toggles: `off`, `standard`, `high`
- **FR-2.4 (v1.0+)**: Implement triple-layer onion routing (Entry → Relay → Exit)
- **FR-2.5 (v1.0+)**: Cover traffic baseline noise in high-security mode (e.g., ≥10 req/hour) and stronger traffic analysis resistance

#### FR-3: Anti-Forensic Execution (ClawOS)
**Priority**: P0  
**Description**: v0.1 focuses on eliminating accidental persistence and minimizing secret lifetime; full anti-forensic OS hardening is v1.0+.
- **FR-3.1 (v0.1)**: Ephemeral mode: no sensitive intermediate states (API keys, retrieved data, conversation history) may be written to disk by default
- **FR-3.2 (v0.1)**: Disable/avoid crash artifacts and persistence surfaces where possible (core dumps, verbose logs, unencrypted caches)
- **FR-3.3 (v1.0+)**: Volatile-only execution with systematic key shredding and hardened OS runtime
- **FR-3.4 (v1.0+)**: Plausible deniability storage (hidden volumes) for optional persistence
- **FR-3.5 (v1.0+)**: Secure boot chain with TPM/Libreboot measured boot attestation

#### FR-4: Decentralized Skills Marketplace (ClawBazaar)
**Priority**: P1 (High)  
**Description**: Skills distribution must be censorship-resistant and economically self-sustaining.
- **FR-4.1**: Content addressing: Skills stored on IPFS with Ethereum L2 (Arbitrum Nova) registry
- **FR-4.2**: Economic staking: Developers stake 0.5 ETH per Skill; slashing condition triggered by ZK proof of malicious behavior
- **FR-4.3**: Permissionless listing: No centralized authority can delist Skills; community challenges via optimistic fraud proofs

#### FR-5: Crypto-Native Agent Economy
**Priority**: P1  
**Description**: Agents must transact value without traditional financial rails.
- **FR-5.1**: ERC-4337 Abstract Accounts: Each Agent instance controls a smart contract wallet with programmable access rules
- **FR-5.2**: ZK-payment channels: Micropayments for API usage (per-token billing) via zk-SNARK verified state channels
- **FR-5.3**: Cross-agent hiring: Agent A can cryptographically commission Agent B without revealing principal identity

### 2.2 Non-Functional Requirements (NFR)

#### NFR-1: Performance
- **Latency**: End-to-end agent task execution (API call + processing + response) must complete within 3x baseline latency (e.g., if direct OpenAI call is 500ms, ClawCipher routing must complete <1500ms)
- **Throughput**: Single node must handle 100 concurrent agent sessions with <20% CPU overhead (M1 Pro baseline)

#### NFR-2: Security
- **Threat Model**: Resistant to:
  - Passive global adversary (NSA-level traffic monitoring)
  - Active local adversary (compromised ISP, hostile WiFi)
  - Physical device seizure (cold boot attacks, forensic disk analysis)
  - Malicious Skills developers (supply chain attacks)
- **Cryptographic Standards**: 
  - Asymmetric: Curve25519 (X25519 for ECDH, Ed25519 for signatures)
  - Symmetric: ChaCha20-Poly1305 (AES-GCM fallback for hardware acceleration)
  - ZK-Proofs: Groth16 (for Skills verification, trusted setup acceptable for v0.1; move to PLONK/STARKs in v1.0)

#### NFR-3: Compatibility
- **Backward Compatibility**: Must maintain OpenClaw core API compatibility (drop-in replacement for `openclaw-core` npm package)
- **Platform Support**: Tier 1: Linux (hardened kernel), Tier 2: macOS (SIP-disabled or T2-secured), Tier 3: Windows (WSL2 only, no native support for security reasons)

#### NFR-4: Legal Resilience
- **Jurisdiction**: Core development team must operate from non-extradition or privacy-friendly jurisdictions (Dubai, Singapore, Switzerland, Portugal)
- **Asset Protection**: 40% of treasury held in cold-storage multisig (3-of-5) with legal defense fund allocation
- **Code Liability**: All contributions under MIT license with explicit "experimental software" disclaimer; no corporate entity ownership (DAO-governed only)

---

## 3. Technical Architecture

### 3.1 High-Level Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Skills     │  │   User CLI   │  │   MCP Tools    │      │
│  │  (ZK-Verified)│  │  (Terminal)  │  │  (Sandboxed)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                    ClawCipher Protocol Layer                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  ZK-Verifier │  │ ClawTunnel   │  │   ClawBazaar │      │
│  │  (RISC Zero) │  │(Onion Router)│  │  (IPFS+L2)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                    OS/Runtime Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  ClawOS      │  │  RAM-only    │  │  Secure Enclave│      │
│  │(Hardened Linux│  │  Execution   │  │  (SGX/SEV)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                    Network Layer                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Noise Protocol│  │  I2P/Tor     │  │  Mesh Network │      │
│  │  (WireGuard) │  │  Bridges     │  │  (Optional)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Core Components

#### 3.2.1 ClawTunnel (Transport Privacy)
**Language**: Rust (async-std/tokio)  
**Architecture**:
- **Entry Nodes**: Operated by community volunteers, incentivized via micropayments (LN/BTC)
- **Mixing Strategy**: Loopix protocol implementation with Poisson-distributed cover traffic
- **Handshake**: Noise Protocol Framework (XX pattern) with ephemeral keys rotated every 5 minutes
- **Exit Policy**: Whitelist-only destinations (OpenAI API, Anthropic API, IPFS gateways); all other traffic dropped

**Threat Mitigation**:
- **Tagging Attacks**: Cryptographic checksums on cells; any bit-flipping detected causes circuit destruction
- **Timing Analysis**: Constant-rate dummy cell injection (200 cells/minute baseline)

#### 3.2.2 ZK-Skills Verifier (RISC Zero Integration)
**Architecture**:
- **Guest Program**: Rust-compiled to RISC-V, runs Skills code in isolated VM
- **Host Program**: Generates execution trace, proves memory access constraints and syscall filtering
- **Proof Submission**: Groth16 proof uploaded to Arbitrum Nova (L2) with 200-byte verification cost (~$0.001)

**Security Properties**:
- **Filesystem Confinement**: Proves `open()` syscalls only touch paths declared in `manifest.json`
- **Network Confinement**: Proves `connect()` syscalls only target IPs/Domains in whitelist
- **Reproducibility**: Same Skills code + input always generates same proof (deterministic execution)

#### 3.2.3 ClawOS (Hardened Runtime)
**Base**: Secureblue (hardened Fedora Silverblue) or Qubes OS R4.2+  
**Modifications**:
- **Kernel**: Linux 6.8+ with grsecurity patches, disabled modules, and seccomp-bpf strict mode
- **Storage**: RAMFS for `/tmp/agent_state`, encrypted LUKS2 for persistent storage with hidden volume support
- **Networking**: Network namespace isolation; all egress forced through ClawTunnel TUN interface
- **Hardware Trust**: Support for TPM 2.0 measured boot + Libreboot (for older hardware)

**Anti-Forensic Features**:
- **Memory Wiping**: `memset_explicit()` on free() with overwrite patterns (Gutmann method for SSD swap if enabled)
- **Panic Mode**: USB "kill switch" insertion triggers immediate RAM wipe and power off (<200ms)

### 3.3 Data Flow: Secure Agent Execution

```
1. User Input: "Summarize my ProtonMail inbox"
   ↓
2. ClawCipher Core: Encrypts intent with Ephemeral Key (X25519)
   ↓
3. ZK-Verifier: Loads "ProtonMail-Connector" Skill
   - Verify ZK-Proof: "Only accesses protonmail.com:443, only reads ~/.config/protonmail/"
   - If valid: Execute in Firejail sandbox
   - If invalid: Abort, log on-chain challenge
   ↓
4. ClawTunnel: Wraps HTTPS request in 3-layer onion:
   - Layer 1: Entry Node (Amsterdam)
   - Layer 2: Relay (Singapore)
   - Layer 3: Exit (Reykjavik) → ProtonMail API
   ↓
5. Response Path: Return journey with same onion circuit (but reversed)
   ↓
6. Local Processing: LLM inference via local Llama-3-70B (quantized) OR
   Remote inference via encrypted tunnel (if cloud model required)
   ↓
7. Output Delivery: Decrypted summary displayed in terminal
   ↓
8. State Destruction: Session keys shredded, RAM buffers wiped, no disk write
```

---

## 4. Economic & Governance Model

### 4.1 Tokenomics (ClawToken - CLW)
**Type**: ERC-20 on Arbitrum Nova (L2) with mainnet bridge  
**Distribution**:
- 40%: Protocol Development (3-year vesting, DAO-controlled)
- 30%: Node Operator Incentives (ClawTunnel relays)
- 20%: Security Audit & Bug Bounties
- 10%: Founder/Team (4-year vesting, 1-year cliff)

**Utility**:
- **Gas**: Pay for ZK-Proof verification on-chain
- **Staking**: Skills developers stake CLW; users stake CLW to challenge malicious Skills (slashing mechanism)
- **Governance**: DAO votes on protocol upgrades, node whitelisting, and treasury allocation

### 4.2 Revenue Model (Protocol Sustainability)
Unlike OpenClaw's potential SaaS model, ClawCipher generates revenue from:
1. **Skill Transaction Fees**: 0.01% of value flow when Agent uses Skills (paid by Skill developer)
2. **Privacy Premium**: High-net-worth users pay CLW for dedicated Entry Node bandwidth (SLA guarantees)
3. **Enterprise Licensing**: Optional non-GPL license for enterprises wanting to fork (funding open core)

---

## 5. Development Roadmap

### Phase 0: Genesis (Months 0-6)
**Objective**: Anonymous MVP for hardcore cypherpunks  
**Deliverables**:
- [ ] Fork OpenClaw, strip telemetry (GitHub: `clawcipher/core`)
- [ ] Rust implementation of ClawTunnel (single-hop proxy mode)
- [ ] ZK-Verifier MVP for 5 canonical Skills (email, calendar, file_ops, web_search, api_call)
- [ ] ClawOS Live USB alpha (hardened Arch-based)
- [ ] Closed alpha: 50 Telegram-selected testers

**Funding**: Self-funded 20万 RMB (~$28k) for infrastructure

### Phase 1: Underground Network (Months 6-18)
**Objective**: Distributed node network + Smart contracts  
**Deliverables**:
- [ ] ClawBazaar smart contract deployment (Arbitrum Nova testnet → mainnet)
- [ ] Multi-hop onion routing (3+ volunteer nodes per continent)
- [ ] IPFS integration for Skills storage
- [ ] Bug bounty program: $100k total allocation
- [ ] First enterprise pilot: 3 DAO treasuries using ClawCipher for autonomous payroll

**Funding**: Crypto donations + small grants (Gitcoin, Ethereum Foundation)

### Phase 2: Protocol Standardization (Months 18-30)
**Objective**: Become default transport for OpenClaw ecosystem  
**Deliverables**:
- [ ] IETF Internet-Draft: "ClawCipher: Privacy-Preserving Agent Communication Protocol"
- [ ] Integration with 3 major AI Agent frameworks (OpenClaw, AutoGPT, BabyAGI)
- [ ] Hardware partnership: Pre-installed ClawOS on Purism Librem 14 / MNT Reform
- [ ] Legal defense fund: 40万 RMB (~$55k) reserved for developer protection

### Phase 3: Sovereign Infrastructure (Months 30-36)
**Objective**: Self-sustaining protocol economy  
**Deliverables**:
- [ ] DAO full control (team tokens fully vested, governance decentralized)
- [ ] 1000+ active relay nodes globally
- [ ] ClawCipher as IETF standard (RFC)
- [ ] Resistance testing: Simulated state-level attack (seizure of 30% nodes, protocol must survive)

---

## 6. Risk Analysis & Mitigation

### 6.1 Technical Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| ZK-Proof generation too slow (>30s) | Medium | High | Move to STARKs (no trusted setup, faster proving) or hardware acceleration (GPU/ASIC) |
| Loopix latency unacceptable (>2s) | Low | High | Fallback to simpler 1-hop VPN mode for latency-critical tasks; keep 3-hop for high-security |
| Quantum computing breaks X25519 | Low | Catastrophic | Post-quantum algorithm migration plan (CRYSTALS-Kyber/Dilithium ready in v2.0) |

### 6.2 Legal/Regulatory Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Developer arrest (Pavel Durov scenario) | Medium | High | Distributed team, no single point of custody; legal defense fund; jurisdiction shopping |
| Skills used for CSAM/terror financing | Medium | Reputational | Community slashing + on-chain taint analysis (optional); but no centralized censorship |
| OFAC sanctions on CLW token | Low | Economic | Privacy-preserving DEX integration (no KYC); atomic swaps for untraceable acquisition |

### 6.3 Market Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| OpenClaw abandons open-source | Low | High | Maintain fork independence; ensure ClawCipher works with alternative Agent cores |
| Big Tech (Google/Apple) integrates privacy agents | High | Medium | Focus on "unbackdoored" guarantee; big tech cannot match our non-KYC promise |

---

## 7. Appendices

### Appendix A: Comparison with Existing Solutions
- **Tor Browser**: Only protects browsing, not AI Agent automation; no ZK verification
- **Nym**: Similar mix-net but focused on generic messaging, not Agent-specific Skills attestation
- **Ollama (local LLM)**: Only local inference, no secure cloud fallback, no Skills marketplace
- **OpenClaw**: Baseline functionality, zero privacy guarantees (the problem we solve)

### Appendix B: Cryptographic Specifications
- **Key Exchange**: X25519 (Curve25519)
- **Symmetric Encryption**: ChaCha20-Poly1305 (RFC 8439)
- **Hashing**: BLAKE3 (for ZK circuits), SHA-3-256 (for blockchain anchoring)
- **ZK-Proofs**: Groth16 (BN254 curve) for v0.1; migrate to BLS12-381 for v1.0

### Appendix C: Glossary
- **Agent**: Autonomous AI process executing tasks on behalf of user
- **Skills**: Modular code packages extending Agent capabilities
- **ZK-Proof**: Zero-knowledge succinct non-interactive argument of knowledge
- **ClawTunnel**: ClawCipher's onion-routing network layer
- **ClawBazaar**: Decentralized Skills marketplace

---

**Document Control**  
Author: Anonymous (Pseudonym: RedLobster)  
Review Cycle: Quarterly  
Next Review: 2026-06-20  
Distribution: Core Team Only (GPG encrypted)

**"Privacy is not for the passive; it is the infrastructure of freedom."**
— Paraphrased from Durov, 2013
```
