# EWC RFC 006: Organisational Wallet Unit Attestation - v1.0

**Authors:**

* Mr. Lal Chandran, iGrant.io, Sweden
* Mr. George Padayatti, iGrant.io, Sweden
* Dr. Andreas Abraham, ValidatedID, Spain
* Dr. Martin Westerkamp, Spherity, Germany
* Mr. Ronald Koening, Spherity, Germany

**Reviewers:**

* Ms. Malin Norlander, Bolagsverket, Sweden
* Dr. Nikos Triantafyllou (University of the Aegean, Greece)

**Status:** Approved, but not mandatory

**Table of Contents**
- [EWC RFC 006: Organisational Wallet Unit Attestation - v1.0](#ewc-rfc-006-organisational-wallet-unit-attestation---v10)
- [1.0 Summary](#10-summary)
- [2.0 Motivation](#20-motivation)
- [3.0 Assumptions](#30-assumptions)
- [4.0 Organisational Wallet Unit Attestation](#40-organisational-wallet-unit-attestation)
  - [4.1 Organisational Wallet Unit Attestation JWT](#41-organisational-wallet-unit-attestation-jwt)
  - [4.2 Organisational Wallet Unit Attestation PoP JWT](#42-organisational-wallet-unit-attestation-pop-jwt)
- [5.0 Messages](#50-messages)
  - [5.1 Token Request](#51-token-request)
  - [5.2 Token Response](#52-token-response)
- [6.0 Implementers](#60-implementers)
- [7.0 Reference](#70-reference)

# 1.0 Summary

This specification designs how an organisational wallet instance acquires a wallet unit attestation. The proposal uses the OpenID4VCI protocol defined in, e.g. EWC RFC 001 [1], to interface with the Issuer (Wallet Provider). The attestation must be shared during any interaction with the issuer or verifier (E.g. as per EWC RFC 002 [2]).

It also uses OAuth 2.0 Attestation-Based Client Authentication to provide Wallet Unit Attestation towards organisational wallets as defined within ARF [2] and the EU Digital Wallet Implementing Acts [3] in the context of EWC Pilots.

# 2.0 Motivation

Assuming that a wallet provider operates the organisational wallets, the organisation needs to guarantee the wallet's trustworthiness. A WUA attestation ensures this. The lifecycle of an organisational wallet unit begins when an organisation deploys an organisational wallet solution provided by an authorised Organisational Wallet Provider.

Once created, the Organisational Wallet Provider issues an Organisational Wallet Unit Attestation (OWUA), operationalising the wallet instance. Using the OWUA, any Electronic Attribute Attestation (EAA) provider or Relying Party (RP) can authenticate the wallet unit and verify that it is a trustworthy organisational wallet unit that has not been revoked before proceeding with any transaction.

Attestation providers or relying parties must be able to confirm that the organisational wallet instance delivers the required functional capabilities and meets the necessary security standards for issuance. To facilitate this, the attestation providers or relying parties must receive proof of the wallet’s capabilities and confirmation of its revocation status.

# 3.0 Assumptions

1. The Wallet Provider is certified by an EU member state per the EU Digital Identity Implementing Regulations.
2. The OWUA is issued to and used by an organisation’s holder wallet during all interactions, e.g. with an issuer or a verifier.

# 4.0 Organisational Wallet Unit Attestation

The attestation of a wallet's capabilities through organisational wallet Unit Attestation is based on the OAuth 2.0 Attestation-Based Client Authentication draft 03 [3].

## 4.1 Organisational Wallet Unit Attestation JWT

Any Wallet Unit interacting with any external party (E.g. Another Wallet Unit, an Issuer, a Verifier, etc.) presents the following organisational Wallet Unit Attestation JWT must be presented when sending the token request as defined in EWC RFC001[1].

```json
{
        "typ": "organisation-wallet-unit-attestation+jwt",
        "alg": "ES256",
        "kid": "1"
}
.
{
        "iss": "https://attestation-service.com",
        "sub": "https://wallet-solution.com",
        "iat": 1541493724,
        "status": {
          "status_list": {
            "idx": 0,
            "uri": "https://example.com/statuslists/1"
          },
        },
        "attested_security_context" : "https://eudiwalletconsortium.org/",
        "cnf": {
                "jwk" : {
                        "kty": "EC",
                        "crv": "P-256",
                        "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
                        "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
                }
        }
}
```

This profile defines the following requirements for a Client Attestation JWT from [5] used as an OWUA JWT in this context.

1. The alg used to sign the attestation must be **<code>ES256</code></strong>
2. The kid must identify a JWK which must be resolvable as a [JWK Set](https://datatracker.ietf.org/doc/html/rfc7517#section-5) by appending <strong><code>/.well-known/jwt-issuer</code></strong> to the value of the <strong><code>iss</code></strong> claim
3. The <strong><code>iss</code></strong> claim must be the unique URL of the wallet solutions provider’s attestation service
4. All acceptable URLs for <strong><code>iss</code></strong> claims must be registered to an applicable trust registry.
5. The <strong><code>sub</code></strong> claim must be the client_id of the wallet solution and registered in an applicable trust registry.
6. The attestation should include a status claim which contains a reference to a status list as defined in [6] and which allows the attestation provider to check the state of revocation of the wallet instance
7. The value of attested_security_context must be [https://eudiwalletconsortium.org/](https://eudiwalletconsortium.org/) for the EWC LSP pilots.
8. The <strong><code>cnf</code></strong> claim must contain a JSON Web Key (JWK) object, jwk, which defines the public key corresponding to the private key used for signing the OWUA Proof of Possession (PoP) JWT, as defined in the following section.

## 4.2 Organisational Wallet Unit Attestation PoP JWT

An OWUA PoP JWT must be provided by the organisation holder wallet during any interaction to, e.g. an issuer, another holder or a relying party, as defined by section 4.3 of Attestation-based Client Authentication[5].

See below the example payload:

```json
{
  "iss": "https://wallet-solution.com",
  "aud": "https://relying-party.com",
  "nbf":1300815780,
  "exp":1300819380,
  "jti": "d25d00ab-552b-46fc-ae19-98f440f25064"
}
```

1. The **iss** value must match the **<code>sub</code></strong> claim of the OWUA JWT from section 3.1.
2. The value of <strong><code>aud</code></strong> must be the identifier of the relying party.
3. The value of <strong><code>jti</code></strong> must be a unique identifier that the relying party will store to prevent replay attacks.
4. The value of <strong><code>exp</code></strong> must be set so that the OWUA PoP JWT's maximum lifetime is no longer than 24 hours.

# 5.0 Messages

The following section describes the messaging requirements and extensions for OWUA using OpenID4VCI [1] and OpenID4VP [2].

## 5.1 Token Request

In conformance with HAIP [7], a wallet unit must send an OWUA when sending the token request and use the same public key defined in the **<code>cnf</code></strong> claim of the OWUA to create the Demonstrating Proof of Possession (DPoP) JWT [9]. The authorization server of the attestation provider must ensure that the JWK given in the header of the DPoP Proof JWT is equivalent to the JWK used within the <strong><code>cnf</code></strong> claim of the OWUA JWT.

Example Request:

```
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
OAuth-Client-Attestation: <wallet-unit-attestation-jwt>
OAuth-Client-Attestation-PoP: <wallet-unit-attestation-pop-jwt>

grant_type=authorization_code
&code=SplxlOBeZQQYbYS6WxSbIA
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
&redirect_uri=https%3A%2F%2Fwallet-solution.com%2Fcb
&authorization_details=...
```

## 5.2 Token Response

The response to the token request includes an access token and additional details required for subsequent operations.

Example Response:

```json
{
  "access_token": "eyJraWQiOiJqa3QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIi...",
  "token_type": "DPoP",
  "expires_in": 3600
  "authorization_details": [
    ...
  ]
}
```

# 6.0 Implementers

Please refer to the [implementers table](https://github.com/EWC-consortium/eudi-wallet-rfcs?tab=readme-ov-file#implementers).

# 7.0 Reference

[1]  EWC RFC001: Issue Verifiable Credential - v2.0, Available at: [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc001-issue-verifiable-credential.md](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc001-issue-verifiable-credential.md) (Accessed: 11-Dec-2024)

[2] EWC RFC002: Present Verifiable Credentials, Available at: [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md) (Accessed: 13-Dec-2024)

[3] European Commission. (2023). The European Digital Identity Wallet Architecture and Reference Framework (ARF). Available at: <https://digital-strategy.ec.europa.eu/en/library/european-digital-identity-wallet-architecture-and-reference-framework> (Accessed: 15-Nov-2024)

[4] European Commission. (2023). Commission Implementing Regulation (EU) 2024/2979 of 2023 on the technical specifications and procedures for ensuring the integrity and core functionalities of the European Digital Identity Wallet under Regulation (EU) 910/2014 of the European Parliament and of the Council, Available at: <https://eur-lex.europa.eu/eli/reg_impl/2024/2979/oj>

[5] OAuth 2.0 Attestation-Based Client Authentication Draft 03, Available at: <https://www.ietf.org/archive/id/draft-ietf-oauth-attestation-based-client-auth-03.html> (Accessed: 17-June-2024)

[6] Token Status List, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-status-list-02](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-status-list-02) (Accessed: 17-June-2024)

[7] OpenID High Assurance Interoperability Profile, Available at: [https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html#section-4.3](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html#section-4.3) (Accessed: 17-June-2024)

[8] RFC 7571 JSON Web Key, Available at: [https://datatracker.ietf.org/doc/html/rfc7517#section-5](https://datatracker.ietf.org/doc/html/rfc7517#section-5) (Accessed: 17-June-2024)

[9] RFC 9449 OAuth 2.0 Demonstrating Proof of Possession (DPoP), Available at: [https://www.rfc-editor.org/rfc/rfc9449.html](https://www.rfc-editor.org/rfc/rfc9449.html) (Accessed: 17-June-2024)

[10] EWC RFC002: Present Verifiable Credentials - v1.0. Accessible from: [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md) (Accessed: 2024/10/24)
