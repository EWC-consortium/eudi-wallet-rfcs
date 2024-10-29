# RFC-010 Synchronous Document Signing on a Remote Signing Service Provider using Long-Term Certificates and the EUDI Wallet - Draft 1

## 1.0 Summary:

This Specification defines the procedures for using the EUDI wallet to digitally sign a document, using Long-Term certificates, on a Remote Signing Service Provider (SSP). It specifies a new protocol for executing digital signing, that will ensure successful communication and interoperability with the EUDI wallet, Signing Service Providers, and will support multiple Remote QES (RQES) Providers as per the ARF specification.

### Authors:

- Mr. Kyriakos Giannakis (Intesi Group, Italy | Flare, Greece)

## 2.0: Motivation:

The primary motivation for this specification is to establish a standardized protocol for authorizing the digital signing of documents using the EUDI wallet and Long-Term certificates, on a Remote Signing Service Provider. 
At present, there are no standardized procedures and interfaces for digitally presenting a document to the signer using the EUDI wallet, no procedures for authenticating the signing party to the Signing Signing Service Provider.

## 2.1 Changelog:

- Jul. 19 2024: Initialization of authoring process. Initial draft.
- Sep. 10 2024: Addition of multiple RQES service support through RQES Access VC concept. Further Enrichment of RFC.
- Oct 1 2024: Addition of SSP discovery endpoint. Addition of RQESAC token example. Misc corrections.
- Oct 18, 2024: Addition of Pre-enrollment process. Addition of CSC API call examples on `credentials/list`, `credentials/info`. Refinement of RQESAC. Changes to Signing Process Metadata endpoint to support multiple credentials. Addition of credential selection screen.
- Oct 28, 2024: Addition of credential authorization. Addition of `credentials/info` example and required attributes.
- Oct 29, 2024: Addition of the Signature Creation step. Marking of the Pre-Enrollment step as optional (moved to Annex 2) and restoration of Client Credentials within the RQESAC.

## 3.0 The Signing Architecture:

The architecture covered in this specification follows the process of remotely signing a document using long-term certificates, handled by a Remote QES (or AES) Service, as detailed in D4.8.

The architecture will be broken down in 4 main phases:
1. Phase 1: User Authentication & RQES Delegation
2. Phase 2: Signing Initialization
3. Phase 3: Signature Creation
4. Phase 4: Signature Confirmation, Dispatching of Signature to SSP, Final Document Retrieval and Storage

**Remote QES services shall adhere to the [CSC (Cloud Signature Consortium)](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf) specifications that are also the basis for the JSON part of the ETSI TS 119 432 standard on protocols for remote digital signature creation.**

## 3.1 Signing Process:

```mermaid
   sequenceDiagram
    participant EUDI Wallet
    participant Signing Service Provider
    participant Delegated RQES
    Note right of Delegated RQES: 1. The user should not be presented with a lot of authorization interfaces. They should be authenticated strongly only once, as to not hamper the experience.
    Note over EUDI Wallet, Delegated RQES: Phase 1: Authentication, Delegated RQES Delegation, Initialization

    EUDI Wallet->>Signing Service Provider: PID
    Signing Service Provider->>EUDI Wallet: User is Authenticated
    EUDI Wallet->>Signing Service Provider: RQES Delegation VC (as to support multiple Delegated RQES providers users may have)
    Signing Service Provider->>Delegated RQES: GET credentials/list (userId: 1234, through RQES Delegation VC).
    Delegated RQES->>Signing Service Provider: Credential List
    Signing Service Provider->>Signing Service Provider: Serv. Prov. forms list of Signature Initialization Requests
    Signing Service Provider->>EUDI Wallet: SIRs shown to user (eg through browser)

    EUDI Wallet->>EUDI Wallet: User launches signature initialization URI (QR Code/Deep Link)
    Note over EUDI Wallet,Signing Service Provider: User decides to sign
    EUDI Wallet->>Signing Service Provider: GET Signature Metadata Endpoint
    Signing Service Provider->>Delegated RQES: GET `credential/info` {credentialId: 123412341234134}
    Delegated RQES->>Signing Service Provider: Credential Info Retrieved from Delegated RQES
    Signing Service Provider->>EUDI Wallet: Document Metadata Passed to User
    EUDI Wallet->>EUDI Wallet: User Reviews Document
    EUDI Wallet->>EUDI Wallet: User Authorizes Signing
    EUDI Wallet->>Delegated RQES: Credential Authorization
    EUDI Wallet->>Delegated RQES: POST credential/authorize { credentialId: 123412341234134 }
    Delegated RQES->>EUDI Wallet: SAD
    EUDI Wallet->>Delegated RQES: POST signatures/signHash OR signatures/signDoc { SAD: ... }
    Delegated RQES->>EUDI Wallet: Signatures
    EUDI Wallet->>Signing Service Provider: Signatures OK { signatures: [...] }
```
Figure 1: Signing Procedure diagram.

