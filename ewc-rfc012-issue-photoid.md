# EWC RFC012: Issuing Photo ID Verifiable Credential - v1.0

**Authors:**

* Matteo Marangoni, SICPA, Spain
* Xavi Vila, SICPA, Spain
* Mr. Daniele Ribaudo, Intesi Group, Italy

**Reviewers:**

* Viky Manaila, Intesi Group, Italy

**Status:** Draft v1.0

Copyright © 2025 All Rights Reserved  
Published under a Creative Commons Attribution 4.0 International License

---

## **Table of Contents**
- [1.0 Summary](#10-summary)
- [2.0 Motivation](#20-motivation)
- [3.0 Pre-requisites](#30-pre-requisites)
- [4.0 Issuance Flow](#40-issuance-flow)
- [5.0 Verifiable Credential Type](#50-verifiable-credential-type)
- [6.0 References](#60-references)

---

## **1.0 Summary**
This specification defines the implementation of the "Photo ID profile" defined in Annex C of ISO/IEC TS 23220-4, in mdoc and SD-JWT. The attestation issued is derived from an **electronic Machine-Readable Travel Document (eMRTD)** by a Qualified Trust Service Provider (QTP).

The issuance process includes eID verification of an attestation issued with high level of assurance (LoA High), such as a **Personal ID (PID) credential** and may include additional steps, such as biometric checks. The issuance follows the **OpenID4VC** framework, ensuring interoperability with **EUDI Wallets**.

### **Key Benefits:**

- **Enhancing digital identity verification** while maintaining strong security and privacy guarantees.
- **Facilitating travel and remote identity proofing**, reducing the need for physical document checks.
- **Complying with international standards** such as **SD-JWT**, and **mDoc (ISO/IEC TS 23220-4 Annex C, 2024-08-14)**.
- **Enabling selective disclosure**, allowing users to share only necessary attributes (e.g., verifying age without revealing full birthdate), enhancing privacy and data minimization.

---

## **2.0 Motivation**

A **Photo ID** is a document type used for identity documents that perform the function of a photo-based eID. In practical terms, a **Photo ID attestation** provides a **digital representation of a passport** that can be used for **identity verification** in various online and offline use cases where the biometric verification of the photo is required.

There are 2 possible scenarios for issuing a Photo ID attestation:

1. **Direct issuance by Passport issuing authority**
    - The **Issuance Authority** has all the necessary data required to issue a valid photo ID credential.
    - The **Issuance Authority** performs the required identity verification before issuing the attestation to the citizen (this might include remote identity proofing, authentication using PID or in-person verification).

2. **Issuance by a Qualified Trust Service Provider**  
    - The QTSP requests the required data to the citizen at the moment of issuing the Photo ID attestation.
    - The QTSP performs the required identity verification before issuing the attestation to the citizen (this might include remote identity proofing, authentication using PID or in-person verification).

In this RFC we will be focusing on the **scenario 2)**, where the QTSP will:

- Request the required the electronic Machine-Readable Travel Document (eMRTD) data to the citizen at the moment of issuing the Photo ID attestation.
- Authenticate the citizen's identity using EUDI Wallet and **PID**.
- Perform an attestation-binding, using attributes from both documents like `name` and `date of birth`.


---

## **3.0 Pre-requisites**

In the scope of this RFC, the following conditions must be met before issuing a **Photo ID EAA**:

1. **Personal ID Verification**  
   - The user must present a valid **Personal ID (PID)** issued by a trusted authority.
   
2. **Passport Scan**  
   - The user must scan the **MRZ** or **NFC chip** of their passport.

3. **Liveness Check (Optional)**  
   - The system may require a **selfie** or a **short video** to detect liveness and compare it against the document portrait.

4. **EUDI Wallet Support**  
   - The wallet requesting the credential must support **OpenID4VCI** and **ISO/IEC 18013-5** display properties.

---

## **4.0 Issuance Flow**
The issuance process follows **OpenID4VCI**, ensuring a standardized method for **credential issuance**.

1. **Credential Offer:**  
   - The issuer sends a credential offer to the wallet.
   
2. **Credential Request:**  
   - The wallet requests the **Photo ID Verifiable Credential**, submitting the necessary identity verification data.

3. **Verification Process:**  
   - The issuer verifies the provided **PID credential**, **passport scan (MRZ/NFC)**, and optionally performs **liveness checks**.

4. **Credential Issuance:**  
   - If successful, the issuer signs and issues the **Photo ID Verifiable Credential**, which the wallet stores.

---

## **5.0 Verifiable Credential Type**
The credential follows:
- **SD-JWT format**, as defined in **[`ds013-photo-id.json`](https://github.com/EWC-consortium/eudi-wallet-rulebooks-and-schemas/blob/main/data-schemas/ds013-photo-id.json)**.
- **mDoc format**, as specified in **ISO/IEC TS 23220-4 Annex C (2024-08-14)**.

---

## **6.0 References**
1. **EUDI Wallet JSON Schema**: [`ds013-photo-id.json`](https://github.com/EWC-consortium/eudi-wallet-rulebooks-and-schemas/blob/main/data-schemas/ds013-photo-id.json)
2. **ISO/IEC TS 23220-4 (E) Annex C** *(2024-08-14)*: Defines the **mDoc format** for Photo ID.
3. **ISO/IEC 18013-5**: Specifies **mobile driving licenses and digital identity display properties**.
4. **OpenID4VCI**: [Draft Specification](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html).

---
