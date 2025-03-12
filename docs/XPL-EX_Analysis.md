> **Important note**: This document focuses on detailed analysis of XPL-EX and how this framework works in spoofing Android device information. For an overview of all Android device information spoofing solutions, please refer to the document "[Analysis of Open Source Projects for Android Device Information Spoofing](android_device_spoofing_comprehensive_analysis.md)". The overview document provides information about various projects, alternative methods, and the latest trends in this field.

# Analysis of XPL-EX Project

## Summary of Application for Android-Spoof Project

**Core values for the Android-Spoof project:**
- Comprehensive spoofing mechanism for all Android device information
- Modular architecture with interceptors and randomizers
- Centralized management of spoofed information but distributed across multiple layers
- Method for ensuring consistency of spoofed data

**Main components to integrate:**
1. Device information (Build Properties) spoofing system
2. Network information and WiFi scan spoofing system
3. SIM/IMEI information spoofing system
4. Input device information spoofing system
5. System uptime spoofing mechanism

## Overview of XPL-EX

XPL-EX is a comprehensive Android device information spoofing framework, developed based on XPrivacyLua but significantly expanded. This project uses Xposed to hook into Android system APIs and change the information returned to applications.

## Operating Mechanism of XPL-EX

Through code analysis, XPL-EX operates according to the following principles:

1. **Hook into system APIs**: Use Xposed to hook into Android API methods
2. **Create fake information**: Use randomizer classes to create consistent fake information
3. **Centralized management**: Use interceptors to handle information requests
4. **Customization via Lua**: Allow customization of hook behavior through Lua scripts

## Analysis of Main Source Code

### Overall Project Structure

```
XPL-EX/
├── .git/                  # Git directory
├── .github/               # GitHub configuration
├── .idea/                 # IntelliJ IDEA configuration
├── app/                   # Directory containing main application source code
├── examples/              # Usage examples
├── gradle/                # Gradle configuration
├── repo/                  # Repository
├── tools/                 # Support tools
├── build.gradle           # Main build configuration file
├── gradle.properties      # Gradle properties
├── gradlew                # Gradle script for Linux/Mac
├── gradlew.bat            # Gradle script for Windows
├── settings.gradle        # Gradle project configuration
├── APIHELP.md             # API documentation
├── APPHELP.md             # Application documentation
├── DEFINE.md              # Definitions
├── FAQ.md                 # Frequently asked questions
├── FUNDING.yml            # Funding information
├── LICENSE                # License
├── LUAHELP.md             # Lua guide
├── README.md              # Main README file
└── XPRIVACY.md            # XPrivacy documentation
```

### Detailed app Directory Structure

```
app/
├── build.gradle           # App build configuration
├── proguard-rules.pro     # ProGuard rules
├── .gitignore             # Git configuration for app directory
└── src/                   # Source code
    └── main/              # Main source code
        ├── assets/        # Resources
        ├── java/          # Java source code
        ├── res/           # Android resources
        ├── AndroidManifest.xml     # App manifest
        └── other files...
```

### Detailed java Directory Structure

