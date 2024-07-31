# EWC RFC010: Issue and Present Organisational Credentials - v0.1

**Authors:** 
* Mr Ronald Koenig (Spherity, Germany)
* Mr Ricky Thiermann  (Spherity, Germany)
* Mr Martin Westerkamp  (Spherity, Germany)

**Reviewers:** 
* ???

**Status:** Ready for review

notes:
* exchange of organisation credentials - issue and present
* organisation credentials are credentials about legal entities and their relationships
* legal entities are legal persons (companies) or natural persons (humans related to the company - employee, shareholder, ultimate beneficiary owners, ...)
* enterprise wallets holds organisational credentials and are operated by the company
  * used by a group a persons (employees of the company)
  * creates and holds identity for every user (DID)
  * company is liable for misuse of issued credentials
  * internal role model (visibility of credentials and claims)
  * running 24/7
* complex enterprise credentials  
* signatory rights based on enterprise credentials
* delegation of rights based on power of attorney credentials

structure:
* summary
* motivation
* trust model
  * DIDs
  * trust list (issuer, verifier)
  * credentials and holder binding
  * signatory rights
  * power of attorney
* issue natural person credential
* issue enterprise credential
* issue power of attorney credential
* present organisational credentials


# 1.0	Summary

This specification defines the workflow for issuing an Organisation Credential.
Because the exchange of credentials between organisational server wallets has unique requirements compared to personal wallets, dedicated workflows are required to ensure streamlined processes.
In particular, organisational server wallets are expected to operate autonomously under certain conditions.
Further, actors (human or machine) within organisations need to authenticate and provide authorisation when interacting with remote organisations.
Therefore, protocols with greater informative value and flexibility are required in order to be able to express the complex organisational structures.
For this purpose, this RFC utilizes the Decentralized Identity Foundation's (DIF) Wallet And Credential Interactions (WACI) DIDComm Interop Profile [1] for exchanging credentials.
We also propose to define credentials using the W3C Credential Format [2], which is based on JSON-LD.
This allows the complex semantics of organisational credentials to be expressed in a concise but specific way.
In addition, W3C Verifiable Credentials provide cryptographic flexibility so that we can take advantage of well-established signature schemes such as ECDSA-SD, while retaining the flexibility to use modern schemes such as BBS.
The presented standard processes and formats ensure interoperability across the European Wallet Ecosystem with a standard specification in the EUDI Wallet Ecosystem according to the requirements of the ARF [3].


# 2.0	Motivation

The EWC LSP handles a variety of credentials that are exchanged between different types of organisations.
The goal of this RFC is to provide a standard protocol for requesting and issuing organisational credentials.
Depending on the use case, organisations need certain information about their business partners, such as authorised signatories, tax numbers or ultimate beneficial owners.
This data needs to be validated and kept up to date.
To simplify the management of this master data, we propose the use of organisational credentials, for example issued by an Attestation Provider verifying data from an authentic source such as the relevant commercial register.
We aim to provide greater flexibility in terms of authentication methods and credential types.
Since the credentials in scope are exchanged with a focus on organisations, we anticipate the use of server-based organisational wallets, which have different requirements than personal mobile wallets.
Therefore, we propose the introduction of an organisation-specific identity for natural persons that allows requesting and presenting organisational credentials without the need to interact with a personal wallet.
This will significantly simplify the handling of organisational credentials and enable a wide range of use cases.
Organisation wallets can handle processes automatically without human interaction and present the required credentials independently.
In addition, processes that require the verifiable liability of a responsible person are enabled by signing such requests using the representative's organisation-specific identity.

The defined protocols and standards serve as a foundation for enabling interoperability between organisations throughout the EWC ecosystem.
This RFC assumes that users are familiar with the chosen EWC protocols and standards, and can reference the original specifications when required. 

