# Organisational Wallet Protocol and Format Discussion

**Authors:** 
* Dr Carsten Stöcker (Spherity, Germany)
* Mr Ronald Koenig (Spherity, Germany)
* Dr Martin Westerkamp  (Spherity, Germany)

## 1. Introduction

Organizational EUDI Wallets, also known as Legal Person Wallets, are digital wallets designed specifically for organizations, including companies, NGOs, and government entities. These wallets serve as digital counterparts to traditional organizational credentials, facilitating secure, efficient, and automated processing and exchange of credentials within and across organizations. Unlike Citizen Wallets, which are intended for individuals to manage their personal credentials and interact with services, Organizational Wallets are tailored to meet the unique needs of legal entities in various industry sectors. As a result, protocols and formats are required to meet these specific requirements, as presented in [EWC RFC010: Issue and Present Organisational Credentials](https://github.com/spherity/eudi-wallet-rfcs/blob/rfc-010/ewc-rfc010-issue-and-present-organisational-credentials.md).

In the following we will describe the different requirements of Citizen and Organizational Wallets, discuss candidate protocols and formats, and propose a selection based on the requirements.
We will also briefly mention the preferred credential format and protocol technology choises of the *US Department of Homeland Security (DHS)* as an important reference and benchmark for decisions regarding the EUDI wallet for organizational use cases. 

### 1.1 Context

The Architecture Reference Framework (ARF) [1] and Large Scale Pilots (LSPs) [2] have largely focused on solutions for citizen identity, particularly for individuals using smartphones. While these efforts have advanced Citizen EUDI Wallets with focus on identity cards, driving liences and Gym memberships, they do not meet the needs of Organizational EUDI Wallets.

Organizational EUDI Wallets operate in environments that differ significantly from those of Citizen Wallets. These wallets need to integrate with corporate IT systems, support 24x7 automation, and handle complex data models for global use cases like supply chain management, critical infrastructures and digital product passports. Unlike Citizen Wallets, which are often focused on national or EU-wide applications, Organizational Wallets frequently need to align with global and industry standards and operate across international borders.  

Current architecture decisions, with their focus on human-centric, mobile-based solutions, are not sufficient for the demands of Organizational Wallets. The protocols and data structures designed for mobile use do not support the complex, automated workflows required by organizations, nor do they offer the flexibility needed for large-scale, global operations.

There is now a need for protocols and formats that meet the requirements of Organizational EUDI Wallets. These protocols and formats shall ensure secure and reliable credential management, support complex data models, and provide interoperability with both EU and global standards. They must also integrate seamlessly with existing enterprise systems.

This document presents these protocols and formats, offering a rationale and recommendations to guide informed decision-making within the EWC LSP consortium. Our goal is to ensure that Organizational EUDI Wallets are robust, scalable, and capable of supporting secure, efficient, and compliant operations across various industries and regions.


### 1.2 Differences Between Organizational EUDI Wallets and Citizen Wallets

1. **User Base:**
   - **Citizen Wallets** are designed for individual use, allowing citizens to manage their personal identity data, access public and private services, and securely share credentials like diplomas, licenses, and ID cards.
   - **Organizational Wallets** are intended for legal entities, enabling members of the organization to manage a wide range of credentials related to business operations, such as business licenses, compliance certificates, power of attorney, and contracts. Therefore, Organizational Wallets must provide or interact with a user management system that allows members of the organisation to receive, store, and present credentials.

2. **Credential Types:**
   - **Citizen Wallets** primarily handle personal credentials (e.g., ID cards, diplomas, health records) that are used in interactions between individuals and service providers.
   - **Organizational Wallets** manage business-related credentials (e.g., person credentials, natural person credentials, power of attorney credentials,regulatory certifications, digital product passports, supply chain credentials) that are essential for organizational operations and compliance.

3. **Delegation of Authority:**
   - **Citizen Wallets** are generally managed by the individual user without delegation features.
   - **Organizational Wallets** often include the ability to delegate rights to various individuals within the organization, allowing them to act on behalf of the organization in the scope of the delegated rights. Signatory rights are expressed in a verifiable credential that includes a reference to the representatives' identifiers and scope of representation. This includes representative roles and signatory rules. In an organizational context, a verifiable credential can reference multiple identifiers managed by the Organizational Wallet. As a result, the credential is held by multiple individuals who can present the credential to relying parties using their associated cryptographic material.

4. **Use Cases:**
   - **Citizen Wallets** focus on individual-centric use cases, such as accessing government services, signing documents, or managing personal health data. Requesting and presenting credentials involves human interaction.
   - **Organizational Wallets** support complex, multi-party use cases across industries, such as automated credential exchange in supply chains, compliance reporting, digital product passport management, and KYC processes in financial markets.


## 2. Protocol and Format Alternatives

The purpose of this chapter is to evaluate potential credential exchange protocols and credential formats for implementing Organizational EUDI Wallets. These protocols and formats are critical for determining the wallet's efficiency, security, interoperability, and scalability, particularly in the context of Industry 4.0, Catena-X, Data Spaces, Digital Product Passports, and Supply Chain Use Cases.

- **Credential Formats:**
  - **SD-JWT VC (Selective Disclosure - JSON Web Token based Verifiable Credentials) [3]**
  - **W3C VCDM (Verifiable Credentials Data Model) [4]**

- **Credential Exchange Protocols:**
  - **OpenID4VC (OpenID for Verifiable Credentials) [5]**
  - **Decentralized Identity Foundation (DIF) Wallet and Credential Interaction (WACI) [6]**

It should be noted that another protocol receiving international attention is [W3C CCG VC-API](https://github.com/w3c-ccg/vc-api). This protocol, which is supported and partially funded by the **US Department of Homeland Security (DHS)**, is under active development and is a promising candidate for credential exchange in the near future. For simplicity, we focus on the most mature credential exchange protocols, DIF WACI and OID4VC, in this paper. However, we are actively monitoring the VC-API protocol as part of our ongoing research and development efforts for organizational EUDI Wallets. It offers advantages such as simplicity, standardization within the W3C community, and support from the US DHS. The US DHS also strongly supports **JSON-LD** as the preferred credential format for structured B2B or B2G use cases.

### 2.1 Credential Formats: SD-JWT vs W3C VCDM

**Credential Format** | **Option 1: SD-JWT VC** | **Option 2: W3C VCDM**
--- | --- | ---
**Data Format** | Plain JSON | JSON-LD
**Description** | SD-JWTs enable selective disclosure of credential information, allowing users to share only the necessary data. It is compact and integrates well with existing OAuth2 and OpenID Connect ecosystems. | The W3C VCDM provides increased flexibility by using JSON-LD to express complex data structures and data integrity proofs to enable cryptographic agility. JSON-LD provides a rich, semantic representation of credentials that supports linked data and interoperability across disparate systems. The VCDM is a W3C standard, making it ideal for scenarios requiring detailed and interconnected data structures.

**Selection of Architecture**

**Architecture** | **Pros** | **Cons**
--- | --- | ---
**Option 1: SD-JWT VC** | - **Selective Disclosure:** Allows users to share only specific parts of a credential, enhancing privacy.<br>- **Alignment with OAuth2/OpenID Connect:** Smooth integration with established enterprise security protocols.<br>- **Standardization:** Standardization process in well established Standarization Orgaization IETF ongoing. | - **Limited Expressiveness:** Less suitable for complex data structures or integration with linked data environments.<br>- **No Interoperability with Present Industry Schemas:** May be less effective for data spaces requiring rich, interoperable data.<br>- **Standardization:** Standardization still in draft state.
**Option 2: W3C VCDM** | - **Selective Disclosure:** Allows users to share only specific parts of a credential, enhancing privacy.<br>- **Rich Semantic Representation:** Ideal for complex data and integration with linked data environments, supporting detailed credentialing scenarios.<br>- **Interoperability with Linked Data Ecosystems:** Facilitates integration with the Semantic Web, enhancing interoperability across domains. Establishes interoperbility with existing credential ecosystems that use W3C VCDM such as Gaia-X, Catena-X, or the US DHS. <br>- **Extensibility:** Easily extended to support future standards and requirements.<br>- **W3C Standard:** Ensures broad acceptance and compliance with global web standards. | - **Performance Overhead:** Processing JSON-LD can introduce computational overhead, potentially affecting performance in high-throughput environments.<br>- **Complexity of Implementation:** More sophisticated handling required, particularly for context management and data integrity.

**Recommendations**

For scenarios requiring rich semantic data, such as Digital Product Passports and Data Spaces, **Option 2: W3C VCDM** is recommended due to its ability to handle complex data structures and integrate with linked data ecosystems. For environments where compactness and integration with existing OAuth2/OpenID Connect infrastructures are critical, **Option 1: SD-JWT VC** may be more appropriate.

---

### 2.2 Credential Exchange Protocols: OpenID4VC vs DIF WACI

**Credential Exchange Protocol** | **Option 1: OpenID4VC** | **Option 2: DIF WACI**
--- | --- | ---
**Description** | OpenID4VC builds on the OAuth2 and OpenID Connect frameworks, enabling verifiable credential issuance and verification within established enterprise security infrastructures. | DIF WACI is based on the DIDComm protocol and designed for decentralized identity systems, supporting secure, peer-to-peer communication and native integration of the W3C VCDM.

**Selection of Architecture Alternatives**

**Architecture** | **Pros** | **Cons**
--- | --- | ---
**Option 1: OpenID4VC** | - **Seamless Integration with OAuth2/OpenID Connect:** Leverages existing infrastructure for secure, verifiable credential exchange.<br>- **Established Security Protocols:** Proven security mechanisms, trusted in enterprise environments.<br>- **Interoperability with Existing Systems:** Easy adoption in organizations already using OAuth2/OpenID Connect. | - **Limited Decentralization:** More aligned with federated identity management, which may not suit all peer-to-peer scenarios.<br>- **Complexity in Implementation:** Integration with existing components can add complexity, particularly in managing different system interactions.<br>- **Overhead:** The protocol is designed for credential exchange using Citizen Wallets. Server-to-server communication requires overhead in the form of additional messages.<br>- **Availability:** The protocol does not provide persistent communication channels for automated secure communication.
**Option 2: DIF WACI** | - **Strong Decentralization:** Supports peer-to-peer communication, aligning with SSI principles and minimizing reliance on central authorities.<br>- **Rich Communication Protocol:** Flexible, adaptable, and capable of handling various use cases. Allows the implementation of authenticated & encrypted persitent communication channels. <br>- **Native Support for W3C Verifiable Credentials:** Ensures compatibility with decentralized, standard-compliant credentialing processes.<br>- **Privacy and Security:** Advanced privacy-preserving features, such as pairwise DIDs, enhance security. | - **Complexity in Deployment:** Requires a deeper understanding of decentralized identity concepts.<br>- **Evolving Standards:** As an emerging technology, there may be risks in stability and interoperability.<br>- **Interoperability Challenges:** Potential difficulties in integrating with legacy systems.

**Notes:** 
* It should be understood that OpenID4VC is an edge device centric protocol. Its viability and use for cloud-2-cloud communication has not been analysed, tested or standardised. This means that using the protocol for organisational identity requires significant effort, while offering less functionality and utility for the use cases discussed in this document.


**Recommendations**

For organisations that need to exchange data in a standardized and extensible fashion, **Option 2: W3C VCDM** provides enables building on present schemas and data models. Further, it provides future-proof crypographic agility. Lightweigt secure automated communication is enables by **Option 2: DIF WACI**.
**Option 1: OpenID4VC with sd-JWT VC** is preferable for organizations that already have a robust OAuth2/OpenID Connect infrastructure and require a more centralized, but secure, credential exchange protocol.

---

## 3. Protocol and Format Selection

The following table shows a decision matrix for selecting appropriate protocols and formats.


**Alternative Option** | **Decision** | **Rationale**
--- | --- | ---
**1. Credential Format** | Use **W3C VCDM** based on JSON-LD for scenarios requiring rich semantic data and integration with linked data environments. | **Rich Semantic Representation and Interoperability**: JSON-LD's ability to handle complex data structures and support linked data makes it ideal for advanced organizational use cases such as Digital Product Passports, KYC, KYS, Finance, and Data Spaces. JSON-LD also integrates well with existing generic and industry-specific schemas and vocabularies (e.g., LEI credentials, LPID, EUCC, ...). The cryptographic agility of the W3C VCDM allows the selection of appropriate proof schemes to issue Verifiable Credentials in a future-proof manner.
**2. Credential Exchange Protocol** | Use **DIF WACI** for decentralized and privacy-preserving interactions. | **Strong Decentralization and Privacy**: DIF WACI provides secure and persistent communication channels that enable autonomous credential exchange and natively support W3C Verifiable Credential.

This structured analysis helps in making informed decisions about the most suitable architecture for Organizational EUDI Wallets, ensuring alignment with the specific needs of various industry sectors, including high-regulated environments and critical infrastructures.


## 4. Protocol and Format Implications of ARF 1.4 and other Regulatory Documents

This chapter examines how the selected architecture alternatives for Organizational EUDI Wallets align with the European Digital Identity Wallet Architecture and Reference Framework (ARF) version 1.4 and other relevant regulatory documents. The goal is to demonstrate that the proposed solutions not only meet technical and operational requirements but also adhere to EU regulations and standards.

**1. Alignment with Credential Formats**

The ARF 1.4 emphasizes the importance of using internationally recognized standards for credential formats to ensure interoperability and security across EU member states [Org-ID-001]. The ARF endorses using the W3C VCDM, which is mentioned as a critical component for ensuring compatibility and extensibility across various applications [1, Annex 2.3.12]. The ARF highlights the need for formats that can support complex data structures, particularly for use cases that require detailed semantic data, making JSON-LD an ideal choice for organizational applications like supply chain management and digital product passports.

**2. Support for Secure and Interoperable Protocols**

ARF 1.4 stresses the necessity of secure and interoperable communication protocols to enable seamless credential exchange across the EU. While OpenID4VC is recognized for its strong integration with existing OAuth2/OpenID Connect frameworks, the ARF also acknowledges the need for more flexible protocols that can address the unique demands of different sectors, including organizational contexts. The selection of DIF WACI is in line with the ARF’s guidance on ensuring to meet industry requirements by defining protocols and formats in supplementary rulebooks. DIF WACI is particularly well-suited for environments that require secure, automated credential exchanges, which are essential in industries such as critical infrastructure and regulated sectors [Org-ID-004].

**3. Integration with Corporate IT Systems**

The ARF 1.4 acknowledges the need for EUDI Wallets to integrate effectively with existing IT infrastructures, particularly within organizations that require high availability and automated operations [Org-ID-002, Org-ID-006]. The ARF supports solutions that ensure continuous operations (24x7) and robust integration with corporate IT systems [Org-ID-005], which is critical for Organizational Wallets. The proposed architecture focuses on deep integration with corporate systems, which aligns with the ARF’s recommendations for ensuring that wallets can operate efficiently in demanding environments while adhering to EU regulations such as GDPR and eIDAS [Org-ID-007].

**4. Global Applicability and Compliance**

ARF 1.4 recognizes that while Citizen Wallets primarily address national or EU-wide use cases, Organizational Wallets often need to function on a global scale. The ARF emphasizes the importance of aligning data models and protocols with global standards to facilitate cross-border operations [Org-ID-001]. The W3C VCDM, which leverages JSON-LD and supports interoperability with global standards [Org-ID-011], ensures that Organizational EUDI Wallets are compliant with both EU and international regulations [Org-ID-007], making them suitable for global operations, including complex supply chain management and Industry 4.0 scenarios [Org-ID-003, Org-ID-013].

## Conclusing & Outlook

**Conclusion**

In accordance with the requirements for Organisational Wallets we have drawn up a decision matrix that compares the advantages and disadvantages of the protocols and formats in question. Based on this, we came to the conclusion that the W3C VCDM with JSON-LD and DIF WACI (or VC-API) best meet the requirements of the use cases that will be mapped by organisation wallets in the future. This applies in particular to interoperability with existing international standards and data formats. The practicability of these protocols and formats is illustrated in RFC010. We have also explained the compatibility of the approach with ARF 1.4, in which protocols and formats are defined within rulebooks in order to provide the necessary flexibility for a wide range of use cases on the one hand and to be able to define concrete interoperability profiles on the other.

**Outlook**

To further advance this initiative, the next step is to draft a rulebook that defines the specific credential formats, protocols, and other technical standards for Organizational EUDI Wallets and EAAs. This rulebook will provide the necessary guidelines to ensure consistent implementation and interoperability across different sectors, supporting the widespread adoption of Organisational EUDI wallets within Industrial and Global Applications.

## **References**

1. **European Commission (2024)**, The European Digital Identity Wallet Architecture and Reference Framework (2024-04, v1.4.0) [Online]. Available at: https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/tree/v1.4.0.

2. **European Commission (2024)**, Large Scale Pilots [Online]. Available at: https://ec.europa.eu/digital-building-blocks/sites/display/EUDIGITALIDENTITYWALLET/What+are+the+Large+Scale+Pilot+Projects

3. **Internet Engineering Task Force (IETF) (2024)**, SD-JWT-based Verifiable Credentials (SD-JWT VC). Available at: https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/

4. **World Wide Web Consortium (W3C) (2024)**, Verifiable Credentials Data Model v2.0, Available at: https://www.w3.org/TR/vc-data-model-2.0/.

5. **OpenID Foundation (2024)**, OpenID for Verifiable Credentials (OID4VC), Available at: https://openid.net/sg/openid4vc/specifications/

6. **Decentralized Identity Foundation (DIF) (2023)**, Wallet And Credential Interactions (WACI) DIDComm Interop Profile, Available at: https://identity.foundation/waci-didcomm/.

## Further Reading

1. **European Commission (2024)**, European Digital Identity Regulation - Detailed guidelines on digital identity regulation, Available at: https://ec.europa.eu/info/law/better-regulation/have-your-say/initiatives_en?text=European%20Digital%20Identity%20Wallets.

2. **European Commission (2024)**, European Digital Identity Regulation - Specifications for credential formats and security protocols, Available at: https://ec.europa.eu/info/law/better-regulation/have-your-say/initiatives_en?text=European%20Digital%20Identity%20Wallets.

3. **European Commission (2024)**, European Digital Identity Regulation - Compliance and integration requirements for organizational identity solutions, Available at: https://ec.europa.eu/info/law/better-regulation/have-your-say/initiatives_en?text=European%20Digital%20Identity%20Wallets.

4. **Decentralized Identity Foundation (DIF) (2023)**, Credential Manifest, Available at: https://identity.foundation/credential-manifest.

5. **European Commission (2024)**, Regulation (EU) 910/2014 on electronic identification and trust services for electronic transactions in the internal market and repealing Directive 1999/93/EC, Available at: https://eur-lex.europa.eu/eli/reg/2014/910/oj.

6. **Catena-X Automotive Network (2023)**, JSON-LD Credential Formats in Catena-X Standards, Available at: https://catena-x.net/de/.

7. **Catena-X Automotive Network (2023)**, CX-0128 Demand and Capacity Management Data Exchange v2.0.0, Available at: https://catenax-ev.github.io/.

8. **Catena-X Automotive Network (2023)**, JSON-LD in Digital Product Passport Use Cases, Available at: https://catenax-ev.github.io/.

9. **Gaia-X (2023)**, Federation Services - Gaia-X Architecture Document - 21.12 Release, Available at: https://docs.gaia-x.eu/.

10. **Gaia-X (2023)**, Verifiable Credentials / Presentations in Gaia-X, Available at: https://gaia-x.eu/.

11. **International Data Spaces Association (IDSA) (2024)**, New IDSA Publication Crystallizes Shared Approach with GAIA-X, Available at: https://internationaldataspaces.org/.

12. **Battery Pass Consortium (2024)**, Battery Passport Technical Guidance, Version 1.0, March 2024. Available at: https://thebatterypass.eu/.


### **Appendix - Key Requirements for Organizational EUDI Wallets**

The following list of key requirements for Organizational Wallets is derived from the EWC requirements for Wallet Applications and Wallet Core Components, as well as from the use cases in the scope of the EWC.

| **ID**            | **Requirement**                           | **Description**                                                                                                                                                   |
|-------------------|-------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Org-ID-001**    | **Interoperability and Standardization**  | Must enable cross-border recognition of credentials and adhere to EU regulations, ensuring interoperability across different systems and member states.            |
| **Org-ID-002**    | **Automation and Scalability**            | Should support automated credential processing and scalable infrastructure to manage large volumes of transactions, particularly in industrial contexts.           |
| **Org-ID-003**    | **Support for Complex Workflows**         | Must cater to both human and system users, allowing for seamless interaction and delegation of tasks within the organization, supporting complex business processes. |
| **Org-ID-004**    | **Security and Privacy**                  | Requires robust cryptographic security, secure key management, and privacy-preserving features to protect organizational credentials from unauthorized access.       |
| **Org-ID-005**    | **Integration with Existing Systems**     | Must integrate with existing organizational systems, such as ERP and CRM, through APIs and external interfaces, facilitating smooth communication and interoperability. |
| **Org-ID-006**    | **Availability and Reliability**          | Must ensure 24x7x365 availability and include high availability and redundancy features to prevent downtime, ensuring continuous operation.                       |
| **Org-ID-007**    | **Compliance with Regulatory Requirements** | Must adhere to all relevant EU regulations, including GDPR, eIDAS, and sector-specific mandates, ensuring legal compliance and auditability.                        |
| **Org-ID-008**    | **Usability**                             | Must provide intuitive user interfaces and developer-friendly APIs, making it easy for both human and system users to manage credentials and interact with other organizations. |
| **Org-ID-009**    | **Cost Efficiency**                       | The solution should minimize the total cost of ownership through efficient resource usage, minimal downtime, and low maintenance requirements.                   |
| **Org-ID-010**    | **Key Management**                        | Strong key management practices are required, including secure generation, storage, rotation, and backup, particularly for high-regulated industries.              |
| **Org-ID-011**    | **Adaptability to Industry Needs**        | The solution should be flexible enough to meet the varying levels of regulatory requirements across different sectors, including high-regulated industries and critical infrastructures. |
| **Org-ID-012**    | **User Types: System and Human Users**    | The wallet shall support both system users (automated processes) and human users, ensuring seamless interaction and functionality for both types of users.          |
| **Org-ID-013**    | **Support for Industry Semantic Models**  | The wallet must support industry-specific semantic models to ensure compatibility and integration with domain-specific data structures and standards.              |
| **Org-ID-014** | **User Management** | Organizational Wallets are used by a group of individuals. Proper user management must be in place to receive, store, and present credentials. Each user must be authenticated before being granted access to the wallet. The Organizational Wallet must ensure that credentials are only accessible in defined realms.