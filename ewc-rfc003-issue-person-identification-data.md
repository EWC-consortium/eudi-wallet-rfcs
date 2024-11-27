# EWC RFC003: Issue Person Identification Data (PID) - v1.1

**Authors:**

* Mr Leone Riello (Infocert, Italy)
* Mr Matteo Mirabelli (Infocert, Italy)
* Mr Lal Chandran (iGrant.io, Sweden)

**Reviewers:**

* Mr George Padayatti (iGrant.io, Sweden)
* Mr Jaromir Talir (NIC.cz, Czech Republic)
* Mr Roger Fagerud (DIGG, Swden)
* Mr Antti Laine (DVV, Finland)

**Status:** approved

**Table of Contents**

- [EWC RFC003: Issue Person Identification Data (PID) - v1.1](#ewc-rfc003-issue-person-identification-data-pid---v11)
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
- [Appendix B: SD-JWT PID example](#appendix-b-sd-jwt-pid-example)


# 1.0 Summary

This specification implements the OID4VCI workflow for issuing Person Identification Data (PID) credentials by government-approved identity providers within the European Wallet Ecosystem. It defines a standard process to minimize risks and ensure interoperability in issuing high-assurance PIDs across the EUDI wallet ecosystem, adhering to the requirements set forth in the ARF [2].

# 2.0 Motivation

The EWC LSP must align with the standard protocol for issuing PID from trusted and accredited sources. This uniform approach serves as the foundation for enabling interoperability between identity providers and wallet holders throughout the EWC ecosystem. This RFC assumes that users are familiar with the chosen EWC protocols and standards, and can reference the original specifications when required.

# 3.0 Messages

The PID credential issuance process incorporates comprehensive steps to ensure the security, reliability, and compliance. This includes both an authorization flow and a pre-authorized flow, with additional preliminary and post-issuance steps to align with regulatory standards and security best practices. The process is illustrated below, incorporating the critical steps of Wallet Conformity, Trust Anchor Verification, Reliable Data Acquisition, PID Generation, Secure Issuance and Storage; Renewal and Revocation Policies Management it's not in scope of this rfc.

### Preliminary Steps for PID Issuance

1. **Wallet Conformity:** Before initiating the PID issuance, the user's wallet must be confirmed to comply with established standards. This includes possessing an internal certificate from Certification Assessment Bodies (CAB) that validates its conformity, ensuring the wallet's capability to securely manage the PID and associated qualified electronic attestations.

2. **Trust Anchor Verification:** The issuing entity's authorization within the Trust Anchor framework must be validated, ensuring it is listed as an authorized actor, thus guaranteeing that only verified entities can issue the PID.

3. **Data Acquisition from Reliable Sources:** Personal data used for PID generation must be sourced from authentic and current databases, such as civil registries, ensuring the PID credentials are based on accurate and up-to-date information.

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

```

Figure 1: PID Issuance Process Incorporating Preliminary Checks

The process highlights the integration of the new preliminary steps with the traditional authorization code flow and pre-authorized code flow, adhering to the OID4VCI specification. It ensures a robust framework for digital identity issuance, from initial compliance verification to the secure generation and storage of PID credentials, followed by ongoing management.

### Post-Issuance Verification and Management

Following the issuance of the PID, initial and periodic verification procedures are crucial to maintain the validity and integrity of the PID and its related electronic attestations. This includes checking for revocation status and ongoing compliance of both the wallet and issuer within the Trust Anchor framework. Additionally, policies for the renewal and revocation of PIDs and electronic attestations must be established to address changes in the individual's status, data breaches, or compliance issues.

## 3.1 Credential offer

For PID credential issuance, this RFC allows both authorization and preauthorized flows. PID issuer will use the credential_offer_uri parameter as shown below:

```
openid-credential-offer://?credential_offer_uri=https://identity-provider.gov/pid-credential-offer

```

In this case, the `credential_offer_uri` query parameter contains the URL where the credential offer from the government-approved identity provider can be resolved. This approach ensures a streamlined user experience while maintaining the necessary information exchange for the PID issuance process. The holder wallet obtains the above by scanning a QR code for cross-device workflows or via a deeplink for same-device workflows.

## 3.2 Credential offer response

On resolving the `credential_offer_uri` query parameter, the issuer responds with details of the PID credential offer. The response format is adapted to the specific requirements of PID issuance and may include information such as the credential type related to personal identification and the applicable trust framework. The response can be in one of the following formats:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "credential_configuration_ids": ["eu.europa.ec.eudi.pid.1"],
  "grants": {
    "authorization_code": {
      "issuer_state": "eyJhbGciOiJSU0Et...FYUaBy"
    }
  }
}
```

The holder's wallet retrieves this JSON response and processes it accordingly. The format of the credential (e.g., jwt_vc, vc+sd-jwt) is specified, focusing on the PID. This process ensures that the credential issuance aligns with the stringent requirements for PID within the EWC ecosystem.

For the pre-authorized flow, the credential response format is adapted to include the necessary grants for PID issuance:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "credential_configuration_ids": ["eu.europa.ec.eudi.pid.1"],
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

The holder's wallet initiates a request to discover the government identity provider’s authorization server configurations, essential for PID credential issuance. To obtain the issuer's configurations, the wallet resolves the /.well-known/openid-credential-issuer endpoint using the credential_issuer URI found in the PID credential offer response (as per EWC RFC001):

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
&authorization_details%3D%5B%7B%22format%22%3A%22jwt_vc%22%2C%22locations%22%3A%5B%22https%3A%2F%2Fissuer.example.com%22%5D%2C%22type%22%3A%22openid_credential%22%2C%22types%22%3A%5B%22eu.europa.ec.eudi.pid.1%22%5D%7D%5D
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
            "eu.europa.ec.eudi.pid.1"
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

Following this protocol, the holder wallet is expected to respond with an id_token signed by its DID to the direct post endpoint, completing the authentication:

```http
POST /direct_post
Content-Type: application/x-www-form-urlencoded
&id_token=eyJraWQiOiJkaW...a980484ecae1
```

If no additional details are requested, the identity provider issues an authorization response containing a `code` parameter with a short-lived authorization code. This streamlined response facilitates a quick and secure exchange, vital for the sensitive nature of PID credential issuance:

```http
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?code=SplxlOBeZQQYbYS6WxSbIA
```

> [!NOTE]
> The above can be deeplinked to the EUDI wallet as well.

## 3.7 Token request

In this step wallet trustwothiness in verified. 
The validation mechanism is delegated to RFC004, still a draft in this stage. 
Wallet unit attestations received within token request will be verified; Wallet provider could be validated against trust framework and the wallet instance could be verified against a  trustlist for valid and not revoked wallet versions published by the wallet provider, if available. 
> Note: The validation of wallet is based on wallet unit attestation (rif RFC004 (WIP) [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc004-individual-wallet-attestation.md])

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

To request the PID credential, the holder’s wallet sends a request to the PID Endpoint as follows:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "format": "vc+sd-jwt",
   "vct": "eu.europa.ec.eudi.pid.1",
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJraW...KWjceMcr"
   }
}
```

This request specifies the format and type of credential being requested, along with a JWT proof of the holder’s identity.

## 3.10 Credential response

The issuance of PID credentials may proceed directly or be deferred, contingent on the issuer's readiness to issue the credential immediately or require additional processing time.

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

The holder can later use the acceptance_token to request the credential once it's ready for issuance.

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
2. European Commission (2024) The European Digital Identity Wallet Architecture and Reference Framework (2024-09, v1.4.1)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: October 16, 2024).
3. OAuth 2.0 Rich Authorization Requests, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11) (Accessed: February 01, 2024)
4. Proof Key for Code Exchange by OAuth Public Clients, Available at: [https://datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) (Accessed: February 01, 2024)
5. OpenID4VC High Assurance Interoperability Profile with SD-JWT VC - draft 1.0, Available at [https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html) (Accessed: February 16, 2024)

# Appendix A: Public key resolution

For a JWT there are multiple ways for resolving the public key using the `kid` header claim:

* If the key identifier is a DID then use a DID resolver to obtain the public key
* If the key identifier is not a DID, then resolve the JWKs endpoint in the AS configuration and match the public key from the JWK set using the key identifier.

Additionally, it is possible to specify JWK directly in the header using `jwk` header claim.

# Appendix B: SD-JWT PID example

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
  "18": {
    "value": true,
    "disclose": true
  },
  "vct": {
    "value": "eu.europa.ec.eudi.pid.1",
    "disclose": false
  },
  "_sd_alg": {
    "value": "sha3-256",
    "disclose": false
  },
  "iss": {
    "value": "https://issuer-backend.eudiw.dev",
    "disclose": false
  },
  "cnf": {
    "value": {
      "jwk": {
        "kty": "RSA",
        "e": "AQAB",
        "use": "sig",
        "kid": "d7185eca-8a8b-40a2-933b-b3f15b9b5a1e",
        "iat": 1731668592,
        "n": "zAMxa2TxJg3hIKj4WGyEmSMcDljQclNTAndDflwTNvYnWXlCA-XBogzRzArR48odJJ8b-Nr9f5oYHIp8WdNoAs4hve1NBQuwSuYNi-SLVkcND4hn1gVmyiewzNlvR6LL7JoBRDtTe3PaR9XAoBHRycq4jMy8r3xLoX0XxmAoctxtm6r7ey2h951vUEehVknX58-RO2JjpYl1sRWIMbQC_hTYqvH0sVFwUxtnQqa53eSz9UZO5ZkzJa7Vs9gSMCcXRtz8XBXTwWNM9h8dDhjNzEXRpzYpgIaM4TIDd4i-_UPs9eN3LWsUisiY-jAS6kEx6apSGkZit-8bv1Hy-2WdOw"
      }
    },
    "disclose": false
  },
  "exp": {
    "value": 1734260593,
    "disclose": false
  },
  "iat": {
    "value": 1731668593,
    "disclose": false
  },
  "age_equal_or_over": {
    "value": {
      "_sd": [
        "ipS83-3PYUk-EsyXzpaC-PtXYSMgRZpqMdnU_BS4Wu0"
      ]
    },
    "disclose": false
  },
  "issuance_date": {
    "value": "2024-11-15",
    "disclose": true
  },
  "given_name": {
    "value": "Tyler",
    "disclose": true
  },
  "family_name": {
    "value": "Neal",
    "disclose": true
  },
  "birthdate": {
    "value": "1955-04-12",
    "disclose": true
  },
  "gender": {
    "value": 1,
    "disclose": true
  },
  "age_in_years": {
    "value": 70,
    "disclose": true
  },
  "birthdate_year": {
    "value": "1955",
    "disclose": true
  },
  "country": {
    "value": "AT",
    "disclose": true
  },
  "region": {
    "value": "Lower Austria",
    "disclose": true
  },
  "locality": {
    "value": "Gemeinde Biberbach",
    "disclose": true
  },
  "postal_code": {
    "value": "3331",
    "disclose": true
  },
  "street_address": {
    "value": "101 Trauner",
    "disclose": true
  },
  "address": {
    "value": {
      "_sd": [
        "-d0L7NlZCrqTQm69Yh2SkUvgizjYtrtpg6_qEmquoTc",
        "6QtSVWFVGfDBh_YmxR2XqVX76fWR1bsb_lVISMycPbU",
        "WhJkGsJpdbT623hSGyKUuG3He31HlQIcbDuvbeOHzwQ",
        "ZeKDZ8oslHvtKsJX3Ncl0LsqBY1VLgwlYHkeI7a0LdE",
        "kgBUkYONl82uIu0nCG2CiJ9nfgd_ZfBOwCd1iqRJTnU"
      ]
    },
    "disclose": true
  }
}
```