# 3.0 Initialisation
This section describes the requirements for setting up the organisational wallet and adding users to the system.
Users in the system are members of the organisation.
Cryptographic material is managed by the Organisation Wallet through a key management system.
In addition, the wallet implementation shall ensure that only the authorised user has access to the private key.
Private keys shall not be retrievable and shall only be used for cryptographic operations such as signing credentials or decrypting messages.
In addition, each user is assigned a unique Decentralised Identifier (DID).
The user's organisation-specific DID can be used to issue, request and present credentials.

## 3.1 Organisation wallet initialisation
Before an organisational wallet can be used, it must be initialised.
In this process, the user management procedures are defined and an organisation-specific DID and key material are generated.
The organisation-specific DID enables the implementation of actions that do not require the traceability of a responsible person.
Such actions can be either part of automated processes or triggered by authorised persons.

```mermaid
sequenceDiagram
    %%{init: { 'sequence': {'noteAlign': 'left'} }}%%
    autonumber

    participant Admin as Administrator
    participant OrgWallet as Organisation Wallet
    Admin->>Admin: Defines configuration
    note right of Admin:  - service endpoints<br/>- authentication methods (password, OIDC, LDAP, ...<br/>- ...
    Admin->>OrgWallet: creates wallet (configuration)
    OrgWallet->>OrgWallet: create Key Pair
    note left of OrgWallet:  - DID of the legal entity used as<br/> subject information id<br/>in the enterprise credential<br/>private key accessible by any user of the wallet
    OrgWallet->>OrgWallet: create DID
    OrgWallet -->> Admin: notify: wallet created
```

## 3.2 User enrollment
Once the Organisation Wallet has been initialised, users (natural persons) can be enrolled.
Enrolling a user involves generating a user-specific DID and key material.
Note that the generated DID is different from the DID a user may have in a personal wallet.
As a result, members of an organisation benefit from role-specific credentials in their day-to-day work without having to rely on their personal wallet.
In addition, the generated material is securely stored in the organisation's key management system and can only be accessed by the assigned user for credential requests and presentations.

In the following sample workflow, an administrator invites a user to create a dedicated account in the Organisation Wallet.
Alternatively, user management can be performed remotely.
Either way, the Organisation Wallet maintains a dedicated DID and key material for each user.

```mermaid
sequenceDiagram
    autonumber

    participant Admin as Administrator
    participant OrgWallet as Organisation Wallet
    participant NP as Natural Person

    Admin ->> OrgWallet: Invite user
    OrgWallet ->> NP: Invite
    NP ->> OrgWallet: Request DID

    OrgWallet ->> OrgWallet: Create Key Pair and DID for Natural Person
    OrgWallet ->> OrgWallet: Store Key Pair and DID for Natural Person

    OrgWallet -->> NP: Request approved
    OrgWallet -->> Admin: Notify
```

# 3.0 Issuing Natural Person Credentials
Natural persons with signatory rights in the organisation are issued with a Natural Person Credential, which can be used to prove their registration with a trusted authority such as a business registry.
This credential can also be used for authentication purposes, for example when requesting an Organisation Credential.
The Natural Person Credential is linked to the Natural Person's DID generated in the Organisation Wallet.
The credential is stored in the Organisation Wallet and can only be presented by the authenticated natural person.

Signatory rights refer to the legal entitlements and responsibilities held by individuals or entities that sign a document or agreement.
These rights typically include the authority to enter into binding agreements, the responsibility to adhere to the terms and conditions of the agreement, and the ability to enforce the agreement against other parties.
Signatory rights ensure that the signer is recognized as a legitimate party to the contract, with the capacity to fulfill the obligations and claim benefits stipulated within the agreement.
In legal contexts, these rights are crucial for validating the enforceability of contracts and other legal documents.

