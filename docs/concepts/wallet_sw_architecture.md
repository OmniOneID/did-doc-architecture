<!-- Individual documents may be merged in the future, so the table of contents is not used. -->

# Wallet SW Architecture

- Subject: Wallet SW Architecture
- Author: OpenSource Development Team
- Date: 2025-06-23
- Version: v2.0.0

| Version | Date       | Changes                                     |
|--------|------------|---------------------------------------------|
| v2.0.0 | 2025-06-23 | Added ZKP and Reflected the SDK integration |
| v1.0.0 | 2024-10-18 | Initial version                             |


<br>

CA (Certified App) can implement functions for each protocol by referencing Wallet, Utility, Communication, and DataModel components to provide reliable wallet functions, such as user registration, VC issuance(including ZKP), VP submission and ZKP submission.

![wallet_sw_archietecture](./images/wallet_sw_architecture.svg)


### 1. Components Description
The above SDK structure is a relationship structure diagram between the OpenDID client's authorization app and wallet software, and the representative classes for each components are explained below.

- Wallet
  - OpenDID Wallet component, and provides functions for creating, storing, and managing the WalletToken, Lock/Unlock, Key, DID Document (DID Document), Verifiable Credential (hereinafter referred to as VC) and Zero-knowledge Proof(hereinafter referred to as ZKP) information required for Open DID.
- Core
  - OpenDID Core component, providing functions to generate, store, and manage the keys, DID Document, Verifiable Credential (VC) and Zero-knowledge Proof(ZKP) information required for Open DID.
- Communication
  - OpenDID Communication component, which provides a communication interface to manage HTTP requests and responses, supporting GET and POST methods with JSON payloads.
- Utility
  - Using the Crypto Utility component, providing various encryption, 
- DataModel
  - Using the DataModel component, defining the data model used in the app and wallet SDK.

| Component Group | Classes        | Features                                                                       |
|----------------|----------------|--------------------------------------------------------------------------------|
| Wallet         | WalletService  | - Processing business logic for user registration, VC issuance(including ZKP), VP submission and ZKP submission<br>- Generating Proof using Signature Value |
|                | WalletCore     | - Generating key, DID/VC, VP and ZKP <br>- sign                                        |
|                | LockManager    | - Change Wallet Type and Status                                                |
|                | WalletToken    | - Create and verify walletToken                                                |
|                | DBManager      | - ser information, WalletToken, CA list management                             |
| Core           | KeyManager     | - User key generation, authentication, and signing                             |
|                | DIDManager     | - DIDDocument creation management                                              |
|                | VCManager      | - VC Management and VP Creation                                                |
|                | ZKPManager     | - ZKP Creation Management                                                      |
| Communication  | NetworkClient  | - CA and Wallet common communication module                                    |
| Utility        | DigestUtils    | - hash common utility                                                          |
|                | MultibaseUtils | - En/Decode Common Utility                                                     |
|                | CryptoUtils    | - Generate random values ​​and key pairs, encryption/decryption common utility |
| DataModel      | VO Objects     | - Data formats and serialization/deserialization                               |





## 1.1 User registration Description
For detailed explanation, refer to the user registration protocol. [user-registration](./User%20Registration.md)
![wallet_sw_archietecture](./images/wallet_sw_architecture_reg_user.svg)
## 1.2 VC Issuance Description
For detailed explanation, refer to the VC issuance protocol. [VC-Issuance](./VC%20Issuance.md)
![wallet_sw_archietecture](./images/wallet_sw_architecture_issue_vc.svg)
## 1.3 VP Submission Description
lease refer to the VP Submission Protocol for detailed instructions. [Presentation-VP](./Presentation%20of%20VP.md)
![wallet_sw_archietecture](./images/wallet_sw_architecture_submit_vp.svg)
## 1.4 ZKP Submission Description
lease refer to the ZKP Submission Protocol for detailed instructions. [Presentation-ZKP](./Presentation%20of%20ZKP%20Proof.md)
![wallet_sw_archietecture](./images/wallet_sw_architecture_submit_zkp.svg)
<br>
