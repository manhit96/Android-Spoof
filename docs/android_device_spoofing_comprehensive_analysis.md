================================================================================
ANALYSIS OF OPEN SOURCE PROJECTS FOR ANDROID DEVICE INFORMATION SPOOFING
================================================================================

================================================================================
TABLE OF CONTENTS
================================================================================

1. INTRODUCTION

2. ANALYSIS OF NOTABLE PROJECTS
   2.1 XPL-EX – Device information hook framework (Xposed)
   2.2 PlayIntegrityFix (PIF) – Bypass Google Play Integrity & SafetyNet (Zygisk module)
   2.3 TrickyStore – Key attestation certificate chain emulation (Magisk/KernelSU module)
   2.4 Tricky Addon – Update Target List
   2.5 BootloaderSpoofer

3. MAGISK/KERNELSU MODULES RELATED TO DEVICE INFORMATION SPOOFING

4. DIRECT MODIFICATION OF ANDROID SOURCE CODE (WITHOUT HOOKS)

5. KEY ATTESTATION SPOOFING & HARDWARE-BACKED KEYSTORE INFORMATION

6. SELINUX BYPASS AND DROIDGUARD EVASION

7. SPOOFING INTEGRATION INTO CUSTOM ROMS

8. ROM REVERSE ENGINEERING METHODS TO CHANGE SYSTEM INFORMATION

9. TOOLS FOR DETECTING AND ANALYZING GOOGLE'S DEVICE CHECKING MECHANISMS (2023-2024)

DETAILED REFERENCES

CONCLUSION

1. INTRODUCTION
--------------------------------------------------------------------------------
Android device integrity checking applications (SafetyNet, Play Integrity API) 
are continuously being improved by Google to detect rooted or modified devices.
The open source community has developed numerous spoofing projects aimed at "faking" 
device information, hiding root/hook traces, and bypassing these checks. This document 
analyzes notable projects (XPL-EX, PlayIntegrityFix, TrickyStore, Tricky-Addon-Update-Target-List, etc.) 
and the latest solutions (2023-2024) from sources such as XDA-Developers, GitHub, as well as 
ROM modification methods and reverse engineering approaches to change system information.

================================================================================
2. ANALYSIS OF NOTABLE PROJECTS

2.1 XPL-EX – Device information hook framework (Xposed)
--------------------------------------------------------------------------------
- Project URL: GitHub – 0bbedCode/XPL-EX

- Description: XPL-EX is an open-source Xposed hook framework focused on privacy. This application blocks other apps from accessing real device information, providing fake data instead (e.g., reporting 900GB RAM instead of 8GB, spoofing the model as iPhone or Pixel). XPL-EX supports Android 6.0+ and allows users to write Lua scripts to customize hooks to hide most identifying information on the device.

- Advantages: 
  • Completely free, no ads, no data sent externally.
  • Very rich collection of hooks (more than competing solutions like original XPrivacyLua), covering many types of information (device IDs, serial numbers, sensors, location, etc.).
  • Supports flexible simulation through Lua scripts, suitable for both security purposes and app customization/hacking.
  • Actively updated (forked from XPrivacyLua by M66B but has changed >50k lines of code) and compatible with modern LSPosed/Xposed.
  • Provides superior number of hooks compared to other tools.
  • Free, does not collect user data.
  • Supports diverse information from software (does not interfere with native code).

- Disadvantages: 
  • Requires a rooted device with Xposed/LSPosed installed, thus risking detection by some apps (needs to be combined with Xposed hiding tools like Hide My Applist).
  • Configuring many complex hooks can be difficult for beginners.
  • Only hooks at the Java level (does not support native hooks) – although most device checks are through Java APIs, some specific cases using native APIs might slip through.
  • Only works at the Java level, does not support native hooks.
  • Requires Xposed environment (LSPosed, etc.) – needs root.

- Update status: The project is being actively maintained. As of 03/2024, XPL-EX is still updated (very active dev – e.g., version 03/26/2024). Has a supporting Telegram community.

