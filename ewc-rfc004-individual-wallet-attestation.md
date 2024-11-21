# EWC RFC004: Individual Wallet Unit Attestation - v1.0

**Authors:** 

* Mr Sebastian Bickerle (Lissi GmbH, Germany)
* Mr Edward Curran (Lissi GmbH, Germany)
* Mr George Padayatti (iGrant.io, Sweden)

**Reviewers:**

* Mr Lal Chandran (iGrant.io, Sweden)
* Dr Andreas Abraham (ValidatedID, Spain)
* Dr Nikos Triantafyllou (University of the Aegean, Greece)

**Status:** Approved

**Table of Contents**
- [EWC RFC004: Individual Wallet Unit Attestation - v1.0](#ewc-rfc004-individual-wallet-unit-attestation---v10)
- [1.0	Summary](#10summary)
- [2.0	Motivation](#20motivation)
- [3.0	Wallet Unit Attestation (WUA)](#30wallet-unit-attestation-wua)
  - [3.1 Wallet Unit Attestation JWT](#31-wallet-unit-attestation-jwt)
  - [3.2 Wallet Unit Attestation PoP JWT](#32-wallet-unit-attestation-pop-jwt)
- [4.0	Messages](#40messages)
  - [4.1 Token request](#41-token-request)
  - [4.2 Token response](#42-token-response)
- [5.0	Implementers](#50implementers)
- [6.0	References](#60references)

# 1.0	Summary

This specification outlines how an individual wallet instance obtains a Wallet Unit Attestation from the Wallet Provider. It employs OAuth 2.0 Attestation-Based Client Authentication to deliver Wallet Unit Attestation to organisational wallets, as defined within the European ARF [1] and the EU Digital Identity Wallet Implementing Acts [2], in the context of EWC Pilots.

# 2.0	Motivation

An EUDI Wallet Instance lifecycle begins when an individual installs an EUDI Wallet app provided by a certified EUDI Wallet Provider. Upon installation, the EUDI Wallet Provider issues a Wallet Unit Attestation (WUA), enabling the instance to become operational.

Using the WUA, an Electronic Attribute Attestation (EAA) provider can authenticate a wallet unit and verify that it is a valid EUDI wallet that has not been revoked before issuing an EAA.

An attestation provider must be able to determine whether a wallet instance possesses the required functional capabilities and satisfies the security requirements for issuance. To achieve this, the attestation provider must receive proof of the wallet’s capabilities and evidence of non-revocation for the wallet instance.

# 3.0	Wallet Unit Attestation (WUA)

The attestation of a wallet's capabilities through Wallet Unit Attestation is based on the OAuth 2.0 Attestation-Based Client Authentication draft 03 [3].

## 3.1 Wallet Unit Attestation JWT

The following Wallet Unit Attestation JWT must be presented when sending the token request defined in RFC001[8] to issuers.

The header block is as given below:

```json
{
  "typ": "wallet-unit-attestation+jwt",
  "alg": "ES256",
  "kid": "1"
}
```

And the claim block:

```json
{
  "iss": "https://attestation-service.com",
  "sub": "https://wallet-solution.com",
  "iat": 1541493724,
  "status": {
    "status_list": {
      "idx": 0,
      "uri": "https://example.com/statuslists/1"
    }
  },
  "attested_security_context": "https://eudiwalletconsortium.org/",
  "cnf": {
    "jwk": {
      "kty": "EC",
      "crv": "P-256",
      "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
      "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
    }
  }
}
```

This profile defines the following requirements for a Client Attestation JWT from [3] used as a WUA JWT in this context.

1. The `alg` used to sign the attestation must be `ES256`
2. The `kid` must identify a JWK which must be resolvable as a JWK Set [5] by appending `./well-known/jwt-issuer` to the value of the `iss` claim
3. The `iss` claim must be the unique URL of the wallet solutions provider’s attestation service
4. All acceptable URLs for `iss` claim must be registered within wallet provider trust list.
5. The `sub` claim must be the `client_id` of the wallet solution and registered within wallet provider trust list.
6. The attestation should include a `status` claim which contains a reference to a status list as defined in [4] and which allows the attestation provider to check the state of revocation of the wallet instance
7. The value of `attested_security_context` must be `https://eudiwalletconsortium.org/`
8. The `cnf` claim must contain a Json Web Key (JWK) object `jwk` which must define the public key corresponding to the private key used for signing the WUA PoP JWT as defined in the following section.

## 3.2 Wallet Unit Attestation PoP JWT

A WUA Proof of Possession JWT must be provided as defined by section 4.3 of Attestation-based Client Authentication [3].

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

1. The value of `iss` must exactly match the `sub` claim of the the WUA JWT from section 3.1
2. The value of `aud` must be the identifier of the relying party
3. The value of `jti` must be a unique identifier that the relying party will store to prevent replay attacks
4. The value of `exp` must be set so that the WUA PoP JWT's maximum lifetime is no longer than 24 hours

# 4.0	Messages

The following messages of EWC RFC001 [8] must be extended with the corresponding wallet attestations to enable secure communication within OID4VCI. Wallet Unit Attestations (WUAs) are critical for ensuring the integrity and authenticity of token exchanges.

## 4.1 Token request

Based on this profile and in conformance with HAIP section 4.3 [6], a wallet instance must send a Wallet Unit Attestation (WUA) when sending the token request. It must use the same public key defined in the WUA's `cnf` claim to create the Demonstrating Proof of Possession (DPoP) JWT [7].

The authorization server of the attestation provider must ensure that the JWK given in the header of the DPoP Proof JWT is equivalent to the JWK used within the `cnf` claim of the WUA JWT.

Example Request:
```http
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

## 4.2 Token response

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

# 5.0	Implementers

Please refer to the [implementers table](https://github.com/EWC-consortium/eudi-wallet-rfcs?tab=readme-ov-file#implementers).

# 6.0	References

[1] European Commission. (2023). The European Digital Identity Wallet Architecture and Reference Framework (ARF). Available at: https://digital-strategy.ec.europa.eu/en/library/european-digital-identity-wallet-architecture-and-reference-framework (Accessed: 15-Nov-2024)

[2] European Commission. (2023). Commission Implementing Regulation (EU) 2023/XXXX of 2023 on the technical specifications and procedures for ensuring the integrity and core functionalities of the European Digital Identity Wallet under Regulation (EU) 910/2014 of the European Parliament and of the Council.

[3] OAuth 2.0 Attestation-Based Client Authentication Draft 03, Available at: https://www.ietf.org/archive/id/draft-ietf-oauth-attestation-based-client-auth-03.html (Accessed: 17-June-2024)

[4] Token Status List, Available at: https://datatracker.ietf.org/doc/html/draft-ietf-oauth-status-list-02 (Accessed: 17-June-2024)

[5] RFC 7571 JSON Web Key, Available at: https://datatracker.ietf.org/doc/html/rfc7517#section-5 (Accessed: 17-June-2024)

[6] OpenID High Assurance Interoperability Profile, Available at: https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html#section-4.3 (Accessed: 17-June-2024)

[7] RFC 9449 OAuth 2.0 Demonstrating Proof of Possession (DPoP), Available at: https://www.rfc-editor.org/rfc/rfc9449.html (Accessed: 17-June-2024)

[8] EWC RFC001: Issue Verifiable Credential - v2.0, Available at: https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc001-issue-verifiable-credential.md (Accessed: 17-June-2024)