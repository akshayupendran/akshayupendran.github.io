# ECU Trust Model

- [ECU Trust Model](#ecu-trust-model)
  - [Resources](#resources)
  - [Requirements](#requirements)
  - [Assumptions](#assumptions)
  - [Technical Details](#technical-details)
    - [Algorithms](#algorithms)
    - [Overall Task List](#overall-task-list)
    - [Certificate / Key List](#certificate--key-list)
      - [Trust Model](#trust-model)
      - [Secure Boot](#secure-boot)

## Resources

- [SSA Integration Information 322.0.3 2023.10.12](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=ERbM45OpaJNJp-xl05BTQwAB7kV-gSvHZ9U5Ft2ODC24GA&e=SvIVfb)
- [MSS10796 Standard Security Specification 3.0.0 2020.05.29](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=ESsU1ZyjLiFGmikXFFVyuRsByn_yLUR41F-cfSpAq9ccSQ&e=PTlaTc)
- [MSS10798 ECU Trust Model Supplier How To 1.1.0 2023.01.25](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=ERlHwpGXGtxLqFZzVLQ66asBHS6rMw2R4SzizB4LwVWFEQ&e=15bNkP)
- [MSS10815 SSA 3.0.3 2021.06.07](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=Eeyj_mYQc4pLgokwzHMKowMBRuEUX0LbkJ7Tie8quGNiYA&e=C6HmPn)
- [MSS10820 Specification for SHA512 and Ed25519ph 1.1.0 2017.07.13](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=Ee0qDMmX73hFkL50VWYser4BsDC4TmbmmlUhS7BVugtBng&e=UztNuL)
- [MSS10824 Specification for Certificate Structure 2.5.0 2022.01.19](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=EaUK36-DCB5JrCG2-k8WvvEB9AxOPJjaG5ldKcWFQS80cg&e=gi6g81)
- [Started kit readme 1.2.8.0 2023.07.26](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=EYLSJWmWbUJAhES9mR1yYfUBUOEKOSo1zg05fEB0NWNmiw&e=2s5Rup)
- [Trust Model Certificate Package 6.0.1](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=ETV27PDHv2NDpW0i8IenzzQB3ZgVRbI8oVNNTk9jMOGZUw&e=MlG8DB)

## Requirements

- Variant of car: [Star 3.5](https://visteon.atlassian.net/browse/MB22330758-4610)
- [SW Requirements](https://jda.visteon.com/single?snap=321491)
- [Sys Requirements](https://jda.visteon.com/single?snap=310771&view=48)

## Assumptions

- No TLS / Trust Model Chain.
- No Security Event Logging.
- Only SecOC use case.

- No Diagnostic Chain
- No Interim Chain

## Technical Details

### Algorithms

| Functionality            | Algorithm Configured | Algorithm Required | CSM_API                             |
| ------------------------ | -------------------- | ------------------ | ----------------------------------- |
| Hash                     | SHA 512              |                    |                                     |
| Mac Generation(SecOC)    | MAC Generate         |                    |                                     |
| Mac Verification(SecOC)  | MAC Verification     |                    |                                     |
| Mac Verification(BcTp)   | MAC Verification     |                    |                                     |
| Signature Generation     | Ed25519ph/ED25519    |                    |                                     |
| Signature Verification   | Ed25519ph/ED25519    |                    |                                     |
| Decrypt                  | AES CBC 128          |                    |                                     |
| Random Number Generation | TRNG                 |                    |                                     |
| Key Exchange Protocol    | Key Set/Get          |                    | Csm_KeyElementSet/Csm_KeyElementGet |

### Overall Task List

| S No | Tasks                                                                                         | Effort estimation | Person Responsible | Date of completion |
| ---- | -----                                                                                         | ----------------- | ------------------ | ------------------ |
| 1    | Migration of 1.2.8.1                                                                          |                   |Shreya              |                    |
| 2    | Key Pair Generation API to be checked in HSM , Test and confirm from HSM                      |                   |Sathish             | 15/07/2024         |
| 3    | Integration of Key Get and Set in CSM                                                         |                   |                    |                    |
| 4    | Writing the Initial values of the certificates                                                |                   |                    |                    |
| 5    | Test the algorithms by independent code from Application for non-tested one's                 |                   |                    |                    |
| 6    | List of Keys/Certificates that has to be written. Need to get from Application/Vector/Systems |                   |                    |                    |
| 7    | Mapping to SSA as per the keys in the task6 above                                             |                   |                    |                    |
| 8    | Confirmation of the certificates to be given by PL's                                          |                   |                    |                    |

### Certificate / Key List

#### Trust Model

| Certificate Name                     | Size(in bytes)    | Generated by ECU | Injected from external                  |
| ------------------------------------ | --------------    | ---------------- | --------------------------------------- |
| Trust Model Root Certificate         | 292(ED25519)(DER) |                  | Yes (MB Delivered in Visteon SW and MB) |
| Trust Model Replacement Key          | 32(ED25519)(bin)  |                  | Yes (MB Delivered in Visteon SW)        |
| Trust Model Backend Certificate      | 325(ED25519)(DER) |                  | Yes (MB Delivered in Visteon SW and MB) |
| Trust Model Intermediate Certificate | 325(ED25519)(DER) |                  | Yes (MB Delivered in Visteon SW and MB) |
| Trust Model ECU Certificate          | 1024              |                  | To be fed at MB                         |

#### Secure Boot

| Key/Security asset                                  | Location       | Memory(Bytes) | Status              |
| ------------------                                  | -------------- | ------------- | ------------------- |
| SecureBoot RoT Key                                  | HSM Work flash | 16            | Generated           |
| MAC for Secure boot RoT                             | HSM Work flash | 16            | Generated           |
| SecureBoot CoT                                      | HSM Work flash | 16            | Generated           |
| MAC for BM                                          | HSM Work flash | 16            | Generated           |
| MAC for BL                                          | HSM Work flash | 16            | Generated           |
| MAC for APP                                         | HSM Work flash | 16            | Generated           |
| ed25519ph - seed/key secure access                  | HSM Work flash | 32            | Provisioned         |
| ed25519ph - PK of VSM (key exchange)                | HSM Work flash | 32            | Injected By MB      |
| ed25519ph ETM private                               | HSM Work flash | 32            | Generated           |
| ed25519ph ETM public                                | HSM Work flash | 32            | Generated           |
| ed25519ph For Signature verification in SW update   | HSM Work flash | 32            | Provisioned         |
| AES 128 (GCM) for image encryption/decryption       | HSM Work flash | 16            | Provisioned         |
| IV for the above (12 bytes nonce + 4 bytes counter) | HSM Work flash | 16            | Provisioned         |
| AES 128 (CBC) Memory encryption                     | HSM Work flash | 16            | Generated           |
| IV for the above                                    | HSM Work flash | 16            | Generated           |
| Diagnostic Root CA                                  | HSM Work flash | 1024          | Injected by Visteon |
| Diagnostic Backend CA                               | HSM Work flash | 1024          | Injected by Visteon |
| SecOC Keys                                          | HSM Work flash | 128           | Generated: Application PDUs - 2 Diag Auth broadcast - 1 Tick count broadcast - 1 Real-Time offset Broadcast - 1 VIN Broadcast - 1 Note: Additional 2 key space is added. |
| SecOC Key Checksum                                  | HSM Work flash | 32            | Generated      |
| Car specific secret hash                            | HSM Work flash | 4             | Generated      |
