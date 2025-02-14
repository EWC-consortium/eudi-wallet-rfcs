# EWC RFC008: Payment Data Confirmation - v1.0

**Authors:**

* Lal Chandran, iGrant.io, Sweden
* George Padayatti, iGrant.io, Sweden
* Edward Curran, Lissi GmbH, Germany
* Ranjiva Prasad, Visa Europe, UK
* Stefan Kauhaus, Visa Europe, Germany

**Reviewers:**

* Marie Austenaa, Visa Europe, UK
* Laurent Bailly, Visa Europe, France

**Status:** Approved v1.0 for release

Copyright © 2024 All Rights Reserved
Published under a Creative Commons Attribution 4.0 International License

**Table of Contents**

- [EWC RFC008: Payment Data Confirmation - v1.0](#ewc-rfc008-payment-data-confirmation---v10)
- [1.0 Summary](#10-summary)
- [2.0 Motivation](#20-motivation)
- [3.0 Pre-requisites](#30-pre-requisites)
- [4.0 Messages](#40-messages)
  - [4.1 Authorisation request](#41-authorisation-request)
  - [4.2 Authorisation response](#42-authorisation-response)
- [5.0 Validation Rules](#50-validation-rules)
  - [5.1 Holder Wallet](#51-holder-wallet)
  - [5.2 Verifier](#52-verifier)
- [6.0 Schema definitions](#60-schema-definitions)
  - [6.1 Presentation definition](#61-presentation-definition)
  - [6.2 Transaction data](#62-transaction-data)
  - [6.3 Key binding JWT](#63-key-binding-jwt)
- [7.0 References](#70-references)

# 1.0 Summary

This specification defines how a Relying Party (RP) can interact with an EUDI wallet to receive verifiable confirmation that a user has agreed to specific details of an (intended) payment transaction. The RP could be the user’s bank (two-party model) or an intermediary like a merchant or a Payment Initiation Service Provider (three-party model). In the latter case, the way how the intermediary relays the data to the user’s bank is out of the scope of this specification but would typically happen through industry-specific protocols such as e.g. EMV 3-D Secure [1] or the Berlin Group’s suite of open banking APIs [2].

This specification builds upon using the OpenID4VP protocol defined in EWC RFC 002 [3]. It also leverages the <strong><code>transaction_data</code></strong> feature recently introduced to OpenID4VP [4] to add a dynamic element to the presentation. Lastly, this specification uses the Payment Wallet Attestation (PWA) specified in EWC RFC 007 [5] as the credential referencing the wallet's key to confirm the payment transaction details. The credential format used in this document is SD-JWT VC, but others may also be supported.

This specification further details concepts developed and documented by the EWC Payment Task Force in their “Payment Authentication (SCA) using EU DI Wallets – Implementation Guide” [6].

# 2.0 Motivation

Support for high-frequency use cases relevant to citizens' daily lives, like payments, will be an important success factor for EU DI wallets. eIDAS recognises this and includes a provision requiring wallets to act as an authentication and confirmation tool when initiating payments [7]. EWC has, therefore, selected payments as one of its focus areas.

Beyond eIDAS, the solution described in this document aims to satisfy other applicable requirements from sector-specific regulations, namely Strong Customer Authentication and Dynamic Linking, as laid out in PSD2 [8], and its delegated and implementing acts.

# 3.0 Pre-requisites

The following are the pre-requisites for this RFC:

1. Issuers, wallets, and relying parties have implemented and complied with the necessary EWC RFCs, specifically the ones mentioned in this document, as well as the underlying industry standards.
2. The EUDI wallet user has a card or account with their PSP and has acquired the Payment Wallet Attestation based on EWC RFC 007 [5] or subsequent versions. This attests that the user is empowered to confirm payments for the respective cards and/or accounts.
3. While OID4VP [4] allows for multiple <strong><code>transaction_data</code></strong> objects to be processed in a single presentation flow, this document has been written with one such object in mind.

# 4.0 Messages

## 4.1 Authorisation request

The authorisation request defined in EWC RFC 002 [3] is extended to support the new parameter <strong><code>transaction_data</code></strong>. This enables a binding agreement between the user's identification/authentication and authorisation (see [Chapter 5.1 - Presentation Definition](#6-1-presentation-definition-6)) to complete confirmation of a payment transaction. It is defined as an array of strings, each of which is a base64url encoded JSON object containing a typed parameter set with details about the transaction that the Verifier requests the individuals to authorise.
The <strong><code>transaction_data</code></strong> parameter must contain a transaction data object (base64url encoded JSON object) that includes the following properties:

<table>
  <tr>
   <td colspan="3" ><strong><code>type</code></strong>
   </td>
   <td>REQUIRED. The string that identifies the type of transaction data. This value determines parameters that can be included in the transaction_data object. It is RECOMMENDED that collision-resistant names be used for type values. The value must be set to ‘payment_data’.
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong><code>credential_ids</code></strong>
   </td>
   <td>REQUIRED. An array of strings, each referencing a Credential requested by the Verifier that can be used to authorize this transaction. In Presentation Exchange, the string matches the id field in the Input Descriptor. If there is more than one element in the array, the Wallet MUST use only one of the referenced Credentials for transaction authorization.
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong><code>transaction_data_hashes_alg</code></strong>
   </td>
   <td>OPTIONAL. An array of strings, each representing a hash algorithm identifier. If this parameter is absent, a default value of sha-256 MUST be used.
   </td>
  </tr>
  <tr>
   <td colspan="3" ><strong><code>payment_data</code></strong>
   </td>
   <td>REQUIRED. An object related to payment transactions. This is introduced in EWC to support payment transactions.
<p>
This object contains the parameters given below.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td colspan="2" ><strong><code>payee</code></strong>
   </td>
   <td>REQUIRED. The name of the payee to whom the payment is being made.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td colspan="2" ><strong><code>currency_amount</code></strong>
   </td>
   <td>REQUIRED. An object conforming to the PaymentCurrencyAmount dictionary specification as defined in the W3C Payment Request API [9].
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td><strong><code>currency</code></strong>
   </td>
   <td>REQUIRED. A well-formed 3-letter alphabetic code, e.g. EUR.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td><strong><code>value</code></strong>
   </td>
   <td>REQUIRED. A valid decimal monetary value containing a monetary amount. e.g. 23.50.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td colspan="2" ><strong><code>recurring_schedule</code></strong>
   </td>
   <td>OPTIONAL. If present, it indicates a recurring payment with the following details:
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td><strong><code>start_date</code></strong>
   </td>
   <td>REQUIRED If the parent object is present. Indicates the date (ISO 8601) when the series of recurring payments has been agreed to commence.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td><strong><code>expiry_date</code></strong>
   </td>
   <td>OPTIONAL. Indicates the date (ISO 8601) when the series of recurring payments has been agreed to end. If absent, no expiration date has been agreed upon.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td><strong><code>frequency</code></strong>
   </td>
   <td>REQUIRED If the parent object is present, indicates the minimum number of days between single occurrences of the recurring schedule.
   </td>
  </tr>
</table>

## 4.2 Authorisation response

The authorisation response, as defined in EWC RFC 002 [3], is extended to include the following parameters in the key binding JWT (Defined in Chapter 6.3) as top-level claims for the IETF SD-JWT VC credential format:

<table>
  <tr>
   <td><strong><code>transaction_data_hashes</code></strong>
   </td>
   <td>An array of hashes, where each hash is calculated using a hash function over the strings received in the <strong><code>transaction_data</code></strong> request parameter. Each hash value ensures the integrity of and maps to the respective transaction data object.
   </td>
  </tr>
  <tr>
   <td><strong><code>transaction_data_hashes_alg</code></strong>
   </td>
   <td>REQUIRED when this parameter was present in the transaction_data request parameter. A string representing the hash algorithm identifier calculates hashes in the <strong><code>transaction_data_hashes</code></strong> response parameter.
   </td>
  </tr>
</table>

# 5.0 Validation Rules

## 5.1 Holder Wallet

The holder wallet must display all attributes (keys and values) contained in the <strong><code>payment_data</code></strong> object to the user as part of releasing the PWA credential and obtaining an explicit user action, signalling they confirm the details presented. It may localise (translate) keys, localise (adapt) number formats and replace or amend currency codes with symbols where appropriate and unambiguous.

## 5.2 Verifier

To verify the VP token received in the presentation response, the verifier needs to have the following information.
This information may have originated from themselves, or been relayed to them by the intermediary in a three-party model:

* The base64url encoded <strong><code>transaction_data</code></strong> string that was sent to the wallet as part of the authorization request.
* In a three party model - The original input from the user on the payment details, i.e. payee, amount, currency at least, plus any optional parameters. Obtained from the intermediary through industry-specific protocols like 3DS or open banking APIs.

With this information at hand, plus the VP token presented by the wallet, the verifier performs the following:

1. Validate the VP Token as described in OpenID4VP [4] for the IETF SD-JWT VC credential format.

* This should include validating the key binding JWT, as a PWA is always bound to a key.
* This should include checking that the timestamp <code>(iat</code>) in the key binding JWT is close (e.g. ±5 minutes) to receiving the input from the user (two-party model) or intermediary (three-party model), respectively.

2. Validate the `transaction_data`

* Validate that the original base64url encoded <strong><code>transaction_data</code></strong> string results in the same hash value when hashed with the given function as the hash contained in `transaction_data_hashes` in the key binding JWT.
* In a three party model - Validate that the payment data in the <strong><code>transaction_data</code></strong> string (once decoded) correspond to the original input from the user obtained through the intermediary.

3. Validate the suitability of the PWA

* Ensure that the PWA was issued by a suitable entity, most likely the issuer should be the same entity as verifier.
* Ensure that the PWA is valid for the funding source (card or account) in question (including non-revoked). Most likely the contents of the PWA can be cross-referenced against the verifiers internal records.

In addition to the above, the verifier may want to validate the user’s Wallet Unit Attestation. This part is outside the scope of this specification. Please see EWC RFC 004 [10] for further details.

# 6.0 Schema definitions

## 6.1 Presentation definition

Below is a non-normative example of a presentation definition to request the Payment Wallet Attestation by specifying the credential type. If the relying party has further information about other attribute values, additional constraints may be used to identify a particular PWA in the user’s wallet.
The id of the input descriptor requesting the PWA, here `7c94e62d-82c2-41d7-a649-6c204bf59d1c`, should be referenced in `credential_ids` of the authorization request.

```json
{
  "id": "8dd03977-74e4-4b10-ad6d-05a681f44fc9",
  "format": {
    "dc+sd-jwt": {
      "sd-jwt_alg_values": [
        "ES256"
      ],
      "kb-jwt_alg_values": [
        "ES256"
      ]
    }
  },
  "input_descriptors": [
    {
      "id": "7c94e62d-82c2-41d7-a649-6c204bf59d1c",
      "constraints": {
        "fields": [
          {
            "path": [
              "$.vct"
            ],
            "filter": {
              "type": "string",
              "contains": {
                "const": "PaymentWalletAttestation"
              }
            }
          },
          {
            "path": [
              "$.accounts"
            ]
          },
          {
            "path": [
              "$.account_holder_id"
            ]
          }
        ]
      }
    }
  ]
}
```

## 6.2 Transaction data

Below is a non-normative example of a <strong><code>payment_data</code></strong> object providing the payload for <strong><code>transaction_data</code></strong>.

```json
{
  "payee": "Merchant XYZ",
  "currency_amount": {
    "currency": "EUR",
    "value": "23.58"
  },
  "recurring_schedule": {
    "start_date": "2024-11-01",
    "expiry_date": "2025-10-31",
    "frequency": 30
  }
}
```

## 6.3 Key binding JWT

Below is a non-normative example of a key-binding JWT with transaction data:

```json
{
  "nonce": "n-0S6_WzA2Mj",
  "aud": "https://example.com/verifier",
  "iat": 1709838604,
  "sd_hash": "Dy-RYwZfaaoC3inJbLslgPvMp09bH-clYP_3qbRqtW4",
  "transaction_data_hashes": [ "fOBUSQvo46yQO-wRwXBcGqvnbKIueISEL961_Sjd4do" ]
}
```

# 7.0 References

1. EMV 3-D Secure overview. Available at: [https://www.emvco.com/emv-technologies/3-d-secure/](https://www.emvco.com/emv-technologies/3-d-secure/) (Last Accessed: 15 Nov, 2024)

2. Berlin Group overview. Available at: [https://www.berlin-group.org/](https://www.berlin-group.org/) (Last Accessed: 15 Nov, 2024)

3. EWC RFC 002: Present Verifiable Credentials - v1.0. Available at: [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md) (Last Accessed: 10 Nov, 2024)

4. OpenID for Verifiable Presentations - draft 22. Available at: [https://openid.net/specs/openid-4-verifiable-presentations-1_0-22.html](https://openid.net/specs/openid-4-verifiable-presentations-1_0-22.html) (Last Accessed: 10 Nov 2024)

5. EWC RFC 007: Payment Wallet Attestation - v1.0. Available at:  [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/payment-rfcs/ewc-rfc007-payment-wallet-attestation.md](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/payment-rfcs/ewc-rfc007-payment-wallet-attestation.md) (Last Accessed: 10 Nov, 2024)

6. EWC: Payment Authentication (SCA) using EU DI Wallets – Implementation Guide. Available at: [https://github.com/EWC-consortium/eudi-wallet-rfcs/tree/main/payment-rfcs/implementation-guides](https://github.com/EWC-consortium/eudi-wallet-rfcs/tree/main/payment-rfcs/implementation-guides) (Last Accessed: 15 Nov, 2024)

7. European Parliament and Council, 2024. Regulation (EU) 2024/1183 amending Regulation (EU) No 910/2014 as regards establishing the European Digital Identity Framework. Official Journal of the European Union, L 1183, pp. 1–45. Available at: [https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=OJ:L_202401183#d1e38-1-1](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=OJ:L_202401183#d1e38-1-1) (Last Accessed: 15 Nov, 2024)

8. European Parliament and Council, 2015. Directive (EU) 2015/2366 of the European Parliament and of the Council of 25 November 2015 on payment services in the internal market, amending Directives 2002/65/EC, 2009/110/EC and 2013/36/EU and Regulation (EU) No 1093/2010, and repealing Directive 2007/64/EC. Official Journal of the European Union, L 337, 23.12.2015, p. 35–127. Available at: [https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32015L2366](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32015L2366) (Last Accessed: 15 Nov, 2024)

9. W3C: Payment Request API, Editor’s Draft 09 September 2024. Available at: [https://w3c.github.io/payment-request/#paymentcurrencyamount-dictionary](https://w3c.github.io/payment-request/#paymentcurrencyamount-dictionary) (Last accessed: 18 Nov 2024)

10. EWC RFC 004: Individual Wallet Unit Attestation - v1.0. Available at:  [https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc004-individual-wallet-attestation.md](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc004-individual-wallet-attestation.md) (Last Accessed: 18 Nov, 2024)