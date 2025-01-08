# EWC RFC003: Issue Person Identification Data (PID) - v1.2

**Authors:**

* Mr Leone Riello (Infocert, Italy)
* Mr Matteo Mirabelli (Infocert, Italy)
* Mr Lal Chandran (iGrant.io, Sweden)

**Reviewers:**

* Mr George Padayatti (iGrant.io, Sweden)
* Mr Jaromir Talir (NIC.cz, Czech Republic)
* Mr Roger Fagerud (DIGG, Swden)
* Mr Antti Laine (DVV, Finland)

**Status:** Approved

**Table of Contents**

- [EWC RFC003: Issue Person Identification Data (PID) - v1.2](#ewc-rfc003-issue-person-identification-data-pid---v12)
- [1.0 Summary](#10-summary)
- [2.0 Motivation](#20-motivation)
- [3.0 Messages](#30-messages)
    - [Preliminary Steps for PID Issuance](#preliminary-steps-for-pid-issuance)
    - [PID Credential Issuance Process](#pid-credential-issuance-process)
    - [Post-Issuance Verification and Management](#post-issuance-verification-and-management)
  - [3.1 Credential offer](#31-credential-offer)
  - [3.2 Credential offer response](#32-credential-offer-response)
  - [3.3 Discover request](#33-discover-request)
  - [3.4 Discover response](#34-discover-response)
  - [3.5 Authorization request](#35-authorization-request)
  - [3.6 Authorization response](#36-authorization-response)
  - [3.7 Token request](#37-token-request)
    - [3.7.1 Authorisation code flow](#371-authorisation-code-flow)
    - [3.7.2 Pre-authorised code flow](#372-pre-authorised-code-flow)
  - [3.8 Token response](#38-token-response)
  - [3.9 Credential request](#39-credential-request)
  - [3.10 Credential response](#310-credential-response)
    - [3.10.1  In-time](#3101--in-time)
    - [3.10.2 Deferred](#3102-deferred)
  - [3.11 Issuer Authorization Verification](#311-issuer-authorization-verification)
  - [3.12 Check Wallet's Conformity](#312-check-wallets-conformity)
- [4.0 Alternate response format](#40-alternate-response-format)
- [5.0 Implementers](#50-implementers)
- [6.0 Reference](#60-reference)
- [Appendix A: Public key resolution](#appendix-a-public-key-resolution)
- [Appendix C: PID attribute schema according to IA](#appendix-c-pid-attribute-schema-according-to-ia)
- [Appendix C: SD-JWT PID example](#appendix-c-sd-jwt-pid-example)


# 1.0 Summary

This specification implements the OID4VCI workflow for issuing Person Identification Data (PID) credentials by government-approved identity providers within the European Wallet Ecosystem. It defines a standard process to minimize risks and ensure interoperability in issuing high-assurance PIDs across the EUDI wallet ecosystem, adhering to the requirements set forth in the ARF [2] and in the implementing act 2024/2977 for PID and EAA issuance [6].

# 2.0 Motivation

The EWC LSP must align with the standard protocol for issuing PID from trusted and accredited sources. This uniform approach serves as the foundation for enabling interoperability between identity providers and wallet holders throughout the EWC ecosystem. This RFC assumes that users are familiar with the chosen EWC protocols and standards, and can reference the original specifications when required.

# 3.0 Messages

The PID credential issuance process incorporates comprehensive steps to ensure the security, reliability, and compliance. This includes both an authorization flow and a pre-authorized flow, with additional preliminary and post-issuance steps to align with regulatory standards and security best practices. The process is illustrated below, incorporating the critical steps of Wallet Conformity, Trust Anchor Verification, Reliable Data Acquisition, PID Generation, Secure Issuance and Storage; Renewal and Revocation Policies Management and communication to the user in accordance with regulation [6] is not in scope of this rfc.

### Preliminary Steps for PID Issuance

1. **Wallet Conformity:** Before initiating the PID issuance, the wallet unit must be confirmed to comply with established standards. This includes possessing an internal certificate from Certification Assessment Bodies (CAB) that validates its conformity, ensuring the wallet's capability to securely manage the PID and associated qualified electronic attestations.

2. **Trust Anchor Verification:** The authorization of PID provider within the Trust Anchor framework must be validated, ensuring it is listed as an authorized actor, thus ensuring that only verified entities can issue the PID.
3. **Data Acquisition from Reliable Sources:** Personal data used for PID generation must be sourced from authentic and current databases, such as civil registries, ensuring the PID credentials are based on accurate and up-to-date information.
   
4.  **PID provider authentication vs the wallet unit** The PID provider must be authenticated by the wallet using its relying party certificate. 

### PID Credential Issuance Process

The PID issuance follows detailed steps starting from the discovery of issuer capabilities, through authentication and authorization, leading to the actual credential issuance. The process is adapted to include the preliminary steps, ensuring a secure and compliant issuance path.

```mermaid
  sequenceDiagram
    participant I as Individual using EUDI Wallet
    participant TA as Trust Anchor
    participant O as PID Provider
    participant AS as Authentic Source
        
    Note over I,O: Discovery of Issuer Capabilities 
    I->>O: GET: Credential Offer URI
    I->> O: GET: /.well-known/openid-credential-issuer
    O-->> I: OpenID credential issuer configuration
    I->> O: GET: /.well-known/oauth-authorization-server
    O-->>I: OAuth authorization server metadata
    I->> O: get PID provider RP access certificate
    I <<->> TA: wallet relying party access certificate validation 
        
    Note over I,TA: Issuer Authorization Verification
    I->>TA: Request Issuer Authorization Status
    TA-->>I: Confirm Issuer is Trusted
    
    Note over I,O: Authenticate, Authorize, Check Wallet's Conformity
    opt authorization flow
    I->>O: Authorization request
    
    Note over O,AS: User Authentication
        opt user data verified vs authentic source
          O->>AS: Request Personal Identifier Data
          AS-->>O: Provide Personal Identifier Data
      end
      O-->>I: Authorization response 
    end
    
    I->> O: Token request
    Note right of I: WTA and WIA should be sent as parameters on token request
    O-->>O: Wallet Unit attestation validation
    O-->>TA: Wallet Provider verification against Trust Framework
      opt wallet attestations not valid
          O-->>I: Error message response
      end
    O->>O: authorization/pre-authorized code validation
    O-->>I: Token response
        
    Note over I,O: PID Generation and Secure Issuance
    I->>O: POST: Credential request with access token
    Note over O,AS: Data Acquisition from Authentic Source <BR> or temporary storage (userInfo)
    O->>AS: Request Personal Identifier Data
    AS-->>O: Provide Personal Identifier Data
    
    O-->>I: Credential response with PID, stored securely in wallet
    I->> I: Validation of the signature certificate vs pid provider certificate

```

Figure 1: PID Issuance Process Incorporating Preliminary Checks

The process highlights the integration of the new preliminary steps with the traditional authorization code flow and pre-authorized code flow, adhering to the OID4VCI specification. It ensures a robust framework for digital identity issuance, from initial compliance verification to the secure generation and storage of PID credentials, followed by ongoing management.

### Post-Issuance Verification and Management

Following the issuance of the PID, initial and periodic verification procedures are crucial to maintain the validity and integrity of the PID and its related electronic attestations. This includes checking for revocation status and ongoing compliance of both the wallet and issuer within the Trust Anchor framework. Additionally, policies for the renewal and revocation of PIDs and electronic attestations must be established to address changes in the individual's status, data breaches, or compliance issues.

## 3.1 Credential offer

For PID credential issuance, this RFC allows both authorization and preauthorized code flows. 
Credential offer can be provided by reference and by value.
In the first case the `credential_offer_uri` query parameter contains the URL where the credential offer from the government-approved identity provider can be resolved. 
This approach ensures a streamlined user experience while maintaining the necessary information exchange for the PID issuance process. The wallet user obtains the above by scanning a QR code for cross-device workflows or via a deeplink for same-device workflows.

Here there's an example:

```
openid-credential-offer://?credential_offer_uri=https://identity-provider.gov/pid-credential-offer

```
Options regarding credential offer are well described at https://issuer.eudiw.dev/
## 3.2 Credential offer response

On resolving the `credential_offer_uri` query parameter, the issuer responds with details of the PID credential offer. The response format is adapted to the specific requirements of PID issuance and may include information such as the credential type related to personal identification and the applicable trust framework. The response can be in one of the following formats:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "credential_configuration_ids": ["urn:eu.europa.ec.eudi:pid:1"],
  "grants": {
    "authorization_code": {
      "issuer_state": "eyJhbGciOiJSU0Et...FYUaBy"
    }
  }
}
```

The wallet instance retrieves this JSON response and processes it accordingly. The format of the credential (e.g., jwt_vc, vc+sd-jwt) is specified, focusing on the PID. This process ensures that the credential issuance aligns with the stringent requirements for PID within the EWC ecosystem.

For the pre-authorized flow, the credential response format is adapted to include the necessary grants for PID issuance:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "credential_configuration_ids": ["urn:eu.europa.ec.eudi:pid:1"],
  "grants": {
      "urn:ietf:params:oauth:grant-type:pre-authorized_code": {
        "pre-authorized_code": "asfdasfdsadfsa",
        "tx_code": {
          "length": 4,
          "input_mode": "numeric",
          "description":
            "Please provide onetimecode....",
        }
      }
   }
}
```

## 3.3 Discover request

The wallet instance initiates a request to discover the government identity provider’s authorization server configurations, essential for PID credential issuance. To obtain the issuer's configurations, the wallet resolves the /.well-known/openid-credential-issuer endpoint using the credential_issuer URI found in the PID credential offer response (as per EWC RFC001):

```http
GET https://identity-provider.gov/.well-known/openid-credential-issuer
```

Subsequently, the wallet requests the authorization server metadata endpoint to retrieve metadata (openid and oauth standard samples):

```http
GET https://identity-provider.gov/.well-known/openid-configuration
GET https://identity-provider.gov/.well-known/oauth-authorization-server
```

## 3.4 Discover response

Upon resolving the well-known endpoints, the **identity provider** responds with its configuration, tailored to support PID credential issuance. The response includes details about supported credentials, endpoints for issuing and managing credentials. It also specifies the cryptographic methods and trust frameworks applicable for PID credentials, as defined by [1]:
[https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID1.html#name-credential-issuer-metadata-p](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID1.html#name-credential-issuer-metadata-p)

Once the well-known endpoint for **authorization servers** configuration is resolved, the response will follow the oauth standard or openid specification

## 3.5 Authorization request

The authorization request seeks permission to access the PID credential endpoint. Here is an adapted example of this request, specifically aimed at PID issuance by a government identity provider:

```http
GET https://identity-provider.gov/auth/authorize?

&response_type=code
&scope=openid
&issuer_state=uniqueStateIdentifier
&state=client-state
&client_id=did%3Akey%3Az2dmzD81cgPx8Vki7JbuuMmFYrWPgYoytykUZ3eyqht1j9KbsEYvdrjxMjQ4tpnje9BDBTzuNDP3knn6qLZErzd4bJ5go2CChoPjd5GAH3zpFJP5fuwSk66U5Pq6EhF4nKnHzDnznEP8fX99nZGgwbAh1o7Gj1X52Tdhf7U4KTk66xsA5r
&authorization_details%3D%5B%7B%22format%22%3A%22jwt_vc%22%2C%22locations%22%3A%5B%22https%3A%2F%2Fissuer.example.com%22%5D%2C%22type%22%3A%22openid_credential%22%2C%22types%22%3A%5B%22urn%3Aeu.europa.ec.eudi%3Apid%3A1%22%5D%7D%5D
&redirect_uri=openid%3A
&nonce=glkFFoisdfEui43
&code_challenge=YjI0ZTQ4NTBhMzJmMmZhNjZkZDFkYzVhNzlhNGMyZDdjZDlkMTM4YTY4NjcyMTA5M2Q2OWQ3YjNjOGJlZDBlMSAgLQo%3D
&code_challenge_method=S256
&client_metadata=%7B%22vp_formats_supported%22%3A%7B%22jwt_vp%22%3A%7B%22alg%22%3A%5B%22ES256%22%5D%7D%2C%22jwt_vc%22%3A%7B%22alg%22%3A%5B%22ES256%22%5D%7D%7D%2C%22response_types_supported%22%3A%5B%22vp_token%22%2C%22id_token%22%5D%2C%22authorization_endpoint%22%3A%22openid%3A%2F%2F%22%7D

Host: https://identity-provider.gov
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
   <td>As specified in OAuth 2.0 Rich Authorization Requests specification to specify fine-grained access [4]. An example is as given below:

   ```json
   {
      "type": "openid_credential",
      "locations": [
         "https://credential-issuer.example.com"
      ],
      "format": "jwt_vc_json",
      "credential_definition": {
         "type": [
            "urn:eu.europa.ec.eudi:pid:1"
         ]
      }
   }
   ```
  
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

> Note 1: the wallet trust attestation and the wallet instance attestation shall be verified during token request step. 

> Note 2: In the authorization flow, we assume that the user will be asked to authenticate in order to provide his identity and optionally personal data will be collected and stored by identity provider. 

## 3.6 Authorization response

The credential issuer can **optionally** request additional details to authenticate the client e.g. DID authentication. In this case, the authorisation response will contain a `response_mode` parameter with the value `direct_post`. A sample response is as given:

```http
HTTP/1.1 302 Found
Location: http://localhost:8080?state=22857405-1a41-4db9-a638-a980484ecae1&client_id=https://example.server.com&redirect_uri=https://example.server.com/direct_post&response_type=id_token&response_mode=direct_post&scope=openid&nonce=a6f24536-b109-4623-a41a-7a9be932bdf6&request_uri=https://example.server.com/request_uri
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

Following this protocol, the wallet instance is expected to respond with an id_token signed by its DID to the direct post endpoint, completing the authentication:

```http
POST /direct_post
Content-Type: application/x-www-form-urlencoded
&id_token=eyJraWQiOiJkaW...a980484ecae1
```
## 3.7 Token request

In this step wallet trustwothiness in verified. 
The validation mechanism is delegated to RFC004 [7]. 
Wallet unit attestations received within token request will be verified; Wallet provider could be validated against trust framework and the wallet instance could be verified against a trustlist for valid and not revoked wallet solutions (and their versions) published by the wallet provider, if available. 
> Note: The validation of wallet instance is based on wallet unit attestation [7] 

### 3.7.1 Authorisation code flow

For PID credential issuance, the token request using the authorization code flow is structured as follows:

```http
POST /token HTTP/1.1
Host: identity-provider.gov
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

&grant_type=authorization_code
&code=SplxlOBeZQQYbYS6WxSbIA
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
&redirect_uri=https%3A%2F%2FWallet.example.org%2Fcb
```

In order to simplify the PID issuance pilot, the validation of Wallet attestations is not mandatory. Whether this validation could be performed, two headers must be included<br>
```http
OAuth-Client-Attestation: <wallet-unit-attestation-jwt>
OAuth-Client-Attestation-PoP: <wallet-unit-attestation-pop-jwt>
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

In scenarios where a pre-authorized code is used, the token request is structured as follows:

```http
POST /token HTTP/1.1
Host: identity-provider.gov
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
   <td><code>pre-authorized_code</code>
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

The token response for PID credential issuance includes:

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
This response grants the wallet an access token and a refresh token to be used  for the request of PID credential.

## 3.9 Credential request

To request the PID credential, the wallet instance sends a request to the PID Endpoint as follows:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "format": "vc+sd-jwt",
   "vct": "urn:eu.europa.ec.eudi:pid:1",
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJraW...KWjceMcr"
   }
}
```

This request specifies the format and type of credential being requested, along with a JWT proof of the user’s identity.

## 3.10 Credential response

The issuance of PID credentials may proceed directly or be deferred, contingent on the issuer's readiness to issue the credential immediately or require additional processing time.
The wallet should verify the signature pf the credential against the certificate collected at the beginning and verified against the trusted list.(the certificate used signing the credential must be the same of the one collected at the beginning).

### 3.10.1  In-time

In cases where the PID credential is immediately available, the response is structured as follows:

```json
{
  "format": "vc+sd-jwt",
  "credential": "eyJ0eXAiOi...F0YluuK2Cog", //EncodedPIDCredential
  "c_nonce": "fGFF7UkhLa", //NonceForThisCredential
  "c_nonce_expires_in": 86400
}
```

This response provides the PID credential in an encoded format, ensuring that the recipient can use it straightaway. The c_nonce ensures the response's freshness, enhancing security.
> [!NOTE] 
> A complete example of PID with sd jwt is provided in Appendix B
### 3.10.2 Deferred

Should the credential not be ready for immediate issuance, the response includes an acceptance token, signaling that the PID credential's issuance is deferred:

```json
{
  "acceptance_token": "eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400
}
```

If the response contains `acceptance_token` field, then it indicates the credential is not yet available and will be accessible through a deferred PID credential retrieval process:

```http
POST /deferred-credential
Authorization: BEARER eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ
```

The user can later use the acceptance_token to request the credential once it's ready for issuance.

## 3.11 Issuer Authorization Verification

During this process, the wallet queries the Trust Anchor to ascertain the issuer's trust status, thereby affirming that the issuer has been vetted and is compliant with established standards and regulations governing PID. It ensures that only entities with verified trustworthiness can issue PID. Further details will be added as soon as additional requirements are derived from ongoing discussions.

## 3.12 Check Wallet's Conformity

This verification process involves assessing whether the wallet possesses an internal certificate, issued by Certification Assessment Bodies (CAB), which confirms its compliance with the requisite standards for securely handling PID digital identities and associated qualified electronic attestations. It guarantees the secure storage and management of the PID. Further details will be added as soon as additional requirements are derived from ongoing discussions.

# 4.0 Alternate response format

Standard HTTP response codes shall be supported. Any additional ones can be formulated in the following format.

```
{
  "error": "invalid_request",
  "error_description": "The PID credential request is invalid or expired"
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
   <td>The request was invalid or improperly formatted. E.g. The PID credential is expired
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

# 5.0 Implementers

Please refer to the [implementers table](https://github.com/EWC-consortium/eudi-wallet-rfcs?tab=readme-ov-file#implementers).

# 6.0 Reference

1. OpenID Foundation (2024), 'OpenID for Verifiable Credential Issuance (OID4VCI)', Available at: [https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID1.html](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID1.html) (Accessed: October 10, 2024).
2. European Commission (2024) The European Digital Identity Wallet Architecture and Reference Framework (2024-09, v1.4.1)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: October 16, 2024). Detail of Annex regarding PID issuance is available at: https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md
3. OAuth 2.0 Rich Authorization Requests, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11) (Accessed: February 01, 2024)
4. Proof Key for Code Exchange by OAuth Public Clients, Available at: [https://datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) (Accessed: February 01, 2024)
5. OpenID4VC High Assurance Interoperability Profile with SD-JWT VC - draft 1.0, Available at [https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html) (Accessed: February 16, 2024)
6. Implementing Act 2024/2977, Available at  [http://data.europa.eu/eli/reg_impl/2024/2977/oj](http://data.europa.eu/eli/reg_impl/2024/2977/oj)
7. RFC004 for wallet authentication, Available at https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc004-individual-wallet-attestation.md

# Appendix A: Public key resolution

For a JWT there are multiple ways for resolving the public key using the `kid` header claim:

* If the key identifier is a DID then use a DID resolver to obtain the public key
* If the key identifier is not a DID, then resolve the JWKs endpoint in the AS configuration and match the public key from the JWK set using the key identifier.

Additionally, it is possible to specify JWK directly in the header using `jwk` header claim.

# Appendix C: PID attribute schema according to IA 
This schema has been composed according to the IA 2997 Annex [6].
The description of each attribute is present both on ARF annex [7] and IA 2997 Annex[6] (this if of course the master reference for encoding, formats and so on).
> [!NOTE]
> The json schema format is simple descriptive.

```json
{  
   "title":"PID Schema",
   "type":"object",
   "properties":{  
      "family_name":{  
         "type":"string"
      },
      "given_name":{  
         "type":"string"
      },
      "birth_date":{  
         "type":"string"      
      },
      "birth_place":{  
         "type":"string"      
      },
      "nationality":{  
         "type":"string"      
      },
      "resident_address":{
         "type":"string",
         "description":"The full address of the place where the user to whom the person identification data relates currently resides or can be contacted (street name, house number, city etc.)"
      },
      "resident_country":{  
         "type":"string"      
      },
      "resident_state":{  
          "type":"string"      
      },
      "resident_city":{  
          "type":"string"      
      },
      "resident_postal_code":{  
          "type":"string"      
      },
      "resident_street":{  
          "type":"string"      
      },
      "resident_house_number":{  
          "type":"string"      
      },
      "personal_administrative_number":{  
          "type":"string",      
          "description":"A value assigned to the natural person that is unique among all personal administrative numbers issued by the provider of person identification data. Where Member States opt to include this attribute, they shall describe in their electronic identification schemes under which the person identification data is issued, the policy that they apply to the values of this attribute, including, where applicable, specific conditions for the processing of this value."
      },
      "personal_administrative_number":{  
          "type":"string",      
          "description":"A value assigned to the natural person that is unique among all personal administrative numbers issued by the provider of person identification data. Where Member States opt to include this attribute, they shall describe in their electronic identification schemes under which the person identification data is issued, the policy that they apply to the values of this attribute, including, where applicable, specific conditions for the processing of this value."
      },
      "portrait":{  
          "type":"string",      
          "description":"Facial image of the wallet user compliant with ISO 19794-5 or ISO 39794 specifications."
      },
      "family_name_birth":{  
          "type":"string",      
          "description":"Last name(s) or surname(s) of the person identification data user at the time of birth."
      },
      "given_name_birth":{  
          "type":"string",      
          "description":"First name(s), including middle name(s), of the person identification data user at the time of birth."
      },
      "sex":{  
          "type":"number",      
          "enum":[0,1,2,3,4,5,6,9],
          "description":"Values shall be one of the following: 0 = not known; 1 = male; 2 = female; 3 = other; 4 = inter; 5 = diverse; 6 = open; 9 = not applicable. For values 0, 1, 2 and 9, ISO/IEC 5218 applies."
      },
      "email_address":{  
          "type":"string",      
          "description":"Electronic mail address of the user to whom the person identification data relates [in conformance with RFC 5322]."
      },
      "mobile_phone_number":{  
          "type":"string",      
          "description":"Mobile telephone number of the user to whom the person identification data relates, starting with the ‘+’ symbol as the international code prefix and the country code, followed by numbers only."
      }
  },
   "required":[  
      "family_name",
      "given_name",
      "birth_date",
      "birth_place",
      "nationality"
   ]
}
```

# Appendix C: SD-JWT PID example

This is an example of a PID formatted according to Reference implementation (Nov 2024 ).

```json
{
  "format": "vc+sd-jwt",
  "credential": "eyJ0eXAiOiJ2YytzZC1qd3QiLCJhbGciOiJFUzI1NiJ9.eyJfc2QiOlsiNF9QeEs3blhUY2FqYWFUWXRuVXlUVVpjTmZaX2xwLTZuX2xYeFNHa3lFSSIsIjl0ekNvNXNrN2JhN0NkZUN2akdySnlCbjhKZHY0UjJMQzhWRndPUm5ja0UiLCJBVHY0VkNzZDlSTzVxWEFFX0VLMXgwTmtjR1FBT05JSWI1OGtWRG82SU1VIiwiREdYYWl2U0FwbEtVa3Q5NmZUV29CQ3dIVUV1Rk5ROTlmMi1KeUZDV01qTSIsIk15dk1HX0IyVzltSy1Wa0FEMFJFY3BwZFJxNF9IbG5rRFlSbHNoNXBrbG8iLCJQbmg2V0JkTUREeXhkRks1YVQ4X1p3ajlpRTYyakM4RE00UDNYc0xFLUxVIiwiUWh5TDN5aXR1LURyUUZfNjhvOXoxWmJ0c0FGeWw1ckVzV0NSakxoNEpHayIsIlVudG04Sm1NT0FIYlg1TXZKcWY4LWM4Ynd2OGV6U0ZSczhjZDVEbWtOblEiXSwidmN0IjoiZXUuZXVyb3BhLmVjLmV1ZGkucGlkLjEiLCJfc2RfYWxnIjoic2hhMy0yNTYiLCJpc3MiOiJodHRwczovL2lzc3Vlci1iYWNrZW5kLmV1ZGl3LmRldiIsImNuZiI6eyJqd2siOnsia3R5IjoiUlNBIiwiZSI6IkFRQUIiLCJ1c2UiOiJzaWciLCJraWQiOiJkNzE4NWVjYS04YThiLTQwYTItOTMzYi1iM2YxNWI5YjVhMWUiLCJpYXQiOjE3MzE2Njg1OTIsIm4iOiJ6QU14YTJUeEpnM2hJS2o0V0d5RW1TTWNEbGpRY2xOVEFuZERmbHdUTnZZbldYbENBLVhCb2d6UnpBclI0OG9kSko4Yi1OcjlmNW9ZSElwOFdkTm9BczRodmUxTkJRdXdTdVlOaS1TTFZrY05ENGhuMWdWbXlpZXd6Tmx2UjZMTDdKb0JSRHRUZTNQYVI5WEFvQkhSeWNxNGpNeThyM3hMb1gwWHhtQW9jdHh0bTZyN2V5Mmg5NTF2VUVlaFZrblg1OC1STzJKanBZbDFzUldJTWJRQ19oVFlxdkgwc1ZGd1V4dG5RcWE1M2VTejlVWk81Wmt6SmE3VnM5Z1NNQ2NYUnR6OFhCWFR3V05NOWg4ZERoak56RVhScHpZcGdJYU00VElEZDRpLV9VUHM5ZU4zTFdzVWlzaVktakFTNmtFeDZhcFNHa1ppdC04YnYxSHktMldkT3cifX0sImV4cCI6MTczNDI2MDU5MywiaWF0IjoxNzMxNjY4NTkzLCJhZ2VfZXF1YWxfb3Jfb3ZlciI6eyJfc2QiOlsiaXBTODMtM1BZVWstRXN5WHpwYUMtUHRYWVNNZ1JacHFNZG5VX0JTNFd1MCJdfX0.8WXGs3v-9drBO_6DiwKZ92DrCeNyAsAIgKidFZtIzBPVj_v5idjUJimqG3GzqRgSESCo28M6WliOu31bD2QoZw~WyJVai1pTDlsX19CRktFUWp5cTJEVDRnIiwiaXNzdWFuY2VfZGF0ZSIsIjIwMjQtMTEtMTUiXQ~WyJRaEwwRXpHSXVpRlJWY0NlQ3NrY0h3IiwiZ2l2ZW5fbmFtZSIsIlR5bGVyIl0~WyJZQXlENGt2MUQ3aTJ3dUJKMl93TFNRIiwiZmFtaWx5X25hbWUiLCJOZWFsIl0~WyJIMjBSQXpDN0MzMHpqNXBYbl9QOWZRIiwiYmlydGhkYXRlIiwiMTk1NS0wNC0xMiJd~WyJMam1ManNoVUFfNzBGX2d1Mm1HWnVBIiwiMTgiLHRydWVd~WyIxMEpKTDR2ZjZrMk9kSlV2VnY0OV93IiwiZ2VuZGVyIiwxXQ~WyJBY0RWdVN2RHp1N1pvWXJvdWE4ekxRIiwiYWdlX2luX3llYXJzIiw3MF0~WyJiMEJ3Y1E5cUdxSVZQVjkzSHAtdjVBIiwiYmlydGhkYXRlX3llYXIiLCIxOTU1Il0~WyJ4QmdaX0RxS3RYR19DRllCTE01cmxRIiwiY291bnRyeSIsIkFUIl0~WyJNajJ5M0p3R29BQzhqaEIxRFc5Zjl3IiwicmVnaW9uIiwiTG93ZXIgQXVzdHJpYSJd~WyJPUHduYkpCT0s2Vll0alBMMDZWeHNRIiwibG9jYWxpdHkiLCJHZW1laW5kZSBCaWJlcmJhY2giXQ~WyJadnVxeHcxSDBwbVJMN0VWSVRDb1VnIiwicG9zdGFsX2NvZGUiLCIzMzMxIl0~WyJleDBxQmVhRXRZeU5IV2ZsaDRGTG1nIiwic3RyZWV0X2FkZHJlc3MiLCIxMDEgVHJhdW5lciJd~WyJDY0xtekpPREJGMVJKOXdyMG1NaEV3IiwiYWRkcmVzcyIseyJfc2QiOlsiLWQwTDdObFpDcnFUUW02OVloMlNrVXZnaXpqWXRydHBnNl9xRW1xdW9UYyIsIjZRdFNWV0ZWR2ZEQmhfWW14UjJYcVZYNzZmV1IxYnNiX2xWSVNNeWNQYlUiLCJXaEprR3NKcGRiVDYyM2hTR3lLVXVHM0hlMzFIbFFJY2JEdXZiZU9IendRIiwiWmVLRFo4b3NsSHZ0S3NKWDNOY2wwTHNxQlkxVkxnd2xZSGtlSTdhMExkRSIsImtnQlVrWU9ObDgydUl1MG5DRzJDaUo5bmZnZF9aZkJPd0NkMWlxUkpUblUiXX1d~", //EncodedPIDCredential
  "c_nonce": "fGFF7UkhLa", //NonceForThisCredential
  "c_nonce_expires_in": 86400
}
```
This credential can be decoded through [https://sdjwt.info/ ]
The disclosed payload
```json

{
  "iss": "https://example.com/issuer",
  "iat": 1683000000,
  "exp": 1883000000,
  "vct": "urn:eu.europa.ec.eudi:pid:1",
  "cnf": {
    "jwk": {
      "kty": "EC",
      "crv": "P-256",
      "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
      "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
    }
  },
  "is_over_18": true,
  "is_over_21": true,
  "is_over_65": true,
  "address": {
    "street_address": "123 Main St",
    "locality": "Anytown",
    "region": "Anystate",
    "country": "US"
  },
  "email": "johndoe@example.com",
  "phone_number": "+1-202-555-0101",
  "family_name": "Doe",
  "birthdate": "1940-01-01",
  "given_name": "John"
}
```

