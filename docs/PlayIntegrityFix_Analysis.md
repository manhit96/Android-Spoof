> **Important note**: This document focuses on detailed analysis of PlayIntegrityFix and its related ecosystem. For an overview of all Android device information spoofing solutions, please refer to the document "[Analysis of Open Source Projects for Android Device Information Spoofing](android_device_spoofing_comprehensive_analysis.md)". The overview document provides information about various projects, alternative methods, and the latest trends in this field.

# Analysis of PlayIntegrityFix and Related Ecosystem

## Summary of Application for Android-Spoof Project

**Core values for the Android-Spoof project:**
- Provides a mechanism to spoof bootloader information and bypass Play Integrity verification
- Key attestation spoofing technology to achieve MEETS_STRONG_INTEGRITY
- Modular architecture model that can be applied to the project
- Automatic update mechanism for device information from the latest Pixel devices

**Main components to integrate:**
1. Spoofing bootloader information and system properties
2. Spoofing package signatures and key attestation
3. Automatic fingerprint update mechanism
4. Keybox and security patch level management

## Overview

PlayIntegrityFix is a Zygisk module designed to bypass Google Play Integrity and SafetyNet integrity verification mechanisms on rooted Android devices. This module is particularly important for the Android-Spoof project as it addresses one of the main objectives: "Bypass verification mechanisms such as Google Play Integrity, SafetyNet".

## Objectives of PlayIntegrityFix

- Spoof bootloader status (to report as "locked")
- Achieve verification levels:
  - MEETS_BASIC_INTEGRITY ✅
  - MEETS_DEVICE_INTEGRITY ✅
  - MEETS_STRONG_INTEGRITY ❌ (requires additional keybox from TrickyStore)
- In SafetyNet:
  - basicIntegrity: true
  - ctsProfileMatch: true
  - evaluationType: BASIC

## PlayIntegrityFix Ecosystem

PlayIntegrityFix works with other modules to achieve the most comprehensive verification level:

1. **PlayIntegrityFix (PIF)**: Core module that helps achieve MEETS_BASIC_INTEGRITY and MEETS_DEVICE_INTEGRITY
2. **TrickyStore**: Supplementary module that helps achieve MEETS_STRONG_INTEGRITY
3. **Tricky-Addon-Update-Target-List**: TrickyStore configuration management tool
4. **playcurlNEXT**: Supplement to PIF, automatically updates fingerprint
5. **BootloaderSpoofer**: Supplement to PIF, spoofs locked bootloader status

### 1. PlayIntegrityFix (PIF)

PlayIntegrityFix is the core component of the ecosystem, developed by chiteroman. This is a Zygisk module aimed at patching SafetyNet/Play Integrity API results, helping devices pass attestation at BASIC and DEVICE levels.

#### Project Structure

The PlayIntegrityFix project has the following structure:

