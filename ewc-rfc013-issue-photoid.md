# EWC RFC013: Issuing Photo ID Verifiable Credential - v1.0

**Authors:**

* Matteo Marangoni, SICPA, Spain
* Xavi Vila, SICPA, Spain
* Dimitroglo, Veaceslav, SICPA, Spain
* Mr. Daniele Ribaudo, Intesi Group, Italy

**Reviewers:**

* Viky Manaila, Intesi Group, Italy

**Status:** Draft v1.0

Copyright © 2025 All Rights Reserved  
Published under a Creative Commons Attribution 4.0 International License

---

## **Table of Contents**
- [1 Summary](#1-summary)
- [2 Motivation](#2-motivation)
- [3 Scope](#3-scope)
- [4 Pre-requisites](#4-pre-requisites)
- [5 Issuance Flow](#5-issuance-flow)
  - [5.1 Actors](#51-actors)
  - [5.2 Flow Details](#52-flow-details)
    - [5.2.1 Setup Phase](#521-setup-phase)
    - [5.2.2 Credential Issuance Request (OpenID4VCI)](#522-credential-issuance-request-openid4vci)
    - [5.2.3 Dynamic Credential Request - Step 1: PID Validation](#523-dynamic-credential-request---step-1-pid-validation)
    - [5.2.4 Dynamic Credential Request - Step 2: Passport Validation](#524-dynamic-credential-request---step-2-passport-validation)
- [6 Verifiable Credential Type](#6-verifiable-credential-type)
- [7 References](#7-references)

---

## **1 Summary**
This specification defines the implementation of the "Photo ID profile" defined in Annex C of ISO/IEC TS 23220-4, in mdoc and SD-JWT. The attestation issued is derived from an **electronic Machine-Readable Travel Document (eMRTD)** by a Qualified Trust Service Provider (QTSP).

The issuance process includes verifying an eID attestation issued with a high Level of Assurance (LoA High), such as a **Personal ID (PID) credential** and may include additional steps, such as biometric checks. The issuance follows the **OpenID4VC** framework, ensuring interoperability with **EUDI Wallets**.

### **Key Benefits:**

- **Enhancing digital identity verification** while maintaining strong security and privacy guarantees.
- **Facilitating travel and remote identity proofing**, reducing the need for physical document checks.
- **Complying with international standards** such as **SD-JWT**, and **mDoc (ISO/IEC TS 23220)**.
- **Enabling selective disclosure**, allowing users to share only necessary attributes (e.g., verifying age without revealing full birthdate), enhancing privacy and data minimization.


## 2 Motivation

### 2.1 The need of a photo ID

The need for a photo ID arises from the limitations of the current Personal ID (PID) issued within the European Digital Identity Wallet (EUDI Wallet) ecosystem. Specifically, PIDs may not always include a photo, which is a required attribute in certain use cases where biometric verification is necessary or a physical presence of the individual is required. Furthermore, some use cases, such as traveling, require specific documentation (e.g., passport) that includes a document number, which is not included in the PID schema. Therefore, a separate photo ID attestation is necessary to fulfill these requirements.

### 2.2 ISO/IEC TS 23220 and Photo ID Issuance Process

The ISO/IEC TS 23220 series provides a comprehensive framework for identity management via mobile devices. Specifically, in ISO/IEC TS 23220-2, the standard outlines the data model and encoding rules required for generic eID systems, which forms the foundation for our Photo ID implementation.

Section 5 of ISO/IEC TS 23220-2 describes the identity data collection and confirmation process, highlighting how "ID documents are issued by binding an applicant with a real-life identity." The standard emphasizes that "an issuer collects evidence to verify the attributes provided by the applicant, and this process is called identity proofing." This corresponds directly to the identity verification procedures an issuer will perform when issuing Photo ID attestations.

![identity-data-collection-and-issuance](images/identity-data-collection-and-issuance.png)

ISO/IEC TS 23220-2 Figure 1 illustrates the issuing process of an eID document, showing how an applicant provides an application form and evidence (such as ID cards issued by an Authority) to the issuer. The issuer then collects other evidence if needed, proves the applicant's identity, binds that identity with the holder, and confirms the applicant through photo ID or by person of authority.

Furthermore, Section 6 of ISO/IEC TS 23220-2 defines the data model specification that we will implement for the Photo ID, including the meta attributes for person entity (Section 6.3.1) which specifies the data elements that express attributes for describing a natural person, including crucial elements like family name, given names, date of birth, portrait, and biometric templates.

### 2.3 ETSI TS 119 461 and Identity Proofing Requirements

The identity proofing and verification procedures mentioned above are further elaborated and standardized in ETSI TS 119 461, which specifies "policy and security requirements for trust service components providing identity proofing of trust service subjects." This standard is particularly relevant to our implementation as it defines the requirements for identity proofing at two levels: Baseline and Extended Level of Identity Proofing (LoIP).

The Baseline level of assurance is sufficient for use cases where the risk of identity fraud is low, such as online services that require a basic level of identity verification. In contrast, the Extended level of assurance could be required for use cases where the risk of identity fraud is higher, such as online services that require a high level of identity verification, or where the user is requesting access to sensitive information.

Section 9 of ETSI TS 119 461 defines various use cases for identity proofing, including:

* Physical presence (Section 9.2.1)
* Attended remote identity proofing using identity documents (Section 9.2.2)
* Unattended remote identity proofing using identity documents (Section 9.2.3)
* Identity proofing using eID means (Section 9.2.4)
* Identity proofing using digital signature with certificate (Section 9.2.5)

In our implementation, the passport validation step described in Section 4.2.4 of this RFC will follow the unattended remote identity proofing requirements specified in Section 9.2.3 of ETSI TS 119 461. This includes requirements related to automated validation of digital identity documents (VAL-8.3.2 requirements) and binding to applicant by automated face biometrics (BIN-8.4.3 requirements).

By adhering to both the ISO/IEC TS 23220 data model and the ETSI TS 119 461 identity proofing requirements, our Photo ID issuance process ensures a high level of interoperability within the European identity ecosystem.

## 3 Scope

We consider 2 possible scenarios when issuing a Photo ID attestation:

1. The **Issuer** has all the necessary data required to issue a valid photo ID credential. This **Issuance Authority** performs the required identity proofing before issuing the attestation to the citizen (this might include remote identity proofing, authentication using PID or in-person verification).

2. **Issuer** requests the required data from the citizen at the moment of issuing the photo ID attestation. In this case the Issuer (e.g. a **QTSP**) performs the required identity verification before issuing the attestation to the citizen.

In this RFC we will be focusing on the **scenario 2)**.

### 3.1 Photo ID as an Electronic Attestation of Attributes

Depending on the recognition needs and handling within the digital identity ecosystem, Photo ID could be issued as **Qualified** or as **non-Qualified** attestation of attributes as per eIDAS regulation.

* Non-qualified attestations, on the other hand, are issued by a broader range of providers, operate under potentially diverse legal and contractual rules, and their trustworthiness and recognition depend on the specific context and agreements in place.
* Qualified attestations are issued by accredited entities, adhere to specific legal and technical standards ensuring a high level of trust and legal validity, and are integrated within a formal trust framework with trusted lists. 

## 4 Pre-requisites

### 4.1 Photo ID as a EAA

// TODO 

### 4.2 Photo ID as a QEAA

* **Issuer**: QEAA must be issued by a Qualified Trust Service Provider (QTSP).
* **Identity Proofing**: The QTSP must verify the identity of the natural person to whom the QEAA is to be issued.
* **Attribute Verification**: The QTSP must ensure with complete certainty the correctness and accuracy of the attested attributes of the person to whom the QEAA is issued. 

Annex C of the ETSI TS 119 461 specifies requirements for identity proofing targeted explicitly to fulfil requirements of the amended eIDAS regulation for issuing of qualified certificates and qualified electronic attestation of attributes.

In this RFC, 2 identity proofing use cases (of the Annex C.3 - Use cases for issuing of QEAA) are considered:

- ID proofing with PID: Use case for identity proofing by authentication using eID means (C.3.2):

  - Requires Extended LoIP.
  - Requires a notified eID with level **high** (EUDI Wallet with PID).
  - The other attributes would be obtained from the Machine-Readable Travel Document (eMRTD) data, authenticated by the SOD signature.
  - Perform user binding verification by comparing attributes from the PID such as first and last name and date of birth, to those in the identity document.

- ID proofing without PID: Use case for identity proofing by other identification means (C.3.4):

  - Perform an unattended remote identity proofing using a digital identity document (eMRTD).
  - Requires Extended LoIP.
  - The ID proofing process must follow the unattended remote identity proofing requirements specified in Section 9.2.3 of ETSI TS 119 461. This includes requirements related to automated validation of digital identity documents and binding to applicant by automated face biometrics.



## **5 Issuance Flow**

The issuance process follows **OpenID4VCI** Authorisation Code flow as described in the [EWC RFC001: Issue Verifiable Credential - v2.0](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc001-issue-verifiable-credential.md), and extends it by implementing Dynamic Credential Request (OpenID4VCI draft 13), ensuring a standardized method for **credential issuance**.

It follows the OpenID4VC High Assurance Interop profile, where it is recommended that the Credential Issuer and the Wallet MUST support the authorization code flow. So no pre-authorized code flow is used.

### High-level flow diagram.

This diagram provides a high-level overview of the credential issuance process, highlighting the main steps involved.

```mermaid
sequenceDiagram

    participant W as EUDI Wallet
    participant IS as PhotoId Issuer(QTSP)
    participant PS as Passport Scanner App
    
    Note over W,IS: Setup

    W->>IS: Discover Request
    IS-->>W: Discover Response

   
    Note over W,IS: Issuance request (OpenID4VCI)
    W->>+IS: Auth Request 
    
    Note over W,IS: Dynamic credential request: 1) PID
    IS->>+W: Request PID 
    W-->>-IS: Present PID
    IS-->>IS: Check if PID is valid
    
    Note over PS,IS: Dynamic credential request: 2) Passport
    IS->>+PS: Passport scan 
    PS-->>-IS: Passport data
    IS-->>IS: Match data with PID presentation
    IS-->>-W: Auth Response 

    Note over W,IS: Complete Issuance  
    W->>+IS: Credential Request
    IS-->>-W: Credential Response

    IS->>IS: Evict User Data
```

### Detailed flow diagram

This diagram provides a more detailed breakdown of the credential issuance process, highlighting the individual steps and interactions involved.

```mermaid
sequenceDiagram
    participant UA as USER's User Agent
    participant W as EUDI Wallet
    participant IS as PhotoId Issuer(QTSP)
    participant QS as Qualified Seal (QTSP)
    participant PS as Passport Scanner App
    participant ES as Passport Scanner BE <br> (External Source)
    Note over W,IS: Setup

    W->>IS: Discover Request
    IS-->>W: Discover Response


    Note over W,IS: Issuance request (OpenID4VCI)
    W->>UA: Open
    UA->>+IS: Authorisation Request

    Note over W,IS: Dynamic credential request: 1) PID
    IS->>UA: PID Presentation Authorisation Request
    UA->>W: Open
    W->>IS: PID Presentation Authorisation Response
    IS-->>W: Direct Post Response
    W->>UA: Open
    UA->>IS: Follow Redirect
    IS->>IS: Check if PID is valid

    Note over UA,ES: Scan Passport And get claims for PhotoId:
    IS->>UA: Open
    UA->>+ES: Authorization Request
    ES->>UA: Passport Scan Request
    UA->>PS: Open

    PS->>PS: Scan passport
    PS->>ES: Scanned Data
    PS->>UA: Open
    UA->>ES: Follow Redirect
    ES-->>UA: Authorization Response
    UA->>IS: Open
    IS->>ES: Token Request
    ES-->>IS: Opaque Token Response
    IS->>ES: Get Claims from userinfo via Access Token
    ES-->>-IS: Claims Response
    IS->>IS: Store data temporarily
    IS->>IS: Match data with PID presentation


    IS-->>-UA: Authorisation Response
    UA->>W: Open

    Note over W,QS: Complete Issuance

    W->>+IS: Token Request
    IS-->>-W: Token Response

    W->>+IS: Credential Request
    IS->>+QS: Seal Attestation
    QS-->>-IS: Attestation sealed
    IS-->>-W: Credential Response

    IS->>IS: Evict User Data
```

## 5.1 Actors

| Actor | Description |
|--------|------------|
| **UA (User Agent)** | The user's browser used to interact with different entities. |
| **W (EUDI Wallet)** | The user's digital wallet storing credentials. |
| **IS (PhotoID Issuer - QTSP)** | A Qualified Trust Service Provider (QTSP) issuing the attestations. |
| **QS (Qualified Seal - QTSP)** | The QTSP responsible for digitally sealing/verifying the issued attestations. |
| **PS (Passport Scanner App)** | The app used to scan the user's passport and extract relevant data. |
| **ES (External Source - Passport Scanner BE)** | A backend service for processing and validating scanned passport data. |

---

## 5.2 Flow Details

### 5.2.1. Setup Phase
Before any attestations are issued, the **EUDI Wallet (W)** and the **PhotoID Issuer (IS)** establish a connection.

1. **Wallet (W) discovers Issuer (IS)**
   - **W → IS:** Discover Request
   - **IS → W:** Discover Response

   The Wallet and Issuer exchange metadata to establish connection endpoints and cryptographic parameters necessary for secure communication. This typically includes OpenID discovery mechanisms.

### 5.2.2. Credential Issuance Request (OpenID4VCI)

The issuance process starts when the user (UA) scans an OpenID4VCI credential offer QR or clicks on a deeplink.
**Wallet** will invoke **UA (User Agent)** to send Authorisation Request to the **Authorization Server (QTSP ISSUER)**.

```json
//TODO add credential offer example
```

After receiving Authorisation Request the **IS (PhotoID Issuer - QTSP)** will start the dynamic credential request to obtain the required data ( PID and Passport) to obtain PhotoID attestation.

```json
//TODO add Authorisation Request example
```

Dynamic credential request will consist of two steps, described in the following sections.

### 5.2.3. Dynamic Credential Request – Option A: ID proofing with PID 

#### Step 1: User authentication with PID

First steps will use OPENID4VP – as described in [EWC RFC002: Present Verifiable Credentials - v1.0
](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md) to request and verify the user’s PID.

```json
//TODO add Authorisation Request example with redirect uri
```

#### Step 2: Passport attributes Validation

After PID validation, the second step of the **Dynamic Credential Request** triggers an OAuth2-like flow to obtain Passport data.

```json
//TODO add redirect request example
```

The user will be redirected to a Passport Reading Service application and prompted to scan their Passport alongside additional biometric checks to ensure the holder of the physical document is managing the application. Finally, the user will grant permission to share Passport data with the issuer.

### 5.2.4. Dynamic Credential Request - Option B: Unattended remote ID Proofing using eMRTD 

### 5.2.5. Credential Issuance Completion

Issuer performs an attribute-based verification to ensure PID data corresponds and matches a subset of the Passport’s claims. 
Once identity is verified, the **attestation is issued and sealed**.

## **6.0 Electronic Attestation Type**
The attestation is issued in one of the follows:
- **SD-JWT format**, as defined in **[`ds013-photo-id.json`](https://github.com/EWC-consortium/eudi-wallet-rulebooks-and-schemas/blob/main/data-schemas/ds013-photo-id.json)**.
- **mDoc format**, as specified in **ISO/IEC TS 23220-4 Annex C (2024-08-14)**.

## **7.0 References**
1. **EUDI Wallet JSON Schema**: [`ds013-photo-id.json`](https://github.com/EWC-consortium/eudi-wallet-rulebooks-and-schemas/blob/main/data-schemas/ds013-photo-id.json)
2. **ISO/IEC TS 23220-4 (E) Annex C** *(2024-08-14)*: Defines the **mDoc format** for Photo ID.
3. **ISO/IEC 18013-5**: Specifies **mobile driving licenses and digital identity display properties**.
4. **OpenID4VCI**: [Draft Specification](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html).

