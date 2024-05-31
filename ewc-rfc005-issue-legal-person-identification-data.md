# EWC RFC005: Issue Legal Person Identification Data (LPID) - v0.4

**Authors:** 
* Mrs Malin Norlander (Bolagsverket, Sweden)
* Mr Lars Stenlund  (Bolagsverket, Sweden)
* Mr Björn Ström  (Bolagsverket, Sweden)

**Reviewers:** 
* Mr Lal Chandran (iGrant.io, Sweden)

**Status:** Ready for review

<!--
**Table of Contents**

- [EWC RFC005: Issue Legal Person Identification Data (LPID) - v1.0](#ewc-rfc005-issue-legal-person-identification-data-pid---v004)
- [1.0	Summary](#10summary)
- [2.0	Motivation](#20motivation)
- [3.0 LPID Issuance Process](#30lpid-issiance-process)
- [4.0 LPID Issuance to a wallet instance](#40lpid-issuance-to-a-wallet-instance)
	- [4.1 LPID Issuance Process](#40lpid-issiance-process)
- [5.0	Messages](#50messages)
  	- [5.1 Discover request](#53-discover-request)
  	- [5.2 Discover response](#54-discover-response)
  	- [5.3 Credential offer](#51credential-offer)
  	- [5.4 Credential offer response](#52credential-offer-response)
  	- [5.5 Authorisation request](#55-authorisation-request)
  	- [5.6 Authorisation response](#56-authorisation-response)
  	- [5.7 Token request](#57-token-request)
    		- [5.7.1 Authorisation code flow](#571-authorisation-code-flow)
    		- [5.7.2 Pre-authorised code flow](#572-pre-authorised-code-flow)
  	- [5.8 Token response](#58-token-response)
  	- [5.9 Credential request](#59-credential-request)
  	- [5.10 Credential response](#510-credential-response)
    		- [5.10.1  In-time](#5101--in-time)
    		- [5.10.2 Deferred](#5102-deferred)
  	- [5.11 Issuer Authorization Verification](#511-issuer-authorization-verification)
  	- [5.12 Check Wallet's Conformity](#512-check-wallets-conformity)
- [6.0	Alternate response format](#60alternate-response-format)
- [7.0	Implementers](#70implementers)
- [8.0	Reference](#80reference)
- [Appendix A: Public key resolution](#appendix-a-public-key-resolution)

-->

# 1.0	Summary

This specification implements the OID4VCI workflow for issuing Legal Person Identification Data (LPID) credentials by government-approved identity providers within the European Wallet Ecosystem. It defines a standard process to minimize risks and ensure interoperability in issuing high-assurance LPIDs across the EUDI wallet ecosystem, adhering to the requirements set forth in the ARF [2].

# 2.0	Motivation

The EWC LSP must align with the standard protocol for issuing LPID from trusted and accredited sources. This uniform approach serves as the foundation for enabling interoperability between identity providers and wallet holders throughout the EWC ecosystem. This RFC assumes that users are familiar with the chosen EWC protocols and standards, and can reference the original specifications when required. 

# 3.0 LPID Issuance Process

The LPID issuance process is implemented by PID providers according to national law. The process may differ between member states but some verifications must be implemented in an LPID issuance process across all PID Providers. These verifications include:

1. **Authentication:** The natural person requesting a LPID on behalf of a legal person must ne authenticated using LoA High.

2. **Authorization:** The representative rights of the natural person requesting a LPID must be verified.

3. **Signing:** The natural person requesting a LPID might have to sign the LPID application. This step is **optional** and only applies if regulated in national law.

4. **Status:** The status of the legal person must be verified. These states are not yet regulated, but at the very least, the legal person for which an LPID is requested must be registered at a business register.

The issuance process can either start in:
1. a wallet application, the grahical user interface of the wallet *or*
2. the eService of the PID provider.

The first approach requires the wallet provider to implement support for a lookup of all PID Providers. It also requires the User (natural person or legal person, as defined in eIDAS2 [8]) to have a wallet solution [8] that includes a wallet application. Wallet applications are optional and not all legal persons may opt for a wallet solution that includes a graphical user interface. An end user representing the legal person chooses an PID Provider in the wallet application to apply for/request an LPID from. The wallet application then redirects the end user to the eService of the PID Pprovider for the issuance process. After completeing the eService procedure, the natural person is redirected back to the wallet application from where a offer reqest is initiated.

The second approach has no dependency to the implementaion of the wallet or the wallet solution of the User. An end user representing the legal person goes directly to the eService of the PID Provider to apply for/request an LPID. After completeing the eService procedure, the PID Provider initiates a offer response to the wallet instance of the legal person which endpoint the representative of the lagal person supplied. The endpoint can be a url or a DID for instance. All communication between wallets is done in the backend.

The picture below shows a generic outline of the LPID issuance process, the actual implemntation depends on national regulations.
![Generic LPID issuance process](images/Generic%20LPID%20issuance%20process.png)

*Figure 1. Generic LPID issuance process*

**Process steps:**
Start: Natural person visists PID providers eService for LPID issuance
1. The natural person authenticates using LoA High (any LoA High certified method). (Not done during pilot as there is no LoA High available yet.)
2. PID Provider verifies the natural person credentials.
3. The natural person signs and submits the application for an LPID. (signing using existing methods)
4. PID Provider verifies the signature of the natural person.
5. PID Provider verifies the powers of the natural person to represent the legal person for which the LPID is requested (according to assumptions as national regulation isn’t in place yet).
6. PID Provider verifies the status of the legal person (according to assumptions as national regulation isn’t in place yet).
7. Verification of results from steps 4-6.
8. PID Provider asks the natural person for a wallet id or endpoint for the wallet of the legal person. (Endpoint in first phase to avoid DID registration)
9. The natural person submits wallet id/endpoint. 
10. PID Provider creates the schema with data for the LPID.
11. PID Provider publishes revocation information for the LPID to be issued.
12. PID Provider issues the LPID to the wallet instance of the legal person appointed by the natural person. (done by the PID Provider wallet, details shown in the following chapter.
End: LPID is stored in wallet instance

Once an application/request has been approved in the LPID process of a PID Provider, the creation and issuance of the LPID is done in the wallet of the PID provider. The transctions between the wallets of PID Provider and legal person are described in the chapter below.

# 4.0 LPID issuance to a wallet instance

In order to issue an LPID from an issuer wallet instance, som preliminary steps must be completed.

**Preliminary Steps for LPID Issuance:**

5. **Wallet Conformity:** Before initiating the LPID issuance, the user's wallet must be confirmed to comply with established standards. This includes possessing an wallet instance attestaion (WIA), wallet trust evidence (WTE), both issued by the wallet provider [ARF], and PID issued by a PID Provider.

6. **Trust Anchor Verification:** The issuing entity's authorization within the Trust Anchor framework must be validated, ensuring it is listed as an authorized actor, thus guaranteeing that only verified entities can issue the LPID.

7. **Data Acquisition from Reliable Sources:** Data used for LPID generation must be supplied from authentic sorces, such as business registries, ensuring the LPID credentials are based on accurate and up-to-date information. In use cases where the PID provider differs from the authentic source the PID Provider must have access to information in the authentic source.

## 4.1 Issuance

The LPID issuance process starts with a natural person applying for, or requesting depending on national regulation, an LPID on behalf of a legal person.

The LPID issuance follows detailed steps starting from the discovery of issuer capabilities, through authentication and authorization, leading to the actual credential issuance. The process is adapted to include the preliminary steps, ensuring a secure and compliant issuance path.

The sequence diagram below descibes the second option, as outlined in the chapter above, where an end user enters the eService of a PID provider directly, without use of a wallet application. For the first flow, the transactions are in a slightly different order and a discovery of the PID providers offers must be done as well, GET: /.well-known/openid-credential-issuer.

```mermaid
---
config:
  theme: base
  themeVariables:
    actorBkg: "lightgrey"
    actorBorder: "darkgrey"
    loopTextColor: "black"
    labelBoxBkgColor: "lightblue"
    labelBoxBorderColor: "grey"
---

sequenceDiagram

    autonumber
    participant NP as Natural person
    participant PP as PID Provider/authentic source
    participant PPW as PID Provider wallet
	participant CW as Legal person wallet
    participant VDR 
    participant AS as Authorization server
    
    
    NP->>PP: Apply for/request LPID at eService
    PP->>PPW: "Send LPID" request (credential schema, client wallet endpoint)
    
    Note over PPW,VDR: Wallet conformity check
    PPW->>CW: Authorization request (Presentation definition for L/PID, WIA, WTE)
    CW-->>PPW: Authorization response (VP Token with Verifiable Presentation(s) of PID, WIA, WTE)
    CW->>PPW: Authorization request (Presentation definition for L/PID, WIA)
    PPW-->>CW: Authorization response (VP Token with Verifiable Presentation(s) of PID, WIA)
   
    Note over PPW,VDR: Verification & validation
    PPW->>VDR: Request verification & validation information
    VDR-->>PPW: Information
    CW->>VDR: Request verification & validation information
    VDR-->>CW: Information
    PPW->>PPW: Verify & validate presentations (including issuers)
    CW->>CW: Verify & validate presentations (including issuers)
    
    alt Legal person wallet is Valid
    Note over PPW, CW: Credential offer
    PPW->>PPW: Create credential offer response
    PPW->>CW: Send credential offer response
       
    Note over CW,PP: Discovery of issuer capabilities
    CW->>PP: GET: /.well-known/oauth-authorization-server
    PP-->>CW: OAuth authorization server metadata

    Note over CW, AS: Authenticate & authorize
    alt Preauthorized flow
    CW->>AS: Token request
    AS-->>CW: Access token
    end
    alt Authorization flow
    CW->>AS: Authorization request
    AS-->>CW: Athorization response
    CW->>AS: Token request
    AS-->>CW: Access Token
    end
    CW->>PPW: LPID Credential request (access token)
    
    Note over PPW, CW: LPID generation, issuance and storage
    PPW->>PPW: Create credential
    PPW->>PPW: Seal credential
    PPW->>PPW: Encrypt credential
    PPW-->>CW: LPID Credential
    CW->>CW: Accept and store LPID
    
    Note over CW, NP: Confirmation
    CW->>PPW: ACK
    PPW->>PP: ACK
    end
    alt Legal person wallet is not Valid
    PPW->>PP: WARN ("Client wallet doesn't validate", CODE)
    end
    PP->>NP: Inform user
```
*Figure 2. Transactions between wallets of PID provider and legal person wallet for LPID issuance*

The sequence diagram highlights the integration of the new preliminary steps (3-7, 10-14) using OpenID4VP with the traditional authorization code flow and pre-authorized code flow, adhering to the OID4VCI specification. It ensures a robust framework for digital identity issuance, from initial compliance verification to the secure generation and storage of LPID credentials, followed by ongoing management. 

Not included in the diagram is the revocation information that must be published to a VDR. It is the responsibility of an issuer to revoke credentials when needed, this also applies to the LPID. Revocation information can be published using the PID provider wallet or directly by the PID provider to the VDR.

**Sequence diagram steps:**
1. A natural person uses the eService of an PID Provider to apply for/request an LPID.
2. The application/request for an LPID in the eService has been approved after verification checks described in chapter 3 above. The PID provider requests the PID Provider wallet to create and send an LPID. The request sent includes the information and schema needed for credential construction and the endpoint for the Client wallet. The endpoint is either given by the natural person or included in the redirection url from the wallet application.
3. The PID provider wallet requests the Client wallet for WIA, WTE, PID using the endpoint either submitted by the natural person in the first step or by information in the redirect url if the LPID process started in a wallet application with a redirect.
4. Client wallet returns presentations of WTE, WIA, (PID).
5. PID provider wallet requests information from VDR for verifiacation and validation.
6. VDR returns requested information.
7. PID provider wallet verifies and validates presentations of and issuer of WTE and WIA (and PID).
ALT - Client wallet is Valid
8. wallet creates LPID **offer response**.
9. Wallet sends LPID **offer response** to Client wallet.
10. Client wallet requests WIA and PID from PID provider wallet.
11. PID provider wallet returs presentations of PID and WIA.
12. Client wallet requests information from VDR for verifiacation and validation.
13. VDR returns requested information.
14. Client wallet verifies and validates presentations of and issuer of WTE and WIA (and PID).
ALT - Preauthorized flow
15. Client wallet requests token from autorization server.
16. Authorization server return access token.
END ALT - Preauthorized flow
ALT - Authorization flow
17. Client wallet requests access from Authorization server.
18. Authorization server returns access response.
29. Client wallet requests access from autorization server.
20. Authorization server return access token.
END ALT - Authorization flow
21. Client wallet requests LPID credential, including access token, from PID Provider wallet.
22. PID provider wallet creates LPID.
23. PID provider wallet encrypts LPID.
24. PID provider wallet  seals LPID.
25. PID provider wallet sends LPID to Client wallet.
26. Client wallet accepts LPID and stores it.
27. Client wallet sends ACK to PID provider wallet.
28. PID provider wallet sends ACK to PID provider.
END ALT - Client wallet is Valid
ALT - Client wallet is NOT Valid
29. PID provider wallet sends a warning to the PID Provider including warning code and warning text message.
END ALT - Client wallet is NOT Valid
30. PID provider informs the natural person of the result from application/request for LPID.

## 4.2 Post-Issuance Verification and Management:

Following the issuance of the LPID, policies for the renewal and revocation of LPIDs must be established by the PID Provider to reflect changes in the legal person's registered information.

# 5.0	Messages

The LPID credential issuance incorporates comprehensive steps to ensure the security, reliability, and compliance. This includes both an authorization flow and a pre-authorized flow, with additional preliminary and post-issuance steps to align with regulatory standards and security best practices. The transactions of the LPID issuance is illustrated below, incorporating the critical steps of **Wallet Conformity, Trust Anchor Verification, Reliable Data Acquisition, PID Generation, Secure Issuance and Storage, Initial and Periodic Verification, and Renewal and Revocation Policies Management**.


## 5.1 Discover request

The holder's wallet initiates a request to discover the government identity provider’s authorization server configurations, essential for LPID credential issuance. To obtain the issuer's configurations, the wallet resolves the /.well-known/openid-credential-issuer endpoint using the credential_issuer URI found in the LPID credential offer response (as per EWC RFC001):

```http
GET https://identity-provider.gov/.well-known/openid-credential-issuer
```

Subsequently, the wallet requests the `/.well-known/oauth-authorization-server` endpoint to retrieve the authorization server metadata:

```http
GET https://identity-provider.gov/.well-known/oauth-authorization-server
```

## 5.2 Discover response

Upon resolving the well-known endpoints, the **identity provider** responds with its configuration, tailored to support LPID credential issuance. The response includes details about supported credentials, endpoints for issuing and managing credentials. It also specifies the cryptographic methods and trust frameworks applicable for LPID credentials, as defined by [6]:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "authorization_server": "https://identity-provider.gov",
  "credential_endpoint": "https://identity-provider.gov/credential",
  "deferred_credential_endpoint": "https://identity-provider.gov/credential_deferred",
  "display": [
    {
      "name": "Government Identity Provider",
      "location": "Country",
      "locale": "en-GB",
      "cover": {
        "url": "https://identity-provider.gov/cover.jpeg",
        "alt_text": "Government Identity Provider"
      },
      "logo": {
        "url": "https://identity-provider.gov/logo.jpg",
        "alt_text": "Government Identity Provider"
      },
      "description": "For inquiries about how we manage your personal identification data, please contact our Data Protection Officer."
    }
  ],
  "credentials_supported": {
    "PersonIdentificationData": {
      "format": "vc+sd-jwt",
      "scope": "PersonIdentificationData",
      "cryptographic_binding_methods_supported": [
        "jwk"
      ],
      "cryptographic_suites_supported": [
        "ES256"
      ],
      "display": [
        {
          "name": "Personal Identification Data",
          "locale": "en-GB",
          "background_color": "#000000",
          "text_color": "#FFFFFF"
        }
      ],
      "credential_definition": {
        "vct": "PersonIdentificationData",
        "claims": {
          "given_name": {
            "display": [
              {
                "name": "Given Name",
                "locale": "en-GB"
              }
            ]
          },
          "surname": {
            "display": [
              {
                "name": "Surname",
                "locale": "en-GB"
              }
            ]
          }
          // Additional LPID-specific claims based on PID Rulebook
        }
      }
    }
  }
}

```

> [!NOTE]
> In order to support all EBSI conformant wallets, the following format for the response is also valid, but **optional** to be supported:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "authorization_server": "https://identity-provider.gov",
  "credential_endpoint": "https://identity-provider.gov/credential",
  "deferred_credential_endpoint": "https://identity-provider.gov/credential_deferred",
  "display": {
    "name": "Government Identity Provider",
    "location": "Country",
    "locale": "en-GB",
    "cover": {
      "url": "https://identity-provider.gov/cover.jpeg",
      "alt_text": "Government Identity Provider"
    },
    "logo": {
      "url": "https://identity-provider.gov/logo.jpg",
      "alt_text": "Government Identity Provider"
    },
    "description": "For inquiries about how we manage your personal identification data, please contact our Data Protection Officer."
  },
  "credentials_supported": [
    {
      "format": "jwt_vc",
      "types": [
        "VerifiableCredential",
        "PersonIdentificationData"
      ],
      "trust_framework": {
        "name": "ewc-issuer-trust-list",
        "type": "Accreditation",
        "uri": "Link to the trust framework accreditation"
      },
      "display": [
        {
          "name": "Person Identification Data",
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
  "issuer": "https://identity-provider.gov",
  "authorization_endpoint": "https://identity-provider.gov/authorize",
  "token_endpoint": "https://identity-provider.gov/token",
  "jwks_uri": "https://identity-provider.gov/jwks",
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
    "did:key:jwk_jcs-pub",
    "did:ebsi:v1",
    "did:ebsi:v2"
  ],
  "subject_trust_frameworks_supported": [
    "ebsi",
    "ewc-issuer-trust-list"
  ],
  "id_token_types_supported": [
    "subject_signed_id_token",
    "attester_signed_id_token"
  ]
}
```
Currently, we retain the trust framework specified by EBSI. Subsequently, we will specify an additional RFC defining the EWC trusted issuer list. 

## 5.3	Credential offer

For LPID credential issuance, the member state LPID issuer will adopt RFC001 for credential offer pre-authorised code flow, using the credential_offer_uri parameter as shown below:

```
openid-credential-offer://?credential_offer_uri=https://identity-provider.gov/pid-credential-offer

```

In this case, the `credential_offer_uri` query parameter contains the URL where the credential offer from the government-approved identity provider can be resolved. This approach ensures a streamlined user experience while maintaining the necessary information exchange for the LPID issuance process. The holder wallet obtains the above by scanning a QR code for cross-device workflows or via a deeplink for same-device workflows.

## 5.4	Credential offer response

On resolving the `credential_offer_uri` query parameter, the issuer responds with details of the LPID credential offer. The response format is adapted to the specific requirements of LPID issuance and may include information such as the credential type related to personal identification and the applicable trust framework. The response can be in one of the following formats:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "credentials": [
    "PersonIdentificationData"
  ],
  "grants": {
    "authorization_code": {
      "issuer_state": "eyJhbGciOiJSU0Et...FYUaBy"
    }
  }
}
```

> [!NOTE]
> To ensure compatibility with all wallets conforming to the European Blockchain Services Infrastructure (EBSI) standards, the following response format is also valid but **optional** to support:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "credentials": [
    {
      "format": "jwt_vc",
      "types": [
        "VerifiableCredential",
        "PersonIdentificationData"
      ],
      "trust_framework": {
        "name": "ewc-issuer-trust-list",
        "type": "Accreditation",
        "uri": "Link to the trust framework accreditation"
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

The holder's wallet retrieves this JSON response and processes it accordingly. The format of the credential (e.g., jwt_vc, vc+sd-jwt) is specified, focusing on the LPID. This process ensures that the credential issuance aligns with the stringent requirements for LPID within the EWC ecosystem.

For the pre-authorized flow, the credential response format is adapted to include the necessary grants for LPID issuance:

```json
{
  "credential_issuer": "https://identity-provider.gov",
  "credentials": [
    {
      "format": "vc+sd-jwt",
      "types": [
        "VerifiableCredential",
        "PersonIdentificationData"
      ],
      "trust_framework": {
        "name": "ewc-issuer-trust-list",
        "type": "Accreditation",
        "uri": "Link to the issuer trust list"
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

## 5.5 Authorisation request

The authorization request seeks permission to access the LPID credential endpoint. Here is an adapted example of this request, specifically aimed at LPID issuance by a government identity provider:

```http
GET https://identity-provider.gov/auth/authorize?

&response_type=code
&scope=openid
&issuer_state=uniqueStateIdentifier
&state=client-state
&client_id=did%3Akey%3Az2dmzD81cgPx8Vki7JbuuMmFYrWPgYoytykUZ3eyqht1j9KbsEYvdrjxMjQ4tpnje9BDBTzuNDP3knn6qLZErzd4bJ5go2CChoPjd5GAH3zpFJP5fuwSk66U5Pq6EhF4nKnHzDnznEP8fX99nZGgwbAh1o7Gj1X52Tdhf7U4KTk66xsA5r
&authorization_details=%5B%7B%22format%22%3A%22jwt_vc%22%2C%22locations%22%3A%5B%22https%3A%2F%2Fissuer.example.com%22%5D%2C%22type%22%3A%22openid_credential%22%2C%22types%22%3A%5B%22VerifiableCredential%22%2C%22VerifiableAttestation%22%2C%22VerifiablePortableDocumentA1%22%5D%7D%5D
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
            "VerifiableCredential",
            "PersonIdentificationData"
         ]
      }
   }
   ```
   > [!NOTE]
   > You may also use the earlier version as supported by EBSI.

   ```json
   {
    "format": "jwt_vc",
    "locations": [
      "https://issuer.example.com"
    ],
    "type": "openid_credential",
    "types": [
      "VerifiableCredential",
      "VerifiableAttestation",
      "PersonIdentificationData"
    ]
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


## 5.6 Authorisation response

In the context of LPID credential issuance, the government identity provider may **optionally** request additional details for enhanced authentication, such as DID verification. In scenarios necessitating this heightened security, the authorization response will include a `response_type` parameter set to `direct_post`. An example of such a response is:

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


Following this protocol, the holder wallet is expected to respond with an id_token signed by its DID to the direct post endpoint, completing the authentication:

```http
POST /direct_post
Content-Type: application/x-www-form-urlencoded
&id_token=eyJraWQiOiJkaW...a980484ecae1
```

If no additional details are requested, the identity provider issues an authorization response containing a `code` parameter with a short-lived authorization code. This streamlined response facilitates a quick and secure exchange, vital for the sensitive nature of LPID credential issuance:

```http
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?code=SplxlOBeZQQYbYS6WxSbIA
```

> [!NOTE]
> The above can be deeplinked to the EUDI wallet as well. 

## 5.7 Token request

### 5.7.1 Authorisation code flow

For LPID credential issuance, the token request using the authorization code flow is structured as follows:

```http
POST /token HTTP/1.1
Host: identity-provider.gov
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

### 5.7.2 Pre-authorised code flow

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

## 5.8 Token response

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

This response grants the wallet an access and a refresh token for requesting the LPID credential.


## 5.9 Credential request

To request the LPID credential, the holder’s wallet sends a request to the LPID Endpoint as follows:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "format": "vc+sd-jwt",
   "credential_definition": {
      "vct": "PersonIdentificationData"
   },
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJraW...KWjceMcr"
   }
}
```

This request specifies the format and type of credential being requested, along with a JWT proof of the holder’s identity.

> [!NOTE]
> To support all EBSI conformant wallets, the format for the request can **optionally** include specifications relevant to EBSI standards but adapting to PID-specific credential types.

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
  "format": "jwt_vc_json",
  "proof": {
    "jwt": "eyJraWQiOiJkaWQ6a2...su7UFClz9NQnw",
    "proof_type": "jwt"
  },
  "types": [
    "VerifiableCredential",
    "VerifiableAttestation",
    "PersonIdentificationData"
  ]
}
```

## 5.10 Credential response

The issuance of LPID credentials may proceed directly or be deferred, contingent on the issuer's readiness to issue the credential immediately or require additional processing time.

### 5.10.1  In-time

In cases where the LPID credential is immediately available, the response is structured as follows:

```json
{
  "format": "vc+sd-jwt",
  "credential": "eyJ0eXAiOi...F0YluuK2Cog", //EncodedLPIDCredential
  "c_nonce": "fGFF7UkhLa", //NonceForThisCredential
  "c_nonce_expires_in": 86400
}
```
This response provides the LPID credential in an encoded format, ensuring that the recipient can use it straightaway. The c_nonce ensures the response's freshness, enhancing security.

### 5.10.2 Deferred

Should the credential not be ready for immediate issuance, the response includes an acceptance token, signaling that the LPID credential's issuance is deferred:

```json
{
  "acceptance_token": "eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400
}
```

If the response contains `acceptance_token` field, then it indicates the credential is not yet available and will be accessible through a deferred LPID credential retrieval process:

```http
POST /deferred-credential
Authorization: BEARER eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ
```
The holder can later use the acceptance_token to request the credential once it's ready for issuance.

### 5.10.3 LPID Schema

The LPID attributes: legal name, EUID

<table>
  <tr>
   	<td><strong>Attribute</strong></td>
   	<td><strong>Data element identifier</strong></td>
	<td><strong>Definition</strong></td>
	<td><strong>Presence (Mandatory/Optional)</strong></td>
   	<td><strong>Proposition</strong></td>
  </tr>
  <tr>
   	<td>LegalPersonIdentifier</td>
   	<td>legal_person_identifier</td>
	<td>Unique id for legal persons</td>
	<td>M</td>
	<td>EUID</td>
  </tr>
  <tr>
   	<td>LegalPersonName</td>
   	<td>legal_person_name</td>
	<td>Name of the legal person</td>
	<td>M</td>
	<td>One statutory name</td>
  </tr>
</table>

THE LPID metadata: .....

<table>
  <tr>
   	<td><strong>Attribute</strong></td>
   	<td><strong>Data element identifier</strong></td>
	<td><strong>Definition</strong></td>
	<td><strong>Presence (Mandatory/Optional)</strong></td>
   	<td><strong>Proposition</strong></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
  <tr>
   	<td></td>
   	<td></td>
	<td></td>
	<td></td>
	<td></td>
  </tr>
</table>

```json
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "title": "EWC - Legal PID",
    "description": "Schema for Legal PID -  EWC WP3",
    "type": "object",
    "properties": {
        "issuer_name": {
            "description": "Name of issuer from the MS that issued the ODI instance",
            "type": "string"
        },
        "issuer_id": {
            "description": "ODI Id of the issuing authority. (Business register identifier for BRIS)",
            "type": "string"
        },
        "issuer_country": {
            "description": "Alpha-2 country code, as defined in ISO 3166-1, of the issuing country or territory.",
            "type": "string",
            "minLength": 2,
            "maxLength": 2
        },
        "issuance_date": {
            "description": "Date and possibly time of issuance",
            "type": "string",
            "format": "datetime"
        },
        "expiry_date": {
            "description": "Date and possibly time of ODI expiration",
            "format": "datetime"
        },
        "authentic_source_id": {
            "description": "ODI Id of the issuing authority. (Business register identifier for BRIS)",
            "type": "string"
        },
        "authentic_source_name": {
            "description": "Name of issuer from the MS that issued the ODI instance",
            "type": "string"
        },
        "credentialSubject": {
            "description": "Attributes representing a Legal PID",
            "type": "object",
            "properties": {
                "legal_person_id": {
                    "description": "Unique id for organisation according to EUID",
                    "type": "string"
                },
                "legal_person_name": {
                    "description": "Legal person name",
                    "type": "string"
                }
            },
            "required": [
                "legal_person_id",
                "legal_person_name"
            ]
        },
        "credentialStatus": {
            "description": "Defines suspension and/or revocation details for the issued credential. Further redefined by the type extension",
            "type": "object",
            "properties": {
                "id": {
                    "description": "Exact identity for the credential status",
                    "type": "string",
                    "format": "uri"
                },
                "type": {
                    "description": "Defines the revocation type extension",
                    "type": "string"
                }
            },
            "required": [
                "id",
                "type"
            ]
        },
        "credentialSchema": {
            "description": "One or more schemas that validate the Verifiable Credential.",
            "anyOf": [
                {
                    "$ref": "#/$defs/credentialSchema"
                },
                {
                    "type": "array",
                    "items": {
                        "$ref": "#/$defs/credentialSchema"
                    }
                }
            ]
        }
    },
    "required": [
        "credentialSubject",
        "credentialStatus",
        "credentialSchema",
        "issuer_name",
        "issuer_id",
        "issuer_country",
        "issuance_date",
        "expiry_date",
        "authentic_source_id",
        "authentic_source_name"
    ],
    "$defs": {
        "credentialSchema": {
            "description": "Contains information about the credential schema on which the issued credential is based",
            "type": "object",
            "properties": {
                "id": {
                    "description": "References the credential schema stored on the Trusted Schemas Registry (TSR) on which the Verifiable Authorisation is based on",
                    "type": "string",
                    "format": "uri"
                },
                "type": {
                    "description": "Defines credential schema type",
                    "type": "string",
                    "enum": [
                        "FullJsonSchemaValidator2021"
                    ]
                }
            },
            "required": [
                "id",
                "type"
            ]
        }
    }
}
```
## 5.11 Issuer Authorization Verification

During this process, the wallet queries the Trust Anchor to ascertain the issuer's trust status, thereby affirming that the issuer has been vetted and is compliant with established standards and regulations governing LPID. It ensures that only entities with verified trustworthiness can issue LPID. Further details will be added as soon as additional requirements are derived from ongoing discussions.

## 5.12 Check Wallet's Conformity

This verification process involves assessing whether the wallet possesses an internal certificate, issued by Certification Assessment Bodies (CAB), which confirms its compliance with the requisite standards for securely handling LPID digital identities and associated qualified electronic attestations. It guarantees the secure storage and management of the LPID. Further details will be added as soon as additional requirements are derived from ongoing discussions.

# 6.0	Alternate response format

Standard HTTP response codes shall be supported. Any additional ones can be formulated in the following format.

```
{
  "error": "invalid_request",
  "error_description": "The LPID credential request is invalid or expired"
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
   <td>The request was invalid or improperly formatted. E.g. The LPID credential is expired
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

# 7.0	Implementers

Please refer to the [implementers table](https://github.com/EWC-consortium/eudi-wallet-rfcs?tab=readme-ov-file#implementers).

# 8.0	Reference

1. OpenID Foundation (2023), 'OpenID for Verifiable Credential Issuance (OID4VCI)', Available at: [https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html]) (Published: February 8, 2024).
2. European Commission (2023) The European Digital Identity Wallet Architecture and Reference Framework (2024-04, v1.3.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: May 14, 2024).
3. OpenID Foundation (2023), 'Self-Issued OpenID Provider v2 (SIOP v2)', Available at: [https://openid.net/specs/openid-connect-self-issued-v2-1_0.html]) (Published: November 28, 2023)
4. OAuth 2.0 Rich Authorization Requests, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11) (Accessed: February 01, 2024)
5. Proof Key for Code Exchange by OAuth Public Clients, Available at: [https://datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) (Accessed: February 01, 2024)
6. OpenID4VC High Assurance Interoperability Profile with SD-JWT VC - draft 00, Available at [https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html) (Accessed: February 16, 2024)
7. Definition of wallet solution, [https://github.com/malinnorlander/eudi-wallet-rfcs/blob/main/images/Concept%20model.png], as defined in EWC.
8. eIDAS2, add online resource.

# Appendix A: Public key resolution

For a JWT there are multiple ways for resolving the public key using the `kid` header claim:

* If the key identifier is a DID then use a DID resolver to obtain the public key
* If the key identifier is not a DID, then resolve the JWKs endpoint in the AS configuration and match the public key from the JWK set using the key identifier.

Additionally, it is possible to specify JWK directly in the header using `jwk` header claim.