- Special features: 
  • Dynamic hooking via Lua – users can write scripts to intervene in app behavior.
  • Allows spoofing of almost all device information (from hardware specifications to manufacturer identity, operating system version).
  • This is a continuation of XPrivacyLua development, significantly expanding capabilities (over 40k additional lines of code).
  • Expected to support hooking at the native level in the future to become an "all-in-one" solution for device protection and comprehensive root/Xposed hiding.
  • Allows customization of spoofed information through scripts, simulating the device as other brands (e.g., iPhone, Pixel).

2.2 PlayIntegrityFix (PIF) – Bypass Google Play Integrity & SafetyNet (Zygisk module)
--------------------------------------------------------------------------------
- Project URL: GitHub – chiteroman/PlayIntegrityFix (Magisk/KernelSU module)

- Description: PlayIntegrityFix is a Zygisk module designed to patch SafetyNet/Play Integrity API results, helping devices pass attestation at BASIC and DEVICE levels (ctsProfileMatch=true, MEETS_DEVICE_INTEGRITY=true). The module intervenes in Google Play Services to spoof necessary values (such as fingerprint, bootloader status...) but does not hide root – users need to combine it with other root hiding tools (Shamiko, Zygisk-Assistant) if needed. PIF supports both Magisk and KernelSU (or APatch) as long as there is a Zygisk environment.

- Advantages: 
  • New solution, frequently updated to keep pace with Play Integrity changes (emerged in late 2023 and still updated today).
  • Easy to use: just flash the module, reboot, and check results with YASNAC or Integrity Checker app.
  • Does not require manual build.prop modifications as before.
  • Works well with KernelSU (with ZygiskNext) so it can be used on devices rooted with KernelSU.
  • After installation, most devices can achieve "MEETS_DEVICE_INTEGRITY ✅" without actually locking the bootloader (sufficient for Google Wallet, banking apps to work).
  • Continuous updates (many releases, latest v18.7).
  • Supports Android 8 – 14, compatible with Magisk and KernelSU.
  • Simple installation, only affects GMS without impacting the system.

- Disadvantages: 
  • Cannot achieve Strong Integrity: PIF by default cannot pass MEETS_STRONG_INTEGRITY (❌) as it does not modify hardware certificates.
  • To achieve strongIntegrity, TrickyStore and keybox need to be combined (complex, see below).
  • Additionally, PIF does not hide root or Xposed traces – apps with their own detection mechanisms outside Play Integrity can still identify modified devices (PIF notes it will not support cases where apps detect root themselves).
  • The module also requires ROMs signed with private keys (not test-keys) to work – meaning some custom ROMs using test-keys may need patching.
  • Google's frequent API changes require PIF to update continuously; if Google updates unexpectedly, PIF may temporarily become ineffective until a new patch is released.
  • Does not help achieve "Strong Integrity" (requires valid key attestation).
  • Does not hide root signs; needs to be combined with other root hiding tools.

- Update status: Being very actively maintained. PIF has frequent updates (XDA discussions show the dev updates promptly when Google makes changes) and has a Telegram community of over 20k members. The latest version (late 2024) adds options to bypass new Play Integrity mechanisms (e.g., spoofing Android SDK to bypass bootloader locked requirements on Android 13+). Very active (large community, updates until 2025).

- Special features: 
  • PIF focuses specifically on "certifying" the device (device certified) without affecting the system.
  • The module does not interfere globally with all apps but targets only SafetyNet/Integrity services in Google Play Services, thus avoiding causing errors in other parts of the system.
  • Additionally, PIF allows advanced configuration via the pif.json file (e.g., spoofVendingSdk to fake Android 12 for Play Store if needed to bypass bootloader locked requirements on Android 13+).
  • This is an open-source solution continuing the ideas of kdrag0n (SafetyNet Fix) and Displax, with many improvements suitable for the new Play Integrity context.
  • PIF also provides clear guidance on verdicts (Basic/Device/Strong) and how to check status via third-party apps.
  • Integrates scripts for automatic fingerprint updates, flexible JSON configuration to spoof information only for Google Play Services.

PlayIntegrityFix – related forks and complementary tools:
- PlayIntegrityNEXT & playcurl: Initially there was a PlayIntegrityFix NEXT fork aimed at automating fingerprint updates. Currently this fork has been replaced by the "playcurl" module – installed alongside official PIF to automatically download valid fingerprints from the web when old fingerprints are invalidated by Google. This approach helps users avoid having to manually change fingerprints or wait for new PIF versions. The playcurl project (formerly PIF NEXT) is also open source on GitHub and has ~1.2k stars, showing community interest. Pro: Automatically updates device profiles, convenient for end users; Con: Depends on external servers for fingerprint data (potential risk if the server stops). Currently playcurl is recommended to be installed with official PIF rather than using a separate PIF fork.

