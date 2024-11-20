<h1 align="center">
    EWC - EUDI Wallet Request for Comments (RFCs) 
</h1>

<p align="center">
    <a href="/../../commits/" title="Last Commit"><img src="https://img.shields.io/github/last-commit/EWC-consortium/eudi-wallet-rfcs?style=flat"></a>
    <a href="/../../issues" title="Open Issues"><img src="https://img.shields.io/github/issues/EWC-consortium/eudi-wallet-rfcs?style=flat"></a>
    <a href="./LICENSE" title="License"><img src="https://img.shields.io/badge/License-Apache%202.0-yellowgreen?style=flat"></a>
</p>

<p align="center">
  <a href="#about">About</a> •
  <a href="#contributing">Contributing</a> •
  <a href="#approved-rfcs">Approved RFCs</a> •
  <a href="#ewc-wallet-implementers-itb---compliant-to-ewc-release-20">Implementers</a> •
  <a href="#licensing">Licensing</a>
</p>

## About

This repository is created to align the implementation of digital wallets across the EWC LSP consortium wallet providers. This also ensures all wallet providers can self-test and sign off against the EWC Interoperability Test Bed (ITB).

## Contributing

Feel free to improve the plugin and send us a pull request. If you find any problems, please create an issue in this repo. Once implemented and aligned with the RFCs, wallet providers can raise a PR.

## RFC Process Summary for EWC Large Scale Pilots (LSPs)

RFCs progress through the following process towards the Large Scale Pilots (LSPs):

