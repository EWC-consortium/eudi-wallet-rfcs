# EWC RFC001: Issue Verifiable Credential - v3.0

**Authors:**
* Mr George Padaytti (iGrant.io, Sweden)
* Mr Lal Chandran (iGrant.io, Sweden)
* Dr Andreas Abraham (ValidatedID, Spain)

**Reviewers:**
* Dr Nikos Triantafyllou (University of the Aegean, Greece)
* Mr Florin Coptil (Bosch, Germany)
* Mr Matteo Mirabelli (Infocert, Italy)
* Dr Mikael Linden (Vero, Finland)
* Mr Renaud Murat (Archipels, France)
* Mr. Sebastian Bickerle (Lissi ID, Germany)
* Mr. Quentin Drouot (Archipels, France)
* Mr. Edward Curran (Lissi ID, Germany)
* Mr. Björn Astrom (BankID, Sweden)
* Mr. Björn Molin (DIGG, Sweden)
* Mr. Pär W (BankID, Sweden)

**Status:** Approved

**Table of Contents**

- [EWC RFC001: Issue Verifiable Credential - v3.0](#ewc-rfc001-issue-verifiable-credential---v30)
- [1.0 Summary](#10-summary)
- [2.0 Motivation](#20-motivation)
- [3.0 Overview of Endpoints](#30-overview-of-endpoints)
- [4.0 Core Concepts](#40-core-concepts)
  - [4.1 Holder Binding](#41-holder-binding)
  - [4.2 Identifying Credentials Throughout the Flow](#42-identifying-credentials-throughout-the-flow)
- [5.0 Flows](#50-flows)
  - [5.1 Authorization Code Flow](#51-authorization-code-flow)
  - [5.2 Pre-Authorized Code Flow](#52-pre-authorized-code-flow)
- [6.0 Messages and Endpoints](#60-messages-and-endpoints)
  - [6.1 Credential Offer](#61-credential-offer)
    - [6.1.1 Credential Offer Parameters](#611-credential-offer-parameters)
    - [6.1.2 Credential Offer Request (by Reference)](#612-credential-offer-request-by-reference)
    - [6.1.3 Credential Offer Response](#613-credential-offer-response)
  - [6.2 Issuer and Authorization Server Metadata Discovery](#62-issuer-and-authorization-server-metadata-discovery)
    - [6.2.1 Discovery Request](#621-discovery-request)
    - [6.2.2 Credential Issuer Metadata Response](#622-credential-issuer-metadata-response)
    - [6.2.3 Authorization Server Metadata Response](#623-authorization-server-metadata-response)
  - [6.3 Authorization Request (Authorization Endpoint)](#63-authorization-request-authorization-endpoint)
    - [6.3.1 Using `authorization_details`](#631-using-authorization_details)
    - [6.3.2 Using `scope`](#632-using-scope)
    - [6.3.3 Additional Authorization Request Parameters](#633-additional-authorization-request-parameters)
  - [6.4 Authorization Response (Authorization Endpoint)](#64-authorization-response-authorization-endpoint)
    - [6.4.1 Successful Response](#641-successful-response)
    - [6.4.2 Error Response](#642-error-response)
  - [6.5 Token Request (Token Endpoint)](#65-token-request-token-endpoint)
    - [6.5.1 Authorization Code Flow Token Request](#651-authorization-code-flow-token-request)
    - [6.5.2 Pre-authorized Code Flow Token Request](#652-pre-authorized-code-flow-token-request)
    - [6.5.3 Client Authentication](#653-client-authentication)
  - [6.6 Token Response (Token Endpoint)](#66-token-response-token-endpoint)
    - [6.6.1 Successful Response (with `authorization_details`)](#661-successful-response-with-authorization_details)
    - [6.6.2 Successful Response (with `scope`)](#662-successful-response-with-scope)
    - [6.6.3 Token Error Response](#663-token-error-response)
  - [6.7 Credential Request (Credential Endpoint)](#67-credential-request-credential-endpoint)
    - [6.7.1 Credential Request Parameters](#671-credential-request-parameters)
    - [6.7.2 Proof of Possession](#672-proof-of-possession)
    - [6.7.3 Proof Types](#673-proof-types)
    - [6.7.4 Credential Request Examples](#674-credential-request-examples)
  - [6.8 Credential Response (Credential Endpoint)](#68-credential-response-credential-endpoint)
    - [6.8.1 Immediate Credential Response](#681-immediate-credential-response)
    - [6.8.2 Deferred Credential Response](#682-deferred-credential-response)
    - [6.8.3 Credential Error Response](#683-credential-error-response)
  - [6.9 Nonce Request (Nonce Endpoint)](#69-nonce-request-nonce-endpoint)
  - [6.10 Nonce Response (Nonce Endpoint)](#610-nonce-response-nonce-endpoint)
  - [6.11 Deferred Credential Request (Deferred Credential Endpoint)](#611-deferred-credential-request-deferred-credential-endpoint)
  - [6.12 Deferred Credential Response (Deferred Credential Endpoint)](#612-deferred-credential-response-deferred-credential-endpoint)
  - [6.13 Deferred Credential Error Response](#613-deferred-credential-error-response)
  - [6.14 Notification Request (Notification Endpoint)](#614-notification-request-notification-endpoint)
  - [6.15 Notification Response (Notification Endpoint)](#615-notification-response-notification-endpoint)
- [7.0	Implementers](#70implementers)
- [8.0	Reference](#80reference)
- [Appendix A: Public key resolution](#appendix-a-public-key-resolution)

# 1.0 Summary

This specification implements OID4VCI workflow for any issuer as per reference specification [1] (OpenID for Verifiable Credential Issuance - Draft 15). This minimises risks towards interoperability across the European Wallet Ecosystem with a standard specification in the EUDI wallet ecosystem as per the ARF [2] requirements.

# 2.0 Motivation

The EWC LSP must align with the standard protocol for issuing credentials. This is the basis of interoperability between Issuers and Holders across the EWC LSPs. The assumption is that the user is familiar with the EWC-chosen protocols and standards and can refer to original standards references when necessary.

# 3.0 Overview of Endpoints

The OID4VCI specification defines an API for Credential issuance provided by a Credential Issuer, potentially interacting with an Authorization Server. The API comprises the following endpoints:

*   **Credential Endpoint:** Issues the Verifiable Credential(s).
*   **Token Endpoint:** Issues Access Tokens.
*   **Authorization Endpoint:** Handles End-User authentication and consent.
*   **Credential Offer Mechanism:** Allows the Issuer to initiate the flow.
*   **Nonce Endpoint:** Provides fresh nonces for proof binding.
*   **Deferred Credential Endpoint:** Handles issuance when processing takes time.
*   **Notification Endpoint:** Allows the Wallet to notify the Issuer about the status of issued Credentials.
*   **Metadata Endpoints:** Publish Issuer and AS capabilities.

# 4.0 Core Concepts

## 4.1 Holder Binding

Issued Credentials SHOULD be bound to the End-User (Holder) possessing them. This allows Verifiers to confirm that the presenter is the legitimate owner.

**Cryptographic Binding:** Proving control over a private key linked to a public key in the Credential, typically via a `proof` in the Credential Request.

## 4.2 Identifying Credentials Throughout the Flow

The specific Credential(s) being requested/issued are identified at different stages:

1.  **Credential Offer:** The Issuer uses `credential_configuration_ids` to identify offered configurations listed in its metadata.
2.  **Authorization Request:**
    *   If using `authorization_details`, the Wallet uses `credential_configuration_id` or `format` (+ format-specific params) within the `openid_credential` object. [4]
    *   If using `scope`, the scope value(s) identify the requested configuration(s).
3.  **Token Response:**
    *   If `authorization_details` was used in the *Authorization Request*, the AS MUST return `authorization_details` containing `credential_identifiers`.
    *   If `scope` was used, the AS MAY return `credential_identifiers` in `authorization_details`.
4.  **Credential Request:**
    *   If `credential_identifiers` were received in the Token Response, the Wallet MUST use the `credential_identifier` parameter for each instance.
    *   Otherwise (scope used, no identifiers returned), the Wallet MUST use the `credential_configuration_id` parameter.

# 5.0 Flows

The credential issuance can use an Authorization Code flow or a Pre-Authorized Code flow.

## 5.1 Authorization Code Flow

The standard OAuth 2.0 Authorization Code flow is used, involving redirection via the user agent for authentication and consent.

```mermaid
sequenceDiagram
  autonumber
  participant W as Wallet (Client/Holder)
  participant AS as Authorization Server
  participant CI as Credential Issuer (Resource Server)

  W->>CI: (1a/1b) Initiate Flow (via Credential Offer or direct interaction)
  W->>CI: (2) GET /.well-known/openid-credential-issuer (+ optional AS metadata)
  CI-->>W: Issuer Metadata (incl. AS URI, Credential Endpoint)

  W->>AS: (3) Authorization Request (via User Agent redirect or PAR)
  AS->>W: (User Interaction: Authentication/Consent)
  AS-->>W: (4) Authorization Response (with code) (via User Agent redirect)

  W->>AS: (5) POST Token Request (incl. code, PKCE verifier)
  AS-->>W: Token Response (incl. Access Token, optionally c_nonce, credential_identifiers)

  W->>CI: (6) POST Credential Request (incl. Access Token, proof)
  CI-->>W: Credential Response (Credential(s) or transaction_id)
```
Figure 1: Issuance using Authorisation Code Flow based on [1]

## 5.2 Pre-Authorized Code Flow

This flow bypasses the Authorization Endpoint. The Issuer handles End-User authentication/consent out-of-band and provides a `pre-authorized_code` in the Credential Offer. The Wallet exchanges this directly for an Access Token.

```mermaid
sequenceDiagram
    autonumber
    participant W as Wallet (Client/Holder)
    participant CI as Credential Issuer
    participant AS as Authorization Server

    CI->>W: (1) Credential Offer (incl. pre-authorized_code, optional tx_code info)

    W->>CI: (2) GET /.well-known/openid-credential-issuer (+ optional AS metadata)
    CI-->>W: Issuer Metadata (incl. AS URI, Token Endpoint, Credential Endpoint)

    W->>AS: (3) POST Token Request (incl. pre-authorized_code, optional tx_code, client_id if needed)
    AS-->>W: Token Response (incl. Access Token, optionally c_nonce, credential_identifiers)

    W->>CI: (4) POST Credential Request (incl. Access Token, proof)
    CI-->>W: Credential Response (Credential(s) or transaction_id)
```
Figure 2: Issuance using Pre-Authorisation Code Flow based on [1]

# 6.0 Messages and Endpoints

## 6.1 Credential Offer

An optional mechanism for an Issuer to initiate issuance, often via a URI or QR code. The offer can be passed by value (`credential_offer`) or by reference (`credential_offer_uri`). Passing by reference is RECOMMENDED for QR codes.

### 6.1.1 Credential Offer Parameters

The JSON object representing the Credential Offer contains:

| Field                              | Req / Opt | Description                                                                                                                                                                                                            |
| :--------------------------------- | :-------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`credential_issuer`**            | REQUIRED  | URL of the Credential Issuer. Used to fetch Issuer Metadata.                                                                                                                                                           |
| **`credential_configuration_ids`** | REQUIRED  | Array of strings identifying offered configurations from the Issuer's metadata (`credential_configurations_supported` map key).                                                                                        |
| **`grants`**                       | OPTIONAL  | Object indicating supported Grant Types (`authorization_code`, `urn:ietf:params:oauth:grant-type:pre-authorized_code`). Contains grant-specific parameters (see below). If absent, Wallet determines from AS metadata. |

**Grant-Specific Parameters within `grants`:**

*   **For `authorization_code`:**
    | Field                      | Req / Opt | Description                                                                                                                      |
    | :------------------------- | :-------- | :------------------------------------------------------------------------------------------------------------------------------- |
    | **`issuer_state`**         | OPTIONAL  | Opaque string from Issuer to correlate the Authorization Request. If received, Wallet MUST send it in the Authorization Request. |
    | **`authorization_server`** | OPTIONAL  | Identifier of the AS to use if Issuer metadata lists multiple. MUST match an entry in the `authorization_servers` array.         |
*   **For `urn:ietf:params:oauth:grant-type:pre-authorized_code`:**
    | Field                      | Req / Opt | Description                                                                                                                                                                                                                                                                 |
    | :------------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | **`pre-authorized_code`**  | REQUIRED  | Short-lived, single-use code representing Issuer authorization. Wallet MUST send in the Token Request.                                                                                                                                                                      |
    | **`tx_code`**              | OPTIONAL  | Object describing requirements for a Transaction Code (user PIN/OTP) to prevent replay. If this object is present (even if empty), a `tx_code` MUST be sent in the Token Request. See OID4VCI [1] Section 4.1.1 for sub-parameters (`input_mode`, `length`, `description`). |
    | **`interval`**             | OPTIONAL  | Deprecated in draft 15, but previously indicated polling interval for the token endpoint in pending state.                                                                                                                                                                  |
    | **`authorization_server`** | OPTIONAL  | Identifier of the AS to use if Issuer metadata lists multiple. MUST match an entry in the `authorization_servers` array.                                                                                                                                                    |

### 6.1.2 Credential Offer Request (by Reference)

If the offer uses `credential_offer_uri`, the Wallet fetches the Credential Offer object.

```
GET /credential-offer HTTP/1.1
Host: server.example.com
Accept: application/json
```

The initiating URI often uses a custom scheme:
`openid-credential-offer://?credential_offer_uri=https://server.example.com/credential-offer`

### 6.1.3 Credential Offer Response

The Issuer responds with the JSON Credential Offer object.

**Example for Authorization Code Flow:**
```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "credential_issuer": "https://server.example.com",
  "credential_configuration_ids": [
    "UniversityDegreeCredential"
  ],
  "grants": {
    "authorization_code": {
      "issuer_state": "eyJhbGciOiJSU0Et...FYUaBy"
    }
  }
}
```

**Example for Pre-Authorized Code Flow (with tx_code):**
```json
HTTP/1.1 200 OK
Content-Type: application/json

{
   "credential_issuer": "https://server.example.com",
   "credential_configuration_ids": [
      "VerifiablePortableDocumentA1"
   ],
   "grants": {
      "urn:ietf:params:oauth:grant-type:pre-authorized_code": {
         "pre-authorized_code": "oaKazRN8I0IbtZ0C7JuMn5",
         "tx_code": {
            "length": 4,
            "input_mode": "numeric",
            "description": "Please provide the one-time code that was sent via e-mail or offline"
         }
      }
   }
}
```

> [!NOTE]
> Credential Offers by reference MUST return `application/json`. Credential Offers cannot be signed JWTs. (OID4VCI [1] Section 4.1.3, 4.2).

## 6.2 Issuer and Authorization Server Metadata Discovery

The Wallet discovers Issuer and AS capabilities via well-known endpoints.

### 6.2.1 Discovery Request

1.  Fetch Issuer Metadata: `GET {credential_issuer}/.well-known/openid-credential-issuer`
2.  Fetch AS Metadata: `GET {authorization_server}/.well-known/oauth-authorization-server` (where `{authorization_server}` is from Issuer Metadata or the `credential_issuer` URL itself if `authorization_servers` is absent).

```http
GET /.well-known/openid-credential-issuer HTTP/1.1
Host: server.example.com
Accept-Language: en-GB, en;q=0.9
```

```http
GET /.well-known/oauth-authorization-server HTTP/1.1
Host: server.example.com (or specific AS host)
```

### 6.2.2 Credential Issuer Metadata Response

Contains Issuer capabilities. Key parameters defined in OID4VCI Section [1] 11.2.3:

| Parameter                                 | Req / Opt | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| :---------------------------------------- | :-------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`credential_issuer`**                   | REQUIRED  | Issuer identifier URL.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **`authorization_servers`**               | OPTIONAL  | Array of AS identifiers used by this Issuer. If omitted, the Issuer acts as its own AS.                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **`credential_endpoint`**                 | REQUIRED  | URL of the Credential Endpoint.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **`deferred_credential_endpoint`**        | OPTIONAL  | URL of the Deferred Credential Endpoint.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **`nonce_endpoint`**                      | OPTIONAL  | URL of the Nonce Endpoint. Required if proofs needing `c_nonce` are supported.                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **`notification_endpoint`**               | OPTIONAL  | URL of the Notification Endpoint.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| **`display`**                             | OPTIONAL  | Array of language-specific Issuer display properties (name, locale, logo, etc.).                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| **`credential_configurations_supported`** | REQUIRED  | Object mapping configuration identifiers (e.g., "VerifiablePortableDocumentA1") to configuration details. Each configuration detail includes: `format` (REQUIRED), `scope` (OPTIONAL), `cryptographic_binding_methods_supported` (OPTIONAL), `credential_signing_alg_values_supported` (OPTIONAL), `proof_types_supported` (OPTIONAL), `display` (OPTIONAL), Format-specific parameters (REQUIRED/OPTIONAL), and `claims` (OPTIONAL - Array of claim description objects using `path` syntax, see OID4VCI Appendix B.2). |

**Example for SD-JWT VC Credential:**
```json
{
  "credential_issuer": "https://server.example.com",
  "authorization_servers": [
    "https://server.example.com"
  ],
  "credential_endpoint": "https://server.example.com/credential",
  "deferred_credential_endpoint": "https://server.example.com/credential_deferred",
  "display": [
    {
      "name": "Example Issuer",
      "locale": "en-US"
    }
  ],
  "credential_configurations_supported": {
    "IdentityCredential": {
      "format": "dc+sd-jwt",
      "scope": "IdentityCredential",
      "cryptographic_binding_methods_supported": [ "jwk" ],
      "credential_signing_alg_values_supported": [ "ES256" ],
      "proof_types_supported": {
         "jwt": {
             "proof_signing_alg_values_supported": [ "ES256" ]
         }
      },
      "display": [
        {
          "name": "Identity Credential",
          "locale": "en-US",
          "logo": {
            "uri": "https://university.example.edu/public/logo.png",
            "alt_text": "a square logo of a university"
          },
          "background_color": "#12107c",
          "text_color": "#FFFFFF"
        }
      ],
      "vct": "IdentityCredential",
      "claims": [
         {
           "path": ["given_name"],
           "display": [
             {"name": "Given Name", "locale": "en-US"},
             {"name": "Vorname", "locale": "de-DE"}
           ]
         },
         {
           "path": ["family_name"],
           "display": [
             {"name": "Surname", "locale": "en-US"},
             {"name": "Nachname", "locale": "de-DE"}
           ]
         },
         { "path": ["email"] },
         { "path": ["phone_number"] },
         {
           "path": ["address"],
           "display": [
             {"name": "Place of residence", "locale": "en-US"},
             {"name": "Wohnsitz", "locale": "de-DE"}
           ]
         },
         { "path": ["address", "street_address"] },
         { "path": ["address", "locality"] },
         { "path": ["address", "region"] },
         { "path": ["address", "country"] },
         { "path": ["birthdate"] }
       ]
    }
  }
}
```
**Example for ISO18013-5 mdl Credential:**
```json
{
  "credential_configurations_supported": {
    "credential_issuer": "https://server.example.com",
    "authorization_servers": [
      "https://server.example.com"
    ],
    "credential_endpoint": "https://server.example.com/credential",
    "deferred_credential_endpoint": "https://server.example.com/credential_deferred",
    "display": [
      {
        "name": "Example Issuer",
        "locale": "en-US"
      }
    ],
    "eu.europa.ec.eudi.pid.1": {
      "format": "mso_mdoc",
      "doctype": "eu.europa.ec.eudi.pid.1",
      "cryptographic_binding_methods_supported": [
        "cose_key"
      ],
      "credential_signing_alg_values_supported": [
        "ES256"
      ],
      "display": [
        {
          "name": "Mobile Driving License",
          "locale": "en-US",
          "logo": {
            "uri": "https://state.example.org/public/mdl.png",
            "alt_text": "state mobile driving license"
          },
          "background_color": "#12107c",
          "text_color": "#FFFFFF"
        },
        {
          "name": "Digitaler Führerschein",
          "locale": "de-DE",
          "logo": {
            "uri": "https://state.example.org/public/mdl.png",
            "alt_text": "digitaler Führerschein"
          },
          "background_color": "#12107c",
          "text_color": "#FFFFFF"
        }
      ],
      "claims": [
        {
          "path": ["org.iso.18013.5.1","given_name"],
          "display": [
            {
              "name": "Given Name",
              "locale": "en-US"
            },
            {
              "name": "Vorname",
              "locale": "de-DE"
            }
          ]
        },
        {
          "path": ["org.iso.18013.5.1","family_name"],
          "display": [
            {
              "name": "Surname",
              "locale": "en-US"
            },
            {
              "name": "Nachname",
              "locale": "de-DE"
            }
          ]
        },
        {
          "path": ["org.iso.18013.5.1","expiry_date"],
          "mandatory": true
        }
      ]
    }
  }
}
```

### 6.2.3 Authorization Server Metadata Response

Standard OAuth AS metadata ([RFC8414]), potentially including:

| Parameter                                             | Req / Opt   | Description                                                                                     |
| :---------------------------------------------------- | :---------- | :---------------------------------------------------------------------------------------------- |
| **`issuer`**                                          | REQUIRED    | Identifier of the Authorization Server.                                                         |
| **`authorization_endpoint`**                          | REQUIRED    | URL of the AS's OAuth 2.0 Authorization Endpoint. Required for Auth Code flow.                  |
| **`token_endpoint`**                                  | REQUIRED    | URL of the AS's OAuth 2.0 Token Endpoint.                                                       |
| **`jwks_uri`**                                        | REQUIRED    | URL of the AS's JSON Web Key Set [JWK] document. Required if using signed responses/tokens.     |
| **`pushed_authorization_request_endpoint`**           | RECOMMENDED | URL of the AS's Pushed Authorization Request Endpoint.                                          |
| **`require_pushed_authorization_requests`**           | OPTIONAL    | Boolean value specifying whether the AS requires PAR requests.                                  |
| **`scopes_supported`**                                | OPTIONAL    | JSON array containing a list of the OAuth 2.0 [RFC6749] scope values that this server supports. |
| **`response_types_supported`**                        | REQUIRED    | JSON array containing a list of the OAuth 2.0 response_type values that this server supports.   |
| **`token_endpoint_auth_methods_supported`**           | OPTIONAL    | JSON array containing a list of Client Authentication methods supported by this Token Endpoint. |
| **`pre-authorized_grant_anonymous_access_supported`** | OPTIONAL    | Boolean indicating if Pre-Auth Code flow works without client authentication. Default `false`.  |

```json
{
  "issuer": "https://server.example.com",
  "authorization_endpoint": "https://server.example.com/authorize",
  "pushed_authorization_request_endpoint": "https://server.example.com/par",
  "require_pushed_authorization_requests": true,
  "token_endpoint": "https://server.example.com/token",
  "jwks_uri": "https://server.example.com/.well-known/jwks.json",
  "response_types_supported": [
    "code",
    "vp_token",
    "id_token"
  ],
  "subject_types_supported": [
    "public",
    "pairwise"
  ],
  "id_token_signing_alg_values_supported": [
    "ES256"
  ],
  "pre-authorized_grant_anonymous_access_supported": true,
  "token_endpoint_auth_methods_supported": [
    "none"
  ]
}
```

## 6.3 Authorization Request (Authorization Endpoint)

Sent via user agent redirect or PAR (Pushed Authorization Request, RECOMMENDED). Requests access to the Credential Endpoint using either `authorization_details` or `scope`.

> [!NOTE]
> HAIP[6] chapter 4.2 mandatorily requires Pushed Authorisation Request as per [RFC9126] [7].

The following table describes the query parameters used in the Authorization Request:

| Parameter               | Req / Opt   | Description                                                                                                                                                                                                                                                                                                                                                                                                                   | Approach        |
| :---------------------- | :---------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------- |
| `response_type`         | REQUIRED    | MUST be `code` to request an authorization code. Other values like `id_token` or `vp_token` might be used for SIOPv2/OpenID4VP interactions during issuance (see Section 6.4).                                                                                                                                                                                                                                                | Common          |
| `client_id`             | REQUIRED    | The identifier for the Wallet (client) making the request.                                                                                                                                                                                                                                                                                                                                                                    | Common          |
| `redirect_uri`          | REQUIRED    | The redirection endpoint where the Authorization Server will send the User-Agent after authorization is complete. Although marked OPTIONAL in the input, it is REQUIRED by [RFC6749] for the code flow unless pre-registered and only one is registered.                                                                                                                                                                      | Common          |
| `code_challenge`        | REQUIRED    | The code challenge used for Proof Key for Code Exchange (PKCE) as specified in [RFC7636] [5].                                                                                                                                                                                                                                                                                                                                 | Common          |
| `code_challenge_method` | OPTIONAL    | The method used to transform the code verifier (`S256` or `plain`). Defaults to `plain`. RECOMMENDED to use `S256`, as defined in [RFC7636] [5].                                                                                                                                                                                                                                                                              | Common          |
| `state`                 | RECOMMENDED | Opaque value used by the client to maintain state between the request and callback, providing protection against CSRF.                                                                                                                                                                                                                                                                                                        | Common          |
| `scope`                 | REQUIRED    | Specifies the scope of access requested. Used to identify the requested Credential Configuration(s) when not using `authorization_details`. The value can be consistent across multiple credential configuration objects. The Authorization Server must uniquely identify the Credential Issuer based on the scope value. Scope values in Issuer metadata may overlap with those in the `scopes_supported` of the AS.         | Scope           |
| `authorization_details` | REQUIRED    | JSON array containing detailed authorization requests, including `openid_credential` type objects. Used instead of `scope` for fine-grained requests. See [RFC9396] [4]. <br/><br/> **W3C VC Example:** <br/>`[{"type":"openid_credential", "credential_configuration_id":"...", "credential_definition":{...}}]` <br/><br/> **SD-JWT VC Example:** <br/> `[{"type":"openid_credential", "format":"dc+sd-jwt", "vct":"..."}]`<br/><br/> **ISO18013-5 mDoc Example:** <br/> `[{"type":"openid_credential", "credential_configuration_id":"eu.europa.ec.eudi.pid.1", "format": "mso_mdoc", "doctype": "eu.europa.ec.eudi.pid.1", claims":"..."}]` | Authz Details   |
| `resource`              | RECOMMENDED | URI identifying the target Resource Server (Credential Issuer). SHOULD be included if the AS protects multiple Issuers (i.e., `authorization_servers` is present in Issuer metadata). See [RFC8707].                                                                                                                                                                                                                          | Scope (usually) |
| `issuer_state`          | OPTIONAL    | A string value representing a specific processing context at the Credential Issuer. Usually provided in a Credential Offer and passed back by the Wallet.                                                                                                                                                                                                                                                                     | Common          |
| `wallet_issuer`         | OPTIONAL    | String containing the Wallet's identifier. RECOMMENDED for Dynamic Credential Requests.                                                                                                                                                                                                                                                                                                                                       | Common          |
| `user_hint`             | OPTIONAL    | Opaque hint about the End-User. RECOMMENDED for Dynamic Credential Requests.                                                                                                                                                                                                                                                                                                                                                  | Common          |

### 6.3.1 Using `authorization_details`

Uses the `authorization_details` parameter (described in the table above) with one or more objects of `type: "openid_credential"`.

```http
GET /authorize?
  response_type=code
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &authorization_details=[{"type": "openid_credential", "
    credential_configuration_id": "VerifiablePortableDocumentA1"}]
  &redirect_uri=https://client.example.org/cb
Host: server.example.com
```

### 6.3.2 Using `scope`

Uses the standard OAuth `scope` parameter (described in the table above). Values are typically obtained from the `scope` field within the `credential_configurations_supported` in Issuer metadata.

*   If the Issuer metadata includes `authorization_servers`, the `resource` parameter containing the Credential Issuer Identifier SHOULD be included.

```http
GET https://my-issuer.rocks/auth/authorize?
  response_type=code
  &scope=VerifiablePortableDocumentA1
  &resource=https%3A%2F%2Fcredential-issuer.example.com
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
Host: server.example.com
```


### 6.3.3 Additional Authorization Request Parameters

| Field              | Req / Opt | Description                                                        |
| :----------------- | :-------- | :----------------------------------------------------------------- |
| **`issuer_state`** | OPTIONAL  | Opaque string received in Credential Offer, passed back to Issuer. |

## 6.4 Authorization Response (Authorization Endpoint)

The Authorization Server responds to the Wallet's Authorization Request. The format depends on whether the request was successful and the requested `response_type`.

### 6.4.1 Successful Response

A successful response redirects the User-Agent back to the client's `redirect_uri`. The parameters included in the redirect depend on the `response_type` used in the request:

1.  **`response_type=code` (Standard Authorization Code Flow):**
    If standard authorization code flow was requested and successful, the credential issuer redirects the User-Agent to the client's `redirect_uri` with the `code` parameter containing the short-lived authorization code. A `state` parameter MUST be included if it was present in the request.

    **Example:**
    ```http
    HTTP/1.1 302 Found
    Location: https://Wallet.example.org/cb?code=SplxlOBeZQQYbYS6WxSbIA&state=af0ifjsldkj
    ```

2.  **`response_type=id_token` (SIOPv2 Pseudonymous Authentication):**
    If the credential issuer requests pseudonymous authentication (e.g., DID Authentication) via SIOPv2 [3], the `response_type` might be `id_token`. Often, `response_mode=direct_post` is used, where the AS instructs the Wallet (via the User-Agent redirect) to `POST` the resulting `id_token` directly to the AS or Issuer's `redirect_uri`.

    **Example Redirect (Instructing Wallet to POST):**
    ```http
    HTTP/1.1 302 Found
    Location: {Wallet-Specific-URI}?state=22857405-1a41-4db9-a638-a980484ecae1&client_id=https://example.server.com&redirect_uri=https://example.server.com/direct_post&response_type=id_token&response_mode=direct_post&scope=openid&nonce=a6f24536-b109-4623-a41a-7a9be932bdf6&request_uri=https://example.server.com/request_uri
    ```
    *(Note: The initial part of the Location might be a Wallet-specific custom scheme or universal link)*

    **Query Parameters in the redirect:**

    | Parameter       | Description                                                                                                                                                 |
    | :-------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `state`         | Opaque value used by the client to maintain state. Included if present in the request.                                                                      |
    | `client_id`     | Identifier for the Credential Issuer acting as the SIOPv2 Relying Party.                                                                                    |
    | `redirect_uri`  | The endpoint where the Wallet should `POST` the `id_token`.                                                                                                 |
    | `response_type` | Indicates `id_token` is expected.                                                                                                                           |
    | `response_mode` | Indicates the response delivery method, e.g., `direct_post`.                                                                                                |
    | `scope`         | Must include `openid`.                                                                                                                                      |
    | `nonce`         | String value used to associate a Client session with an ID Token, and to mitigate replay attacks.                                                           |
    | `request_uri`   | OPTIONAL. This is intended for scenarios where the authorization request is large. The URI can be used by the holder to retrieve the authorization request. |

    The holder wallet then generates an `id_token` (signed using the Holder's key, typically derived from a DID) and sends it to the specified `redirect_uri` (which acts as the direct post endpoint).

    **Example Wallet POST:**
    ```http
    POST /direct_post HTTP/1.1
    Host: example.server.com
    Content-Type: application/x-www-form-urlencoded

    id_token=eyJraWQiOiJkaW...a980484ecae1&state=22857405-1a41-4db9-a638-a980484ecae1
    ```

3.  **`response_type=vp_token` (Dynamic Credential Request via OpenID4VP):**
    If the credential issuer requests a presentation during issuance (Dynamic Credential Request) via OpenID4VP, the `response_type` includes `vp_token`. The AS redirects the User-Agent to the Wallet's `redirect_uri`. This redirect contains parameters that constitute an OpenID4VP Authorization Request, instructing the Wallet what needs to be presented. Refer to the OpenID4VP specification for the exact parameters (like `presentation_definition`).

### 6.4.2 Error Response

If the request fails, the Authorization Server redirects the User-Agent back to the client's `redirect_uri` with error information encoded in the query parameters, as defined in [RFC6749] Section 4.1.2.1.

| Parameter           | Description                                                                                                                                                                                     |
| :------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `error`             | REQUIRED. A single ASCII error code (e.g., `invalid_request`, `unauthorized_client`, `access_denied`, `unsupported_response_type`, `invalid_scope`, `server_error`, `temporarily_unavailable`). |
| `error_description` | OPTIONAL. Human-readable ASCII text providing additional information.                                                                                                                           |
| `error_uri`         | OPTIONAL. URI identifying a human-readable web page with information about the error.                                                                                                           |
| `state`             | REQUIRED if a `state` parameter was present in the client authorization request.                                                                                                                |

**Example:**
```http
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?error=access_denied&error_description=User%20denied%20access&state=af0ifjsldkj
```

## 6.5 Token Request (Token Endpoint)

Exchanges authorization grant (code or pre-authorized code) for tokens.

### 6.5.1 Authorization Code Flow Token Request

Standard OAuth 2.0 request ([RFC6749] Section 4.1.3).

```http
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

&grant_type=authorization_code
&code=SplxlOBeZQQYbYS6WxSbIA
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
&redirect_uri=https%3A%2F%2FWallet.example.org%2Fcb
```

### 6.5.2 Pre-authorized Code Flow Token Request

Uses the `urn:ietf:params:oauth:grant-type:pre-authorized_code` grant type.

```http
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

&grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
&pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
&tx_code=493536
```

### 6.5.3 Client Authentication

*   **Auth Code Flow:** Client authentication MAY be required based on AS policy and client type (e.g., `client_secret_basic`, `private_key_jwt`).
*   **Pre-Auth Code Flow:** Client authentication is OPTIONAL. If the AS supports `pre-authorized_grant_anonymous_access_supported: true`, `client_id` can be omitted. Otherwise, client authentication is required.

## 6.6 Token Response (Token Endpoint)

Standard OAuth 2.0 response ([RFC6749] Section 5.1) plus OID4VCI extensions.

| Field                       | Req / Opt                                    | Description                                                                                                                                                                                                                                                                                                                                                                                  |
| :-------------------------- | :------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`access_token`**          | REQUIRED                                     | The Access Token for the Credential Endpoint.                                                                                                                                                                                                                                                                                                                                                |
| **`token_type`**            | REQUIRED                                     | Typically `Bearer`.                                                                                                                                                                                                                                                                                                                                                                          |
| **`expires_in`**            | RECOMMENDED                                  | Lifetime in seconds.                                                                                                                                                                                                                                                                                                                                                                         |
| **`refresh_token`**         | OPTIONAL                                     | Used to obtain new Access Tokens.                                                                                                                                                                                                                                                                                                                                                            |
| **`scope`**                 | OPTIONAL                                     | Scope associated with the Access Token.                                                                                                                                                                                                                                                                                                                                                      |
| **`authorization_details`** | REQUIRED (if requested with `authz_details`) | Array of objects detailing the granted authorization. [4] For `openid_credential` type, MUST include: <br/> - `type`: `openid_credential` <br/> - `credential_configuration_id`: Identifier from metadata <br/> - `credential_identifiers` (REQUIRED): Array of unique strings identifying the specific Credential Dataset instance(s) authorized by this token. Used in Credential Request. |

### 6.6.1 Successful Response (with `authorization_details`)

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
  "token_type": "bearer",
  "expires_in": 86400,
  "authorization_details": [
    {
      "type": "openid_credential",
      "credential_configuration_id": "VerifiablePortableDocumentA1",
      "credential_identifiers": [ "VerifiablePortableDocumentA1-Spain", "VerifiablePortableDocumentA1-Sweden", "VerifiablePortableDocumentA1-Germany" ]
    }
  ]
}
```

### 6.6.2 Successful Response (with `scope`)

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI4a5k..zEF",
    "token_type": "bearer",
    "expires_in": 86400,
    "id_token": "eyJodHRwOi8vbWF0dHIvdGVuYW50L..3Mz"

}
```

### 6.6.3 Token Error Response

Standard OAuth 2.0 errors ([RFC6749] Section 5.2). OID4VCI [1] Section 6.3 provides specific interpretations for Pre-Authorized Code flow:

*   **`invalid_request`**: `tx_code` provided but not expected, or expected but not provided.
*   **`invalid_grant`**: `tx_code` expected but wrong value provided; `pre-authorized_code` is invalid or expired.
*   **`invalid_client`**: Client authentication required for Pre-Auth flow but not provided or failed.

## 6.7 Credential Request (Credential Endpoint)

`POST` request to the `credential_endpoint` using `application/json`. Requires Bearer Access Token authorization.

### 6.7.1 Credential Request Parameters

| Field                             | Req / Opt                                         | Description                                                                                                                                                                                                        |
| :-------------------------------- | :------------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`credential_identifier`**       | REQUIRED (if received in Token Resp.)             | String identifying the specific Credential Dataset instance from the `credential_identifiers` array in the Token Response. Mutually exclusive with `credential_configuration_id`.                                  |
| **`credential_configuration_id`** | REQUIRED (if `credential_identifier` not used)    | String identifying the Credential Configuration from Issuer metadata. Used when `scope` was used for authorization and no `credential_identifiers` were returned. Mutually exclusive with `credential_identifier`. |
| **`proof`**                       | OPTIONAL (but required if Issuer supports proofs) | Object containing a single proof of possession for the key material the Credential instance should be bound to. MUST contain `proof_type`.                                                                         |

### 6.7.2 Proof of Possession

If the Issuer requires proof of possession (indicated by `proof_types_supported` in metadata for the configuration), the `proof` parameter MUST be included.

*   The proof MUST incorporate the **Credential Issuer Identifier** as the audience (`aud` claim in JWT proof).
*   If a `c_nonce` was retrieved from the Nonce endpoint, it MUST be included in the proof (`nonce` claim in JWT proof).

### 6.7.3 Proof Types

OID4VCI [1] Section 8.2.1 defines:

*   **`jwt`:** A JWT signed by the key to be bound.
    *   *JOSE Header:* `alg` (REQUIRED, asymmetric), `typ` (REQUIRED, `openid4vci-proof+jwt`), `jwk` or `kid` (RECOMMENDED). Optional: `key_attestation`, `trust_chain`.
    *   *JWT Body:* `aud` (REQUIRED, Issuer Identifier), `iat` (REQUIRED), `nonce` (REQUIRED, if `c_nonce` received). Optional: `iss` (Client ID, if not anonymous Pre-Auth).

### 6.7.4 Credential Request Examples

**For W3C VC with credential format** `jwt_vc_json` and `proof_type` of `jwt`:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "credential_identifier": "urn:eu.europa.ec.eudi:pid:1", 
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJraWQiOiJkaWQ6ZX..zM"
   }
}
```
> [!NOTE]
> In the above, the credentialSubject is optional and is not considered within the scope of EWC LSP.

**For IETF SD-JWT VC with credential format** `dc+sd-jwt` and `proof_type` of `jwt`:


```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "credential_identifier": "urn:eu.europa.ec.eudi:pid:1",
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJ0eXAiOiJvc..1WlA"
   }
}
```
**For ISO18013-5 mDL with credential format** `mso_mdoc` and `proof_type` of `jwt`:


```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "credential_configuration_id": "eu.europa.ec.eudi.pid.1",
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJraWQiOiJkaWQ6...QifQ"
   }
}
```

## 6.8 Credential Response (Credential Endpoint)

Response containing the issued Credential(s) or indicating deferral.

*   **Success (Immediate):** HTTP 200 OK.
*   **Success (Deferred):** HTTP 202 Accepted.
*   **Media Type:** `application/json` (unless encrypted).
*   **Encrypted Response:** If requested and supported, the response body is a JWE ([RFC7516]) and the media type is `application/jwt`.

| Field                    | Req / Opt                                          | Description                                                                                                                                                                                                     |
| :----------------------- | :------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`format`**             | OPTIONAL                                           | Credential format identifier. May be needed if multiple formats possible for the configuration. (Removed in draft 15, but might be contextually useful).                                                        |
| **`credentials`**        | REQUIRED (for immediate single or batch issuance)  | Array of objects, each containing a `credential` field with one issued Credential instance. Mutually exclusive with `transaction_id`. See OID4VCI [1] 8.3.                                                    |
| **`transaction_id`**     | REQUIRED (for deferred issuance)                   | String identifying the deferred transaction. Used in subsequent requests to the Deferred Credential Endpoint. Mutually exclusive with `credential` and `credentials`.                                           |
| **`notification_id`**    | OPTIONAL (if Notification Endpoint supported/used) | String identifying this issuance transaction for later use with the Notification Endpoint.                                                                                                                      |


### 6.8.1 Immediate Credential Response

```json
{
  "credentials": [
    {
      "credential": "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L"
    }
  ]
}
```

### 6.8.2 Deferred Credential Response

*Status Code: 202 Accepted*
```json
{
  "transaction_id": "8xLOxBtZp8"
}
```

### 6.8.3 Credential Error Response

If the Access Token is invalid/expired, return standard OAuth errors ([RFC6750]). For payload errors, use HTTP 400 Bad Request with `application/json` body:

| Error Code                      | Description                                                                                                   |
| :------------------------------ | :------------------------------------------------------------------------------------------------------------ |
| `invalid_request`               | Generic request error (SHOULD use more specific codes below).                                                 |
| `invalid_credential_request`    | Request malformed, missing required parameter, unsupported parameter, repeated parameter.                     |
| `unsupported_credential_type`   | Requested Credential Type (e.g., `vct`) not supported.                                                        |
| `unsupported_credential_format` | Requested Credential Format not supported.                                                                    |
| `invalid_proof`                 | `proof` parameter is invalid (missing, signature invalid, required binding info missing like audience/nonce). |
| `invalid_nonce`                 | Proof contains an invalid `c_nonce` value. Wallet should request a new nonce.                                 |
| `credential_request_denied`     | Issuer declined the request for policy reasons.                                                               |

Also includes optional `error_description` (ASCII text) and `error_uri`.

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "invalid_proof",
  "error_description": "Proof validation failed: signature mismatch"
}
```

## 6.9 Nonce Request (Nonce Endpoint)

Optional endpoint (`nonce_endpoint` in metadata) to get a fresh `c_nonce`.

```http
POST /nonce HTTP/1.1
Host: credential-issuer.example.com
Content-Length: 0
(Requires Access Token if endpoint is protected)
```

## 6.10 Nonce Response (Nonce Endpoint)

```json
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "c_nonce": "wKI4LT17ac15ES9bw8ac4"
}
```

## 6.11 Deferred Credential Request (Deferred Credential Endpoint)

Optional endpoint (`deferred_credential_endpoint` in metadata) to retrieve a previously deferred credential. Requires the *same* Access Token as the initial request.

```http
POST /deferred_credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "transaction_id": "8xL0xBtZp8"
}
```

## 6.12 Deferred Credential Response (Deferred Credential Endpoint)

If successful, the response is the same as an immediate Credential Response, containing the `credential(s)`. HTTP 200 OK.

## 6.13 Deferred Credential Error Response

Uses HTTP 400 Bad Request with `application/json` body:

| Error Code               | Description                                                                                                                         |
| :----------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `invalid_request`        | Generic request error.                                                                                                              |
| `invalid_transaction_id` | The `transaction_id` is invalid, expired, or already used.                                                                          |
| `issuance_pending`       | Credential issuance is still pending. Response SHOULD include `interval` (seconds) for Wallet to wait before retrying (default 5s). |
| `unauthorized_client`    | Access token invalid or insufficient.                                                                                               |

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "issuance_pending",
  "interval": 30
}
```

## 6.14 Notification Request (Notification Endpoint)

Optional endpoint (`notification_endpoint` in metadata) for the Wallet to notify the Issuer about the outcome of an issuance flow. Requires the *same* Access Token as the issuance request.

```http
POST /notification HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "notification_id": "3fwe98js",
  "event": "credential_accepted", // or credential_failure, credential_deleted
  "event_description": "Credential stored successfully by user." // Optional
}
```

| Field                   | Req / Opt | Description                                                                                                                                                                                                                                           |
| :---------------------- | :-------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`notification_id`**   | REQUIRED  | String received in the Credential/Deferred Response.                                                                                                                                                                                                  |
| **`event`**             | REQUIRED  | String indicating the event type: <br/> - `credential_accepted`: Credential successfully stored. <br/> - `credential_failure`: Storage failed (not due to user deletion). <br/> - `credential_deleted`: User action caused issuance failure/deletion. |
| **`event_description`** | OPTIONAL  | Human-readable ASCII text with details.                                                                                                                                                                                                               |

## 6.15 Notification Response (Notification Endpoint)

*   **Success:** HTTP 2xx (RECOMMENDED: 204 No Content).
*   **Error (Auth):** Standard OAuth errors ([RFC6750]).
*   **Error (Payload):** HTTP 400 Bad Request with `application/json`:
    *   `invalid_notification_id`: ID is invalid.
    *   `invalid_notification_request`: Request malformed or missing required parameter.

```http
HTTP/1.1 204 No Content
```

# 7.0	Implementers

Please refer to the [implementers table](https://github.com/EWC-consortium/eudi-wallet-rfcs?tab=readme-ov-file#implementers).

# 8.0	Reference

1. OpenID Foundation (2023), 'OpenID for Verifiable Credential Issuance', Available at: [https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID2.html](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID2.html) (Accessed: May 04, 2025).
2. European Commission (2023) The European Digital Identity Wallet Architecture and Reference Framework (2023-04, v1.1.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: October 16, 2023).
3. OpenID Foundation (2023), 'Self-Issued OpenID Provider v2 (SIOP v2)', Available at: [https://openid.net/specs/openid-connect-self-issued-v2-1_0.html](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html) (Accessed: October 01, 2023)
4. OAuth 2.0 Rich Authorization Requests, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11) (Accessed: February 01, 2024)
5. Proof Key for Code Exchange by OAuth Public Clients, Available at: [https://datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) (Accessed: February 01, 2024)
6. OpenID4VC High Assurance Interoperability Profile with SD-JWT VC - draft 00, Available at [https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html) (Accessed: February 16, 2024)
7. Lodderstedt, T., Campbell, B., Sakimura, N., Tonge, D., and F. Skokan, "OAuth 2.0 Pushed Authorization Requests", RFC 9126, DOI 10.17487/RFC9126, September 2021, <https://www.rfc-editor.org/info/rfc9126>.

# Appendix A: Public key resolution

Refer [RFC012](/ewc-rfc012-trust-mechanism.md) for key resolution and trust mechanims. 

For a JWT (e.g., Request Object, VP Token, ID Token), there are multiple ways for resolving the public key needed for signature verification:

1.  **`jwk` Header:** The public key JWK is directly embedded in the JOSE header.
2.  **`kid` Header:** The header contains a Key ID. The key needs to be retrieved:
    *   If the `kid` is a DID URL (e.g., `did:example:123#key-1`), use DID resolution to obtain the verification method (public key) from the DID Document.
    *   If the issuer provides an AS Metadata endpoint (`iss` claim), fetch the `jwks_uri` from the metadata and find the key matching the `kid` in the JWK Set.
3.  **`x5c` Header:** The header contains the X.509 certificate chain containing the public key. Validate the chain according to PKI rules.