- Universal SafetyNet Fix (USNF): This is a Magisk module developed by kdrag0n, the predecessor of PIF, aimed at bypassing the old SafetyNet and part of Play Integrity. USNF intervenes to block hardware attestation and update CTS profile logic. Requires the device to pass basic CTS already (must have the right fingerprint). Pro: Very useful on Android 8-12, forces Google to use Basic Attestation instead of hardware. This module is also pre-integrated in some ROMs (like ProtonAOSP). Con: No longer updated for Android 14+ (supports up to Android 13); cannot handle new changes in Play Integrity (which is why the community switched to PIF). Currently USNF mainly has historical value or for old devices, and kdrag0n has stopped maintaining it (ProtonAOSP also stopped development).

- MagiskHide Props Config: Magisk module (shell script) that is no longer developed but was once popular for editing build.prop (ro.product.) to impersonate other devices to pass CTS. Pro: Simple, allows selecting Google-certified model fingerprints (e.g., Pixel 5) to fix CTS profile mismatch issues. Con: Does not handle attestation hardware – only effective for old Basic SafetyNet. Project discontinued because it couldn't keep up with new Play Integrity. Currently users can modify props manually or use newer modules (such as Sensitive Props by Pixel-Props team to auto reset sensitive props at boot).

2.3 TrickyStore – Key attestation certificate chain emulation (Magisk/KernelSU module)
--------------------------------------------------------------------------------
- Project URL: GitHub – 5ec1cff/TrickyStore (Magisk/KernelSU module)

- Description: TrickyStore is a module that manipulates the security key attestation certificate chain. Specifically, it modifies the certificate chain generated when Android performs hardware attestation, allowing spoofing of hardware key certificates from other devices. The purpose is to fool the SafetyNet/Play Integrity system at a deeper level – e.g., providing a certificate chain corresponding to locked bootloader and unrevoked devices. Supports Android 10 and above. After installing the module, users can add a keybox.xml file containing hardware keys (unrevoked key) to the module directory to achieve Device Integrity or even Strong Integrity depending on the key. Additionally, TrickyStore allows configuring target.txt to select which applications will have spoofing applied (avoiding system-wide effects).

- Advantages: 
  • Deep bypass level: unlike PIF or Xposed that only intervene at the software level, TrickyStore intervenes in the attest results from TEE/HSM.
  • When combined with a valid keybox (e.g., obtained from an unlocked device), TrickyStore can help achieve MEETS_STRONG_INTEGRITY = true, which other solutions struggle to achieve.
  • The module also intelligently recognizes broken TEE devices or those that cannot extract certs – it will switch to "generate certificate" mode instead (generate mode) to still provide an attest chain (though only achieving deviceIntegrity).
  • Can be flexibly applied per app (only banking apps, GMS, etc.), avoiding interfering with other apps.
  • Supports customizing security patch level reported in attest (spoofing newer patch level if needed).
  • Allows achieving "Strong Integrity" if accurate keybox is available.
  • Supports spoofing at the system level, harder to detect.
  • Custom configuration for each application package.

- Disadvantages: 
  • Difficult to fully implement – to achieve strongIntegrity requires a valid keybox, and leaked hardware keys are rare and quickly revoked by Google.
  • In other words, few users have good keyboxes to fully leverage TrickyStore.
  • In practice, most only achieve deviceIntegrity level with public AOSP keybox (Google provides AOSP keys for devices without TEE).
  • Additionally, TrickyStore from version 1.1.0 has become closed-source due to the dev's dissatisfaction with misuse and lack of contributions – the final 1.0 open-source version is still on GitHub but newer versions are distributed as binaries.
  • This may raise concerns about transparency in later versions.
  • Finally, TrickyStore installation/setup is more complex (must understand keybox concepts, certificate chain).
  • Cannot hide root, so still needs to be combined with Shamiko/denylist if apps check for root beyond attestation.
  • Requires valid keybox (usually difficult for ordinary users to obtain).
  • Complex installation, high risk (can cause bootloop if configured incorrectly).
  • Open source development stopped at v1.0.x, then switched to closed development.

