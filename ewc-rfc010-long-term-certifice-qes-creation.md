# RFC-010 Document Signing on a Remote Signing Service Provider using Long-Term Certificates

**Status: Approved**

**Authors:**

- Mr. Kyriakos Giannakis (Intesi Group, Italy | Flare, Greece)

**Reviewers:**

- Dr. Andreas Abraham (ValidatedID, Spain)
- Mr. Leone Riello (Infocert, Italy)

**Table of Contents:**

# Table of Contents

- [RFC-010 Document Signing on a Remote Signing Service Provider using Long-Term Certificates (v1)](#rfc-010-document-signing-on-a-remote-signing-service-provider-using-long-term-certificates-v1)
    - [1.0 Summary](#10-summary)
    - [2.0 Motivation](#20-motivation)
    - [3.0 The Signing Architecture](#30-the-signing-architecture)
        - [3.1 Architecture Phases](#31-architecture-phases)
    - [4. Signing Process](#4-signing-process)
        - [4.0 Overview](#40-overview)
        - [4.1 Phase 1: Signing Service User Registration](#41-phase-1-signing-service-user-registration)
            - [4.1.1 QESAC Claims Example](#411-qesac-claims-example)
            - [4.1.2 Inclusion of Credential ID in the QESAC (Optional)](#412-inclusion-of-credential-id-in-the-qesac-optional)
            - [4.1.3 VCT of QESAC](#413-vct-of-qesac)
        - [4.2 Phase 2: Service Provider Access & User Authentication](#42-phase-2-service-provider-access--user-authentication)
            - [4.2.1 Service Access by User](#421-service-access-by-user)
            - [4.2.2 User Authentication](#422-user-authentication)
        - [4.3 Phase 3: Certificate Listing and Selection (Optional)](#43-phase-3-certificate-listing-and-selection-optional)
            - [4.3.1 credentials/list](#431-credentialslist)
            - [4.3.2 credentials/info (optional)](#432-credentialsinfo-optional)
        - [4.4 Phase 4: Signature Confirmation & Private Key Unlocking (Credential Authorization)](#44-phase-4-signature-confirmation--private-key-unlocking-credential-authorization)
            - [4.4.1 Signing Confirmation as Willful Act](#441-signing-confirmation-as-willful-act)
            - [4.4.2 Private Key Unlocking (Credential Authorization)](#442-private-key-unlocking-credential-authorization)
                - [4.4.2.1 Authorization Code Flow (oauth2code)](#4421-authorization-code-flow-oauth2code)
                - [4.4.2.2 Explicit Flow (explicit)](#4422-explicit-flow-explicit)
        - [4.5 Phase 5: Signature Creation](#45-phase-5-signature-creation)
        - [4.6 Phase 6: Signed Document Formation and Retrieval](#46-phase-6-signed-document-formation-and-retrieval)
    - [5. Reference](#5-reference)


**Changelog:**

- Nov. 11 2024: Initialization of authoring process.
- Nov. 28 2024: Phase 4,5 authoring. Reformatting of headings and content. Addition of references.
- Nov. 29 2024: Added Overview section.
- Dec. 3 2024: Refinements on User Authn/z. Added registration phase.

# 1.0 Summary:

This Specification defines the procedures for using the EUDI wallet to digitally sign a document, using Long-Term certificates, on a Remote Signing Service Provider (SSP). It uses the user's PID to identify the user's Certificates. The Signer's Private Keys are stored safely in a Remote Qualified Electronic Signature (RQES) service.

**Remote QES services shall adhere to the [CSC (Cloud Signature Consortium)](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf) specifications that are also the basis for the JSON part of the ETSI TS 119 432 standard on protocols for remote digital signature creation.**

# 2.0 Motivation:

The motivation for this specification is to provide a robust and secure framework for enabling remote digital document signing using long-term certificates and the EUDI Wallet as a means of user authentication. As organizations and individuals increasingly transition to digital workflows, the need for verifiable and legally binding electronic signatures has grown significantly. This document aims to establish a standardized and interoperable approach that ensures trust, security, and ease of use and implementation in the signing process.

# 3.0 The Signing Architecture:

The architecture covered in this specification follows the process of remotely signing a document using long-term certificates, handled by a Remote QES (or AES) Service, as detailed in D4.8.

The architecture will be broken down in 6 main phases:
1. Phase 1: Signing Service User Registration
2. Phase 2: Service Provider Access & User Authentication
3. Phase 3: Certificate Listing and Selection
4. Phase 4: Signature Confirmation & Private Key Unlocking (Credential Authorization)
5. Phase 5: Signature Creation
6. Phase 6: Signed Document Formation and Retrieval

**Remote QES services shall adhere to the [CSC (Cloud Signature Consortium)](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf) specifications that are also the basis for the JSON part of the ETSI TS 119 432 standard on protocols for remote digital signature creation.**

> **Note**: The “Signature Creation Application” is shown as a separate Signing Service but may be integrated into the Service Provider. This depends on available software that the service provider can use.

> **Note**: The Signer's Document (SD) uploading process is out of scope of this RFC. The SD can be uploaded either by the user or the Service Provider, prior to the execution of the signing procedure.

# 4. Signing Process:

## 4.0 Overview:

```mermaid
   sequenceDiagram
    participant User
    participant EUDI Wallet
    participant Service Provider
    box CSC Protocol Usage
    participant Signing Service
    participant RQES Provider
    end
    
    Note over User, Signing Service: Phase 1: Signing Service User Registration
    User->>Signing Service: User Login/Registration to Signing Service (eg. via Username, Password, VCs)
    Signing Service->>User: PID Presentation (Binding) Request via OID4VP
    EUDI Wallet->>Signing Service: PID Presentation
    Note over User, Signing Service: At this point the user has been strongly authenticated.

    Signing Service->>User: Credential Offer Deeplink for QESAC
    EUDI Wallet->>Signing Service: Credential Request for QESAC
    Signing Service->>EUDI Wallet: QESAC Issuance

    Note over User, Signing Service: Phase 2: Service Provider Access
    User->>Service Provider: Service Access
    Service Provider->>Signing Service: Request Signing of Document
    Signing Service->>User: PID, QESAC Presentation Request (Authentication)
    EUDI Wallet->>Signing Service: PID, QESAC Presentation

    Note over Signing Service, RQES Provider: Phase 3: Certificate Listing and Selection
    Signing Service->>RQES Provider: POST /csc/v2/credentials/list
    RQES Provider->>Signing Service: { credentialIDs: [...], credentialInfos: [...] }

    Note over User, RQES Provider: Phase 4: Signature Confirmation & Private Key Unlocking (Credential Authorization)
    Signing Service->>User: PID, QESAC Presentation Request (Signature Confirmation)
    EUDI Wallet->>Signing Service: PID, QESAC Presentation

    alt oauth2code Credential Authz
    User->>RQES Provider: Credential Authorization (for oauth2code flow)
    activate RQES Provider
    else explicit Credential Authz
    activate Signing Service
    User->>Signing Service: Credential Authorization (for explicit flow)
    deactivate Signing Service
    
    

    Signing Service->>RQES Provider: POST /csc/v2/credentials/authorize
    deactivate RQES Provider
    RQES Provider->>Signing Service: SAD
    end

    Note over Signing Service, RQES Provider: Phase 5: Signature Creation
    Signing Service->>RQES Provider: POST /csc/v2/signatures/signHash
    RQES Provider->>Signing Service: Signed Hash

    Note over User, Signing Service: Phase 6: Signed Document Formation and Retrieval
    Signing Service->>Service Provider: Signed Document
    Service Provider->>User: Signed Document
```

## 4.1 Phase 1: Signing Service User Registration

```mermaid
   sequenceDiagram
  participant User
  participant EUDI Wallet
  participant Signing Service

  User->>Signing Service: User Login/Registration to Signing Service (eg. via Username, Password)
  Signing Service->>User: PID Presentation (Binding) Request via OID4VP
  EUDI Wallet->>Signing Service: PID Presentation
  Note over User, Signing Service: At this point the user has been strongly authenticated.
  
  Signing Service->>User: Credential Offer Deeplink for QESAC
  EUDI Wallet->>Signing Service: Credential Request for QESAC
  Signing Service->>EUDI Wallet: QESAC Issuance
```

Before the user can get access to the Signing Service to be able to sign documents, the user will need to be registered and authorized with the Signing Service.

During the registration flow, the signing service should request identification and authentication data from the user in a way that the unique identification of the user is ensured (eg, a combination of Username, Password, VCs).

During the registration procedure, the Signing Service must also request the user's PID to be presented with, at least, the [mandatory attributes](https://eu-digital-identity-wallet.github.io/eudi-doc-architecture-and-reference-framework/1.4.0/annexes/annex-3/annex-3.01-pid-rulebook/#23-pid-attributes) (`family_name`, `given_name`, `birth_date`, `age_over_18`, `issuance_date`, `expiry_date`, `issuing_authority`, `issuing_country`) included in the presentation.

The Signing Service should bind the user’s PID to its corresponding and uniquely identified user (by utilizing the authentication data) and issue a QES Auth Credential (QESAC). The QESAC must contain a `token` claim, bound to the user's profile.

For the issuance of the QESAC, the process detailed in [RFC-001 (Issue Verifiable Credential)](ewc-rfc001-issue-verifiable-credential.md) must be used.

### QESAC Claims Example:

```json
{
  "token": "XRfEU4QYJGhnmgpu3LftecA4197QR78n06gQf9QdbvwAdYX9eucNbep6wiwL259L"
}
```

Other claims can be added as needed by the Signing Service, however **the `token` claim is required**.

### Inclusion of Credential ID in the QESAC (Optional):

If needed, the user's credential ID can also be included in the QESAC to assist the Signing Service on the selection of the credential during the signing process, should the user have more than one Signing Credentials (Certificates):

```json
{
  "token": "XRfEU4QYJGhnmgpu3LftecA4197QR78n06gQf9QdbvwAdYX9eucNbep6wiwL259L",
  "credential_id": "GX0112348"
}
```

### VCT of QESAC:

Since each Signing Service has its own requirements and processes and should not be used by another Signing Service, the `vct` of the QESAC can be set by each Signing Service accordingly.
 
## 4.2 Phase 2: Service Provider Access & User Authentication for Signing

#### Overview:

```mermaid
   sequenceDiagram
  participant User
  participant EUDI Wallet
  participant Service Provider
  participant Signing Service

  User->>Service Provider: Service Access
  Service Provider->>Signing Service: Request Signing of Document
  Signing Service->>User: PID, QESAC Presentation Request via OID4VP
  EUDI Wallet->>Signing Service: PID, QESAC Presentation
```

In the second part of the signing procedure, the **User** accesses (through their browser) the **Service Provider** to request a document to be signed.

### 4.2.1: Service Access by User:

Initially, the User accesses the Service Provider through their browser. Upon arrival, the user should be presented with an Authentication screen, should they not have already been strongly authenticated (during phase 1).

### 4.2.2: User Authentication for Signing

The Signing Service should require the user is authenticated. Authentication of the user happens through a presentation of their PID and their QESAC.

The Signing Service should, at minimum, make the following verifications during authentication:

- The VPs of the QESAC, PID are valid.
- The `token` inside the QESAC is valid and not recalled.
- The user profile binded to the presented PID matches the profile binded to the QESAC.
- If a `credential_id` is included in the QESAC, the credential with the specified ID exists.

## 4.3 Phase 3: Certificate Listing and Selection (Optional)

The User's Signing Certificates (`Credentials`) from the Remote QES Service are listed:

**credentials/list**

```mermaid
%% credentialList
  sequenceDiagram
    participant Signing Service
    participant RQES Provider
    Signing Service->>RQES Provider: POST /csc/v2/credentials/list
    RQES Provider->>Signing Service: { credentialIDs: [...], credentialInfos: [...] }
```

**Sample Request**:
```http request
POST /csc/v2/credentials/list HTTP/1.1
Host: rqes.example.com
Authorization: Bearer ...
Content-Type: application/json
{
    "credentialInfo": true,
    "certificates": "chain",
    "certInfo": true,
    "authInfo": true
}
```

**Sample Response**:

```http request
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "credentialIDs": [
    "GX0112348",
    "HX0224685"
  ],
  "credentialInfos": [
    {
      "credentialID": "GX0112348",
      "key": {
        "status": "enabled",
        "algo": [
          "1.2.840.113549.1.1.11",
          "1.2.840.113549.1.1.10"
        ],
        "len": 2048
      },
      "cert": {
        "status": "valid",
        "certificates": [
          "<Base64-encoded_X.509_end_entity_certificate>",
          "<Base64-encoded_X.509_intermediate_CA_certificate>",
          "<Base64-encoded_X.509_root_CA_certificate>"
        ],
        "issuerDN": "<X.500_issuer_DN_printable_string>",
        "serialNumber": "5AAC41CD8FA22B953640",
        "subjectDN": "<X.500_subject_DN_printable_string>",
        "validFrom": "20200101100000Z",
        "validTo": "20230101095959Z"
      },
      "auth": {
        "mode": "explicit",
        "expression": "PIN AND OTP",
        "objects": [
          {
            "type": "Password",
            "id": "PIN",
            "format": "N",
            "label": "PIN",
            "description": "Please enter the signature PIN"
          },
          {
            "type": "Password",
            "id": "OTP",
            "format": "N",
            "generator": "totp",
            "label": "Mobile OTP",
            "description": "Please enter the 6 digit code you received by SMS"
          }
        ]
      },
      "multisign": 5,
      "lang": "en-US"
    }
  ]
}
```

After the listing of the credentials, the Signing Service can determine the one to be used through an internal policy, show a credential selection screen to the user or, if provided, use the `credential_id` claim inside the QESAC to identify the credential to be used.

**credentials/info (optional)**:

SSPs can utilize the `credentials/info` endpoint to receive info about a specific credential:

**Sample Request**:
```http request
POST /csc/v2/credentials/info HTTP/1.1
Host: rqes.example.com
Authorization: Bearer ...
Content-Type: application/json
{
    "credentialID": "GX0112348",
    "certificates": "chain",
    "certInfo": true,
    "authInfo": true
}
```

The actual process of the certificate selection is not detailed in this RFC, as different Signing Services might use different methods for certificate labeling and mapping to User data.

> **Note**: Authentication to the RQES Provider is out of scope. Implementors will need to follow the CSC API Spec Guidelines for Service Authentication & Authorization. The user might need to be redirected to the RQES Provider to complete authorization.

## 4.4: Phase 4: Signature Confirmation & Private Key Unlocking (Credential Authorization)

During this step of the process, the Signing of the Signer's Document (SD) must be confirmed by the user and the Private Key of the User's Certificate will need to be unlocked (authorized for use), in order to obtain the `Signature Activation Data (SAD)`.

### 4.4.1: Signing Confirmation

In order to confirm that the signature approval is a willful act, a second PID and QESAC Presentation must be requested by the Signing Service:

```mermaid
   sequenceDiagram
  participant User
  participant EUDI Wallet
  participant Signing Service

  Signing Service->>User: PID, QESAC Presentation Request via OID4VP
  EUDI Wallet->>Signing Service: PID, QESAC Presentation
```

During this step, the Signing Service must confirm that the PID and QESAC attributes presented exactly match the PID, QESAC attributes presented in Phase 2. Any deviation should result in the immediate termination of the process.

### 4.4.2: Private Key Unlocking (Credential Authorization)

The Signing Service will need to parse the `auth.mode` object of the user's credential to determine the mode of credential authorization (see `credentials/list` response example in Phase 3). Credential Authorization can support either of the following methods, according to the CSC API Spec:

#### Authorization Code Flow (oauth2code):

If the auth mode is set to follow the **OAuth2 Authorization Code Flow**, the Signing Service will need to redirect the user to the RQES Provider's
`oauth2/authorize` and the `oauth2/token` endpoints, as defined by [RFC-6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1) and while following the procedure in the CSC API v2 Spec.

**oauth2/authorize:**

```http request
GET https://rqes.example.com/oauth2/authorize?
    response_type=code&
    client_id=<OAuth2_client_id>&
    redirect_uri=<OAuth2_redirect_uri>&
    scope=credential&
    code_challenge=K2-ltc83acc4h0c9w6ESC_rEMTJ3bww-uCHaoeK1t8U&
    code_challenge_method=S256&
    credentialID=GX0112348&
    numSignatures=1&
    hashes=MTIzNDU2Nzg5MHF3ZXJ0enVpb3Bhc2RmZ2hqa2zDtnl4&
    hashAlgorithmOID=2.16.840.1.101.3.4.2.1&state=12345678
```

**oauth2/token:**

```http request
POST https://rqes.example.com/oauth2/token HTTP/1.1
Host: www.domain.org
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=FhkXf9P269L8g&
client_id=<OAuth2_client_id>&
client_secret=<OAuth2_client_secret>&
redirect_uri=<OAuth2_redirect_uri>
```

#### Explicit Flow (explicit):

In the case of `explicit` credential authorization, the Signing Service will need to parse the `expression` parameter of the respective
credential and present the required authorization prompts to the User (for example, a PIN prompt).

For each step of the authorization, the specific CSC API endpoints will need to be queried by the Signing Service (for example, the
`credentials/getChallenge` endpoint, to receive an OTP).

**credentials/authorize:**

After the respective input from the user has been collected, the `credentials/authorize` endpoint can be queried by the Signing Service
to finalize the authorization process:

```mermaid
%% Signature Metadata Endpoint
  sequenceDiagram
    participant Signing Service
    participant RQES Provider
    Signing Service->>RQES Provider: POST /csc/v2/credentials/authorize
    RQES Provider->>Signing Service: SAD
```

```http request
POST /csc/v2/credentials/authorize HTTP/1.1
Host: rqes.example.com
Authorization: Bearer ...
Content-Type: application/json
{
  "credentialID": "GX0112348",
  "numSignatures": 1,
  "hashes": [
    "sTOgwOm+474gFj0q0x1iSNspKqbcse4IeiqlDg/HWuI="
  ],
  "hashAlgorithmOID": "2.16.840.1.101.3.4.2.1",
  "authData": [
    {
      "id": "PIN",
      "value": "123456"
    },
    {
      "id": "OTP",
      "value": "738496"
    }
  ]
}
```

> Note: The hash of the document should be passed onto the authorization request, to bind to the SAD to the hash to be signed, as to not be able to be used to sign a different content.

**Sample Response**:

```http request
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "SAD": "_TiHRG-bAH3XlFQZ3ndFhkXf9P24/CKN69L8gdSYp5_pw"
}
```

## 4.5 Phase 5: Signature Creation

After the SAD has been obtained by authorizing the credential for use, the SAD can be used to sign the document's hash (Data To Be Signed Representation, DTBSR) using the `signature/signHash` endpoint of the RQES Provider. The endpoint must respond with the Signed Hash (DSV, Digital Signature Value).

****signatures/signHash:****

```mermaid
%% Signature Metadata Endpoint
  sequenceDiagram
    participant Signing Service
    participant RQES Provider
    Signing Service->>RQES Provider: POST /csc/v2/signatures/signHash
    RQES Provider->>Signing Service: Signed Hash
```

```http request
POST /csc/v2/signatures/signHash HTTP/1.1
Host: rqes.example.com
Content-Type: application/json
Authorization: Bearer 4/CKN69L8gdSYp5_pwH3XlFQZ3ndFhkXf9P2_TiHRG-bA

{
  "credentialID": "GX0112348",
  "SAD": "_TiHRG-bAH3XlFQZ3ndFhkXf9P24/CKN69L8gdSYp5_pw",
  "hashes": [
    "sTOgwOm+474gFj0q0x1iSNspKqbcse4IeiqlDg/HWuI="
  ],
  "hashAlgorithmOID": "2.16.840.1.101.3.4.2.1",
  "signAlgo": "1.2.840.113549.1.1.1",
  "clientData": "12345678"
}
```

**Sample Response:**

```json
{
  "signatures": [
    "KedJuTob5gtvYx9qM3k3gm7kbLBwVbEQRl26S2tmXjqNND7MRGtoew=="
  ]
}
```

## 4.6 Phase 6: Signed Document Formation and Retrieval

After the signing of the hash (SDR) has been completed, the AdES digital signature can be formed and the final document can be sent back to the Service Provider for further handling and/or for download by the user.

The transfer of the document to the Service Provider is out of scope of this RFC.

# 5. Reference:

1. OpenID Foundation (2023), 'OpenID for Verifiable Presentations (OID4VP)', Available at: [https://openid.net/specs/openid-4-verifiable-presentations-1_0-ID2.html](https://openid.net/specs/openid-4-verifiable-presentations-1_0-ID2.html)
2. European Commission (2023) The European Digital Identity Wallet Architecture and Reference Framework (2023-04, v1.1.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases)
3. Cloud Signing Consortium API Specification v2 (2023), Available at: [https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf)
4. ETSI TS 119 432 V1.2.1 (2020), Available at: [https://www.etsi.org/deliver/etsi_ts/119400_119499/119432/01.02.01_60/ts_119432v010201p.pdf](https://www.etsi.org/deliver/etsi_ts/119400_119499/119432/01.02.01_60/ts_119432v010201p.pdf)
