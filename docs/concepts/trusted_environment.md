# OpenDID Trusted Environment

- Subject: OpenDID Trusted Environment  
- Author: Open Source Development Team  
- Date: 2025-06-23  
- Version: v2.0.0

| Version | Date       | Changes   |
|---------|------------|----------------------|
| v2.0.0  | 2025-06-23 | Initial version        |

<br>

## Overview of Trusted Environment

- The OpenDID system is designed based on **Zero Trust principles** to establish **mutual trust** among participants.
- A **decentralized trust chain** is formed, centered around the **TAS (Trust Agent Service)**.
- Anyone can **verify the authenticity** of another participant through their **DID Document**.
- By utilizing a **blockchain-based storage**, the system prevents data tampering and ensures transparent traceability.

---

<br>

## Architecture of Trusted Environment

![Trusted Environment Architecture](images/trusted_environment_architecture.svg)

- **Trust Chain Composition**: The environment is formed with consensus from the Committee.
- **Trust Repository → Authority delegated to TAS**
- **Each Entity → Requests registration of its DID Document**
- **DID Document includes public key, signature, and core identity data**
- **Verification of DID Document → Establishes mutual trust**
- **Membership credential issued → Officially proves entity trust**

---

<br>

## Key Components of the Trusted Environment

| Component              | Description |
|------------------------|-------------|
| **Trust Repository**    | Established by the Committee. A decentralized storage system such as blockchain for permanent, tamper-resistant document storage. |
| **TAS**                 | Trust administrator responsible for entity registration and management. |
| **DID Document**        | Trusted document containing the entity’s DID, public key, signature, and related metadata. |
| **Membership Credential** | Official credential certifying the entity’s authority and trustworthiness. |
| **Entity Registration & Verification** | Process where issuers, verifiers, apps, and other entities are onboarded and verified. |
| **Zero Trust Principle** | Every request is validated; no default trust is assumed. |
| **Digital Signatures & Encryption** | Core technology to ensure data authenticity and integrity. |

> OpenDID’s Trusted Environment builds a **tamper-proof digital trust ecosystem** where all participants can verify one another transparently.

---

<br>

## Trust Environment Establishment Procedure

### 1. TAS Registration
- The TAS acts as a **core authority** for trust registration and management.
- It is registered to the **top-level trust repository**, forming the root of the trust chain.

**Process:**
- TAS requests a **DID Document attestation** from the Committee.  
- The Committee generates the **attestation** for the TAS’s DID Document.  
- Upon registration, a **membership credential** is issued.

---

### 2. Entity Registration
- Each participant requests TAS to register its DID Document and obtain appropriate permissions.
- TAS manages entity onboarding based on **role-based access control**.

**Process:**
- Entity requests **DID Document registration** to the TAS.  
- TAS completes the registration and issues a **membership credential**.

---

### 3. Membership Credential Issuance
- Once registered, the TAS issues a **credential** to certify the entity’s trust and authority.
- This credential serves as an **official proof** of the entity’s identity and role in the system.

---

<br>

## Notes

- All registered DID Documents and credentials are stored in the trust repository with **public visibility and tamper resistance**.
- All interactions in the system are verified using **digital signatures** for authenticity.