- Update status: The project is still maintained (though closed source). Dev 5ec1cff is very active – TrickyStore has thousands of stars (2.3k+) and continuous new versions. Although closed source, the community still follows updates through changelogs. The module is expected to need adaptation to Play Integrity changes (Google has announced increased requirements for deviceIntegrity with bootloader on Android 13+ in 2025, and tools like TrickyStore will be key to bypassing this). Open source version stopped in 2022; supporting addon continues development.

- Special features: 
  • TrickyStore performs "leaf certificate hack" – that is, modifying extension fields of certificates issued by TEE (e.g., removing the Unlocked flag in the certificate) or creating entirely new certificates if needed.
  • It also has "! / ? mode" options for each app in target.txt to force using generate certificate mode or leaf hack depending on the situation.
  • The unique point is the ability to embed arbitrary hardware keyboxes: if users have keybox files from OEM (leaked), the module will inject them into the attest process as if they were the device's keys.
  • This is a creative approach to fool even the hardware attestation system – which is considered "inviolable" without exploits.
  • TrickyStore, combined with BootloaderSpoofer (below), forms a powerful duo to bypass DroidGuard/TEE at both system and application levels.
  • Unique in spoofing hardware-backed attestation through keybox, helping spoof high-level attestation certificates.

2.4 Tricky Addon – Update Target List
--------------------------------------------------------------------------------
- Project URL: GitHub – KOWX712/Tricky-Addon-Update-Target-List

- Description: This is a WebUI interface auxiliary module for TrickyStore, helping users easily configure the target.txt app list and some spoofing settings. Instead of manually editing files, the module provides a visual interface (in browser) to select apps that need spoofing and set special modes. Works on KernelSU (via KSU WebUI) and Magisk (via WebUI open button in Magisk app). Addon for TrickyStore provides a WebUI interface to configure the list of applications to be spoofed, supporting additional options to force or default for each app.

- Advantages: 
  • User friendly: lists apps by name (easy to select), allows long press to select ! or ? mode for each app (switching between leaf hack and generate mode).
  • Can pull lists from existing Magisk DenyList (convenient if previously configured denylist for sensitive apps).
  • Features to filter unnecessary apps to avoid adding extras.
  • Supports quick installation of AOSP keybox with just one button (for those without their own keybox), and allows uploading custom keyboxes from device storage.
  • Automatically configures appropriate verifiedBootHash and security patch values in attest (can be customized if needed).
  • Generally, this addon helps automate many complex configuration steps of TrickyStore.
  • Intuitive web interface, simplifies configuration.
  • Integrates with KernelSU WebUI and Magisk DenyList.
  • Supports keybox import and whitelist options for Shamiko.

- Disadvantages: 
  • The addon itself does not extend new spoofing capabilities, it's just a UI support tool.
  • If users don't install TrickyStore, this module is meaningless.
  • Also does not "guarantee valid keybox" – only supports input, not magically creating new keyboxes.
  • Additionally, requires KernelSU (for web server) or Magisk + WebUI plugin, meaning a slightly more complex root environment than normal.
  • Is an additional addon, not officially supported by the TrickyStore project.
  • Installation requires root access, opening WebUI port may pose potential risks.
  • Complex with many options, not suitable for non-specialized users.

- Update status: Very well maintained. The addon has ~300 stars, 30 releases, latest version v3.6 (March 2025). Dev KOWX712 also contributes code to PIF (action.sh), showing the project is reliable and closely follows updates from TrickyStore/PIF. Regular updates (until 2024).

- Special features: 
  • Tricky-Addon is almost an all-in-one control panel for attestation spoofing.
  • It integrates Shamiko whitelist configuration (planned, although currently disabled), and has ideas to add periodic features to add all apps to target (not enabled) – this suggests future automatic protection for all apps if users want.
  • The good point is the addon supports both Magisk and KernelSU, leveraging KSU WebUI template – thus on ROMs without Magisk there's still a convenient configuration method.
  • It also contributes to bringing attestation spoofing into end-user UI, paving the way for deeper integration into custom ROMs (one can imagine future ROMs with built-in interfaces to manage fake attestation keys, similar to what this addon does).
  • Allows visual configuration through WebUI and automatically processes spoof lists based on Magisk DenyList.

