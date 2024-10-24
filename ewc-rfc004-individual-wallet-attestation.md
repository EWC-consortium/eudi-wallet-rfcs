# EWC RFC004: Individual Wallet Attestation - v1.0

**Authors:** 
* Mr Sebastian Bickerle (Lissi GmbH, Germany)
* Mr Edward Curran (Lissi GmbH, Germany)

**Reviewers:**
* Dr Andreas Abraham (ValidatedID, Spain)
* Mr George Padaytti (iGrant.io, Sweden)
* Mr Lal Chandran (iGrant.io, Sweden)
* Dr Nikos Triantafyllou (University of the Aegean, Greece)

**Status:**
Current: OAuth 2.0 Attestation-Based Client Authentication Draft 03 [[1](#60reference)]

**Table of Contents**

- [EWC RFC004: Individual Wallet Attestation - v1.0](#ewc-rfc004-individual-wallet-attestation---v10)
- [1.0	Summary](#10summary)
- [2.0	Motivation](#20motivation)
- [3.0	Wallet Unit Attestation (WUA)](#30wallet-unit-attestation-wua)
  - [3.1 Wallet Unit Attestation JWT](#31-wallet-unit-attestation-jwt)
    - [3.2 Wallet Unit Attestation PoP JWT](#32-wallet-unit-attestation-pop-jwt)
- [4.0	Messages](#40messages)
  - [4.1 Wallet Unit Attestation within OID4VCI](#41-wallet-unit-attestation-within-oid4vci)
    - [4.1.2 Token Request](#412-token-request)
- [5.0	Implementers](#50implementers)
- [6.0	References](#60references)


# 1.0	Summary

This specification defines a profile for using OAuth 2.0 Attestation-Based Client Authentication to provide Wallet Unit Attestation as defined within the implementing acts for the context of EWC Pilots. 

# 2.0	Motivation

An EUDI Wallet Instance's life cycle begins when the individual installs an EUDI Wallet app provided by an authorized EUDI Wallet Provider. Once installed, the EUDI Wallet Provider issues a Wallet Unit Attestation (WUA), and the instance is now operational [Ref: ARF 1.4, Chapter 6.5.3]. 
Using the WUA, an electronice attribute attestation (EAA) provider can authenticate a wallet unit and ensure that it is a valid EUDI wallet that has not been revoked before issuing an EAA.

An attestation provider must be able to determine if a wallet instance offers the requireed functional capabilities and meets the security requirements for issuance. For this purpose, the attestation provider must receive an attestation of the wallet’s capabilities and proof of non-revocation for the wallet instance.

# 3.0	Wallet Unit Attestation (WUA)

The attestation of a wallet's capabilities through Wallet Unit Attestation is based on the OAuth 2.0 Attestation-Based Client Authentication draft 03[1].

## 3.1 Wallet Unit Attestation JWT

The following Wallet Unit Attestation JWT must be presented when sending the token request as defined in RFC001[7].

```
{
        "typ": "wallet-unit-attestation+jwt",
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


This profile defines the following additional requirements for a Client Attestation JWT from [1] used as a WUA JWT in this context.

1. The `alg` used to sign the attestation must be “ES256”
2. The `kid` must identify a JWK which must be resolvable as a JWK Set [https://datatracker.ietf.org/doc/html/rfc7517#section-5] by appending ./well-known/jwt-issuer to the value of the iss claim
3. The `iss` claim must be the unique URL of the wallet solutions provider’s attestation service
4. All acceptable URLs for `iss` claim must be registered within section 5.
5. The `sub` claim must be the client_id of the wallet solution and registered within section 5.
6. The attestation should include a `status` claim which contains a reference to a status list as defined in [3] and which allows the attestation provider to check the state of revocation of the wallet instance
7. The value of `attested_security_context` must be “https://eudiwalletconsortium.org/”
8. The `cnf` claim must contain a Json Web Key (JWK) object `jwk` which must define the public key corresponding to the private key used for signing the WUA PoP JWT as defined in the following section.

### 3.2 Wallet Unit Attestation PoP JWT

A WUA Proof of Possession JWT must be provided as defined by section 4.3 of Attestation-based Client Authentication[1].

See below the example payload:
```
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
## 4.1 Wallet Unit Attestation within OID4VCI
The following messages of RFC001[7] must be extended with the corresponding wallet attestations.

### 4.1.2 Token Request
Based on this profile and in conformance with HAIP section 4.3 [5] a wallet instance must send a WUA when sending the token request and use the same public key defined in the “cnf” claim of the WUA to create the Demonstrating Proof of Possession (DPoP) JWT [6]. The authorization server of the attestation provider must ensure that the JWK given in the header of the DPoP Proof JWT is equivalent to the JWK used within the `cnf` claim of the WUA JWT.


Request:
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

Response:
```
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
<table>
<tr><th>Wallet</th><th>Qualified Redirect URL (client_id / sub)</th><th>Issuer (iss)</th>
<tr><td>Lissi Wallet</td><td>https://oob.lissi.io</td><td>https://wallet.lissi.io</td></tr>
<tr><td>Data Wallet by iGrant.io</td><td>https://igrant.io/datawallet.html</td><td>https://demo-business.igrant.io/</td></tr>
</table>

# 6.0	References

[1] OAuth 2.0 Attestation-Based Client Authentication Draft 03 https://www.ietf.org/archive/id/draft-ietf-oauth-attestation-based-client-auth-03.html
Accessed: 2024/06/17

[2] OpenID for Verifiable Credentials Issuance Draft 13 https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html#section-5.1.4
Accessed: 2024/06/17

[3] Token Status List https://datatracker.ietf.org/doc/html/draft-ietf-oauth-status-list-02
Accessed: 2024/06/17

[4] RFC 7571 JSON Web Key https://datatracker.ietf.org/doc/html/rfc7517#section-5
Accessed: 2024/06/17

[5] OpenID High Assurance Interoperability Profile
https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html#section-4.3
Accessed: 2024/06/17

[6] RFC 9449 OAuth 2.0 Demonstrating Proof of Possession (DPoP)
 https://www.rfc-editor.org/rfc/rfc9449.html
Accessed: 2024/06/17

[7] EWC RFC001: Issue Verifiable Credential - v2.0 https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc001-issue-verifiable-credential.md
Accessed: 2024/10/24
