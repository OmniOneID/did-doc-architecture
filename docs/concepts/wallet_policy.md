# Wallet Policy

- Subject: Wallet Policy
- Author: OpenSource Development Team
- Date: 2026-11-16
- Version: v3.0.0

| Version | Date       | Changes         |
| ------- | ---------- | --------------- |
| v3.0.0  | 2026-11-16 | Unify TA naming |
| v1.0.0  | 2024-10-18 | Initial version |

<br>

## Wallets Available for Use Upon Reinstallation

When reinstalling the app, the types of wallets available to the user and the corresponding policies are as follows.

### File Wallet
- **Use a New Wallet**: If the wallet to be used is a File Wallet when reinstalling the app, a new wallet must be used.
    - **Prevent Data Loss**: Existing wallet data may be lost during the reinstallation process, so it is essential to create a new wallet.
- **Device Restrictions**: The File Wallet does not support a multi-device policy for a single user.
    - **Device Limitation**: One File Wallet can only be used on a single device.

### App Wallet
- **Use on the Same Device**: If the wallet to be used is an App Wallet when reinstalling the app, the previous wallet can only be used on the same device.
    - **Device Verification**: TA must know the device information of the App.
- **Device Restrictions**: App Wallet does not support a multi-device policy for a single user.
    - **Device Limitation**: One App Wallet can only be used on a single device.

### Cloud Wallet
- **Previous Wallet Usable**: If the wallet to be used is a Cloud Wallet when reinstalling the app, the previous wallet can be used regardless of device policy.
    - **Ensure Continuity**: Through the Cloud Wallet, users can use the same wallet on multiple devices.
    - **Enhanced Convenience**: Users can access the previous wallet even after changing devices or reinstalling the app, increasing user convenience.
    - **Data Preservation**: Cloud Wallet stores data in the cloud, ensuring safe data preservation even during reinstallation or device changes.

## Summary
- **File Wallet**: Use a new wallet upon app reinstallation, one-person, one-device policy.
- **App Wallet**: Previous wallet can only be used on the same device, one-person, one-device policy.
- **Cloud Wallet**: Previous wallet can be used regardless of device policy, supports a one-person, multiple-device policy.

These policies enable users to select the appropriate usage method for each wallet type, facilitating wallet management that considers both security and convenience.