### 3.1.1 Phase 1: User Authentication & RQES Delegation

#### Overview:

```mermaid
%% Example of sequence diagram
  sequenceDiagram
    participant EUDI Wallet
    participant Signing Service Provider
    participant RQES Provider
    Signing Service Provider->>EUDI Wallet: Presentation Request for PID and RQESAC
    EUDI Wallet->>Signing Service Provider: PID, RQESAC
    Signing Service Provider->>RQES Provider: Service Authorization
    RQES Provider->>Signing Service Provider: Credentials Listing
```

##### 3.1.1.1: User Authentication & Multiple RQES Service Support

The user is authenticated through a Presentation Request requesting 2 credentials:

1. The User's PID or LPID, which contains the user's personal (or business) identification record.
2. An RQES Access Credential (RQESAC): Serves as an access and configuration credential to the user's RQES service. Through the presented RQES Access Credential, the user can choose their preferred RQES service. Example provided in Annex A.

##### 3.1.1.2: Service Authentication of the SSP, Listing of the User's Credentials and SRU Preparation:

After User Authentication and RQESAC Verification, the Signing Service Provider must then query the user's preferred RQES Provider to request the user's available credentials and construct the final Signing Request URIs.

The Signing Service Provider must query the RQES service using the `rqes_provider.api` parameter to acquire information about the service and its supported authentication methods.

Authentication must be done using the **OAuth2 Client Credentials** located inside the RQESAC (see Annex 1). Should the `service_auth` object not be present inside the RQESAC, the SSP must be pre-enrolled with the RQES Provider, to proceed (see Annex 2).

**credentials/list**

```mermaid
%% credentialList
  sequenceDiagram
    participant Signing Service Provider
    participant RQES Provider
    Signing Service Provider->>RQES Provider: POST /csc/v2/credentials/list
    RQES Provider->>Signing Service Provider: { credentialIDs: [...], credentialInfos: [...] }
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

### 3.1.2 Phase 2: Signing Initialization

#### Overview

```mermaid
%% Example of sequence diagram
  sequenceDiagram
    participant Browser
    participant EUDI Wallet
    participant Signing Service Provider
    Signing Service Provider->>Browser: Signing Request URIs (SRUs) 
    Browser->>Browser: SRU Launch
    EUDI Wallet->>Signing Service Provider: GET Signature Request Metadata
    EUDI Wallet->>Signing Service Provider: GET Document for Preview
    EUDI Wallet->>EUDI Wallet: Credential Validation
    EUDI Wallet->>EUDI Wallet: Credential Selection by User (Optional)
