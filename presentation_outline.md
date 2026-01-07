
---

# Slide 1: Title

**Democratizing Visibility into Personal Device Data Flows**

*A Technical Feasibility Study*

- Systems & Security Research Perspective
- January 2026

---

# Slide 2: The Problem

## Users Can't See Their Own Data

- Encryption hides content from users, not just attackers
- Apps transmit data with no visibility to device owners
- Existing tools show *permissions granted*, not *data flowing*
- Gap between user intent and network behavior is opaque

---

# Slide 3: The Vision

## A Transparency System That Shows:

| Question | Answer Provided |
|----------|-----------------|
| What left my device? | Correlated packet metadata |
| When? | Timestamped event logs |
| From which app? | Process attribution |
| Why? | Semantic context from user actions |

**Key Principle:** Correlation, not decryption

---

# Slide 4: System Architecture

```
┌────────────────────────────────────────────┐
│           USER-OWNED DEVICE                │
├────────────────────────────────────────────┤
│  [Input Observer] → [Entry Point Log]      │
│         ↓                                  │
│  [Correlation Engine] ← [Network Observer] │
│         ↓                                  │
│  [User Dashboard]                          │
└────────────────────────────────────────────┘
```

---

# Slide 5: Observation Layers

## Four Points of Visibility

1. **Input Layer** — Keystrokes, touch events, IME commits
2. **UI Layer** — Form submissions, button actions, text fields
3. **Application Layer** — Message composition, file selections
4. **Network Layer** — Packet metadata, timing, destinations

---

# Slide 6: Correlation Signals

## How We Link Actions to Packets (Without Decryption)

| Signal | Method |
|--------|--------|
| **Timing** | Input at *t₀* → packet at *t₀ + δ* |
| **Process** | Match PID of input receiver to socket owner |
| **Size** | Plaintext length → expected ciphertext size |
| **Destination** | Known service endpoints narrow interpretation |

---

# Slide 7: Platform Feasibility Summary

| Platform | Feasibility | Key Constraint |
|----------|-------------|----------------|
| Windows | ✅ High | Minimal restrictions |
| Linux (X11) | ✅ High | Full API access |
| macOS | ⚠️ Moderate | TCC permission gates |
| Android (rooted) | ✅ High | Full observability |
| Android (non-rooted) | ⚠️ Limited | No process attribution |
| iOS | ❌ Not viable | Architectural blocks |

---

# Slide 8: Desktop Platforms

## Windows & Linux: Most Permissive

**Available:**
- Global keyboard hooks
- Process-socket mapping via OS APIs
- eBPF/ETW for rich correlation

**Constraints:**
- Wayland (Linux) blocks global input capture
- Security software may flag behavior

---

# Slide 9: Mobile Platforms

## Android: Partial Success

- ✅ AccessibilityService for input capture
- ✅ VpnService for network observation
- ❌ No UID from VPN API (non-rooted)
- ⚠️ Play Store policy friction

## iOS: Architecturally Blocked

- ❌ No third-party input observation APIs
- ❌ No process info from VPN framework
- Only viable on jailbroken devices

---

# Slide 10: Correlation Accuracy

| Scenario | Accuracy | Limiting Factor |
|----------|----------|-----------------|
| Interactive messaging | 70-85% | Background keepalives |
| Web form submission | 80-90% | Clear request pattern |
| File upload | 60-75% | Chunking, parallelism |
| Background sync | 30-50% | No user trigger |

---

# Slide 11: False Attribution Risks

## When Correlation Fails

1. **Temporal collision** — Background service sends during user action
2. **Multiplexing** — HTTP/2, QUIC combine streams
3. **Encryption overhead** — Padding obscures size
4. **Batching** — Apps buffer before sending
5. **Indirect causation** — Input triggers delayed processing

---

# Slide 12: What's Fundamentally Unobservable

## Hard Limits Regardless of Implementation

- Application decision logic (compiled, no hooks)
- Deferred transmission queues (in-process memory)
- Encrypted payload semantics (by design)
- Pre-encryption compression (destroys size correlation)
- Secure enclave operations (hardware isolation)

---

# Slide 13: Unique Value Proposition

## What This Provides That Existing Tools Don't

| Existing Tool | Limitation | Our Advantage |
|---------------|------------|---------------|
| Permission prompts | Shows access, not data | Shows actual flows |
| Network monitors | No semantic context | Links to user actions |
| Privacy reports | Aggregate counts | Content-level correlation |
| Firewall apps | Binary allow/deny | Explains *why* traffic occurred |

---

# Slide 14: Vendor Response Scenarios

## How Platforms Might React

| Response | Likelihood | Impact |
|----------|------------|--------|
| Ignore | Low | Status quo |
| Block/restrict | Medium | Tighten APIs |
| Co-opt | High | First-party "Privacy Dashboard 2.0" |
| Collaborate | Low | Official transparency APIs |

---

# Slide 15: Research Critique — Gaps Identified

## What's Missing for Academic Rigor

1. ❌ No explicit threat model
2. ❌ No formal correctness guarantees
3. ❌ No evaluation methodology
4. ❌ No legal analysis
5. ❌ No human factors study
6. ❌ No boundary specification vs. stalkerware

---

# Slide 16: Threat Model Gap

## Questions That Must Be Answered

- **Who is the adversary?** Curious apps? Malicious apps? OS vendor?
- **What's in the TCB?** Do we trust the kernel? Hardware?
- **What guarantees?** Process X sent bytes ≠ payload contained Y

