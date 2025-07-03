# EWC RFC003: Issue Person Identification Data (PID) - v2.1

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

- [EWC RFC003: Issue Person Identification Data (PID) - v2.1](#ewc-rfc003-issue-person-identification-data-pid---v21)
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
  - [3.5 Issuer Authorization verification](#35-issuer-authorization-verification)
  - [3.6 Authorization request](#36-authorization-request)
  - [3.7 Authorization response](#37-authorization-response)
  - [3.8 Token request](#38-token-request)
    - [3.8.1 Authorization code flow](#381-authorization-code-flow)
    - [3.8.2 Pre-authorized code flow](#382-pre-authorized-code-flow)
  - [3.9 Token response](#39-token-response)
  - [3.10 Credential request](#310-credential-request)
  - [3.11 Credential response](#311-credential-response)
    - [3.11.1  In-time](#3111--in-time)
    - [3.11.2 Deferred](#3112-deferred)
  - [3.11 Issuer Authorization Verification](#311-issuer-authorization-verification)
  - [3.13 Check Wallet's Conformity](#313-check-wallets-conformity)
- [4.0 Alternate response format](#40-alternate-response-format)
- [5.0 Implementers](#50-implementers)
- [6.0 Reference](#60-reference)
- [Appendix A: Public key resolution](#appendix-a-public-key-resolution)
- [Appendix B: PID attribute schema according to IA and ARF](#appendix-b-pid-attribute-schema-according-to-ia-and-arf)
- [Appendix C: SD-JWT PID example](#appendix-c-sd-jwt-pid-example)


# 1.0 Summary

This specification implements the OID4VCI workflow for issuing Person Identification Data (PID) credentials by government-approved identity providers within the European Wallet Ecosystem. It defines a standard process to minimize risks and ensure interoperability in issuing high-assurance PIDs across the EUDI wallet ecosystem, adhering to the requirements set forth in the ARF [2] and in the implementing act 2024/2977-2979 for PID and EAA issuance [6].

# 2.0 Motivation

The EWC LSP must align with the standard protocol for issuing PID from trusted and accredited sources. This uniform approach serves as the foundation for enabling interoperability between identity providers and wallet holders throughout the EWC ecosystem. This RFC assumes that users are familiar with the chosen EWC protocols and standards, and can reference the original specifications when required.

# 3.0 Messages

The PID credential issuance process incorporates comprehensive steps to ensure the security, reliability, and compliance. This includes both an authorization flow and a pre-authorized flow, with additional preliminary and post-issuance steps to align with regulatory standards and security best practices. The process is illustrated below, incorporating the critical steps of Wallet Conformity, Trust Anchor Verification, Reliable Data Acquisition, PID Generation, Secure Issuance and Storage; Renewal and Revocation Policies Management it's not in scope of this rfc.

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
                
    Note over I,TA: Issuer Authorization Verification
    I->> I: validate signed metadata
    I->>TA: Request Issuer Authorization Status
    TA-->>I: Confirm PID provider is trusted 
    
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
    I->> I: Validation of the signature vs pid provider signature certificate

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

The wallet instance retrieves this JSON response and processes it accordingly. The format of the credential (e.g., jwt_vc, dc+sd-jwt) is specified, focusing on the PID. This process ensures that the credential issuance aligns with the stringent requirements for PID within the EWC ecosystem.

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
In the case that PID Issuer is capable of issuing PID in multiple formats, all these formats are listed in array credential_configuration_ids. 

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

## 3.5 Issuer Authorization verification 
According to IA 2977 draft [6] the PID provider must be authenticated to the wallet (art 3 comma 8,9) using a relying party access certificate (The same requirement is expressed in ETSI TS 119 471 [8] (REQ-EAASP-4.2.2.1-21)).
The authentication of the PID provider is a critical step in this process because it ensures that data of the user would not be shared with unauthorized subjects, so it's mandatory that it will take place at the beginning of the process itself. This authentication is based on the RP access certificate validation performed by the wallet instance.

PID Issuer must sign the metadata hash and must add the signed payload to a signed_metadata attribute together with the certificate. 
Taking as example the RFC001 par 3.4 Discover response, the final result could be like the following:

```json
{
  "metadata1":"xyz",
  "metadata2":"wwe",  
   ......
  "signed_metadata": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```
The wallet has 
1. to validate the signature and so the integrity of the jwt, 
2. to validate the payload against the metadata list that is in the json
3. to extract from the header the certificate and to validate it against a built-in PID issuer trusted list reference endpoint (an official authoritative source) and it validates its trusted CA root.

In any case the signature of the credential, issued at the end of the process and delivered to the wallet, must be validated against the pid provider signature certificate.
> Note: Pid Provider authentication and the definition of rp access certificates is still under development so it won't be mandatory in EWC integration test bed.

At this stage, not all PID issuer will support this authentication method: signed_metadata won't be present and the wallet should raise an alert to the user in order to inform that the PID provider authentication will not take place.
On the other side the wallet could not support this function, so signed_metadata will be ignored if present.

## 3.6 Authorization request

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

## 3.7 Authorization response

Is is not expected that PID provider requires additional data from the Holder to issue PID as PID is expected to be first credential in the wallet upon initialization. After user authentication (usually by existing notified eID mean), PID provider issues an authorization response containing a `code` parameter with a short-lived authorization code. This streamlined response facilitates a quick and secure exchange, vital for the sensitive nature of PID credential issuance:

```http
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?code=SplxlOBeZQQYbYS6WxSbIA
```

> [!NOTE]
> The above can be deeplinked to the EUDI wallet as well.

## 3.8 Token request

In this step wallet trustwothiness in verified. The validation mechanism is delegated to RFC004. 
Wallet unit attestations received within token request will be verified; Wallet provider could be validated against trust framework and the wallet instance could be verified against a  trustlist for valid and not revoked wallet versions published by the wallet provider, if available. 
The binding ow wallet instance and WUA must be verified too, in a similar manner of which the PID is bound to the wallet key in step 3.10 Credential Request/Response.
In order to simplify the PID issuance pilot, the validation of Wallet attestations is not mandatory. 

> Note: The validation of wallet is based on wallet unit attestation (rif RFC004 (WIP) [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc004-individual-wallet-attestation.md]) based on IETF attestation based client identification [11].

### 3.8.1 Authorization code flow

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
Whether the wallet attestation could be performed, two headers must be included<br>
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

### 3.8.2 Pre-authorized code flow

In scenarios where a pre-authorized code is used, the token request is structured as follows:

```http
POST /token HTTP/1.1
Host: identity-provider.gov
Content-Type: application/x-www-form-urlencoded

&grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
&pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
&tx_code=493536
```
Whether the wallet attestation could be performed, two headers must be included<br>
```http
OAuth-Client-Attestation: <wallet-unit-attestation-jwt>
OAuth-Client-Attestation-PoP: <wallet-unit-attestation-pop-jwt>
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
   <td><code>tx_code</code>
   </td>
   <td>The end user pin is decided by the issuer and sent to the holder through an out-of-band process. E.g. Email, SMS
   </td>
  </tr>
</table>

## 3.9 Token response

The token response for PID credential issuance includes:

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI4a5k..zEF",
    "token_type": "bearer",
    "expires_in": 86400,
}
```
This response grants the wallet an access token and a refresh token to be used  for the request of PID credential.

## 3.10 Credential request

To request the PID credential, the wallet instance sends a request to the PID Endpoint as follows:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
  "credential_identifier": "urn:eu.europa.ec.eudi:pid:1",
  "proofs": {
    "jwt": [
      "eyJ0eXAiOiJvcGVuaWQ0dmNpLXByb29mK2p3dCIsImFsZyI6IkVTMjU2IiwiandrIjp7Imt0eSI6IkVDIiwiY3J2IjoiUC0yNTYiLCJ4IjoiblVXQW9BdjNYWml0aDhFN2kxOU9kYXhPTFlGT3dNLVoyRXVNMDJUaXJUNCIsInkiOiJIc2tIVThCalVpMVU5WHFpN1N3bWo4Z3dBS18weGtjRGpFV183MVNvc0VZIn19",
      "eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ"
    ]
  }
}
```

This request specifies the format and type of credential being requested, along with a JWT proof of the wallet key.
PID issuer extracts from the jwt of the proof , the public key and the nonce in order to verify it correctness and integrity.

> [!NOTE] 
> The nonce of the proof is handle differently in oidc4vci versions (the one used in EWC is the 13rd).
> In version 13 before credential request, pid issuer should send a nonce to the wallet that should be signed and set in the proof. In the latest published version a nonce endpoint should be provided to the wallet to get a nonce to be signed.

If there are multiple credential_configuration_ids in credential offer, wallet sends multiple credential requests for each credential_configuration_ids in credential offer with the same access token.

## 3.11 Credential response

The issuance of PID credentials may proceed directly or be deferred, contingent on the issuer's readiness to issue the credential immediately or to require additional processing time.
This step will follow [RFC001 Chapter 3.9 - 3.10](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc001-issue-verifiable-credential.md#39-credential-request) using an in-time response and the `credential_identifier` parameter. The credential identifier values are obtained from the token response in 3.9. 

There are legal requirements (CIR 2024/2977 Article 3.5) for PID issuers to implement key binding - PID issued must be bound to the key from the wallet:

**Providers of person identification data shall ensure that person identification data that they issue is cryptographically bound to the wallet unit to which it is issued.**

In OpenID4VCI protocol this is realized via proofs: [https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html#name-proof-types](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html#name-proof-types). 
This proof is strongly tied with nonce generated by issuer.

For PID issuers there are several requirements based on this:
1. to implement nonce endpoint
2. to implement proof verification - check signature over nonce 
3. to implement cryptographic binding - add public key into PID as "cnf" attribute according to [https://www.ietf.org/archive/id/draft-ietf-oauth-selective-disclosure-jwt-15.html#name-key-binding](https://www.ietf.org/archive/id/draft-ietf-oauth-selective-disclosure-jwt-15.html#name-key-binding). Optionally, the key may also be required to be the same as one discovered during the presentation (if performed) so that the presentation can be used for wallet holder authentication.

For PID example ref to Appendix C.
> [!NOTE]
> It would be inline with HAIP to set we accept only JWT proofs that contains public key specified in JWK that should be included in cnf attribute of SD-JWT credential.
> There are several types of proofs and to ensure interoperability we should probably agree on one type that we will implement. 

### 3.11.1  In-time

In cases where the PID credential is immediately available, the response is structured as follows:

```json
{
  "credentials": [
    {
      "credential": "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L"
    },
    {
      "credential": "YXNkZnNhZGZkamZqZGFza23....29tZTIzMjMyMzIzMjMy"
    }
  ]
}
```

This response provides the PID credential in an encoded format, ensuring that the recipient can use it straightaway. The c_nonce ensures the response's freshness, enhancing security.
> [!NOTE] 
> A complete example of PID with sd jwt is provided in Appendix B
### 3.11.2 Deferred

Should the credential not be ready for immediate issuance, the response includes an acceptance token, signaling that the PID credential's issuance is deferred:

```json
{
  "transaction_id": "8xLOxBtZp8"
}
```
If the response contains `transaction_id` field, then it indicates the credential is not yet available and will be accessible through a deferred PID credential retrieval process:

```http
POST /deferred-credential
Host: server.example.com
Content-Type: application/json
Authorization: BEARER eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ
{
  "transaction_id": "8xLOxBtZp8"
}
```
## 3.11 Issuer Authorization Verification

During this process, the wallet queries the Trust Anchor to ascertain the issuer's trust status, thereby affirming that the issuer has been vetted and is compliant with established standards and regulations governing PID. It ensures that only entities with verified trustworthiness can issue PID. Further details will be added as soon as additional requirements are derived from ongoing discussions.

## 3.13 Check Wallet's Conformity

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

1. OpenID Foundation (2025), 'OpenID for Verifiable Credential Issuance (OID4VCI)', Available at: [https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID2.html](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID2.html) (Accessed: April 10, 2025).
2. European Commission (2025) The European Digital Identity Wallet Architecture and Reference Framework (2025-04, v1.8.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: April 10, 2025). Detail of Annex regarding PID issuance is available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md)
3. OAuth 2.0 Rich Authorization Requests, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11) (Accessed: February 01, 2024)
4. Proof Key for Code Exchange by OAuth Public Clients, Available at: [https://datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) (Accessed: February 01, 2024)
5. OpenID4VC High Assurance Interoperability Profile with SD-JWT VC - draft 1.0, Available at [https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-1_0.html](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-1_0.html) (Accessed: March 16, 2025)
6. Implementing Act 2024/2977, Available at  [http://data.europa.eu/eli/reg_impl/2024/2977/oj](http://data.europa.eu/eli/reg_impl/2024/2977/oj)
7. RFC004 for wallet authentication, Available at [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc004-individual-wallet-attestation.md](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc004-individual-wallet-attestation.md)
8. ETSI 119.471 v 0.0.11 [https://docbox.etsi.org/esi/Open/Latest_Drafts/ETSI%20DRAFT%20TS_119_471v0.0.11-public.pdf] (https://docbox.etsi.org/esi/Open/Latest_Drafts/ETSI%20DRAFT%20TS_119_471v0.0.11-public.pdf)
9. IANA JWT claim registry [https://www.iana.org/assignments/jwt/jwt.xhtml](https://www.iana.org/assignments/jwt/jwt.xhtml)
10. ARF tech specification for Provider Info [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md)
11. IETF Attestation based client identification [https://datatracker.ietf.org/doc/draft-ietf-oauth-attestation-based-client-auth/](https://datatracker.ietf.org/doc/draft-ietf-oauth-attestation-based-client-auth/)

# Appendix A: Public key resolution

For a JWT there are multiple ways for resolving the public key using the `kid` header claim:

* If the key identifier is a DID then use a DID resolver to obtain the public key
* If the key identifier is not a DID, then resolve the JWKs endpoint in the AS configuration and match the public key from the JWK set using the key identifier.

Additionally, it is possible to specify JWK directly in the header using `jwk` header claim.

# Appendix B: PID attribute schema according to IA and ARF
The PID scheme has been composed according to the CIR 2024/2977 Annex [6] and merging the information of the pid rulebook of the ARF [2]. Its definition reference is [https://github.com/EWC-consortium/eudi-wallet-rulebooks-and-schemas/blob/main/data-schemas/ds012-person-identification-data.json](https://github.com/EWC-consortium/eudi-wallet-rulebooks-and-schemas/blob/main/data-schemas/ds012-person-identification-data.json)
The description of each attribute is present both on ARF annex [7] and IA 2977 Annex[6] (this if of course the master reference for encoding, formats and so on).
The optional attributes that are only present in the ARF PID rulebook have been marked.
> [!NOTE]
 The json schema format is simple descriptive, and it includes both data and metadata.
 In EWC we use json shemes that do not refers specifically sdjwt or mdoc cases, but they simply describe the functional content of the PID and the other credentials.
 The ds012 (PID scheme), follows SD-JWT VC-based encoding of PID description in the ARF [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md#5-sd-jwt-vc-based-encoding-of-pid](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md#5-sd-jwt-vc-based-encoding-of-pid).
 Subject attributes are collected on credentialSubject object, while credential metadata are flattened as root object.
 Credential lifecycle status and references is described using the credentialStatus  object.
  
 As now (April 2025) the PID Rulebook is structured as follows:

1. Chapter 2 contains generic high-level requirements, which are valid for all PIDs regardless of the encoding used.
2. Chapter 3 describes the PID attributes and metadata on a generic level, regardless of the encoding used for the PID. Most of the content of this chapter is a direct copy of the Annex of Commission Implementing Regulation 2024/2977 on PID and EAA. However, a few additional attributes are specified in this chapter.
3. Chapter 4 specifies how the PID attributes and metadata are encoded in case the PID complies with [ISO/IEC 18013-5].[https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md#4-isoiec-18013-5-compliant-encoding-of-pid](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md#4-isoiec-18013-5-compliant-encoding-of-pid)
4. Chapter 5 specifies how the PID attributes and metadata are encoded in case the PID complies with [SD-JWT VC].[https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md#5-sd-jwt-vc-based-encoding-of-pid](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-3/annex-3.01-pid-rulebook.md#5-sd-jwt-vc-based-encoding-of-pid)

# Appendix C: SD-JWT PID example

This is the example of a PID formatted according to ARF PID Rulebook .

```json
{
   "credentials": [
     {
        "credential": "eyJ0eXAiOiJ2YytzZC1qd3QiLCJhbGciOiJFUzI1NiJ9.eyJfc2QiOlsiNF9QeEs3blhUY2FqYWFUWXRuVXlUVVpjTmZaX2xwLTZuX2xYeFNHa3lFSSIsIjl0ekNvNXNrN2JhN0NkZUN2akdySnlCbjhKZHY0UjJMQzhWRndPUm5ja0UiLCJBVHY0VkNzZDlSTzVxWEFFX0VLMXgwTmtjR1FBT05JSWI1OGtWRG82SU1VIiwiREdYYWl2U0FwbEtVa3Q5NmZUV29CQ3dIVUV1Rk5ROTlmMi1KeUZDV01qTSIsIk15dk1HX0IyVzltSy1Wa0FEMFJFY3BwZFJxNF9IbG5rRFlSbHNoNXBrbG8iLCJQbmg2V0JkTUREeXhkRks1YVQ4X1p3ajlpRTYyakM4RE00UDNYc0xFLUxVIiwiUWh5TDN5aXR1LURyUUZfNjhvOXoxWmJ0c0FGeWw1ckVzV0NSakxoNEpHayIsIlVudG04Sm1NT0FIYlg1TXZKcWY4LWM4Ynd2OGV6U0ZSczhjZDVEbWtOblEiXSwidmN0IjoiZXUuZXVyb3BhLmVjLmV1ZGkucGlkLjEiLCJfc2RfYWxnIjoic2hhMy0yNTYiLCJpc3MiOiJodHRwczovL2lzc3Vlci1iYWNrZW5kLmV1ZGl3LmRldiIsImNuZiI6eyJqd2siOnsia3R5IjoiUlNBIiwiZSI6IkFRQUIiLCJ1c2UiOiJzaWciLCJraWQiOiJkNzE4NWVjYS04YThiLTQwYTItOTMzYi1iM2YxNWI5YjVhMWUiLCJpYXQiOjE3MzE2Njg1OTIsIm4iOiJ6QU14YTJUeEpnM2hJS2o0V0d5RW1TTWNEbGpRY2xOVEFuZERmbHdUTnZZbldYbENBLVhCb2d6UnpBclI0OG9kSko4Yi1OcjlmNW9ZSElwOFdkTm9BczRodmUxTkJRdXdTdVlOaS1TTFZrY05ENGhuMWdWbXlpZXd6Tmx2UjZMTDdKb0JSRHRUZTNQYVI5WEFvQkhSeWNxNGpNeThyM3hMb1gwWHhtQW9jdHh0bTZyN2V5Mmg5NTF2VUVlaFZrblg1OC1STzJKanBZbDFzUldJTWJRQ19oVFlxdkgwc1ZGd1V4dG5RcWE1M2VTejlVWk81Wmt6SmE3VnM5Z1NNQ2NYUnR6OFhCWFR3V05NOWg4ZERoak56RVhScHpZcGdJYU00VElEZDRpLV9VUHM5ZU4zTFdzVWlzaVktakFTNmtFeDZhcFNHa1ppdC04YnYxSHktMldkT3cifX0sImV4cCI6MTczNDI2MDU5MywiaWF0IjoxNzMxNjY4NTkzLCJhZ2VfZXF1YWxfb3Jfb3ZlciI6eyJfc2QiOlsiaXBTODMtM1BZVWstRXN5WHpwYUMtUHRYWVNNZ1JacHFNZG5VX0JTNFd1MCJdfX0.8WXGs3v-9drBO_6DiwKZ92DrCeNyAsAIgKidFZtIzBPVj_v5idjUJimqG3GzqRgSESCo28M6WliOu31bD2QoZw~WyJVai1pTDlsX19CRktFUWp5cTJEVDRnIiwiaXNzdWFuY2VfZGF0ZSIsIjIwMjQtMTEtMTUiXQ~WyJRaEwwRXpHSXVpRlJWY0NlQ3NrY0h3IiwiZ2l2ZW5fbmFtZSIsIlR5bGVyIl0~WyJZQXlENGt2MUQ3aTJ3dUJKMl93TFNRIiwiZmFtaWx5X25hbWUiLCJOZWFsIl0~WyJIMjBSQXpDN0MzMHpqNXBYbl9QOWZRIiwiYmlydGhkYXRlIiwiMTk1NS0wNC0xMiJd~WyJMam1ManNoVUFfNzBGX2d1Mm1HWnVBIiwiMTgiLHRydWVd~WyIxMEpKTDR2ZjZrMk9kSlV2VnY0OV93IiwiZ2VuZGVyIiwxXQ~WyJBY0RWdVN2RHp1N1pvWXJvdWE4ekxRIiwiYWdlX2luX3llYXJzIiw3MF0~WyJiMEJ3Y1E5cUdxSVZQVjkzSHAtdjVBIiwiYmlydGhkYXRlX3llYXIiLCIxOTU1Il0~WyJ4QmdaX0RxS3RYR19DRllCTE01cmxRIiwiY291bnRyeSIsIkFUIl0~WyJNajJ5M0p3R29BQzhqaEIxRFc5Zjl3IiwicmVnaW9uIiwiTG93ZXIgQXVzdHJpYSJd~WyJPUHduYkpCT0s2Vll0alBMMDZWeHNRIiwibG9jYWxpdHkiLCJHZW1laW5kZSBCaWJlcmJhY2giXQ~WyJadnVxeHcxSDBwbVJMN0VWSVRDb1VnIiwicG9zdGFsX2NvZGUiLCIzMzMxIl0~WyJleDBxQmVhRXRZeU5IV2ZsaDRGTG1nIiwic3RyZWV0X2FkZHJlc3MiLCIxMDEgVHJhdW5lciJd~WyJDY0xtekpPREJGMVJKOXdyMG1NaEV3IiwiYWRkcmVzcyIseyJfc2QiOlsiLWQwTDdObFpDcnFUUW02OVloMlNrVXZnaXpqWXRydHBnNl9xRW1xdW9UYyIsIjZRdFNWV0ZWR2ZEQmhfWW14UjJYcVZYNzZmV1IxYnNiX2xWSVNNeWNQYlUiLCJXaEprR3NKcGRiVDYyM2hTR3lLVXVHM0hlMzFIbFFJY2JEdXZiZU9IendRIiwiWmVLRFo4b3NsSHZ0S3NKWDNOY2wwTHNxQlkxVkxnd2xZSGtlSTdhMExkRSIsImtnQlVrWU9ObDgydUl1MG5DRzJDaUo5bmZnZF9aZkJPd0NkMWlxUkpUblUiXX1d~", //EncodedPIDCredential
     }
   ]
}
```
This credential can be decoded through [https://sdjwt.info/ ]
The disclosed payload
```json

{
    "vct": "urn:eu.europa.ec.eudi:pid:1",
    
    "given_name": "Jean",
    "family_name": "Dupont",
    "birthdate": "1980-05-23",

    "age_equal_or_over": {
        "12": true,
        "14": true,
        "16": true,
        "18": true,
        "21": true,
        "65": false
    },
    "age_in_years": 44,
    "age_birth_year": 1980,

    "address": {
        "street_address": "123 Via Appia",
        "locality": "Rome",
        "region": "Lazio",
        "postal_code": "00100",
        "country": "IT"
    },

    "nationalities": ["FR"],

    "sex": 5,

    "place_of_birth": {
        "country": "DD"
    },

    "cnf": {
        "jwk": {
            "kty": "EC",
            "crv": "P-256",
            "x": "52aDI_ur05n1f_p3jiYGUU82oKZr3m4LsAErM536crQ",
            "y": "ckhZ-KQ5aXNL91R8Eufg1aOf8Z5pZJnIvuCzNGfdnzo"
        }
    },

    "issuing_authority": "DE",
    "issuing_country": "DE"
}
```