```

#### 3.1.2.1: Signing Process Initialization through Signature Request URI (SRU)

After service authentication and authorization, the Signing Service Provider can form the "Signature Request" URIs (SRU), responsible for **initiation of the signing process**.

Each SRU contains a reference to the Signing Service Provider's **Signature Metadata Endpoint** with a token, authenticating the user and their credential. Each token MUST be bound to the user's profile and their specific credentials,
obtained through the CSC compatible endpoints `credentials/list` and `credentials/info`.

A **one-time-use, secret token** is embedded in the URL, to authenticate the user and to bind the signature to the user and their credentials. Signing services should keep track of these tokens and delete them after some time being unused. This token should not have any other purpose and must be kept secret from other users.

The signing process is initiated by the user either clicking a link or scanning a QR code with a SRU, provided by the Signing Service Provider, after acquiring the available credentials of the user.

Sample Signature Request URI:

```
eudi-sig-request://?signature_url=https://signing_server_url/signing?token=<signature_access_token>
```

#### 3.1.2.3 Signing Process Metadata:

The EUDI Wallet executes the following GET request to obtain the metadata about the signing process and to draw the final preview and approval UI, to show the user.

```mermaid
%% Signature Metadata Endpoint
  sequenceDiagram
    participant EUDI Wallet
    participant Signing Service Provider
    EUDI Wallet->>Signing Service Provider: GET https://signing_server_url/signing?token=<signature_access_token>
    Signing Service Provider->>EUDI Wallet: Signing Process Metadata
```

```
GET https://signing_server_url/signing?token=<signature_access_token>
```

```http request
GET /signing?token=<signature_access_token> HTTP/1.1
Host: signing_server_url
Content-Type: "application/json"
{
   "document_id": "bf1d1e65-b8cd-4f88-b334-18740380ca38",
   "document_url": "https://...",
   "document_type": "pdf",
   "document_hash": "lk1j23h45l34jkth234...",
   "access_type": "sign",
   "author_details": {
      "given_name": "Kyriakos",
      "family_name": "Giannakis",
      "phone_number": "+30695...",
      "affiliated_with": "IntesiGroup",
      "affiliate_url": "https://www.intesigroup.com/en/"
   },
   "signature_request_metadata": {
      "created_at": "2024-07-18T15:13:56Z",
      "expires_at": "2024-07-18T15:25:56Z",
      "reason": "Please sign this NDA to gain demo access to IG Sign",
      "signing_alg": "RS256",
      "hashing_alg": "SHA-256"
   },
   "document_pages_no": 4,
   "signature_fields": [
      {
         "id": "7fd8bd7b-be35-4f0a-8edc-9d11f159a032",
         "page_number": 4
      }
   ],
   "rqes_provider": {
      "title": "Intesi Group",
      "homepage": "https://...",
      "title_short": "INTESI_GRP_ITA",
      // more details TBD
   },
   "credential_info": [
     {
       // Credential 1
     },
     {
       // Credential 2
     }
   ]
}
```

**`credential_info` Objects**:

Objects inside the `credential_info` list follow the output format of the `credentials/info` endpoint, as denoted on the CSC API v2. 

#### 3.1.2.4: Required Attributes for Supported Credentials:

Credentials need to have the following attributes to be supported for signing:

- `key/status`: `enabled`
- `TBA: Key Algo`
- `cert`:
  - `status`: `valid`
- `auth`:
  - `mode`: `explicit` | `oauth2code`
  - `objects`: REQUIRED if using `explicit` auth mode

> TBA Restrictions on algo use?

#### 3.1.2.5: Credential Selection:

Should the user own more than one credential, the wallet will need to present the user with a selection screen for the user to pick
the credential they wish to use to sign the document.

#### 3.1.2.6: What You See is What You Sign (WYSIWYS):

The EUDI Wallet app can use the attributes of the metadata response to provide a WYSIWYS (What You See Is What You Sign) preview to the user, helping them visualize the final document. 

> Author's Note: More needs TBA regarding signature preview support.

### 3.1.3 Phase 3: Signature Creation

#### Overview

```mermaid
  sequenceDiagram
    participant EUDI Wallet
    participant RQES Provider
    EUDI Wallet->>EUDI Wallet: User Accepts the Signing
    EUDI Wallet->>RQES Provider: Credential Authorization
    RQES Provider->>EUDI Wallet: SAD
    EUDI Wallet->>RQES Provider: Signature Request
    RQES Provider->>EUDI Wallet: Signatures