**Required:** Explicit adversary taxonomy and trust boundary declaration

---

# Slide 17: Correctness & Uncertainty

## Formalizing "Accuracy"

```
Correlation Claim: C(input_i, packet_j) = True
Ground Truth: G(i,j) ∈ {True, False, Unknown}

Metrics needed:
- Precision: P(G=True | C=True)
- Recall: P(C=True | G=True)
- Confidence calibration
```

**Key question:** How do we express uncertainty to users?

---

# Slide 18: Countermeasure Resilience

## How Easily Can This Be Defeated?

| Countermeasure | Effect on Correlation |
|----------------|----------------------|
| 10% random padding | Size correlation breaks |
| 2+ second batching | Timing correlation fails |
| Randomized jitter | Temporal analysis degrades |
| Child process for I/O | Attribution may miss |

**Required:** Quantitative degradation curves

---

# Slide 19: Theoretical Positioning

## This is Partial Observation of a Hidden Markov Model

```
User Intent → Application State → Network Behavior
     ↑              ↑                    ↑
  Observable    Unobservable         Observable
```

- Related to: traffic analysis, information flow control, side-channel research
- **Key insight:** We're doing "benevolent side-channel analysis" on our own device

---

# Slide 20: The Self-Trust Problem

## The Observer Becomes a Risk

| Data Type | Sensitivity | Retention Need |
|-----------|-------------|----------------|
| Raw keystrokes | Extreme | Minimal |
| Packet metadata | High | Medium |
| Timing data | Medium | Medium |

**Question:** What is the *minimum* observation for *meaningful* transparency?

---

# Slide 21: Legal Considerations

## Unaddressed Regulatory Risks

- **GDPR:** Data minimization, purpose limitation
- **Wiretap laws:** "Interception" definitions vary
- **Third-party data:** Messages contain *other people's* information
- **Enterprise:** BYOD, MDM, lawful intercept obligations

**Critical:** User consent may not cover third-party data

---

# Slide 22: Human Factors Risks

## Misinterpretation Taxonomy

| User Thinks | Reality |
|-------------|---------|
| "App sent data when I typed = keylogger" | Normal sync behavior |
| "Connecting to AWS = suspicious" | Standard CDN usage |
| "No correlation = nothing sent" | Batching hid the transmission |
| "Large packet = big leak" | Cached assets |

**Required:** Abstractions that prevent paranoid conclusions

---

# Slide 23: Dual-Use Problem

## This Architecture = Stalkerware Architecture

**Differentiation is in consent mechanisms, not capabilities**

Design requirements:
- Persistent visible indicator (no silent operation)
- Logs never leave device without explicit action
- No cross-user correlation
- Clean uninstallation guaranteed

---

# Slide 24: Evaluation Framework

## Proposed Validation Approach

1. **Ground truth construction**
   - Instrumented test apps with known behavior
   - Synthetic labeled traffic

2. **User studies**
   - Can users correctly identify data sources?
   - Misinterpretation rate measurement

3. **Adversarial testing**
   - Deploy countermeasures, measure degradation

---

# Slide 25: Explicit Non-Goals

## What the System Must Refuse to Do

| Capability | Decision | Rationale |
|------------|----------|-----------|
| Full keystroke retention | ❌ Refuse | Enables abuse |
| Payload storage | ❌ Refuse | Useless + risky |
| Cross-device correlation | ❌ Refuse | Scope creep |
| Third-party export | ❌ Refuse | Fundamental violation |
| Auto-anomaly alerts | ⚠️ Opt-in only | Paranoia risk |

---

# Slide 26: Feasibility Verdict

| Platform | Production | Research |
|----------|------------|----------|
| Windows | ✅ Viable | ✅ Valuable |
| Linux (X11) | ✅ Viable | ✅ Valuable |
| macOS | ⚠️ Moderate | ✅ Valuable |
| Android (rooted) | ✅ Viable | ✅ Valuable |
| Android (stock) | ⚠️ Limited | ⚠️ Partial |
| iOS | ❌ Not viable | ⚠️ Jailbreak only |

---

# Slide 27: Key Contributions

## What This Work Provides

1. **Novel transparency class** — Semantic user intent ↔ network behavior
2. **Platform capability mapping** — What's possible where
3. **Correlation methodology** — Non-decryption-based attribution
4. **Boundary analysis** — What's fundamentally unobservable
5. **Dual-use framework** — Distinguishing transparency from surveillance

---

# Slide 28: Research Agenda

## Required Future Work

1. Formal threat model and guarantee specification
2. Uncertainty quantification framework
3. Ground truth dataset for validation
4. Legal analysis across jurisdictions
5. User study: interpretation accuracy
6. Countermeasure resilience measurement

---

# Slide 29: The Fundamental Tension

> "Platforms increasingly treat transparency as a **vendor-controlled feature** rather than a **user right**."

This system makes that tension visible.

---

# Slide 30: Conclusion

## Summary

- **Technically feasible** on open platforms
- **Architecturally blocked** on iOS
- **Provides unique insight** no existing tool offers
- **Requires careful design** to avoid stalkerware equivalence
- **Research value** extends beyond implementation

**The question is not "can we build this?" but "what guarantees can we provide, and how do we know?"**

---

# Slide 31: Questions?

**Contact / Discussion**

*Thank you*