```
app/src/main/java/
├── org/
│   └── luaj/              # Lua library for Java
└── eu/
    └── faircode/
        └── xlua/          # Main source code of XPL-EX
            ├── hooks/     # Basic hook mechanism
            │   ├── LuaHook.java              # Basic hook for Lua
            │   ├── LuaHookResolver.java      # Lua hook resolver
            │   ├── LuaHookWrapper.java       # Wrapper for Lua hook
            │   ├── LuaLocals.java            # Manage Lua local variables
            │   ├── LuaLog.java               # Logging for Lua
            │   ├── LuaScriptHolder.java      # Hold Lua code
            │   ├── XHookUtil.java            # Hook utilities
            │   ├── XReport.java              # Hook report
            │   └── XReporter.java            # Hook reporter
            │
            ├── interceptors/                 # API interceptors
            │   ├── shell/                    # Shell interceptors
            │   ├── ShellIntercept.java       # Intercept shell commands
            │   └── UserContextMaps.java      # User context maps
            │
            ├── loggers/                      # Logging system
            │
            ├── api/                          # API for Lua
            │
            ├── x/                            # Extension modules
            │   ├── data/                     # Data handling
            │   │   └── utils/                # Data handling utilities
            │   │       └── random/           # Random data generation
            │   │
            │   ├── file/                     # File handling
            │   │
            │   ├── hook/                     # Extended hook system
            │   │   ├── filter/               # Hook filters
            │   │   ├── inlined/              # Inlined hooks
            │   │   │   ├── HashMapHooks.java # Hooks for HashMap
            │   │   │   └── UpTimeHooks.java  # Hooks for uptime
            │   │   │
            │   │   └── interceptors/         # Extended API interceptors
            │   │       ├── build/            # Intercept build information
            │   │       │   └── random/       # Generate random build information
            │   │       │       ├── RandomBaseOs.java
            │   │       │       ├── RandomBuildCodeName.java
            │   │       │       ├── RandomBuildId.java
            │   │       │       ├── RandomBuildRadio.java
            │   │       │       ├── RandomBuildSDK.java
            │   │       │       ├── RandomBuildTags.java
            │   │       │       ├── RandomBuildType.java
            │   │       │       ├── RandomBuildUser.java
            │   │       │       ├── RandomBuildVersion.java
            │   │       │       ├── RandomDevCodeName.java
            │   │       │       ├── RandomRomBootState.java
            │   │       │       ├── RandomRomName.java
            │   │       │       └── RandomRomSecure.java
            │   │       │
            │   │       ├── cell/             # Intercept mobile information
            │   │       │   ├── SubInfoServiceHook.java      # Hook SIM information
            │   │       │   └── TelephonyManagerInterceptor.java # Intercept TelephonyManager
            │   │       │
            │   │       ├── devices/          # Intercept device information
            │   │       │   ├── random/       # Generate random device information
            │   │       │   ├── InputDeviceType.java
            │   │       │   ├── InputDeviceUtils.java
            │   │       │   ├── MockDevice.java              # Mock device
            │   │       │   └── InputDeviceInterceptor.java  # Intercept input device information
            │   │       │
            │   │       ├── hardware/         # Intercept hardware information
            │   │       ├── ipc/              # Intercept inter-process communication
            │   │       ├── network/          # Intercept network information
            │   │       ├── pkg/              # Intercept package information
            │   │       └── zone/             # Intercept zone information
            │   │
            │   ├── network/                  # Network module
            │   │   ├── cell/                 # Mobile network handling
            │   │   │   └── randomizers/      # Generate random mobile network information
            │   │   │       ├── RandomMCC.java              # Generate random MCC
            │   │   │       └── RandomMNC.java              # Generate random MNC
            │   │   │
            │   │   ├── ipv6/                 # IPv6 handling
            │   │   ├── randomizers/          # Generate random network information
            │   │   ├── NetInfoGenerator.java # Generate network information
            │   │   ├── NetRandom.java        # Random for network
            │   │   └── NetUtils.java         # Network utilities
            │   │
            │   ├── process/                  # Process handling
            │   ├── runtime/                  # Runtime environment
            │   ├── ui/                       # User interface
            │   └── xlua/                     # XLua extensions
            │       ├── LibUtil.java          # Library utilities
            │       ├── XposedUtility.java    # Xposed utilities
            │       ├── commands/             # Commands
            │       ├── hook/                 # Hooks
            │       └── settings/             # Settings
            │
            ├── utilities/                    # Utilities
            ├── random/                       # Random generation module
            ├── rootbox/                      # Root module
            ├── tools/                        # Tools
            ├── logger/                       # Logging module
            ├── builders/                     # Object builders
            ├── display/                      # Display
            ├── ui/                           # User interface
            │
            ├── XLua.java                     # Main class of XLua
            ├── XParam.java                   # Parameters for hook
            ├── XUtil.java                    # Utilities
            ├── XposedUtil.java               # Xposed utilities
            ├── XUiGroup.java                 # UI group
            ├── XCommandBridgeStatic.java     # Static command bridge
            ├── XDatabaseOld.java             # Old database
            ├── XNotify.java                  # Notifications
            ├── XPolicy.java                  # Policy
            ├── XSecurity.java                # Security
            ├── XUiConfig.java                # UI configuration
            │
            ├── ActivityBase.java             # Base activity
            ├── ActivityConfig.java           # Configuration activity
            ├── ActivityCpu.java              # CPU activity
            ├── ActivityDatabase.java         # Database activity
            ├── ActivityHelp.java             # Help activity
            ├── ActivityMain.java             # Main activity
            ├── ActivityProperties.java       # Properties activity
            ├── ActivityProps.java            # Props activity
            ├── ActivitySettings.java         # Settings activity
            │
            ├── AdapterApp.java               # App adapter
            ├── AdapterConfig.java            # Configuration adapter
            ├── AdapterCpu.java               # CPU adapter
            ├── AdapterDatabase.java          # Database adapter
            ├── AdapterGroup.java             # Group adapter
            ├── AdapterGroupHooks.java        # Hook group adapter
            ├── AdapterHook.java              # Hook adapter
            ├── AdapterHookSettings.java      # Hook settings adapter
            ├── AdapterProp.java              # Prop adapter
            ├── AdapterProperties.java        # Properties adapter
            ├── AdapterPropertiesGroup.java   # Properties group adapter
            ├── AdapterProperty.java          # Property adapter
            ├── AdapterSetting.java           # Setting adapter
            │
            ├── AppGeneric.java               # Generic application
            ├── ApplicationEx.java            # Extended application
            ├── DebugUtil.java                # Debug utilities
            │
            ├── FragmentAppControl.java       # App control fragment
            ├── FragmentConfig.java           # Configuration fragment
            ├── FragmentCpu.java              # CPU fragment
            ├── FragmentDatabase.java         # Database fragment
            ├── FragmentMain.java             # Main fragment
            ├── FragmentProperties.java       # Properties fragment
            ├── FragmentPropsEx.java          # Extended props fragment
            ├── FragmentSettings.java         # Settings fragment
            │
            ├── GlideHelper.java              # Glide helper
            ├── ReceiverPackage.java          # Package receiver
            ├── SettingDefine.java            # Setting definition
            ├── TextDividerItemDecoration.java# Text divider item decoration
            ├── UberCore888.java              # Uber core
            └── VXP.java                      # Virtual Xposed
```