![RFC Process Workflow](https://github.com/user-attachments/assets/5fc6cf87-9364-47bc-8666-8817b07779df)

To ensure interoperability with other wallets (issuer/holder/verifier) within the **European Wallet Consortium (EWC)**, all wallets, issuer-only, or verifier-only applications must comply with the EWC RFC and complete compliance testing with the **EWC Interoperability Test Bed (ITB)**. For guidance and support, please use the `#wallet-support` channel in the [EWC Slack Workspace](https://eudigitaliden-gax7504.slack.com/archives/C063LNT4L4R).

## Current RFCs

### Approved RFCs

These are the approved RFCs identified to be specified towards wallet providers, producing reference codes and the EWC Interoperability Test Bed (ITB). The supported verifiable credentials and presentations are specified in the [EWC supported format csv file](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-supported-formats.csv).

| **RFC #** | **RFC Title**                                                                                                |
| --------- | ------------------------------------------------------------------------------------------------------------ |
| RFC-001   | [Issue Verifiable Credentials Workflow - v2.0](ewc-rfc001-issue-verifiable-credential.md)                    |
| RFC-002   | [Present Verifiable Credentials Workflow - v1.0](ewc-rfc002-present-verifiable-credentials.md)               |
| RFC-005   | [Issue Legal Person Identification Data (LPID) - v1.0](ewc-rfc005-issue-legal-person-identification-data.md) |
| RFC-007   | [Payment Wallet Attestation - v1.0](payment-rfcs/ewc-rfc007-payment-wallet-attestation.md)                   |
| RFC-100   | [EWC Interoperability Profile Towards ITB - v2.0](ewc-rfc100-interoperability-profile-towards-itb-v1.0.md)   |

### RFCs Under Development

Following are the candidates' RFCs taken up. Note that the title, etc, may change.

| **RFC #** | **RFC Title**                                                                               |
| --------- | ------------------------------------------------------------------------------------------- |
| RFC-003   | [Issue Person Identification Data Issuance](ewc-rfc003-issue-person-identification-data.md) |
| RFC-004   | [Individual Wallet Unit Attestation](ewc-rfc004-individual-wallet-attestation.md)           |
| RFC-006   | Organisational Wallet Unit Attestation                                                      |
| RFC-008   | [Payment Data Confirmation - v0.9](payment-rfcs/ewc-rfc008-payment-data-confirmation.md)    |
| RFC-009   | Payment Transaction Initiation                                                              |
| RFC-010   | Synchronous Document Signing using Long-Term Certificates                                   |

## EWC Wallet Implementers (ITB - Compliant to EWC release 2.0)

NOTE: The wallets scheduled to be compliant with EWC release 2.0 based on the survey concluded on 15 October 2024 are as given:

### Individual Wallets

Date of availability: 15 October 2024

| Wallet                | Link                                                                                                                                                 | Holder | Issuer | Verifier |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | :----: | :----: | :------: |
| iGrant.io Data Wallet | [iOS](https://apple.co/2Mz9nJp), [Android](https://play.google.com/store/apps/details?id=io.igrant.mobileagent)                                      |   ✅    |   ✅    |    ✅     |
| Lissi ID-Wallet       | [iOS](https://testflight.apple.com/join/9AWbZISv), [Android](https://play.google.com/store/apps/details?id=io.lissi.mobile.android.beta)             |   ✅    |        |          |
| ValidatedID           | [iOS](https://apps.apple.com/us/app/id-wallet-lsp/id6504026408), [Android](https://play.google.com/store/apps/details?id=com.vididentity.wallet.lsp) |   ✅    |   ✅    |    ✅     |
| Digidentity         | [iOS](https://apps.apple.com/app/id916749732), [Android](https://play.google.com/store/apps/details?id=com.digidentity)                              |   ✅    |   ✅    |          |

### Organisation Wallets (Holder, Issuers and Verifiers)

Date of availability: 15 November 2024

| Wallet                        | Link                                                                                   | Holder | Issuer | Verifier |
| ----------------------------- | -------------------------------------------------------------------------------------- | :----: | :----: | :------: |
| iGrant.io Organisation Wallet | [https://demo-business.igrant.io/](https://demo-business.igrant.io/)                   |   ✅    |   ✅    |    ✅     |
| SICPA Digital Trust Suite     | [https://docs.dip.sicpa.com/](https://docs.dip.sicpa.com/)                             |   ✅    |   ✅    |    ✅     |
| Archipels                     | [https://app.archipels.io/](https://app.archipels.io/)                                 |        |   ✅    |    ✅     |
| ValidatedID                   | [https://staging-studio.vidchain.net/login](https://staging-studio.vidchain.net/login) |        |   ✅    |    ✅     |
| Mini Suomi (Vero, Finland)    | Link to be provided                                                                    |   ✅    |   ✅    |    ✅     |

## EWC Wallet Implementers (ITB - Compliant to EWC release 1.0)

NOTE: These were the wallets compliant with EWC release 1.0.

### Holder Wallets for Individuals

| Wallet                | Link                                                                                                                                                 | Holder | Issuer | Verifier |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | :----: | :----: | :------: |
| iGrant.io Data Wallet | [iOS](https://apple.co/2Mz9nJp), [Android](https://play.google.com/store/apps/details?id=io.igrant.mobileagent)                                      |   ✅    |   ✅    |    ✅     |
| Lissi ID-Wallet       | [iOS](https://testflight.apple.com/join/9AWbZISv), [Android](https://play.google.com/store/apps/details?id=io.lissi.mobile.android.beta)             |   ✅    |        |          |
| ValidatedID           | [iOS](https://apps.apple.com/us/app/id-wallet-lsp/id6504026408), [Android](https://play.google.com/store/apps/details?id=com.vididentity.wallet.lsp) |   ✅    |   ✅    |    ✅     |

### Organisation Wallets (Holder, Issuers and Verifiers)

| Wallet                          | Link                                                                    | Holder | Issuer | Verifier |
| ------------------------------- | ----------------------------------------------------------------------- | :----: | :----: | :------: |
| iGrant.io Organisation Wallet   | [Organisation Wallet Dashboard](https://demo-business.igrant.io/), [DevDocs](https://docs.igrant.io/docs/organisation-wallet-overview/)            |   ✅    |   ✅    |    ✅     |
| Lissi ID-Wallet Connector       | [lissi.id](https://lissi.id)                                            |   ✅    |   ✅    |    ✅     |
| ValidatedID                     |                                                                         |   ✅    |   ✅    |    ✅     |
| Amadeus                         | [Passport Verfifier](https://tid-wallet-dev.azurewebsites.net/passport) |        |   ✅    |    ✅     |
| DVV Wallet                      |                                                                         |        |   ✅    |    ✅     |
| UAegean (Cyclades Fast Ferries) |                                                                         |        |   ✅    |    ✅     |
| E-Group                         |                                                                         |        |   ✅    |    ✅     |
| Sicpa Digital Trust Suite       | [sicpa.com](https://docs.dip.sicpa.com/)                                |        |   ✅    |    ✅     |

## Funding

![image](https://github.com/EWC-consortium/ewc-wiki/assets/455274/1ac9b4e3-06b9-4c3c-a2af-ec5fbf584517)

The project is co-funded by the European Union. However, the views and opinions expressed are those of the author(s) only and do not necessarily reflect those of the European Union or the granting authority. Neither the European Union nor the granting authority can be held responsible.

## Licensing

Licensed under the Apache 2.0 License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. The IP is handled as part of the EWC IP agreement. Please note that some of the RFCs (Like payments) are not based on Apache 2.0 license.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the LICENSE for the specific language governing permissions and limitations under the License.