# 3.1 Process
In order for users to demonstrate their signatory rights on behalf of the organisation to business partners, they must possess a credential that has been issued by a reliable authority.
To obtain this credential, the user must request it from a trustworthy authority such as a business registry (EAA Provider).
Prior to the issuance of the credential, the credential provider must authenticate the user by requesting the user's PID.
Once the provider has verified the PID, they will review the user's role within the organisation according to a registry, such as the business registry.
If the requested role can be attested, the credential is issued to the user's organisation-specific DID and stored in the organisational wallet.
Consequently, the credential can be used to authenticate to business partners and perform authorised actions.

The following process presumes the Organisation Wallet to provide a list of trustworthy authorities natural persons can use to request signatory rights credentials.

```mermaid
sequenceDiagram
    autonumber

    participant NP as Natural Person
    participant OrgWallet as Organisation Wallet
    participant Issuer as EAA Provider

    NP->>OrgWallet: Request Natural Person Credential

    OrgWallet->>Issuer: Credential Application: Natural Person Credential
    
    Note over NP, Issuer: Natural Person Authentication

    Issuer-)Issuer: Store PID Information and Natural Person DID
    Issuer->>OrgWallet: Credential Fulfilment: Natural Person Credential
    OrgWallet->>Issuer: Ack
```

**Sequence diagram steps:**
1. The Natural Person instructs the Organisation Wallet to request a Natural Person Credential from the relevant authority (EAA Provider).
2. The Organisation Wallet sends a 'Credential Application' message to the EAA Provider.
This message is preferably a DIDComm message sent via an established DIDComm channel.
If no channel exists, the Organisation Wallet resolves the EAA Provider's DID configured in the Wallet to obtain the DID document.
The DID document contains the EAA Provider's accepted protocols and endpoints.
The 'Credential Application' is sent to the endpoint and includes a connection specific Peer DID.
The Peer-DID encodes an ephemeral key specific to the connection and the Organisation Wallet endpoint.
3. Before the EAA Provider can issue the requested credential, the Natural Person must authenticate.
This authentication is outside the scope of this RFC and may be implemented, for example, by presenting a PID using the Natural Person Wallet.
Upon successful authentication, the EAA Provider stores the Natural Person's PID and DID for possible matching in future organisation credential requests.
4. The EAA Provider issues the requested credential via a Credential Fulfilment message sent to the Organisation Wallet.
The Organisation Wallet stores the received credential.
It may then be used by the Natural Person for authentication purposes.
5. The Organisation Wallet acknowledges receipt of the Natural Person Credential.