## Main Spoofing Components

### 1. Device Information Spoofing (Build Properties)

XPL-EX uses classes like `RandomRomName`, `RandomBuildCodeName`, `RandomDevCodeName` to spoof build information. For example:

```java
public class RandomRomName extends RandomElement {
    public static IRandomizer create() { return new RandomRomName(); }
    public static final String[] ROM_NAMES = {
            // Popular Active ROMs
            "AOSP",         // Android Open Source Project
            "LineageOS",    // Formerly CyanogenMod
            "YAAP",         // Yet Another AOSP Project
            "PixelOS",      // AOSP with Pixel features
            // ...many other ROMs
    };

    public RandomRomName() {
        super("Build ROM Name");
        bindSetting("android.rom.name");
    }

    @Override
    public String generateString() { 
        return RandomStringGenerator.randomStringIfRandomElse(
            ROM_NAMES[RandomGenerator.nextInt(0, ROM_NAMES.length)]); 
    }
}
```

### 2. Network Information Spoofing

XPL-EX uses `NetInfoGenerator` to create fake network information such as IP, DNS, domain, etc.:

```java
public class NetInfoGenerator {
    private static final String TAG = "XLua.NetInfoGenerator";

    public static final List<String> PROVIDERS = Arrays.asList(
        "Comcast", "AT&T", "Verizon", "Spectrum", "Cox", 
        "CenturyLink", "Frontier", "Optimum", "Google Fiber", "Windstream"
    );

    private final String provider;
    private String ipv4Address;
    private String netmask;
    private String gateway;
    private List<String> routes;
    private String ipv6Address;
    private List<String> dnsServers;
    private String domain;
    private String dhcpServer;
    
    // ... methods to generate information
}
```

### 3. Input Device Information Spoofing

XPL-EX uses `MockDevice` to spoof device information such as name, descriptor, vendor ID, product ID:

```java
public class MockDevice {
    private static final String TAG = "XLua.MockDevice";

    public static final DynamicField FIELD_DESCRIPTOR = new DynamicField(InputDevice.class, "mDescriptor")
            .setAccessible(true);

    public static final DynamicField FIELD_NAME = new DynamicField(InputDevice.class, "mName")
            .setAccessible(true);

    public static final DynamicField FIELD_VENDOR_ID = new DynamicField(InputDevice.class, "mVendorId")
            .setAccessible(true);

    public static final DynamicField FIELD_PRODUCT_ID = new DynamicField(InputDevice.class, "mProductId")
            .setAccessible(true);

    public String name;
    public String descriptor;
    public int id;
    public int vendorId;
    public int productId;

    public InputDeviceType type;

    public String fakeName;
    public String fakeDescriptor;
    public String fakeDescriptorHash;
    public int fakeVendorId;
    public int fakeProductId;
    
    // ... spoofing methods
}
```

### 4. SIM/IMEI Information Spoofing

XPL-EX has classes like `SubInfoServiceHook`, `RandomMCC`, `RandomMNC` to spoof SIM information:

```java
public class RandomMCC extends RandomElement {
    public static RandomMCC create() { return new RandomMCC(); }
    public RandomMCC() {
        super("Cell MCC");
        bindSetting("gsm.operator.mcc");
    }

    @Override
    public String generateString() { 
        return String.valueOf(RandomGenerator.nextInt(100, 999)); 
    }
}
```

### 5. System Uptime Spoofing

XPL-EX uses `UpTimeHooks` to spoof system uptime information:

```java
public static class UpTimeHooks extends XHook {
    private static final String LOG_TAG = "XLua.UpTimeHooks";
    
    @Override
    protected void before(XParam param) throws Throwable {
        // Change uptime/system time value
        long fakeTime = generateFakeTime();
        param.setResult(fakeTime);
    }
    
    private long generateFakeTime() {
        // Generate appropriate fake time
        // ...
    }
}
```