- **module/**: Contains scripts and configuration files for the Magisk/KernelSU module
  - `action.sh`: Script to automatically update fingerprint
  - `common_func.sh`: Common utility functions
  - `customize.sh`: Module installation script
  - `module.prop`: Module information
  - `pif.json`: Spoofing information configuration file
  - `post-fs-data.sh`: Script that runs after filesystem is mounted
  - `service.sh`: Script that runs when the system boots
  - `sepolicy.rule`: SELinux rules
  - `uninstall.sh`: Module uninstallation script

- **app/src/main/java/es/chiteroman/playintegrityfix/**: Java source code
  - `EntryPoint.java`: Main entry point of the module
  - `CustomKeyStoreSpi.java`: KeyStore spoofing class
  - `CustomPackageInfoCreator.java`: Package information spoofing class
  - `CustomProvider.java`: Provider spoofing class

- **app/src/main/cpp/**: C++ source code
  - `main.cpp`: Zygisk module implementation
  - `zygisk.hpp`: Zygisk API
  - `json.hpp`: JSON processing library

#### Operating Mechanism

PlayIntegrityFix works through the following mechanisms:

1. **Device information spoofing**: The module uses the `pif.json` file to configure spoofed information such as:
```json
{
  "FINGERPRINT": "google/oriole_beta/oriole:Baklava/BP22.250124.009/13034193:user/release-keys",
  "MANUFACTURER": "Google",
  "MODEL": "Pixel 6",
  "SECURITY_PATCH": "2025-02-05",
  "DEVICE_INITIAL_SDK_INT": 21,
  "spoofVendingSdk": 0
}
```

2. **Hook into application initialization process**: Uses the Zygisk framework to intervene in the initialization process of Google applications such as:
   - `com.google.android.gms` (Google Play Services)
   - `com.android.vending` (Google Play Store)
   - `com.google.android.gms.unstable` (DroidGuard)

3. **Spoof signatures and provider**: The module replaces package signatures with valid Google signatures and hooks into the AndroidKeyStore provider to pass verification.

4. **Modify system properties**: Through `post-fs-data.sh` and `service.sh`, the module changes many system properties:
```
# Bootloader and verification status
ro.boot.flash.locked 1
ro.boot.verifiedbootstate green
ro.boot.veritymode enforcing
ro.boot.vbmeta.device_state locked
vendor.boot.verifiedbootstate green
vendor.boot.vbmeta.device_state locked

# Security-related properties
ro.boot.warranty_bit 0
ro.vendor.boot.warranty_bit 0
ro.vendor.warranty_bit 0
ro.warranty_bit 0
ro.debuggable 0
ro.secure 1
ro.adb.secure 1
sys.oem_unlock_allowed 0
```

5. **Hook into system_property_read_callback**: The module hooks into the system property reading function to adjust the returned values.

### 2. TrickyStore

TrickyStore is a Zygisk module supplementary to PlayIntegrityFix, focusing on modifying the certificate chain generated for Android key attestation. This is an advanced method to achieve the MEETS_STRONG_INTEGRITY level.

#### How TrickyStore Works

1. **Modifying Certificate Chain**: TrickyStore modifies the certificate chain generated by Android KeyStore to pass strong verification.

2. **Using keybox.xml**: The module requires a valid keybox.xml file (not revoked) to be placed at `/data/adb/tricky_store/keybox.xml`.

3. **Custom target applications**: Users can specify which applications to apply the trick to via the `/data/adb/tricky_store/target.txt` file.

4. **Two operating methods**:
   - **Leaf Hack Mode**: Hack the root certificate (default)
   - **Certificate Generating Mode**: Create new certificates (for devices with broken TEE)

5. **Custom security patch level**: The security patch level can be customized via the `/data/adb/tricky_store/security_patch.txt` file.

#### keybox.xml Format

```xml
<?xml version="1.0"?>
<AndroidAttestation>
    <NumberOfKeyboxes>1</NumberOfKeyboxes>
    <Keybox DeviceID="...">
        <Key algorithm="ecdsa|rsa">
            <PrivateKey format="pem">
-----BEGIN EC PRIVATE KEY-----
...
-----END EC PRIVATE KEY-----
            </PrivateKey>
            <CertificateChain>
                <NumberOfCertificates>...</NumberOfCertificates>
                    <Certificate format="pem">
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
                    </Certificate>
                ... more certificates
            </CertificateChain>
        </Key>...
    </Keybox>
</AndroidAttestation>
```

### 3. Tricky-Addon-Update-Target-List

This is a helper module to configure TrickyStore through KernelSU's Web UI.

#### Features

- Configure target.txt with display names of applications
- Select `!` or `?` mode for each application
- Select applications from Magisk DenyList
- Deselect unnecessary applications
- Set verifiedBootHash
- Automatically configure security patch
- Provide AOSP Keybox or input custom Keybox

### 4. playcurlNEXT

playcurlNEXT is a supplementary module for PlayIntegrityFix, developed by daboynb. This module automatically updates the fingerprint from the latest Pixel device to ensure PlayIntegrityFix always has valid fingerprint information.

#### Project Structure

The playcurlNEXT project has a simpler structure:

- `service.sh`: Main script, runs when the system boots
- `minutes.txt`: Configuration file for update interval
- `module.prop`: Module information
- `customize.sh`: Module installation script
- `update.json`: Module update information

#### Operating Mechanism

playcurlNEXT works as follows:

1. **Check environment**: The script checks if PlayIntegrityFix is installed and finds the path to busybox.

2. **Copy and set up cron script**: The script copies the PlayIntegrityFix directory to a temporary directory and prepares the `action.sh` script to run periodically.

3. **Read time configuration**: The script reads the value from the `minutes.txt` file to determine the update interval (default is 60 minutes).

4. **Set up cron job**: The script sets up a cron job to run `action.sh` according to the configured interval.

5. **Initialize and run script**: The script runs `action.sh` once at startup and then runs periodically according to the cron job.

### 5. BootloaderSpoofer

BootloaderSpoofer is an Xposed module developed by chiteroman, aimed at spoofing locked bootloader status in attestation certificates. This module operates at the application level, intervening in the local attestation process when an application requests a certificate from KeyStore/TEE.

#### Project Structure

The BootloaderSpoofer project mainly includes one main Java file:

- `app/src/main/java/es/chiteroman/bootloaderspoofer/Xposed.java`: Main source code of the module

#### Operating Mechanism

BootloaderSpoofer works through the following mechanisms:

1. **Initialize KeyPair and Certificate**: The module initializes EC and RSA key pairs, along with the corresponding certificates.

2. **Hook into attestation process**: The module hooks into attestation-related methods in KeyStore to intervene in the certificate creation process.

3. **Modify Extensions in certificate**: The module modifies extensions in the certificate to remove signs of unlocked bootloader.

4. **Create new certificate or hack existing certificate**: The module can create a completely new certificate or modify an existing certificate to trick the application into thinking that the bootloader is locked.

5. **Apply to specific applications**: The module allows selecting specific applications to hook, avoiding affecting the entire system.

## Achieving MEETS_STRONG_INTEGRITY

To achieve the MEETS_STRONG_INTEGRITY level, the following steps must be followed:

1. **Install both PlayIntegrityFix and TrickyStore**:
   - PlayIntegrityFix solves the basic problem of bootloader and device integrity
   - TrickyStore provides a key attestation spoofing mechanism to achieve STRONG_INTEGRITY

2. **Use a valid keybox**:
   - Find a keybox that hasn't been revoked (this is the hardest part)
   - Place the keybox at `/data/adb/tricky_store/keybox.xml`

3. **Configure TrickyStore**:
   - Use Tricky-Addon-Update-Target-List for easier configuration
   - Add applications that need verification to target.txt
   - Configure security patch to match

4. **Note on validity period**:
   - Keyboxes are often quickly detected and revoked by Google
   - When the keybox is revoked, you'll only achieve MEETS_DEVICE_INTEGRITY
   - Regularly update or look for new keyboxes

## Integration between components

The components in the PlayIntegrityFix ecosystem work together as follows:

1. **PlayIntegrityFix (PIF)**: Core component, helps achieve MEETS_BASIC_INTEGRITY and MEETS_DEVICE_INTEGRITY by spoofing device information and hooking into the attestation process.

2. **playcurlNEXT**: Supplement to PIF, automatically updates fingerprint to ensure PIF always has the latest valid fingerprint information.

3. **BootloaderSpoofer**: Supplement to PIF, spoofs locked bootloader status in attestation certificates at the application level.

4. **TrickyStore**: Supplement to PIF, intervenes in the security key certificate chain to achieve MEETS_STRONG_INTEGRITY.

5. **Tricky-Addon-Update-Target-List**: Management tool to configure TrickyStore more easily.

## Application for Android-Spoof Project

### Identifying APIs to Modify

- `android.os.Build`: To change device information (MANUFACTURER, MODEL, etc.)
- `android.security.keystore`: To spoof KeyStore and certificates
- `android.content.pm.PackageManager`: To change package signature information

### Techniques to Integrate into LineageOS

1. **Device information spoofing**: Integrate device information spoofing technique from PlayIntegrityFix into Android framework.

2. **Automatic fingerprint update**: Integrate automatic fingerprint update mechanism from playcurlNEXT into the system.

3. **Bootloader status spoofing**: Integrate bootloader status spoofing technique from BootloaderSpoofer into Android framework.

4. **Security key certificate chain spoofing**: Integrate certificate chain spoofing technique from TrickyStore into Android framework.

### Proposed Integration Architecture

1. **Instead of using Zygisk hooks**:
   - Directly modify classes in the Android framework
   - Add new system services to manage spoofed information
   - Create a mechanism for storing and managing consistent spoofed information

2. **Functional separation**:
   - Each component has a specific task, making it easy to maintain and update
   - Design specialized services for each type of information to be spoofed

3. **Automation**:
   - Integrate mechanism to automatically update new device information
   - Reduce user intervention

4. **Flexibility in configuration**:
   - Provide flexible configuration options
   - Allow users to adjust according to needs

### Challenges to Resolve

- Ensure consistency between APIs
- Bypass root detection mechanisms
- Maintain updates according to changes in Google Play Integrity
- Efficiently manage and update keyboxes
- Ensure all components work consistently
- Build continuous update mechanism

## Conclusion

The PlayIntegrityFix ecosystem provides a comprehensive solution to bypass Google Play Integrity and SafetyNet integrity verification mechanisms. By integrating techniques from PlayIntegrityFix, TrickyStore, Tricky-Addon-Update-Target-List, playcurlNEXT, and BootloaderSpoofer into the Android-Spoof project, we can build a comprehensive, powerful, and more sustainable device spoofing solution capable of passing the highest verification level of Google Play Integrity.

Through deep analysis of these components, we can create a solution integrated directly into LineageOS, ensuring performance, stability, and higher detection resistance compared to current hook-based methods. 