2.5 BootloaderSpoofer
--------------------------------------------------------------------------------
- Project URL: GitHub – chiteroman/BootloaderSpoofer (LSPosed module)

- Description: BootloaderSpoofer is a lightweight Xposed/LSPosed module aimed at spoofing locked bootloader status in attestation certificates. It intervenes in the local attestation process (when an application on the device requests a certificate from KeyStore/TEE) and modifies the extension field of the certificate to remove signs of unlocked bootloader. The main target is apps that check bootloader locks locally (offline) instead of relying on Google's server results. If an app sends certificates to Google's server for verification, BootloaderSpoofer won't help (the dev plainly states "if the app sends certificates to secure servers, then you're out of luck"). The module allows selecting specific applications to hook (recommends only choosing apps that check bootloader, not hooking Google Play Services or frameworks to avoid causing SafetyNet/Integrity failures).

- Advantages: 
  • This is a very specialized solution but useful for difficult cases: some banking apps, e-wallets or games may directly check SafetyNet certificates to see if the bootloader is unlocked, rather than using Google's API.
  • BootloaderSpoofer will trick these apps into thinking the bootloader is locked (by adjusting bits in the certificate).
  • Light and simple: install as an LSPosed module and only configure target apps, does not require system changes or complex keyboxes.
  • Compatible with TrickyStore/PIF – we can use BootloaderSpoofer for a few specific apps without affecting PIF's global mechanism.
  • Open source project by the same author as PIF, usually updated in parallel (new version v3.8 released 01/2024).

- Disadvantages: 
  • Limited scope: only effective with local TEE attestation. If the app completely depends on Google's server (most Play Integrity cases today), this module doesn't help.
  • Moreover, as an Xposed module it requires an LSPosed environment (root, Zygisk...), and some apps can detect LSPosed.
  • However, since BootloaderSpoofer only hooks into target applications, we can limit detection risk (but should still use Shamiko/Hide My Applist to hide LSPosed).
  • Additionally, if hooks are abused indiscriminately (e.g., hooking GMS) it will cause SafetyNet failure as warned by the dev – so users need to carefully configure the right apps.

- Update status: Well maintained. Chiteroman regularly updates BootloaderSpoofer along with PIF. The Xposed repo community confirms the module works on Android 13, 14.

- Special features: 
  • BootloaderSpoofer focuses on certificate extensions – the part containing information such as bootloader lock status, device status (Certified/Unlocked).
  • The module will directly edit certificates returned by TEE before apps receive them, so that the "unlocked" flag disappears.
  • Basically, it bypasses the hardware-backed attestation mechanism at the client.
  • This is an important piece: combining BootloaderSpoofer (hiding unlocked bootloader) + TrickyStore (providing valid keys) can allow bypassing both hardware and software attestation.
  • BootloaderSpoofer also illustrates how to use Xposed to intervene in narrow targets (just a few apps), helping reduce detection risk compared to system-wide patches.

================================================================================
3. MAGISK/KERNELSU MODULES RELATED TO DEVICE INFORMATION SPOOFING
--------------------------------------------------------------------------------
- Universal SafetyNet Fix (kdrag0n):
   • Expected URL: https://github.com/kdrag0n/universal-safetynet-fix
   • Function: Intervenes in hardware attestation, supports SafetyNet information spoofing.

- MagiskHide Props Config:
   • URL: https://forum.xda-developers.com/t/tool-magiskhide-props-config.3654247/
   • Function: Allows changing props such as build fingerprint, model... to spoof device information.

- Shamiko:
   • Expected URL: https://github.com/LSPosed/Shamiko
   • Function: Completely hides root/hook signs from sensitive processes (e.g., Google Play Services).

- Magisk & KernelSU:
   • Magisk URL: https://github.com/topjohnwu/Magisk
   • KernelSU (with ZygiskNext): Supports running Zygisk modules in the KernelSU environment.