### 6. WiFi Scan Spoofing

XPL-EX has a mechanism to spoof WiFi scan results to avoid location tracking:

```java
// Method to generate fake WiFi scan results
private void generateFakeWifiScanResults() {
    // Generate fake scan results
    FIELD_WIFI_SEEN.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomSeen());
    FIELD_WIFI_UNTRUSTED.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomUntrusted());
    FIELD_WIFI_NUM_USAGE.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomNumUsage(numConnections));
    FIELD_WIFI_FLAGS.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomFlags());
    FIELD_WIFI_INFORMATION_ELEMENTS.trySetValueInstanceEx(emptyResult, fakeElements);
}
```

## Proposed Integration into Android-Spoof

### Converting from Hooks to Source Code Modification

The Android-Spoof project needs to convert hook mechanisms from XPL-EX to direct modifications of LineageOS source code:

#### 1. Service Architecture for Device Information

```java
// Instead of hooking Build.java, directly modify it
public class Build {
    private static final SpoofingDeviceInfoService sDeviceInfoService =
        SpoofingDeviceInfoService.getInstance();
    
    public static final String BRAND = sDeviceInfoService.getBuildProperty("ro.product.brand");
    public static final String MODEL = sDeviceInfoService.getBuildProperty("ro.product.model");
    // Các trường khác...
    
    public static class VERSION {
        public static final String RELEASE = sDeviceInfoService.getVersionProperty("ro.build.version.release");
        // Các trường khác...
    }
}
```

#### 2. Service Architecture for Network Information

```java
// Fake network information management service
public class SpoofingNetworkService extends SystemService {
    // Manage WiFi information
    public ScanResult[] getSpoofedWifiScanResults() {
        // Generate fake WiFi scan results
        // Based on WifiRandom from XPL-EX
    }
    
    // Manage network information
    public NetworkInfo getSpoofedNetworkInfo() {
        // Generate fake network information
        // Based on NetInfoGenerator from XPL-EX
    }
}
```

#### 3. Service Architecture for SIM/IMEI Information

```java
// SIM/IMEI information management service
public class SpoofingTelephonyService extends SystemService {
    // Manage IMEI information
    public String getSpoofedImei() {
        // Generate fake IMEI
    }
    
    // Manage SIM information
    public String getSpoofedMcc() {
        // Based on RandomMCC from XPL-EX
    }
    
    public String getSpoofedMnc() {
        // Based on RandomMNC from XPL-EX
    }
}
```

#### 4. Service Architecture for System Time

```java
// Fake time management service
public class SpoofingTimeService extends SystemService {
    // Manage uptime
    public long getSpoofedUptime() {
        // Based on UpTimeHooks from XPL-EX
    }
    
    // Manage system time
    public long getSpoofedSystemTime() {
        // Generate fake system time
    }
}
```

### Advantages of the Proposed Architecture

1. **Centralized Management**: All spoofed information is centrally managed through specialized system services

2. **Consistency**: Ensures all APIs return consistent information

3. **Maintainability**: Each type of information has its own service for easy maintenance and expansion

4. **System Integration**: Works as part of the LineageOS system instead of an external module

### Specific Integration Steps

1. **Analyze and Extract Logic from XPL-EX**:
   - Extract randomizer classes to generate fake information
   - Extract configuration management and consistency mechanisms
   - Separate spoofing logic from hook mechanisms

2. **Build System Services in LineageOS**:
   - Create services as proposed above
   - Integrate into LineageOS startup system
   - Establish storage and configuration management mechanisms

3. **Modify Core System Classes**:
   - Modify Build.java, TelephonyManager.java, etc.
   - Replace default API calls with calls to spoofing services
   - Ensure compatibility with the rest of the system

4. **Build Management Application**:
   - Create interface to configure spoofed information
   - Connect to system services through AIDL
   - Allow users to save and select different device profiles

### Technical Challenges

1. **Ensuring Consistency**:
   - Spoofed information must be consistent across all APIs
   - Need centralized storage and information distribution mechanism

2. **Performance Issues**:
   - Information retrieval methods are called frequently
   - Need optimization to avoid impact on performance

3. **Security**:
   - Need to protect spoofing mechanism from untrusted applications
   - Manage access permissions so only privileged applications can change configuration

## Conclusion

XPL-EX provides a powerful framework for spoofing Android device information, with components that can be integrated into the Android-Spoof project. By converting from hook mechanisms to direct modification of LineageOS source code, we can build a comprehensive spoofing solution capable of controlling all device information to protect user privacy.

A system management application will help users easily set up spoofed information, while system services ensure consistency of information across the entire system. The modular architecture will allow easy expansion and maintenance in the future. 