# Credora

> Portable professional verification infrastructure.

**Version:** 1.0.0
**Status:** Active Development
**License:** Proprietary © Credora

---

## Table of Contents

- [What Credora Is](#what-credora-is)
- [The Problem](#the-problem)
- [How It Works](#how-it-works)
- [Core Principles](#core-principles)
- [Features](#features)
- [Verification Labels](#verification-labels)
- [Security Architecture](#security-architecture)
- [Liability Model](#liability-model)
- [Governance](#governance)
- [Known Limitations](#known-limitations)
- [Technology Stack](#technology-stack)
- [Business Model](#business-model)
- [Geographic Expansion](#geographic-expansion)
- [Roadmap](#roadmap)
- [Contributing & Integration](#contributing--integration)

---

## What Credora Is

Credora is a **credential transport and verification protocol**.

It moves a signed claim from an issuer to a verifier and lets anyone confirm the signature is authentic and unmodified.

```
Issuer signs credential
      ↓
Credora transports and stores it
      ↓
Holder controls who accesses it
      ↓
Verifier checks the signature and revocation status
      ↓
Result: Valid | Invalid | Revoked
```

Credora does not score people. It does not rank candidates. It does not evaluate talent. It confirms whether a specific credential was issued by a specific source and remains unrevoked.

---

## The Problem

### Credentials Are Self-Reported

Modern hiring runs on unverifiable claims. Anyone can fabricate a project, exaggerate experience, or edit a certificate. PDFs have no mechanism to confirm their own authenticity. Recruiters spend significant time on manual verification — a process that is slow, inconsistent, and still fails regularly.

### Verification Is Fragmented

Universities, employers, and certification platforms each operate their own verification processes. None are standardized. None interoperate. Most require manual outreach. There is no shared infrastructure.

### The Signal Problem Is Getting Worse

AI-generated applications and synthetic credentials are increasing in volume. Existing tools were not designed to handle this.

---

## How It Works

### Verification Flow

```
Institution (university, bootcamp, employer)
  └─ Signs credential with HSM-backed private key
        ↓
Credora Transport Layer
  └─ Stores encrypted credential
  └─ Maintains public issuer key registry (CT-style mirrored log)
  └─ Maintains public verification endpoint
        ↓
Holder (student, professional, freelancer)
  └─ Controls access permissions
  └─ Generates Controlled Access Profile link
  └─ Sets expiry and revocation rules
        ↓
Verifier (recruiter, platform, employer)
  └─ Checks signature against issuer public key
  └─ Checks live revocation status
  └─ Receives Cache-Control header enforcing revalidation window
        ↓
Result: Valid | Invalid | Revoked
```

Every verification response includes a timestamp, freshness expiration, and live revocation status. There are no permanently cached results.

### Revocation

Revocation propagates within 60 seconds via webhook to all active verifier sessions.

For verifiers unable to accept webhooks, a fallback polling endpoint is available:

```
GET /v1/revocation-status/{credential_id}
```

All verification API responses include `Cache-Control: max-age=1209600` (14 days) for high-risk credential types (professional licenses, employment verifications) and `max-age=2592000` (30 days) for standard credentials. Revalidation is enforced at the protocol level — not just documented in terms of service.

Verifiers see:

```
✓ Degree Verified
   └ Issued: June 2023 — IIT Bombay
   └ Verified: 14 days ago
   └ Revalidation required: in 16 days
   └ Revocation status: Active (checked 2 hours ago)
```

If a credential is revoked, verifiers see the revocation date and issuing institution — not a silent failure.

### NFC Card (Premium)

```
NFC tap → Credora UIN + Signed Auth Token
      ↓
Server validates signature
      ↓
Permission rules applied
      ↓
Controlled Access Profile delivered
```

---

## Core Principles

### Credora Verifies Claims. Not People.

```
Old:  Human → Score → Decision       (wrong)
New:  Claim → Evidence → Confidence  (correct)
```

Credora confirms whether a specific claim was issued by a verified source and remains active. It does not evaluate the person behind the claim.

### Proof, Not Exposure

Holders control what is visible, who can access it, and how long access lasts. Credora stores proof — not permanent exposure.

### Infrastructure, Not Judgment

Credora is a transport layer. Institutions own issuance liability. Recruiters own hiring decisions. Credora validates signatures and proof integrity — nothing more.

### Honest Scope

Credora removes one specific obstacle from hiring: not knowing whether credentials are genuine. It does not claim to solve hiring.

---

## Features

### Verified Credential Vault

A secure encrypted vault of professional records, each signed by the issuing institution.

| Category | Supported Types |
|---|---|
| Education | Degrees, diplomas, transcripts |
| Employment | Offer letters, experience letters, internship confirmations |
| Certification | Platform certifications, professional licenses |
| Projects | Verified repository contributions, portfolio links |
| Assessments | Proctored skill evaluations |
| Research | Published papers, conference presentations |
| Freelance | Verified client engagements, payment proofs, platform tenure |

**Verification sources:**

- Universities and educational institutions
- Employers and companies
- Certification platforms (Coursera, AWS, Google, etc.)
- Government identity systems via KYC adapter (Aadhaar, DigiLocker, Passport)
- Third-party KYC providers (Persona, Onfido, etc.)
- Freelance platforms via API (Upwork, Toptal, Contra)
- Payment processors for freelance engagement confirmation (Stripe, Razorpay, Wise)

---

### Controlled Access Profiles

Permissioned, expiring, live-updating views of verified credentials.

- Configurable expiry: 24 hours to permanent
- Role-specific views from a single verified identity
- Live revocation — profile updates instantly when a credential status changes
- Recruiter view analytics
- Per-link permission management
- Optional privacy-preserving mode: verifier receives only `{valid | invalid | revoked, timestamp}` and a blinded issuer fingerprint — no credential type, no issuer name, no metadata

**What this guarantees:** permissioned visibility, live state, revocable access, auditability, expiration enforcement.

**What this does not guarantee:** prevention of exfiltration. A recruiter can screenshot or copy information after access is granted. This is stated plainly — not buried.

---

### Issuer Dashboard

Institutions issue, manage, and revoke signed credentials via API or dashboard.

- Credential issuance (individual and bulk)
- Instant revocation with propagation
- Full audit trail
- Key rotation tooling with automated suspension on missed rotation deadlines
- Anomaly alerts on issuance volume and behavioral patterns
- Key liveness challenge responses (see Security Architecture)

**Issuer Reliability Indicators** — operational track record visible to all verifiers:

```
Issuer: Example University
   └ Credentials issued (12 months): 4,200
   └ Revocation responsiveness: avg 8 days
   └ Fraud incidents disclosed (12 months): 3
   └ Verification audit score: 81%
   └ Dispute resolution rate: 94%
   └ Identity confidence level: High (Persona KYC, liveness verified)
   └ Compliance status: SOC 2 Type II verified
```

This is not a prestige ranking. It is an operational reliability record. Issuer trust affects verification confidence — not candidate quality. These are explicitly different things.

---

### Recruiter Tools

- Instant signature verification
- Live revocation status
- Explainable skill matching: which credentials matched, which requirements are unmet, no black-box outputs
- Fairness disclosure when verification-only filters are applied
- Automated flagging when a single verifier runs high-frequency checks on one holder (>10 in 24 hours triggers human review)
- ATS plugin integrations (Greenhouse, Lever, Workday — Phase 2)

---

### Freelancer Verification

Freelance engagement confidence levels:

| Confidence | Requirements |
|---|---|
| `~ Weakly Verified` | Single source (client attestation only) |
| `✓ Verified` | Two independent sources confirmed |
| `✓ Strongly Verified` | Payment processor + client signature + platform API (all three required) |

For `✓ Strongly Verified`: at least one source must be a payment processor (Stripe, Razorpay, Wise), as these run their own KYC on merchants and provide a genuinely independent signal.

Single-source freelance attestations are never displayed as equivalent to institutionally signed credentials.

---

### Data Portability

Users can export all credentials in open standard formats at any time:

- W3C Verifiable Credentials (JSON-LD)
- OpenBadges 3.0
- Signed PDF

This is mandatory, not optional. Users own their verified credentials. Credora is the transport layer.

---

### Credora Points

A verification quality score inspired by CIBIL — anchored to PAN.

**What it scores:** the evidence package, not the person. Credora Points measures how well-verified a profile is — not how good the person is.

Every user onboards with a verified PAN. PAN is the identity anchor that ties all credentials, issuances, and verifications to one confirmed real person. This makes synthetic identity fraud structurally harder and gives Credora Points a reliable, unfakeable foundation.

**Scoring dimensions:**

| Dimension | What it measures |
|---|---|
| Identity anchor | PAN verified? KYC confidence level? |
| Credential coverage | % of submitted credentials institutionally signed vs self-declared |
| Issuer reliability | Are issuers SOC 2 compliant and high-reliability? |
| Recency | How fresh are verifications? Any expired revalidations? |
| Consistency | Do employment dates, education timelines, and tax signals align? |
| Revocation history | Any credentials revoked? Were they disclosed? |

**What it does not score:**
- Which institution someone attended
- Employment or education gaps
- Employer or institution prestige
- Career switches
- Self-declared skills (these remain labeled `~ Self-Declared` and sit outside the model entirely)

**Example breakdown:**

```
Credora Points: 780 / 1000

  Identity anchor:        ██████████  100/100  PAN verified, KYC High confidence
  Credential coverage:    ████████░░   82/100  4 of 5 credentials institutionally signed
  Issuer reliability:     █████████░   91/100  All issuers SOC 2 compliant
  Recency:                ███████░░░   74/100  One credential approaching revalidation
  Consistency:            ████████░░   80/100  Employment timeline aligns with DigiLocker records
  Revocation history:     ██████████  100/100  No revocations
```

**Guardrails:**

Credora Points cannot be used as a standalone filter. Recruiters cannot set a minimum points threshold as a hiring gate. The full dimensional breakdown is always visible — a recruiter who sees a lower score can see exactly which dimension is low and why. The score is a summary entry point, not a rejection mechanism.

The score rewards authenticity and verification depth. It does not punish anyone for what they haven't yet verified.

**v1.0 scope:** India only, PAN-anchored. International equivalents (SingPass, NIN, Emirates ID) added per market in subsequent versions via the KYC adapter layer.

---

## Verification Labels

| Label | Meaning |
|---|---|
| `✓ Verified` | Cryptographic signature confirmed, revocation status active |
| `✓ Strongly Verified` | Multi-source confirmation (freelance) |
| `~ Self-Declared` | User-submitted, no third-party signature |
| `⚠ Pending` | Submitted for institutional confirmation, not yet signed |
| `✗ Revoked` | Previously verified, issuer has since revoked |
| `↻ Expired` | Verification timestamp exceeds policy freshness window |

Every label expands to full issuer metadata, verification method, timestamp, revocation status, and issuer compliance status. The label is the entry point. The metadata is the actual information.

---

## Security Architecture

### Credential Format

Credora uses **W3C Verifiable Credentials 1.1** with Ed25519 cryptographic signatures (RFC 8037).

Stored per credential:
- Encrypted credential payload (holder-key encrypted)
- Cryptographic hash
- Issuer signature
- RFC 3161 trusted timestamp
- Revocation pointer

Not stored publicly:
- Raw personal data
- Government ID payloads
- Sensitive document contents

### Issuer Key Infrastructure

Issuers operate as mini certificate authorities. Onboarding requires third-party compliance attestation before any issuance rights are granted.

**Pre-onboarding requirements:**
- SOC 2 Type II or ISO 27001 certification with explicit HSM controls
- Self-attestation is not accepted — third-party audited only

**Ongoing controls:**
- HSM-backed private key storage (FIPS 140-2 Level 3 minimum)
- 90-day key rotation default; 30-day for high-volume issuers
- Short-lived signing certificates with automatic expiry
- Missed rotation deadlines trigger automatic issuance suspension — not just a dashboard alert
- Anomaly detection on issuance volume and behavioral patterns
- Public key compromise transparency log — disclosures are mandatory, not voluntary

**Key liveness challenges:**
Credora's API issues periodic random nonce challenges to issuers at unpredictable intervals. The issuer must return the nonce signed by their current active key within a defined window. A failed or missing response triggers issuance suspension and human review. This proves the key is live and accessible without revealing any key material — and detects low-and-slow attacks that volume anomaly detection alone would miss.

### KYC Adapter Layer

```
KYC Provider (Aadhaar / DigiLocker / PAN / Passport / Third-party)
      ↓  returns: verification token + confidence score
KYC Adapter Layer
      ↓
Identity Verification Token + Confidence Score
      ↓
Credora UIN (Universal Identity Number)
```

**PAN as the primary identity anchor (India v1.0):**
Every Credora user onboards with a verified PAN. PAN is India's government-issued financial identity number — linked to ITR filings, TDS deductions, employer records, and bank accounts. It is the most reliable single identity anchor available in the Indian market. Faking a PAN that passes Income Tax Department verification requires actual document fraud — not a casual attack.

PAN gives Credora:
- A single verified identity that all credentials, issuances, and verifications tie back to
- Employment history corroboration: if an employer issued a credential and deducted TDS against the same PAN during the same period, that is an independent financial signal confirming the employment was real
- Structural synthetic identity resistance: one PAN = one real person, making multi-identity fraud significantly harder

KYC adapters are required to return a **confidence score** reflecting liveness detection quality and document authenticity. This score surfaces in Issuer Reliability Indicators as Identity Confidence: High / Medium / Low.

**Cross-provider deduplication (privacy-preserving):**
On KYC completion, Credora computes a salted HMAC of normalized biographic data (name + date of birth + document type). The input data is immediately discarded — only the hash is stored. If a new KYC token produces the same hash, the account is flagged for human review as a potential duplicate identity.

**International expansion:** PAN is India-specific. Equivalent anchors per market are added via adapter as Credora expands — SingPass (Singapore), MyKad (Malaysia), Emirates ID (UAE), NIN (UK). The adapter interface is anchor-agnostic by design. Credora Points scoring dimensions work regardless of which anchor is in use.

No single government identity system is a core dependency. New KYC providers are added via adapter without rebuilding the identity layer.

### Issuer Key Registry

The issuer key registry is published as a **Certificate Transparency-style signed append-only log**. Any verifier can mirror a full copy of the registry. Verification works against a local mirror even if Credora's API is temporarily unreachable, as long as the cached registry is within its freshness window.

Registry updates are signed by Credora's own root key. Mirrors can verify they have not been tampered with independently.

This design means no single point of failure for verification, and no single point of attack on the key registry.

### Audit Log Integrity

The audit log is an append-only PostgreSQL table with hash chaining and write-once row-level security. Application layer cannot update or delete rows — only insert.

The audit log contains no personal data — only anonymous event records (event type, credential ID hash, actor ID, timestamp). GDPR erasure deletes personal data from the credentials table. The audit chain entry for that deletion remains, referencing nothing personal.

**External anchoring against insider threat:**
To protect against a compromised database administrator truncating or rewriting the audit log, the hash of the latest chain entry is periodically anchored to an immutable external store:

- **Standard tier:** AWS Glacier Vault Lock or Azure Immutable Blob Storage (WORM — Write Once Read Many). Anchoring frequency: daily. Cost: negligible.
- **Enterprise compliance tier:** Additional anchoring options available per client requirements.

This closes the insider truncation risk without using blockchain and without conflicting with GDPR — because only an anonymous hash of a hash chain entry is anchored externally, not any personal data.

All database admin access requires MFA. Any admin action outside normal application service accounts triggers an immediate alert to a separate, admin-inaccessible monitoring log. Separation of duties: the person with production database credentials cannot also be the person reviewing security alerts.

### No Blockchain

Credora does not use blockchain at any tier.

Verification integrity is provided entirely by:

- **Ed25519 cryptographic signatures** — issuer-signed, independently verifiable against the public issuer key registry. Verification time under 5ms. Cost effectively zero.
- **Append-only PostgreSQL audit log with hash chaining** — tamper-evident, deletable by design, GDPR compliant.
- **RFC 3161 trusted timestamping** — legally recognized, court-admissible in EU, Indian, and US jurisdictions. Stronger legal standing than a blockchain transaction in most jurisdictions.
- **WORM external anchoring** — closes the insider truncation risk for the audit log at near-zero cost, without immutability conflicts on personal data.

| Requirement | Blockchain | Credora's stack |
|---|---|---|
| Tamper-evident records | ✓ | ✓ |
| Independent verifiability | ✓ | ✓ |
| Legal timestamp | Weak | ✓ RFC 3161 — court admissible |
| Audit log insider protection | ✓ | ✓ WORM anchoring |
| GDPR / DPDP erasure compliant | ✗ | ✓ |
| Verification speed | 1–7 minutes | < 5ms |
| Cost per operation | $2–50 gas fees | ~$0 |
| Regulatory risk | High | None |
| Engineering overhead | High | Low |

Immutability — blockchain's defining property — directly conflicts with the right to erasure under GDPR Article 17, India's DPDP Act Section 13, and CCPA. There is no architectural workaround that satisfies both simultaneously. Credora removes the conflict at the source.

### Data Architecture

**PostgreSQL handles all persistent storage:**

| Table | Contents | Notes |
|---|---|---|
| `credentials` | Encrypted payload, issuer signature, metadata | Payload encrypted with holder-derived key |
| `holders` | Encrypted identity token, Credora UIN, KYC confidence score | Raw government ID never stored |
| `issuers` | Public key, key history, reliability metrics, compliance status | Public registry |
| `kyc_dedup` | Salted HMAC of biographic data only | Input data never stored; used for duplicate detection |
| `access_profiles` | Permissions, expiry, view analytics | Per Controlled Access Profile link |
| `audit_log` | Event type, credential ID hash, actor ID, RFC 3161 timestamp, hash chain | Append-only, no personal data, write-once RLS |
| `revocations` | Credential ID, revocation timestamp, issuer ID | Propagates via webhook within 60 seconds |

**Row-level security enforced at database layer:**

```
Holder     → read/write own credentials, delete own data
Issuer     → write issued credentials, revoke only
Verifier   → read explicitly shared credentials during access window only
Admin      → audit log read, key registry management, no payload access
```

No Credora employee can read encrypted credential payloads. Encryption key is derived from the holder's account — architectural constraint, not policy.

**GDPR erasure flow:**

```
User requests deletion
      ↓
Credential payloads deleted from PostgreSQL
      ↓
Identity token and UIN mapping deleted
      ↓
KYC dedup hash deleted
      ↓
Access profile links invalidated
      ↓
Audit log receives anonymous deletion event (no personal data)
      ↓
Issuer notified (they retain their own issuance records per their legal obligations)
      ↓
Timestamped confirmation issued to user
```

Zero personal data retained after full erasure. Audit chain integrity preserved.

### Data Minimization

Verifier API responses expose only verification-required metadata. Behavioral signals — activity frequency, login patterns, profile update history — are never surfaced to verifiers unless explicitly user-approved.

Automated anomaly detection monitors verifier API usage patterns. More than 10 verification requests on a single holder from the same verifier within 24 hours triggers a human review flag for potential behavioral profiling.

Enterprise contracts include addendums explicitly prohibiting behavioral profiling from verification metadata. Policy violations require active enforcement — logging catches patterns; individual misuse requires human review.

---

## Liability Model

Credora's legal position is that of an infrastructure provider — not a truth authority.

The precedent is established:
- Email providers are not liable for fraud transmitted over email
- DocuSign is not liable for the contents of contracts it carries
- Certificate Authorities carry reissuance liability for their own key failures — not the sites that relied on their certificates

**Credora carries:**
- Cyber liability insurance covering breach response, notification, and affected party compensation up to policy limits
- Errors and omissions coverage

**Institutional contracts include:**
- Liability cap at annual contract value or defined maximum (standard enterprise SaaS terms — signable by legal teams)
- Shared security obligations: SOC 2 compliance, HSM key storage, rotation schedules, access controls, breach notification timelines
- Liability shifts toward institutions when breach results from institutional key mismanagement or compliance obligation failure
- Liability shifts toward Credora insurance when breach results from Credora infrastructure failure

Unlimited downstream liability provisions are not in any contract. No institutional legal team signs them — and demanding they do kills deals before they start.

---

## Governance

### Dispute Resolution

| Issue | Path | SLA |
|---|---|---|
| Credential dispute | Holder-initiated appeal → issuer review | 30 days |
| Recruiter abuse | Report → investigation → suspension | 14 days |
| Institution audit | Annual review + ad hoc on fraud incident | Ongoing |
| Profile correction | User-initiated with audit trail | 7 days |
| Key compromise | Immediate suspension → reissuance protocol → breach notification | 24 hours |
| Liveness challenge failure | Automatic suspension → issuer notified → human review | Immediate |

### Transparency

Credora publishes annual transparency reports covering:
- Total credentials issued and active
- Revocation rates by issuer category
- Fraud incidents disclosed
- Dispute volumes and resolution rates
- Key compromise and liveness challenge failure incidents
- Verifier anomaly flags and outcomes

Reports are public. Nothing material is withheld.

---

## Known Limitations

Infrastructure systems always have residual risk. The following are Credora's known limitations, how each is mitigated, and where the honest boundary of mitigation sits.

---

**Issuer key compliance enforcement** — Credora cannot remotely verify that a key physically resides in a FIPS 140-2 Level 3+ HSM, or that a SOC 2 audit accurately reflects current practices. Mitigated by third-party attestation requirements before onboarding, periodic key liveness challenges, and automatic issuance suspension on missed rotation deadlines. A motivated and sophisticated issuer who passes a SOC 2 audit while mismanaging their keys in practice is not detectable by remote means. The liveness challenge detects compromised keys in active use; it does not detect keys stored insecurely that have not yet been exploited.

---

**Revocation for offline verifiers** — verifiers who do not implement webhooks, ignore HTTP cache headers, or operate fully offline may act on stale verification data indefinitely. Mitigated by the fallback polling endpoint, `Cache-Control` enforcement at the API level, and a 14-day revalidation window for high-risk credential types. Credora cannot force correct client implementation. This is a documented risk, not a hidden one.

---

**KYC synthetic identity** — a sophisticated attacker with legitimate identities in multiple jurisdictions can generate multiple valid KYC tokens and use them to simulate independent multi-source freelance verification. Mitigated by privacy-preserving cross-provider deduplication and payment processor KYC requirements for `✓ Strongly Verified` freelance credentials. A determined attacker with two genuinely distinct real-world identities is not detectable by any KYC system currently in existence. Credora raises the cost and complexity of this attack substantially — it does not eliminate it.

---

**Metadata surveillance** — a motivated verifier can correlate verification timestamps, credential types, and issuer IDs to infer behavioral patterns without receiving any explicitly behavioral data. Mitigated by the optional privacy-preserving verification mode, automated anomaly detection on verifier API usage, and enterprise contract addendums. Full cryptographic privacy would require zero-knowledge proofs — a potential Phase 3+ feature. The current mitigation is policy-enforced, not cryptographically enforced.

---

**Audit log insider threat** — a database administrator with direct PostgreSQL access could in principle delete audit log rows and rewrite the hash chain. Mitigated by daily WORM anchoring of the latest chain hash to AWS Glacier Vault Lock or equivalent, MFA on all admin access, break-glass logging, and separation of duties. The anchoring means any truncation of the audit log is detectable by comparing the current chain against the externally anchored checkpoints. This does not prevent a sufficiently motivated and coordinated insider attack — it makes one detectable and therefore evidentiable.

---

**Centralization of the issuer key registry** — until federated mirroring is live (Phase 2), Credora's key registry is a single point of failure. Registry unavailability means verification fails globally. Registry compromise means attacker could substitute issuer public keys. Mitigated by the CT-style mirrored log architecture allowing verifiers to operate from local mirrors during outages. Moving federated mirroring to Phase 2 rather than Phase 4 directly addresses this — it is on the critical path, not a nice-to-have.

---

**AI-generated portfolios** — synthetic GitHub histories, fabricated projects, and AI-written codebases are increasingly indistinguishable from genuine work. Credora verifies provenance, attribution, issuance integrity, and continuity. It does not verify that work attributed to a person was produced by them. This problem will grow significantly. Credora does not claim otherwise.

---

**Recruiter bias amplification** — verification confidence used as a proxy for talent creates systematic disadvantage for self-taught developers, career switchers, and candidates from smaller institutions. Mitigated by fairness disclosures, anti-filtering guardrails, and platform policy. Credora cannot prevent biased decisions — it is designed to not make them easier.

---

**Jurisdiction conflicts** — digital signatures and identity proofs are treated differently across the EU, India, and the US. Mitigated by regional compliance layers, local legal entities, and country-specific KYC providers per market. Credora does not expand to a new market before local legal review is complete.

---

**The fundamental limit** — verified credentials represent verified claims, not verified competence. A person can be genuinely certified, genuinely employed, genuinely verified — and still wrong for the role.

Credora does not verify competence, creativity, judgment, culture fit, or character. It verifies one thing: whether a credential is genuine.

> Credora removes one specific obstacle from hiring: not knowing whether credentials are real.
> That is a valuable and bounded goal.
> The rest of hiring remains the recruiter's job.

---

## Technology Stack

| Layer | Stack |
|---|---|
| Frontend | React, Next.js, Tailwind CSS, Framer Motion |
| Backend | Node.js, FastAPI |
| Primary database | PostgreSQL |
| Caching / sessions | Redis |
| Auth | OAuth2, Passkeys, MFA, Biometric |
| Credential format | W3C Verifiable Credentials 1.1 (Ed25519 / RFC 8037) |
| Export formats | W3C VC JSON-LD, OpenBadges 3.0, Signed PDF |
| Timestamping | RFC 3161 trusted timestamping |
| Audit anchoring | AWS Glacier Vault Lock / Azure Immutable Blob (WORM) |
| KYC adapters | Aadhaar, DigiLocker, Passport, Persona, Onfido (extensible) |
| Blockchain | None |

---

## Business Model

### Revenue Streams

**Issuer SaaS** — primary revenue, closes in days to weeks for bootcamps.

| Tier | Target | Price |
|---|---|---|
| Starter | Bootcamps, certification platforms | $99–299/month |
| Growth | Mid-size institutions | $499–1,499/month |
| Enterprise | Universities, large organizations | Custom |

**Verifier API** — pay-per-call, no contract required. Scales with hiring volume.

**Holder Premium** — recruiter analytics, unlimited profiles, advanced permission controls, privacy-preserving verification mode. $8–15/month.

**Enterprise Compliance Tier** — extended audit trails, WORM anchoring reporting, dedicated SLAs, custom KYC integrations, priority incident response. For regulated industries.

**Premium Physical Card** — one-time purchase. Revenue-generating, not strategically central.

### Sequencing

Bootcamp SaaS closes in weeks. Verifier API requires no contract. Holder premium is immediate. Enterprise contracts are upside — not survival dependency. University integrations use bootcamp deployments as proof of concept, shortening their procurement cycles.

---

## Geographic Expansion

Credora launches in India. Deliberately.

India has the right conditions for v1.0: PAN as a government-backed identity anchor already linked to employment history, the largest volume of engineering graduates globally, a well-documented credential fraud problem in fresher hiring, DigiLocker and Aadhaar APIs available, and a bootcamp and certification platform ecosystem that is digitally native with short procurement cycles.

Expansion follows one principle: **does a government-backed identity anchor exist that enables PAN-equivalent verification in that market?**

| Market | Identity Anchor | Hiring Market | Regulatory Complexity | Version |
|---|---|---|---|---|
| India | PAN + Aadhaar | Very High | Medium | v1.0 |
| Singapore | SingPass | High | Low | v2.0 |
| Malaysia | MyKad | Medium | Low | v2.0 |
| UAE | Emirates ID | High | Low | v3.0 |
| UK | NIN | High | Medium | v3.0 |
| US | SSN (restricted) | Very High | Very High | v4.0 |
| EU | eIDAS (fragmented) | High | Very High | v4.0 |

The US enters last — not because the market is small, but because SSN cannot be used as a verification anchor the way PAN can. US employment verification is a patchwork of state laws, FCRA requirements, and background check regulations requiring a dedicated legal and compliance team to navigate. Entering before product-market fit is established burns capital on compliance with no return.

The UAE and Singapore expansions are prioritized because both have large Indian professional populations already familiar with Credora from v1.0 — natural network-effect expansion rather than cold market entry.

---

## Roadmap

### v1.0 — India: Verification Core *(current)*

**Market:** India
**Identity anchor:** PAN + Aadhaar + DigiLocker
**Pricing:** INR

- Credential issuance API (W3C VC 1.1 compliant)
- Controlled Access Profile links with privacy-preserving verification mode
- Signature verification endpoint
- Revocation system: webhooks + fallback polling endpoint
- `Cache-Control` header enforcement on all verification responses
- Verifier pay-per-call API
- Holder dashboard with view analytics and Credora Points breakdown
- Issuer dashboard with bulk issuance and key rotation tooling
- KYC adapter: PAN, Aadhaar, DigiLocker, Passport
- KYC cross-provider deduplication (salted HMAC)
- Issuer key liveness challenge system
- SOC 2 Type II requirement enforced at issuer onboarding
- Daily WORM audit log anchoring (AWS Glacier Vault Lock)
- Verifier anomaly detection (high-frequency holder checks)
- Credora Points (PAN-anchored, India)

Target: 10–20 Indian bootcamp and certification platform partners.

---

### v2.0 — Trust Infrastructure + Southeast Asia

**New markets:** Singapore (SingPass), Malaysia (MyKad)
**KYC adapters added:** SingPass, MyKad

- CT-style mirrored issuer key registry (federated — moved from Phase 4)
- Offline verification support via local registry mirrors
- Issuer Reliability Indicators with KYC confidence scores
- Explainable skill matching
- ATS integrations (Greenhouse, Lever, Workday)
- Freelancer multi-source verification with payment processor requirement
- Dispute and governance framework
- Fairness disclosures and anti-filtering guardrails
- Additional KYC adapters (Persona, Onfido, SingPass, MyKad)
- Credora Points extended to Singapore and Malaysia

Target: First Indian university partnerships. Southeast Asia bootcamp partners. Key registry no longer a single point of failure.

---

### v3.0 — Identity Ecosystem + Gulf + UK

**New markets:** UAE (Emirates ID), UK (NIN)
**KYC adapters added:** Emirates ID, NIN

- NFC Identity Card
- Institutional HSM onboarding tooling and rotation workflows
- W3C VC and OpenBadges full export compatibility
- International KYC adapter expansion (Emirates ID, NIN)
- Annual transparency report infrastructure
- Extended audit trail tooling for regulated industries
- Zero-knowledge proof verification mode (cryptographic privacy option)
- Credora Points extended to UAE and UK

Target: Enterprise contracts. Gulf and UK market entry via Indian diaspora network effects.

---

### v4.0 — Verification Network + US + EU

**New markets:** United States, European Union
**KYC adapters added:** US third-party identity verification (no SSN direct use), eIDAS-compatible providers

- Cross-platform credential interoperability
- Issuer key registry as open industry standard
- Credora embedded in major hiring platforms as invisible infrastructure
- US FCRA-compliant verification workflows
- EU eIDAS-compatible credential format
- Credora Points extended to US and EU with jurisdiction-specific anchor adapters

Target: The issuer registry is the moat. Hiring platforms verify against it automatically. Credora is the layer nobody thinks about — and that hiring cannot function without.

---

## Contributing & Integration

### Issuer Integration

Institutions integrate via REST API or dashboard. Full API documentation at `docs.credora.io/issuers`.

Key endpoints:

```
POST /v1/credentials/issue
POST /v1/credentials/revoke
GET  /v1/credentials/{id}/status
GET  /v1/issuer/audit-log
POST /v1/issuer/liveness-response
```

### Verifier Integration

Recruiters and platforms verify via API. No contract required for pay-per-call.

```
GET  /v1/verify/{credential_id}
GET  /v1/verify/{credential_id}?mode=privacy-preserving
GET  /v1/revocation-status/{credential_id}
GET  /v1/profile/{access_token}
POST /v1/webhooks/revocation
```

### Standards Compliance

- W3C Verifiable Credentials Data Model 1.1
- OpenBadges 3.0
- Ed25519 signing (RFC 8037)
- RFC 3161 trusted timestamping
- GDPR Article 17 (right to erasure) compliant by design
- India DPDP Act Section 13 compliant by design
- CCPA compliant by design
- eIDAS compatible

---

*Credora. Verified people. Trusted opportunities.*