```


#### 3.1.3.1: Signature Creation

The user can accept the signing of a document using the corresponding acceptance button on their wallet. Upon acceptance, a
series of steps is performed.

#### 3.1.3.2: Credential Authorization

Depending on the `auth/mode` attribute of the credential, **the wallet will need to follow a specific flow to authorize the
credential**. The credential authorization will need to happen on the wallet, as the SSP cannot be trusted with the sensitive
credentials of the user (eg. their PIN).

##### Authorization Code Flow (oauth2code):

If the auth mode is set to follow the **OAuth2 Authorization Code Flow**, the wallet will need to redirect to the RQES Provider's
`oauth2/authorize` and the `oauth2/token`, as defined by [RFC-6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1) and while following the procedure in the CSC API v2 Spec.

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

> Author's Note: There's some questions to be answered reg. Auth Code Flow, that might inhibit the adoption:
> - What is going to be the Client ID? How is the RQES provider going to be aware of the wallet's Client ID prior to the request?
> - Wallet providers will need to prepare for this process and build the required mechanisms, which might prove slow.

MORE TBA

##### Explicit Flow (explicit):

In the case of `explicit` credential authorization, the Wallet will need to parse the `expression` parameter of the respective
credential and present the required authorization prompts (for example, a PIN prompt).

For each step of the authorization, the specific CSC API endpoints will need to be queried by the Wallet (for example, the
`credentials/getChallenge` endpoint, to receive an OTP).

After the respective input from the user has been collected, the `credentials/authorize` endpoint can be queried by the wallet
to enable the finalize the authorization process:

```mermaid
%% Signature Metadata Endpoint
  sequenceDiagram
    participant EUDI Wallet
    participant RQES Provider
    EUDI Wallet->>RQES Provider: POST /csc/v2/credentials/authorize
    RQES Provider->>EUDI Wallet: SAD
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

#### 3.1.3.3: Signature Creation:

After the successful authorization of the user's credential and the retrieval of the SAD, the `signatures/signHash` endpoint of the CSC Compatible API of the RQES Provider can be used to sign the document's hash:

```mermaid
%% Signature Metadata Endpoint
  sequenceDiagram
    participant EUDI Wallet
    participant RQES Provider
    EUDI Wallet->>RQES Provider: POST /csc/v2/signatures/signHash
    RQES Provider->>EUDI Wallet: Document Signatures
```

```http request
POST /csc/v2/signatures/signHash HTTP/1.1
Host: service.domain.org
Content-Type: application/json
Authorization: Bearer 4/CKN69L8gdSYp5_pwH3XlFQZ3ndFhkXf9P2_TiHRG-bA
{
  "credentialID": "GX0112348",
  "SAD": "_TiHRG-bAH3XlFQZ3ndFhkXf9P24/CKN69L8gdSYp5_pw",
  "hashes": [
    "sTOgwOm+474gFj0q0x1iSNspKqbcse4IeiqlDg/HWuI=",
  ],
  "hashAlgorithmOID": "2.16.840.1.101.3.4.2.1",
  "signAlgo": "1.2.840.113549.1.1.1"
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

The wallet can then send the Signature to the SSP, to be attached to the document and for the final document to be created.

### 3.1.4 Phase 4: Signature Confirmation, Dispatching of Signature to SSP, Final Document Retrieval and Storage

TBA

## Annex 1: RQES Access Credential Schema and Example

### Example:

```json
{
  "id": "76b0184c-ac8e-4484-a9e1-9f0a0d68fe0b",
  "holder_name": "Kyriakos Giannakis",
  "rqes_provider": {
    "id": "2cea80dd-18b8-4e6c-9964-f11fc0bc4423",
    "api": "https://services.test4mind.com/csc/v1/",
    "title": "Intesi Group SPA",
    "title_short": "INTESI_GRP_ITA",
    "location": "Milan, IT"
  },
  "service_auth": {
    "grant": "client_credentials",
    "client_id": "...",
    "client_secret": "..."
  }
}
```

## Annex 2: RQES Pre-Enrollment

Should an RQES provider not choose to provide the `Service Authorization` Client Credentials within the RQES Access Credential, the SSP must be pre-enrolled with the RQES Provider.

Pre-enrollment is facilitated through a contractual agreement, outside the scope of this RFC.

Upon pre-enrollment, **OAuth2 Client Credentials** for access to the [CSC Specification Compatible API (v2)](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf) exposed by the RQES provider are issued to the SSP, to facilitate communication between the parties.


### Schema:

TBD