# 3.2 Messages
The message flow for obtaining a natural person credential consists of the request and issuance of the actual credential ([3.2.1 Issue Credential](#321-issue-credential)) as well as a sub-message flow that enables the authentication of the applicant ([3.2.2 Present PID](#322-present-pid)).
It should be noted that the main message flow is controlled by the organisational wallet, while the authentication message flow is carried out by the personal user wallet.
Both workflows are described according to the DIF-WACI protocol [1].

## 3.2.1 Issue Credential


### 3.2.1.1 Credential Manifest
The DIF Credential Manifest is designed to standardize and streamline the issuance and verification of digital credentials.
It specifies a common structure for defining what information (credentials) an issuer needs from a holder, how this information should be presented, and the criteria for validation.
This framework enables interoperability across different systems and providers by ensuring that digital credentials can be easily understood and processed regardless of the issuing entity.
The Credential Manifest facilitates secure, trustworthy, and user-centric identity verification processes, enhancing the efficiency and reliability of digital interactions.

In the following, the Credential Manifest defines the credentials issued by the EAA Provider.
In this case, a Natural Person Credential is offered as defined in the corresponding JSON-LD schema.
The credential manifest also provides information about the provider's DID and the credential formats offered, including the signature algorithms supported.

The Credential Manifest of the provider is assumed to be known to the Organisation Wallet.
If it is not known, it must first be requested by sending a `Propose Credential` message to the provider and receiving the manifest in the subsequent `Offer Credential` message, as defined in the DIF-WACI protocol [1].

```json
{
  "id": "natural-person-manifest",
  "spec_version": "https://identity.foundation/credential-manifest/spec/v1.0.0/",
  "issuer": {
    "id": "did:example:provider",
    "name": "Example EAA Provider"
  },
  "outputDescriptors": [
    {
      "id": "natural-person-credential",
      "schema": "https://spherity.github.io/oid/credentials/v1/schema.jsonld"
      ]
    }
  ],
  "format": {
    "ldp_vc": {
      "proof_type": [
        "JsonWebSignature2020",
        "Ed25519Signature2018",
        "EcdsaSecp256k1Signature2019",
        "ecdsa-sd-2023"
      ]
    }
  }
}
```

### 3.2.1.2 Credential Application (Request)
The `Credential Application` message is sent from the Organisation Wallet to the EAA Provider after the credential request has been made by the user.

If a communication channel exists between the Organisation Wallet and the EAA Provider, the channel specific Peer DIDs are used in the `from` and `to` fields of the DIDComm message.
In this example, we assume that no communication channel exists and a new one needs to be established by generating new Peer DIDs on each side of the channel.
Therefore, the Organisation Wallet generates a new key pair for secure communication with the specific provider.
It then encodes the generated public key and connection information, such as the endpoint being offered, into the Peer DID and places it in the `from` field of the request.
To initiate the channel, the message is delivered to the provider using its public DID.
The provider then announces its Peer DID in the response message for further communication.

The Credential Application references the Credential Manifest of the Provider to indicate what type of credential the Organisational Wallet is requesting.
Furthermore, the applicant is defined by the identity of the user managed by the Organisation Wallet, here `did:key:zDnaeVXmpeF4fafnTY44Fba4yCUMgxhPf85XEoajZbsBxPnEC`.

```json
{
  "type": "https://didcomm.org/issue-credential/3.0/request-credential",
  "id": "c6686159-ef49-45b2-938f-51818da14723",
  "pthid": "6346b86f-c216-42f1-a0dc-ff733de2708d",
  "from": "did:peer:2.VzDnaeXJT2DCDJyzRPXGErHYevjvZw85UT8GKnVxVBieH2mSmi.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMC9kaWRjb21tIiwiYSI6WyJkaWRjb21tL3YyIl0sInIiOlsiZGlkOmtleTp6RG5hZVhKVDJEQ0RKeXpSUFhHRXJIWWV2anZadzg1VVQ4R0tuVnhWQmllSDJtU21pI2tleS0xIl19fQ",
  "to": "did:peer:2.VzDnaeVQ53PrRWHhijjTCwRhez7927X92evThvdnHYQVz6mt4i.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMC9kaWRjb21tIiwiYSI6WyJkaWRjb21tL3YyIl0sInIiOlsiZGlkOmtleTp6RG5hZVZRNTNQclJXSGhpampUQ3dSaGV6NzkyN1g5MmV2VGh2ZG5IWVFWejZtdDRpI2tleS0xIl19fQ",
  "body": {},
  "attachments": [
    {
      "id": "e00e11d4-906d-4c88-ba72-7c66c7113a78",
      "format": "dif/credential-manifest/application@v1.0",
      "media_type": "application/json",
      "data": {
        "json": {
          "id": "c407be9a-6a17-4577-91c7-ed327c8fa8ea",
          "@context": [
            "https://www.w3.org/ns/credentials/v2",
            "https://identity.foundation/credential-manifest/application/v1"
          ],
          "credential_application": {
            "id": "888963b8-c087-4e70-afbb-11fba91e66b3",
            "spec_version": "https://identity.foundation/credential-manifest/spec/v1.0.0/",
            "applicant": "did:key:zDnaeVXmpeF4fafnTY44Fba4yCUMgxhPf85XEoajZbsBxPnEC",
            "manifest_id": "075c7ccf-db02-42fd-bedb-d9fc369438c4",
            "format": {
              "ldp_vc": {
                "proof_type": ["ecdsa-sd-2023"]
              }
            }
          },
          "type": ["CredentialApplication"]
        }
      }
    }
  ]
}
```

### 3.2.1.3 Credential Fulfilment (Issue)
If the applicant is authorised to receive a Natural Person Credential, it will be issued by the EAA Provider and sent to the Organisation Wallet.

In the `Credential Fulfilment` message, the provider indicates which credential has been issued according to the Credential Manifest.

TODO: Why VerifiablePresentation?

```json
{
  "type": "https://didcomm.org/issue-credential/3.0/issue-credential",
  "id": "7a476bd8-cc3f-4d80-b784-caeb2ff265da",
  "thid": "c6686159-ef49-45b2-938f-51818da14723",
  "from": "did:peer:connection-1-provider",
  "to": ["did:peer:connection-1-organisation"],
  "body": {},
  "attachments": [
    {
      "id": "e00e11d4-906d-4c88-ba72-7c66c7113a79",
      "media_type": "application/json",
      "format": "dif/credential-manifest/fulfillment@v1.0",
      "data": {
        "json": {
          "@context": [
            "https://www.w3.org/2018/credentials/v1",
            "https://identity.foundation/credential-manifest/fulfillment/v1"
          ],
          "type": ["VerifiablePresentation", "CredentialFulfillment"],
          "credential_fulfillment": {
            "id": "a30e3b91-fb77-4d22-95fa-871689c322e2",
            "manifest_id": "natural-person-manifest",
            "descriptor_map": [
              {
                "id": "natural-person-credential",
                "format": "ldp_vc",
                "path": "$.verifiableCredential[0]"
              }
            ]
          },
          "verifiableCredential": [
            {
              "@context": [
                "https://www.w3.org/2018/credentials/v1",
                "https://spherity.github.io/oid/credentials/v1/schema.jsonld"
              ],
              "type": ["VerifiableCredential", "NaturalPerson"],
              "id": "urn:uuid:f086cced-1153-4540-993b-b4d52ca499c2",
              "issuanceDate": "2019-12-03T12:19:52Z",
              "expirationDate": "2029-12-03T12:19:52Z",
              "issuer": "did:key:z6MkiY62766b1LJkExWMsM3QG4WtX7QpY823dxoYzr9qZvJ3",
              "credentialSubject": {
                "id": "did:example:natural-person-1",
                "type": "NaturalPerson",
                "givenName": "John",
                "familyName": "Doe",
                "birthDate": "1970-01-01"
              },
              "proof": {
                "created": "2021-06-07T20:02:44.730614315Z",
                "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..NVum9BeYkhzwslZXm2cDOveQB9njlrCRSrdMZgwV3zZfLRXmZQ1AXdKLLmo4ClTYXFX_TWNyB8aFt9cN6sSvCg",
                "proofPurpose": "assertionMethod",
                "type": "Ed25519Signature2018",
                "verificationMethod": "did:orb:EiA3Xmv8A8vUH5lRRZeKakd-cjAxGC2A4aoPDjLysjghow#tMIstfHSzXfBUF7O0m2FiBEfTb93_j_4ron47IXPgEo"
              }
            }
          ],
          "proof": {
            "created": "2021-06-07T20:02:44.730614315Z",
            "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..NVum9BeYkhzwslZXm2cDOveQB9njlrCRSrdMZgwV3zZfLRXmZQ1AXdKLLmo4ClTYXFX_TWNyB8aFt9cN6sSvCg",
            "proofPurpose": "authentication",
            "type": "Ed25519Signature2018",
            "verificationMethod": "did:orb:EiA3Xmv8A8vUH5lRRZeKakd-cjAxGC2A4aoPDjLysjghow#tMIstfHSzXfBUF7O0m2FiBEfTb93_j_4ron47IXPgEo"
          }
        }
      }
    }
  ]
}
```

# 4.0 Issuing Organisational Credentials

# 4.1 Process

# 4.2 Messages

## 4.2.1 Credential Manifest

```json
{
  "id": "organisation-credential-manifest",
  "spec_version": "https://identity.foundation/credential-manifest/spec/v1.0.0/",
  "issuer": {
    "id": "did:example:provider",
    "name": "Example EAA Provider"
  },
  "outputDescriptors": [
    {
      "id": "organisation-credential",
      "schema": [
        {
          "uri": "https://spherity.github.io/oid#LegalEntityCertificate",
          "required": true
        }
      ],
      "name": "Organisation Credential"
    }
  ],
  "format": {
    "jwt": {
      "alg": ["EdDSA", "ES256K", "ES384"]
    },
    "jwt_vc": {
      "alg": ["ES256K", "ES384"]
    },
    "jwt_vp": {
      "alg": ["EdDSA", "ES256K"]
    },
    "ldp_vc": {
      "proof_type": [
        "JsonWebSignature2020",
        "Ed25519Signature2018",
        "EcdsaSecp256k1Signature2019",
        "RsaSignature2018"
      ]
    },
    "ldp_vp": {
      "proof_type": ["Ed25519Signature2018"]
    },
    "ldp": {
      "proof_type": ["RsaSignature2018"]
    }
  }
}
```

## 4.2.2 Credential Application (Request)

```json
{
  "type": "https://didcomm.org/issue-credential/3.0/request-credential",
  "id": "c6686159-ef49-45b2-938f-51818da14723",
  "thid": "c6686159-ef49-45b2-938f-51818da14723",
  "from": "did:example:organisation",
  "to": ["did:example:provider"],
  "body": {},
  "attachments": [
    {
      "id": "e00e11d4-906d-4c88-ba72-7c66c7113a78",
      "media_type": "application/json",
      "format": "dif/credential-manifest/application@v1.0",
      "data": {
        "json": {
          "@context": [
            "https://www.w3.org/2018/credentials/v1",
            "https://identity.foundation/credential-manifest/application/v1"
          ],
          "type": "CredentialApplication",
          "credential_application": {
            "id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
            "spec_version": "https://identity.foundation/credential-manifest/spec/v1.0.0/",
            "applicant": "did:example:organisation",
            "manifest_id": "organisation-credential-manifest",
            "format": {
              "ldp_vc": {
                "proof_type": [
                  "JsonWebSignature2020",
                  "EcdsaSecp256k1Signature2019"
                ]
              }
            }
          }
        }
      }
    }
  ]
}
```

## 4.2.3 Credential Fulfilment (Issue)

```json
{
  "type": "https://didcomm.org/issue-credential/3.0/issue-credential",
  "id": "7a476bd8-cc3f-4d80-b784-caeb2ff265da",
  "thid": "c6686159-ef49-45b2-938f-51818da14723",
  "from": "did:example:provider",
  "to": ["did:example:organisation"],
  "body": {},
  "attachments": [
    {
      "id": "e00e11d4-906d-4c88-ba72-7c66c7113a79",
      "media_type": "application/json",
      "format": "dif/credential-manifest/fulfillment@v1.0",
      "data": {
        "json": {
          "@context": [
            "https://www.w3.org/2018/credentials/v1",
            "https://identity.foundation/credential-manifest/fulfillment/v1"
          ],
          "type": ["VerifiablePresentation", "CredentialFulfillment"],
          "credential_fulfillment": {
            "id": "a30e3b91-fb77-4d22-95fa-871689c322e2",
            "manifest_id": "dcc75a16-19f5-4273-84ce-4da69ee2b7fe",
            "descriptor_map": [
              {
                "id": "driver_license_output",
                "format": "ldp_vc",
                "path": "$.verifiableCredential[0]"
              }
            ]
          },
          "verifiableCredential": [
            {
              "@context": [
                "https://www.w3.org/2018/credentials/v1",
                "https://spherity.github.io/oid/credentials/v1/schema.jsonld"
              ],
              "type": ["VerifiableCredential", "LegalEntityCertificate"],
              "id": "urn:uuid:f086cced-1153-4540-993b-b4d52ca499c2",
              "issuanceDate": "2019-12-03T12:19:52Z",
              "expirationDate": "2029-12-03T12:19:52Z",
              "issuer": "did:example:provider",
              "credentialSubject": {
                "id": "did:example:organisation",
                "type": ["LegalPersonId", "LegalPersonBaseData"],
                "euid": "DEPLPOAG.123456789",
                "companyName": "Flower Power AG",
                "address": {
                  "postalCode": "10119",
                  "addressCountry": "Germany",
                  "addressLocality": "Berlin"
                },
                "registerIdentification": {
                  "registerType": "ABC123",
                  "registerTypeDescription": "Amtsgericht Tiergarten",
                  "registerLocation": "Berlin, Moabit",
                  "registerNumber": "123AVB",
                  "companyId": "ACDC",
                  "companyIdCountryCode": "DE"
                },
                "previousName": ["AL", "DI"],
                "status": "active",
                "liquidationStatus": "none",
                "insolvencyStatus": "none",
                "legalForm": "GmbH",
                "isFoundation": false,
                "isNfrdObligated": false,
                "isTrust": false,
                "businessObject": "Trading Company",
                "registrationDate": "2010-01-01",
                "exchange": {
                  "name": "Boerse Stuttgart",
                  "country": "Germany",
                  "marketIdentificationCode": "AA",
                  "tickerSymbol": "BB"
                },
                "isin": "1234567890",
                "vatId": "22222ABC",
                "lei": "ABC988",
                "leiStatus": "active",
                "leiNextIssuance": "2027-01-10"
              },
              "proof": {
                "created": "2021-06-07T20:02:44.730614315Z",
                "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..NVum9BeYkhzwslZXm2cDOveQB9njlrCRSrdMZgwV3zZfLRXmZQ1AXdKLLmo4ClTYXFX_TWNyB8aFt9cN6sSvCg",
                "proofPurpose": "assertionMethod",
                "type": "Ed25519Signature2018",
                "verificationMethod": "did:orb:EiA3Xmv8A8vUH5lRRZeKakd-cjAxGC2A4aoPDjLysjghow#tMIstfHSzXfBUF7O0m2FiBEfTb93_j_4ron47IXPgEo"
              }
            }
          ],
          "proof": {
            "created": "2021-06-07T20:02:44.730614315Z",
            "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..NVum9BeYkhzwslZXm2cDOveQB9njlrCRSrdMZgwV3zZfLRXmZQ1AXdKLLmo4ClTYXFX_TWNyB8aFt9cN6sSvCg",
            "proofPurpose": "authentication",
            "type": "Ed25519Signature2018",
            "verificationMethod": "did:orb:EiA3Xmv8A8vUH5lRRZeKakd-cjAxGC2A4aoPDjLysjghow#tMIstfHSzXfBUF7O0m2FiBEfTb93_j_4ron47IXPgEo"
          }
        }
      }
    }
  ]
}
```

# 5.0 Presenting Organisation Credentials

# 5.1 Process

# 5.2 Messages

# 6.0	Reference


1. Decentralized Identity Foundation (DIF) (2023), Wallet And Credential Interactions (WACI) DIDComm Interop Profile, Available at: https://identity.foundation/waci-didcomm/ (Accessed at: July 9, 2024).
2. World Wide Web Consortium (W3C) (2024), Verifiable Credentials Data Model v2.0, Available at: https://www.w3.org/TR/vc-data-model-2.0/ (Accessed at: July 9, 2024).
3. European Commission (2023), The European Digital Identity Wallet Architecture and Reference Framework (2024-04, v1.3.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: May 14, 2024).
4. Decentralized Identity Foundation (DIF) (2023), Credential Manifest, Available at: https://identity.foundation/credential-manifest (Accessed at: July 15, 2024).