- Other smaller modules:
  • Android Faker (Xposed): Xposed module supporting Android 8.1 – 13, allowing spoofing of many device information types (IMEI, Android ID, email, etc.) on a per-application basis. Similar to XPrivacy but focused on changing device identity. Pro: easy to use, has interface for selecting fake values, helps bypass basic checks in some apps; Con: not sufficient for advanced SafetyNet (doesn't touch attestation). This project is useful for simulating other devices (like Device ID Masker also similar) rather than bypassing Play Integrity, so it's less mentioned in the context of new SafetyNet.

  • ih8sn (I hate SafetyNet): Script add-on for AOSP ROMs, developed by several LineageOS devs. It runs early at boot to spoof sensitive props at the system level (like MagiskHide Props but implemented in ROM init). The result is a custom ROM that can pass basicIntegrity/CTS without needing Magisk. Pro: Suitable for those using pure ROMs who don't want to install Magisk (e.g., LineageOS + MicroG) – just flash ih8sn and the config file for the right model. Does not depend on background services so harder to detect. Con: Requires configuration setup for each model (must have the right fingerprint, model, patch level... in the config file). Not flexible by app – affects the entire system. Currently LineageOS has not officially integrated it (for legal reasons), users must add it themselves. The project is maintained on GitHub by the Lineage community, but its effectiveness mainly stops at basic SafetyNet level; for advanced Play Integrity, other methods need to be combined.

  • selinux_permissive_hide, BypassRootCheckPro, zygisk-maphide (hiding Magisk file traces on /proc), Zygisk-Assistant (replacing Shamiko for KernelSU/APatch).

================================================================================
4. DIRECT MODIFICATION OF ANDROID SOURCE CODE (WITHOUT HOOKS)
--------------------------------------------------------------------------------
- ProtonAOSP (by kdrag0n):
   • URL: Often discussed on XDA; you can refer to examples at:
     https://forum.xda-developers.com/t/protonaosp-rom-tinh-nang-safetynet.4214223/
   • Function: Custom ROM integrating bootloader spoof patches, fingerprint, and key attestation blocking.

- CalyxOS:
   • URL: https://calyxos.org/
   • Function: Security-focused ROM, integrating model change patches to block hardware attestation.

- ih8sn:
   • Reference URL (XDA): https://forum.xda-developers.com/t/ih8sn-safetynet-fix.4214298/
   • Function: Script/shim changing system props to pass SafetyNet on ROMs like LineageOS.

- microG:
   • URL: https://microg.org/
   • Function: Provides a simulated SafetyNet API for devices not using official GMS.

- Integration into custom ROMs: Some ROMs like ProtonAOSP previously integrated SafetyNet patches: they set fake properties directly in the source (e.g., reporting bootloader locked by overriding kernel props at startup, only sending "standard" fingerprints to Google Play services, removing lineage_ suffix in device names, blocking hardware attestation usage in keystore service). These changes allowed ROMs to pass SafetyNet out-of-the-box without needing Magisk. Pro: end users experience seamless integration, no need to install anything extra; Con: large ROM projects (LineageOS, etc.) are reluctant to do this due to policies – only small custom ROMs or personal builds dare to integrate (ProtonAOSP stopped because Google gradually tightened restrictions and the dev changed direction). However, the current trend is for Pixel Experience ports or MIUI EU to try to pass deviceIntegrity by default (signing ROM with private keys, adjusting fingerprint to match the device...) for user convenience. The community also discusses integrating spoofing frameworks into ROMs (e.g., adding signature spoofing & SafetyNet passing to LineageOS for microG), but generally official projects are wary due to legal risks and cat-and-mouse game with Google.

================================================================================
5. KEY ATTESTATION SPOOFING & HARDWARE-BACKED KEYSTORE INFORMATION
--------------------------------------------------------------------------------
Solutions related to key attestation mainly include:
- Key injection via keybox (as in TrickyStore).
- Blocking key attestation to force the system to use basic attestation.
- Adjusting keystore hardware information via props (e.g.: ro.product.first_api_level).
- Hardware exploits: No successful open source projects due to Google's key revocation mechanism.
  
Note: Key attestation spoofing methods usually rely on using a valid keybox;
if this key is leaked or revoked, it will cause serious risks.

================================================================================
6. SELINUX BYPASS AND DROIDGUARD EVASION
--------------------------------------------------------------------------------
- SELinux bypass:
   • Example: selinux_permissive_hide module (often shared on XDA) helps hide SELinux permissive status
     from SafetyNet checks.

- DroidGuard evasion:
   • Using tools such as Magisk DenyList, Shamiko, XposedHide to hide hook/root traces
     from Google Play Services' DroidGuard.
   • Some DroidGuard analyses:
     https://forum.xda-developers.com/t/droidguard-deep-dive.4214234/

- Bypass DroidGuard & SELinux:
  • Regarding SELinux: Most modern solutions keep SELinux in Enforcing mode and simulate a "clean" environment rather than turning off SELinux (as SELinux permissive will be immediately detected by SafetyNet). For example, MagiskHide/Shamiko hides root services under Enforcing, ProtonAOSP overrides the ro.boot.verifiedbootstate prop to "GREEN". There's no specific public module for "fake SELinux", usually just props adjustments and process hiding.
  
  • Regarding DroidGuard (SafetyNet's hidden checking component): technical research (BlackHat 2019, SSTIC 2021) describes how it collects system data. Bypass solutions mainly create a "trusted" environment so DroidGuard doesn't detect anomalies. Therefore modules like SafetyNet Fix/PIF actually disable some DroidGuard checks by blocking hardware attestation, fixing device props, hiding root... (preventing DroidGuard from seeing abnormal signs). Directly "patching" DroidGuard is very difficult because it runs native code downloaded by Google. The microG project once simulated part of SafetyNet (only returning fake basic OK results), but abandoned it due to legal issues and reliability concerns. In summary, there are no public tools to bypass DroidGuard 100% – instead, users combine root hiding, Xposed hiding, prop spoofing, blocking key attestation as in the projects above to make DroidGuard think the device is valid.
  
  • Some indirectly supporting modules: for example zygisk-maphide (hiding Magisk file traces on /proc), Zygisk-Assistant (replacing Shamiko for KernelSU/APatch)... They don't spoof info but help create a cleaner environment. Sandbox evasion techniques (avoiding virtual environment detection) are extensively studied in academic literature and the community is gradually applying them to practical modules.

================================================================================
7. SPOOFING INTEGRATION INTO CUSTOM ROMS
--------------------------------------------------------------------------------
Solutions directly integrated into ROMs to pass SafetyNet without flashing separate modules:
- LineageOS & ih8sn:
   • Reference thread: https://forum.xda-developers.com/t/lineageos-pass-safetynet-with-ih8sn.4214300/
- CalyxOS:
   • URL: https://calyxos.org/
- PixelExperience, crDroid:
   • PixelExperience URL: https://download.pixelexperience.org/
   • crDroid URL: https://crdroid.net/
- /e/OS:
   • URL: https://e.foundation/
  
Advantages: Reduces risks of flashing separate modules, OTA updates don't remove spoofing features.
Disadvantages: Less flexible for users wanting deep root access, must wait for ROM updates when Google changes.

================================================================================
8. ROM REVERSE ENGINEERING METHODS TO CHANGE SYSTEM INFORMATION
--------------------------------------------------------------------------------
Methods for changing system information include:
- Editing build.prop, default.prop files:
   • Tool: MagiskBoot (https://github.com/topjohnwu/MagiskBoot)
- Smali patching framework:
   • Tool: APKTool (https://ibotpeaches.github.io/Apktool/)
- Hex editing binary files:
   • Tool: HxD (https://mh-nexus.de/en/hxd/)
- Native hooking with Frida/Xposed:
   • Frida URL: https://frida.re/
- Analyzing Google Play Services updates:
   • Tools: JADX (https://github.com/skylot/jadx), Ghidra (https://ghidra-sre.org/)
- Reference additional in-depth articles:
   • Example: "Inside SafetyNet Attestation" (papers at Black Hat, SSTIC).

================================================================================
9. TOOLS FOR DETECTING AND ANALYZING GOOGLE'S DEVICE CHECKING MECHANISMS (2023–2024)
--------------------------------------------------------------------------------
- YASNAC (Yet Another SafetyNet Attestation Checker):
   • URL: https://github.com/LSPosed/YASNAC
- RootBeer & root testing kits:
   • RootBeer URL: https://github.com/scottyab/rootbeer
- Reverse engineering tools:
   • APKT