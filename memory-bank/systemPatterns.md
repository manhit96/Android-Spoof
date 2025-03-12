# Android-Spoof System Architecture and Design Patterns

## Overall Architecture
Android-Spoof uses a modular architecture with a central service, combined with specialized components to create a comprehensive solution. This architecture is designed to:
1. Deeply integrate into the Android system
2. Maintain information consistency
3. Be easily extensible and maintainable
4. Bypass integrity verification mechanisms

## Architecture Model
```
+---------------------+     +-------------------------+     +----------------------+
|                     |     |                         |     |                      |
| System Properties   |     | DeviceSpoofService      |     | Key Attestation      |
| Modifier            <----->  (Core Service)         <----->  Provider            |
|                     |     |                         |     |                      |
+---------------------+     +-------------------------+     +----------------------+
                                        ^
                                        |
                                        v
                             +-------------------------+
                             |                         |
                             | Device Spoof Manager    |
                             |  (User Interface)       |
                             |                         |
                             +-------------------------+
```

## Key Components and Design Patterns

### 1. Core Service Framework (DeviceSpoofService)
**Design Patterns**: Singleton, Facade, Repository
- Central management for all spoofed information
- Provides consistent API for other components
- Manages device profiles and configuration

**Core Source Code**:
```java
// Implemented as a System Service
public class DeviceSpoofService extends SystemService {
    // Singleton instance
    private static DeviceSpoofService sInstance;
    
    // Current profile
    private DeviceProfile mCurrentProfile;
    
    // Cache information to optimize performance
    private final Map<String, String> mPropertyCache = new HashMap<>();
    
    // Manage device information
    public String getProperty(String key) {
        if (mPropertyCache.containsKey(key)) {
            return mPropertyCache.get(key);
        }
        
        String value = mCurrentProfile.getProperty(key);
        mPropertyCache.put(key, value);
        return value;
    }
    
    // Other APIs for network, telephony, and subsystems
    // ...
}
```

### 2. System Properties Modifier
**Design Patterns**: Proxy, Decorator
- Replace or extend system classes (Build, SystemProperties)
- Redirect API calls to DeviceSpoofService
- Modify classes in the Android framework

**Classes to modify**:
- `android.os.Build`
- `android.os.SystemProperties`
- `android.telephony.TelephonyManager`
- `android.net.wifi.WifiInfo`

### 3. Key Attestation Provider
**Design Patterns**: Strategy, Factory
- Manage keybox and attestation certificates
- Implement different strategies for attestation
- Handle attestation requests from Google Play Services

**Core Logic**:
- Create or modify certificate chains
- Manage keybox and rotation
- Integrate with Play Integrity API

### 4. Device Spoof Manager (UI)
**Design Patterns**: MVC/MVVM
- User interface for configuration management
- Display information about the current device
- Manage profiles and settings

## Data Structures
### Device Profile
Device profile data is stored in JSON format with the structure:
```json
{
  "profile_name": "Profile Name",
  "device": {
    "brand": "brand",
    "model": "model",
    "manufacturer": "manufacturer",
    "fingerprint": "full fingerprint"
  },
  "network": {
    // Network configuration
  },
  "telephony": {
    // Telephony configuration
  },
  "keystore": {
    // Keystore configuration
  }
}
```

### File System
```
/data/system/device_spoof/
├── profiles/                 # Directory containing user profiles
│   ├── default.json          # Default profile
│   └── user_profiles/        # User-created profiles
├── settings.json             # System settings
└── keybox/                   # Keybox and attestation keys
    ├── current_keybox.xml    # Current keybox
    └── rotation_history/     # Keybox history
```

## Core Processes

### Boot Process
1. DeviceSpoofService is started during system boot
2. Read default or configured profile information
3. Initialize cache and management components
4. Set up hooks or system modifications

### Information Request Handling Process
1. Application or framework calls system API (e.g., Build.MODEL)
2. System Properties Modifier intercepts API call
3. Redirects to DeviceSpoofService
4. Returns spoofed information from the current profile

### Key Attestation Process
1. Application requests key attestation
2. Android KeyStore redirects request to Key Attestation Provider
3. Provider modifies certificate chain based on current keybox
4. Returns modified certificate resembling the original device

## Integration Strategies
The project uses two integration strategies:

### 1. Direct Source Code Integration (LineageOS)
- Directly modify classes in the Android framework
- Add new services to the system
- Compile into a full LineageOS build

### 2. Integration through Reverse Engineering
- Create bridge module to integrate into stock ROMs
- Use "hybrid integration" technique
- Package as a Magisk Module for stock ROMs 