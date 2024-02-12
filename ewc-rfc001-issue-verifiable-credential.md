
# EWC RFC 001: Issue Verifiable Credential - v1.0

**Authors:** 
* Mr George Padaytti (iGrant.io, Sweden)
* Mr Lal Chandran (iGrant.io, Sweden)
<Please add if you wish to be an author>

**Reviewers:** 

* Dr Nikos Triantafyllou (University of the Aegean, Greece)
* Mr Florin Coptil (Bosch, Germany)
* Mr Matteo Mirabelli (Infocert, Italy)
* Dr Mikael Linden (Vero, Finland) 
* Mr Andreas Abraham (ValidatedID, Spain)
* Mr Renaud Murat (Archipels, France)
* Mr. Sebastian Bickerle (Lissi ID, Germany)
<Please add your name as a reviewer once you review>

**Status:** Work in Progress, Target to finish 15-Feb-2024

**Table of Contents**

- [EWC RFC 001: Issue Verifiable Credential - v1.0](#ewc-rfc-001-issue-verifiable-credential---v10)
- [1.0	Summary](#10summary)
- [2.0	Motivation](#20motivation)
- [3.0	Messages](#30messages)
  - [3.1	Credential offer](#31credential-offer)
  - [3.2	Credential offer response](#32credential-offer-response)
  - [3.3 Discover request](#33-discover-request)
  - [3.4 Discover response](#34-discover-response)
  - [3.5 Authorisation request](#35-authorisation-request)
  - [3.6 Authorisation response](#36-authorisation-response)
  - [3.7 Token request](#37-token-request)
    - [3.7.1 Authorisation code flow](#371-authorisation-code-flow)
    - [3.7.2 Pre-authorised code flow](#372-pre-authorised-code-flow)
  - [3.8 Token response](#38-token-response)
  - [3.9 Credential request](#39-credential-request)
  - [3.10 Credential response](#310-credential-response)
    - [3.10.1  In-time](#3101--in-time)
    - [3.10.2 Deferred](#3102-deferred)
- [4.0	Alternate response format](#40alternate-response-format)
- [5.0	Implementors](#50implementors)
- [Open issues](#open-issues)
- [6.0	Reference](#60reference)


# 1.0	Summary

This specification implements OID4VCI workflow for any issuer as per reference specification [1]. This minimises risks towards interoperability across the European Wallet Ecosystem with a standard specification in the EUDI wallet ecosystem as per the ARF [2] requirements. 

# 2.0	Motivation

The EWC LSP must align with the standard protocol for issuing credentials. This is the basis of interoperability between Issuers and Holders across the EWC LSPs. The assumption is that the user is familiar with the EWC-chosen protocols and standards and can refer to original standards references when necessary.

# 3.0	Messages

The credential issuance can be an authorisation flow or a pre-authorised one. These are depicted in the following diagrams, the assumption is that credential offer is obtained by holder wallet prior to discovery using same device (i.e. clicking on a link) or cross device (i.e scanning a QR code) flows.

```mermaid
  sequenceDiagram
    participant I as Individual using EUDI Wallet
    participant O as Organisational Wallet (Issuer)

    Note over I,O: Discovery issuer capabilities
    I->>O: GET: /.well-known/openid-credential-issuer
    O-->>I: OpenID credential issuer configuration
    I->>O: GET: /.well-known/openid-configuration
    O-->>I: OpenID configuration for authorisation server

    Note over I,O: Authenticate and Authorise
    I->>O: Authorisation request
    O-->>I: Authorisation response
    I->>O: Token request
    O-->>I: Token response

    Note over I,O: Issue credential
    I->>O: POST: Credential request with token
    O-->>I: Credential response with acceptance token
```
Figure 1: Issuance using Authorisation Code Flow based on [1]

```mermaid
  sequenceDiagram
      participant I as Individual using EUDI Wallet
      participant O as Organisational Wallet (Issuer)
      
      Note over I,O: Discovery issuer capabilities
      I->>O: GET: /.well-known/openid-credential-issuer
      O-->>I: OpenID credential issuer configuration
      I->>O: GET: /.well-known/openid-configuration
      O-->>I: OpenID configuration for authorisation server

      Note over I,O: Authenticate and Authorise
      I->>O: POST: Pre-authorised token request with PIN
      O-->>I: Token response

      Note over I,O: Issue credential
      I->>O: POST: Credential request with token
      O-->>I: Credential response with acceptance token
```
Figure 2: Issuance using Pre-Authorisation Code Flow based on [1]

## 3.1	Credential offer

The recommendation is to send the Credential Offer by Reference Using the `credential_offer_uri` Parameter to avoid QR Code data overload. The endpoint that is expected to be embedded in the QR code is:

```
openid-credential-offer://?credential_offer_uri=https://server.example.com/credential-offer
```

Here, the `credential_offer_uri` query param contains the URL in which the credential offer from the issuer can be resolved. 

## 3.2	Credential offer response

Once the `credential_offer_uri` query param is resolved, the response can be either of the following formats. 

```json
{
  "credential_issuer": "https://server.example.com",
  "credentials": [
    "VerifiablePortableDocumentA1"
  ],
  "grants": {
    "authorization_code": {
      "issuer_state": "eyJhbGciOiJSU0Et...FYUaBy"
    }
  }
}
```

> [!NOTE]
> In order to support all EBSI conformant wallets, the following format for the response is also valid, but not **mandatory** to be supported:

```json
{
  "credential_issuer": "https://server.example.com",
  "credentials": [
    {
      "format": "jwt_vc",
      "types": [
        "VerifiableCredential",
        "VerifiableAttestation",
        "VerifiablePortableDocumentA1"
      ],
      "trust_framework": {
        "name": "ewc-issuer-trust-list",
        "type": "Accreditation",
        "uri": "TIR link towards accreditation"
      }
    }
  ],
   "grants": {
        "authorization_code": {
            "issuer_state": "eyJhbGciOiJSU0Et...FYUaBy"
        }
    }
}
```

The holder wallet obtains the JSON and can process it. The format is the credential format. In the context of EWC LSPs, it can be `jwt_vc` or `sd-jwt_vc`.  

For pre-authorised flow, the credential response is as given:


```json
{
  "credential_issuer": "https://server.example.com",
  "credentials": [
    {
      "format": "jwt_vc",
      "types": [
        "VerifiableCredential",
        "VerifiableAttestation",
        "VerifiablePortableDocumentA1"
      ],
      "trust_framework": {
        "name": "ebsi",
        "type": "Accreditation",
        "uri": "TIR link towards accreditation"
      }
    }
  ],
  "grants": {
    "urn:ietf:params:oauth:grant-type:pre-authorized_code": {
      "pre-authorized_code": "eyJhbGciOiJSU0Et...FYUaBy",
      "user_pin_required": true
    }
  }
}
```

## 3.3 Discover request

Here, the holder wallet requests the issuer’s authorisation server configurations.

Resolve `/.well-known/openid-credential-issuer` endpoint for `credential_issuer` URI in the credential offer response.

```http
GET https://server.example.com/.well-known/openid-credential-issuer
```

Resolve `/.well-known/openid-configuration` endpoint for `authorization_server` URI present in the response for the above.

```http
GET https://server.example.com/.well-known/openid-configuration
```

## 3.4 Discover response

Once the well-known endpoint for **issuer server** configuration is resolved, the response is as given below:

```json
{
  "credential_issuer": "https://server.example.com",
  "authorization_server": "https://server.example.com",
  "credential_endpoint": "https://server.example.com/credential",
  "deferred_credential_endpoint": "https://server.example.com/credential_deferred",
  "display": [
    {
      "name": "Issuer",
      "location": "Belgium",
      "locale": "en-GB",
      "cover": {
        "url": "https://server.example.com/cover.jpeg",
        "alt_text": "Issuer"
      },
      "logo": {
        "url": "https://server.example.com/logo.jpg",
        "alt_text": "Issuer"
      },
      "description": "For queries about how we are managing your data please contact the Data Protection Officer."
    }
  ],
  "credentials_supported": {
    "VerifiablePortableDocumentA1": {
      "format": "vc+sd-jwt",
      "display": [
        {
          "name": "Portable Document A1",
          "locale": "en-GB"
        }
      ]
    }
  }
}
```

> [!NOTE]
> In order to support all EBSI conformant wallets, the following format for the `credentials_supported` is also valid, but not **mandatory** to be supported:

```json
{
  "credentials_supported": [
    {
      "format": "jwt_vc",
      "types": [
        "VerifiableCredential",
        "VerifiableAttestation",
        "VerifiablePortableDocumentA1"
      ],
      "trust_framework": {
        "name": "ebsi",
        "type": "Accreditation",
        "uri": "TIR link towards accreditation"
      },
      "display": [
        {
          "name": "Portable Document A1",
          "locale": "en-GB"
        }
      ]
    }
  ]
}
```


Once the well-known endpoint for **authorisation server** configuration is resolved, the response is as given below:

```json
{
  "redirect_uris": [
    "https://server.example.com/direct_post"
  ],
  "issuer": "https://server.example.com",
  "authorization_endpoint": "https://server.example.com/authorize",
  "token_endpoint": "https://server.example.com/token",
  "jwks_uri": "https://server.example.com/jwks",
  "scopes_supported": [
    "openid"
  ],
  "response_types_supported": [
    "vp_token",
    "id_token"
  ],
  "response_modes_supported": [
    "query"
  ],
  "grant_types_supported": [
    "authorization_code"
  ],
  "subject_types_supported": [
    "public"
  ],
  "id_token_signing_alg_values_supported": [
    "ES256"
  ],
  "request_object_signing_alg_values_supported": [
    "ES256"
  ],
  "request_parameter_supported": true,
  "request_uri_parameter_supported": true,
  "token_endpoint_auth_methods_supported": [
    "private_key_jwt"
  ],
  "request_authentication_methods_supported": {
    "authorization_endpoint": [
      "request_object"
    ]
  },
  "vp_formats_supported": {
    "jwt_vp": {
      "alg_values_supported": [
        "ES256"
      ]
    },
    "jwt_vc": {
      "alg_values_supported": [
        "ES256"
      ]
    }
  },
  "subject_syntax_types_supported": [
    "did:key",
    "did:ebsi"
  ],
  "subject_syntax_types_discriminations": [
    "did:key:jwk_jcs-pub",
    "did:ebsi:v1"
  ],
  "subject_trust_frameworks_supported": [
    "ebsi"
  ],
  "id_token_types_supported": [
    "subject_signed_id_token",
    "attester_signed_id_token"
  ]
}
```

## 3.5 Authorisation request

The authorisation request is to grant access to the credential endpoint. Below is an example of such a request:

```http
GET https://my-issuer.rocks/auth/authorize?

&response_type=code
&scope=openid
&issuer_state=tracker%3Dvcfghhj
&state=client-state
&client_id=did%3Akey%3Az2dmzD81cgPx8Vki7JbuuMmFYrWPgYoytykUZ3eyqht1j9KbsEYvdrjxMjQ4tpnje9BDBTzuNDP3knn6qLZErzd4bJ5go2CChoPjd5GAH3zpFJP5fuwSk66U5Pq6EhF4nKnHzDnznEP8fX99nZGgwbAh1o7Gj1X52Tdhf7U4KTk66xsA5r
&authorization_details=%5B%7B%22format%22%3A%22jwt_vc%22%2C%22locations%22%3A%5B%22https%3A%2F%2Fissuer.example.com%22%5D%2C%22type%22%3A%22openid_credential%22%2C%22types%22%3A%5B%22VerifiableCredential%22%2C%22VerifiableAttestation%22%2C%22VerifiablePortableDocumentA1%22%5D%7D%5D
&redirect_uri=openid%3A
&nonce=glkFFoisdfEui43
&code_challenge=YjI0ZTQ4NTBhMzJmMmZhNjZkZDFkYzVhNzlhNGMyZDdjZDlkMTM4YTY4NjcyMTA5M2Q2OWQ3YjNjOGJlZDBlMSAgLQo%3D
&code_challenge_method=S256
&client_metadata=%7B%22vp_formats_supported%22%3A%7B%22jwt_vp%22%3A%7B%22alg%22%3A%5B%22ES256%22%5D%7D%2C%22jwt_vc%22%3A%7B%22alg%22%3A%5B%22ES256%22%5D%7D%7D%2C%22response_types_supported%22%3A%5B%22vp_token%22%2C%22id_token%22%5D%2C%22authorization_endpoint%22%3A%22openid%3A%2F%2F%22%7D

Host: https://server.example.com
```

Query params for the authorisation request are given below:

<table>
  <tr>
   <td><code>response_type</code>
   </td>
   <td>The value must be ‘code’
   </td>
  </tr>
  <tr>
   <td><code>scope</code>
   </td>
   <td>The value must be ‘openid’
   </td>
  </tr>
  <tr>
   <td><code>state</code>
   </td>
   <td>The client uses an opaque value to maintain the state between the request and callback.
   </td>
  </tr>
  <tr>
   <td><code>client_id</code>
   </td>
   <td>Decentralised identifier
   </td>
  </tr>
  <tr>
   <td><code>authorization_details</code>
   </td>
   <td>As specified in OAuth 2.0 Rich Authorization Requests specification to specify fine-grained access [4]
   </td>
  </tr>
  <tr>
   <td><code>redirect_uri</code>
   </td>
   <td>For redirection of the response
   </td>
  </tr>
  <tr>
   <td><code>code_challenge</code>
   </td>
   <td>As specified in PKCE for OAuth Public Client specification [5]
   </td>
  </tr>
  <tr>
   <td><code>code_challenge_method</code>
   </td>
   <td>As specified in PKCE for OAuth Public Client specification
   </td>
  </tr>
  <tr>
   <td><code>client_metadata</code>
   </td>
   <td>Holder wallets are non-reachable and can utilise this field in the Authorisation Request to deliver configuration
   </td>
  </tr>
  <tr>
   <td><code>issuer_state</code>
   </td>
   <td>If present in the credential offer
   </td>
  </tr>
</table>


## 3.6 Authorisation response

The credential issuer can **optionally** request additional details to authenticate the client e.g. DID authentication. In this case, the authorisation response will contain a `response_type` parameter with the value `direct_post`. A sample response is as given:

```http
HTTP/1.1 302 Found
Location: http://localhost:8080?state=22857405-1a41-4db9-a638-a980484ecae1&client_id=https%3A%2F%2Fapi-conformance.ebsi.eu%2Fconformance%2Fv3%2Fauth-mock&redirect_uri=https%3A%2F%2Fapi-conformance.ebsi.eu%2Fconformance%2Fv3%2Fauth-mock%2Fdirect_post&response_type=id_token&response_mode=direct_post&scope=openid&nonce=a6f24536-b109-4623-a41a-7a9be932bdf6&request_uri=https%3A%2F%2Fapi-conformance.ebsi.eu%2Fconformance%2Fv3%2Fauth-mock%2Frequest_uri%2F111d2819-9ab7-4959-83e5-f414c57fdc27
```

Query params for the authorisation response are given below:

<table>
  <tr>
   <td><code>state</code>
   </td>
   <td>The client uses an opaque value to maintain the state between the request and callback.
   </td>
  </tr>
  <tr>
   <td><code>client_id</code>
   </td>
   <td>Decentralised identifier
   </td>
  </tr>
  <tr>
   <td><code>redirect_uri</code>
   </td>
   <td>For redirection of the response
   </td>
  </tr>
  <tr>
   <td><code>response_type</code>
   </td>
   <td>The value must be <code>id_token</code> if the issuer requests DID authentication.
   </td>
  </tr>
  <tr>
   <td><code>response_mode</code>
   </td>
   <td>The value must be <code>direct_post</code>
   </td>
  </tr>
  <tr>
   <td><code>scope</code>
   </td>
   <td>The value must be <code>openid</code>
   </td>
  </tr>
  <tr>
   <td><code>nonce</code>
   </td>
   <td>A value used to associate a client session with an ID token and to mitigate replay attacks
   </td>
  </tr>
  <tr>
   <td><code>request_uri</code>
   </td>
   <td>The authorisation server’s private key signed the request.
   </td>
  </tr>
</table>


The holder wallet then responds with an `id_token` signed by the DID to the direct post endpoint.

```http
POST /direct_post
Content-Type: application/x-www-form-urlencoded
&id_token=eyJraWQiOiJkaW...a980484ecae1
```

If additional details are not requested, the credential issuer will send an authorisation response with a `code` query parameter containing the short-lived authorisation code. A sample response is given below:

```http
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?code=SplxlOBeZQQYbYS6WxSbIA
```

## 3.7 Token request

### 3.7.1 Authorisation code flow

The token request for authorised code flow is as given:

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

This request is made with the following query params:

<table>
  <tr>
   <td><code>grant_type</code>
   </td>
   <td>Grant type for authorisation. E.g. <code>authorization_code</code>
   </td>
  </tr>
  <tr>
   <td><code>client_id</code>
   </td>
   <td>Decentralised identifier
   </td>
  </tr>
  <tr>
   <td><code>code</code>
   </td>
   <td>Authorisation code
   </td>
  </tr>
  <tr>
   <td><code>code_verifier</code>
   </td>
   <td>Wallet-generated secure random token used to validate the original <code>code_challenge</code> provided in the initial Authorization Request
   </td>
  </tr>
</table>

### 3.7.2 Pre-authorised code flow

The token request for pre-authorised code flow is as given:

```http
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

&grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
&pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
&user_pin=493536
```

This request is made with the following query params:

<table>
  <tr>
   <td><code>grant_type</code>
   </td>
   <td>Grant type for authorisation. E.g. <code>urn:ietf:params:oauth:grant-type:pre-authorized_code</code>
   </td>
  </tr>
  <tr>
   <td>pre-<code>authorized_code</code>
   </td>
   <td>Code representing the Credential Issuer's authorisation for the Wallet to obtain Credentials of a certain type. This code must be short-lived and single-use.
   </td>
  </tr>
  <tr>
   <td><code>user_pin</code>
   </td>
   <td>The end user pin is decided by the issuer and sent to the holder through an out-of-band process. E.g. Email, SMS
   </td>
  </tr>
</table>

## 3.8 Token response

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI4a5k..zEF",
    "token_type": "bearer",
    "expires_in": 86400,
    "id_token": "eyJodHRwOi8vbWF0dHIvdGVuYW50L..3Mz",
    "c_nonce": "PAPPf3h9lexTv3WYHZx8ajTe",
    "c_nonce_expires_in": 86400
}
```

## 3.9 Credential request

The Holder wallet makes a Credential Request to the Credential Endpoint as below:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "format": "jwt_vc_json",
   "credential_definition": {
      "type": [
         "VerifiableCredential",
         "VerifiablePortableDocumentA1"
      ],
      "credentialSubject": {
         "given_name": {},
         "family_name": {},
         "nationalities": {}
      }
   },
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJraW...KWjceMcr"
   }
}
```

In the above the `credentialSubject` is optional. If specified, you can request specific fields to be included in the issued credential. If its not specified, all fields in the credential is included. 

> [!NOTE]
> In order to support all EBSI conformant wallets, the following format for the request is also valid, but not **mandatory** to be supported:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
  "format": "jwt_vc",
  "proof": {
    "jwt": "eyJraWQiOiJkaWQ6a2...su7UFClz9NQnw",
    "proof_type": "jwt"
  },
  "types": [
    "VerifiableCredential",
    "VerifiableAttestation",
    "VerifiablePortableDocumentA1"
  ]
}
```

## 3.10 Credential response

The credential response can happen in-time or can be deferred as described below. 

### 3.10.1  In-time

The In-time flow indicates that the credential is available immediately and the response format is as below: 

```json
{
  "format": "jwt_vc",
  "credential": "eyJ0eXAiOi...F0YluuK2Cog",
  "c_nonce": "fGFF7UkhLa",
  "c_nonce_expires_in": 86400
}
```

### 3.10.2 Deferred

If the credential is unavailable, the issuer responds with an acceptance token, indicating credential issuance is deferred.

```json
{
  "acceptance_token": "eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400
}
```

If the response contains `acceptance_token` field, then it can be understood the credential is not available now and should be later available through the deferred credential endpoint. 

```http
POST /deferred-credential
Authorization: BEARER eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ
```

# 4.0	Alternate response format

Standard HTTP response codes shall be supported. Any additional ones can be formulated in the following format.

```
{
  "error": "invalid_request",
  "error_description": "The verifiable credential is expired"
}
```
The table below summarises the success/error responses that can be used:

<table>
  <tr>
   <td><strong>Response format</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>invalid_request
   </td>
   <td>Request failed. E.g. The verifiable credential is expired
   </td>
  </tr>
  <tr>
   <td>invalid_grant
   </td>
   <td>
<ul>

<li>The Authorization Server expects a PIN in the Pre-Authorized Code Flow but the Client provides the wrong PIN.

<li>The end user provides the wrong Pre-Authorized Code or the Pre-Authorized Code has expired.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>invalid_client
   </td>
   <td>The Client tried to send a Token Request with a Pre-Authorized Code without Client ID, but the Authorization Server does not support anonymous access
   </td>
  </tr>
</table>

# 5.0	Implementors

<table>
  <tr>
   <td><strong>Wallet</strong>
   </td>
   <td><strong>Link, if any</strong>
   </td>
   <td><strong>ITB compliance</strong>
   </td>
  </tr>
  <tr>
   <td>Amadeus
   </td>
   <td><a href="https://tid-wallet-dev.azurewebsites.net/">https://tid-wallet-dev.azurewebsites.net/</a> 
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>DVV Wallet
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>iGrant.io Enterprise Wallet
   </td>
   <td><a href="https://business.igrant.io/">https://business.igrant.io/</a> 
   </td>
   <td>Planned Mar’24
   </td>
  </tr>
  <tr>
   <td>iGrant.io Data Wallet
   </td>
   <td><a href="https://apple.co/2Mz9nJp">iOS</a>, <a href="https://play.google.com/store/apps/details?id=io.igrant.mobileagent">Android</a>
   </td>
   <td>Planned Mar’24
   </td>
  </tr>
  <tr>
   <td>ValidatedID
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>UAegean 
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
</table>

# Open issues

1. Where is authentication covered in the OID4VCI workflow? Is it assuming that the user requesting a credential has already been authenticated? 

2. In this spec (01-Feb), we have assumed that the user has not authenticated and the user is using a DID authentication mechanism to get the credential issued to the DID of the wallet. This is done using the `id_token` response in the authorisation request workflow.

# 6.0	Reference

1. OpenID Foundation (2023), 'OpenID for Verifiable Credential Issuance (OID4VCI)', Available at: [https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-12.html](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-12.html) (Accessed: January 10, 2024).
2. European Commission (2023) The European Digital Identity Wallet Architecture and Reference Framework (2023-04, v1.1.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: October 16, 2023).
3. OpenID Foundation (2023), 'Self-Issued OpenID Provider v2 (SIOP v2)', Available at: [https://openid.net/specs/openid-connect-self-issued-v2-1_0.html](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html) (Accessed: October 01, 2023)
4. OAuth 2.0 Rich Authorization Requests, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11) (Accessed: February 01, 2024)
5. Proof Key for Code Exchange by OAuth Public Clients, Available at: [https://datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) (Accessed: February 01, 2024)
