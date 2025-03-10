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
This specification defines the issuance process of a **Photo ID Verifiable Credential**. The credential is derived from a passport’s **Machine-Readable Zone (MRZ)** or **NFC chip**.

The issuance process may include additional steps, such as biometric checks or verifying a credential issued with high level of assurance (LOA HIGH), such as a **Personal ID (PID) credential**. The issuance follows the **OpenID4VCI** framework, ensuring interoperability with **EUDI Wallets**.

---

## **2.0 Motivation**

A **Photo ID Verifiable Credential** provides a **digital representation** of a passport that can be used for **identity verification** in various online and offline use cases. 

### **Key Benefits:**

- **Enhancing digital identity verification** while maintaining strong security and privacy guarantees.
- **Facilitating travel and remote identity proofing**, reducing the need for physical document checks.
- **Complying with international standards** such as **SD-JWT**, and **mDoc (ISO/IEC TS 23220-4 Annex C, 2024-08-14)**.
- **Enabling selective disclosure**, allowing users to share only necessary attributes (e.g., verifying age without revealing full birthdate), enhancing privacy and data minimization.

---

## **3.0 Pre-requisites**

The following conditions must be met before issuing a **Photo ID Verifiable Credential**:

1. **Personal ID Verification**  
   - The user must present a valid **Personal ID (PID) credential** issued by a trusted authority.
   
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
