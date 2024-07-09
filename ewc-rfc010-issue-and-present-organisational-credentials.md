# EWC RFC010: Issue and Present Organisational Credentials - v0.1

**Authors:** 
* Mr Ronald Koenig (Spherity, Germany)
* Mr Ricky Thiermann  (Spherity, Germany)
* Mr Martin Westerkamp  (Spherity, Germany)

**Reviewers:** 
* ???

**Status:** Ready for review


# 1.0	Summary

This specification defines the workflow for issuing an Organisation Credential.
As the exchange of credentials between organisational server wallets has unique requirements, dedicated workflows are required to provide streamlined processes.
While this RFC utilizes the Decentralized Identity Foundation's (DIF) Wallet And Credential Interactions (WACI) for DIDComm Interop Profile [1] for exchanging credentials, it is open to alternative message exchange protocols.
We also propose to define credentials using the W3C Credential Format [2], which is based on JSON-LD.
This allows the complex semantics of organisational credentials to be expressed in a concise but specific way.
In addition, W3C Verifiable Credentials provide cryptographic flexibility so that we can take advantage of well-established signature schemes such as ECDSA-SD, while retaining the flexibility to use modern schemes such as BBS.
The presented standard processes and formats ensure interoperability across the European Wallet Ecosystem with a standard specification in the EUDI Wallet Ecosystem according to the requirements of the ARF [3].


# 2.0	Motivation

The EWC LSP handles a variety of credentials that are exchanged between different types of organizations.
The goal of this RFC is to provide a standard protocol for requesting and issuing organizational credentials.
The goal is to provide greater flexibility in terms of authentication methods and credential types.
Since the credentials are exchanged with a focus on organizations, we anticipate the use of server-based organizational wallets, which have different requirements than personal mobile wallets.
Therefore, we propose the introduction of an organization-specific identity for natural persons that allows requesting and presenting organizational credentials without the need to interact with a personal wallet.
This will significantly simplify the handling of organizational credentials and enable a wide range of use cases.
Organization wallets can handle processes automatically without human interaction and present the required credentials independently.
In addition, processes that require the verifiable liability of a responsible person are enabled by signing such requests using the representative's organization-specific identity.

The defined protocols and standards serve as a foundation for enabling interoperability between organisations throughout the EWC ecosystem.
This RFC assumes that users are familiar with the chosen EWC protocols and standards, and can reference the original specifications when required. 

# 3.0 Issuing Signatory Rights Credentials
In the following, we assume that users have a user-specific identity that is managed in the organisational wallet.
Thus, the cryptographic material is managed by the organisational wallet through a key management system.
Furthermore, the wallet implementation must ensure that only the authorised user has access to the private key.
Private keys shall not be retrievable and shall only be used for cryptographic operations such as signing credentials or decrypting messages.
In addition, each user receives a unique Decentralised Identifier (DID).
The user's organisation-specific DID can be used to issue, request, and present credentials.

# 3.1 Process
In order for users to demonstrate their signatory rights on behalf of the organisation to business partners, they must possess a credential that has been issued by a reliable authority.
To obtain this credential, the user must request it from a trustworthy authority.
Prior to the issuance of the credential, the credential provider must authenticate the user by requesting the user's PID.
Once the provider has verified the PID, they will review the user's role within the organisation according to a registry, such as the business registry.
If the requested role can be attested, the credential is issued to the user's organisation-specific DID and stored in the organisational wallet.
Consequently, the credential can be used to authenticate to business partners and perform authorised actions.

# 3.2 Messages

# 4.0 Issuing Organisational Credentials

# 4.1 Process

# 4.2 Messages

# 5.0 Presenting Organisation Credentials

# 5.1 Process

# 5.2 Messages

# 6.0	Reference


1. Decentralized Identity Foundation (DIF) (2023), Wallet And Credential Interactions (WACI) for DIDComm Interop Profile, Available at: https://identity.foundation/waci-didcomm/ (Accessed at: July 9, 2024).
2. World Wide Web Consortium (W3C) (2024), Verifiable Credentials Data Model v2.0, Available at: https://www.w3.org/TR/vc-data-model-2.0/ (Accessed at: July 9, 2024).
3. European Commission (2023), The European Digital Identity Wallet Architecture and Reference Framework (2024-04, v1.3.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: May 14, 2024).