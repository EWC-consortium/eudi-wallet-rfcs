# eudi-wallet-rfcs

<h1 align="center">
    EWC - EUDI Wallet RFCs 
</h1>

<p align="center">
    <a href="/../../commits/" title="Last Commit"><img src="https://img.shields.io/github/last-commit/EWC-consortium/eudi-wallet-rfcs?style=flat"></a>
    <a href="/../../issues" title="Open Issues"><img src="https://img.shields.io/github/issues/EWC-consortium/eudi-wallet-rfcs?style=flat"></a>
    <a href="./LICENSE" title="License"><img src="https://img.shields.io/badge/License-Apache%202.0-yellowgreen?style=flat"></a>
</p>

<p align="center">
  <a href="#about">About</a> •
  <a href="#contributing">Contributing</a> •
  <a href="#rfcs">RFCs</a> •
  <a href="#implementors">Implementors</a> •
  <a href="#licensing">Licensing</a>
</p>

## About

This repository is created to align the implementation of digital wallets across the EWC LSP consortium wallet providers. This also ensures all wallets providers can self-test and sign off against the EWC Interoperability Test Bed (ITB). 

## Contributing

Feel free to improve the plugin and send us a pull request. If you find any problems, please create an issue in this repo. Wallet providers can raise a PR once they have implemented and aligned with the RFCs. 

## RFCs

These are the RFCs identified to be specified towards ensuring producing reference code as well towards the EWC ITB. The supported verifiable credentials and presentations are specified in the [EWC supported format csv file](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-supported-formats.csv).

| **RFC #** | **RFC Title**                                                                                                                                                           |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RFC-001   | [Issue Verifiable Credentials Workflow](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc001-issue-verifiable-credential.md)                         |
| RFC-002   | [Present Verifiable Credentials Workflow](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc002-present-verifiable-credentials.md)                    |
| RFC-003   | Attest Individual Wallet Instance (TBD) - ARF/OID4 Security                                                                                                             |
| RFC-004   | Attest Enterprise Wallet Instance (TBD) - WP3/3.1.2 Requirements/OID4 Security                                                                                          |
| RFC-100   | [EWC Interoperability Profile Towards ITB - v1.0](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-rfc100-interoperability-profile-towards-itb-v1.0.md) |

## Implementers
| Wallet                          | Link                                                                                                                                         | ITB compliance | Holder | Issuer | Verifier |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | :----: | :----: | :------: |
| Amadeus                         | [Passport Verifier](https://tid-wallet-dev.azurewebsites.net/passport)                                                                       | Mar'24         |        |        |    ✅     |
| DVV Wallet                      |                                                                                                                                              | Mar'24         |   ✅    |   ✅    |    ✅     |
| iGrant.io Enterprise Wallet     | [demo-business.igrant.io/](https://demo-business.igrant.io/)                                                                                 | Mar'24         |   ✅    |   ✅    |    ✅     |
| iGrant.io Data Wallet           | [iOS](https://apple.co/2Mz9nJp), [Android](https://play.google.com/store/apps/details?id=io.igrant.mobileagent)                              | Mar'24         |   ✅    |   ✅    |    ✅     |
| ValidatedID                     | [iOS](https://apps.apple.com/us/app/vidwallet/id1554340592), [Android](https://play.google.com/store/apps/details?id=com.validatedid.wallet) | Mar'24         |   ✅    |   ✅    |    ✅     |
| UAegean (Cyclades Fast Ferries) |                                                                                                                                              | Mar'24         |        |   ✅    |    ✅     |
| Lissi ID-Wallet                 | [iOS](https://testflight.apple.com/join/9AWbZISv), [Android](https://play.google.com/store/apps/details?id=io.lissi.mobile.android.beta)     | TBD            |   ✅    |        |          |
| Lissi ID-Wallet Connector       | [lissi.id](https://lissi.id)                                                                                                                 | TBD            |        |   ✅    |    ✅     |


## Funding

![image](https://github.com/EWC-consortium/ewc-wiki/assets/455274/1ac9b4e3-06b9-4c3c-a2af-ec5fbf584517)

The project is co-funded by the European Union. Views and opinions expressed are, however, those of the author(s) only and do not necessarily reflect those of the European Union or the granting authority. Neither the European Union nor the granting authority can be held responsible for them.

## Licensing

Licensed under the Apache 2.0 License, Version 2.0 (the "License"); you may not use this file except in compliance with the License.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the LICENSE for the specific language governing permissions and limitations under the License.
