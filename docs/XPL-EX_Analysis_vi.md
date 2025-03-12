> **Lưu ý quan trọng**: Tài liệu này tập trung phân tích chi tiết về XPL-EX và cách thức hoạt động của framework này trong việc giả mạo thông tin thiết bị Android. Để có cái nhìn tổng quan về toàn bộ các giải pháp giả mạo thông tin thiết bị Android, vui lòng tham khảo thêm tài liệu "[Phân tích các dự án mã nguồn mở về giả mạo thông tin thiết bị Android](android_device_spoofing_comprehensive_analysis.md)". Tài liệu tổng quan cung cấp thông tin về nhiều dự án khác nhau, các phương pháp thay thế, và xu hướng mới nhất trong lĩnh vực này.

# Phân tích dự án XPL-EX

## Tóm tắt ứng dụng cho dự án Android-Spoof

**Giá trị cốt lõi cho dự án Android-Spoof:**
- Cơ chế spoofing toàn diện cho mọi thông tin thiết bị Android
- Kiến trúc module hóa với các interceptor và randomizer
- Quản lý tập trung thông tin giả mạo nhưng phân tán qua nhiều lớp
- Phương pháp đảm bảo tính nhất quán của dữ liệu giả mạo

**Các thành phần chính cần tích hợp:**
1. Hệ thống spoofing thông tin thiết bị (Build Properties)
2. Hệ thống spoofing thông tin mạng và quét WiFi
3. Hệ thống spoofing thông tin SIM/IMEI
4. Hệ thống spoofing thiết bị đầu vào
5. Cơ chế giả mạo thời gian hệ thống (uptime)

## Tổng quan về XPL-EX

XPL-EX là một framework giả mạo (spoofing) thông tin thiết bị Android toàn diện, được phát triển dựa trên XPrivacyLua nhưng đã được mở rộng đáng kể. Dự án này sử dụng Xposed để hook vào các API hệ thống Android và thay đổi thông tin trả về cho các ứng dụng.

## Cơ chế hoạt động của XPL-EX

Qua phân tích mã nguồn, XPL-EX hoạt động theo các nguyên tắc sau:

1. **Hook vào API hệ thống**: Sử dụng Xposed để hook vào các phương thức của Android API
2. **Tạo thông tin giả**: Sử dụng các lớp randomizer để tạo thông tin giả mạo nhất quán
3. **Quản lý tập trung**: Sử dụng các interceptor để xử lý các yêu cầu thông tin
4. **Tùy chỉnh qua Lua**: Cho phép tùy chỉnh hành vi hook thông qua script Lua

## Phân tích mã nguồn chính

### Cấu trúc tổng thể dự án

```
XPL-EX/
├── .git/                  # Thư mục Git
├── .github/               # Cấu hình GitHub
├── .idea/                 # Cấu hình IntelliJ IDEA
├── app/                   # Thư mục chứa mã nguồn ứng dụng chính
├── examples/              # Ví dụ sử dụng
├── gradle/                # Cấu hình Gradle
├── repo/                  # Kho lưu trữ
├── tools/                 # Công cụ hỗ trợ
├── build.gradle           # Tệp cấu hình build chính
├── gradle.properties      # Thuộc tính Gradle
├── gradlew                # Script Gradle cho Linux/Mac
├── gradlew.bat            # Script Gradle cho Windows
├── settings.gradle        # Cấu hình dự án Gradle
├── APIHELP.md             # Tài liệu hướng dẫn API
├── APPHELP.md             # Tài liệu hướng dẫn ứng dụng
├── DEFINE.md              # Định nghĩa
├── FAQ.md                 # Câu hỏi thường gặp
├── FUNDING.yml            # Thông tin tài trợ
├── LICENSE                # Giấy phép
├── LUAHELP.md             # Hướng dẫn về Lua
├── README.md              # Tệp README chính
└── XPRIVACY.md            # Tài liệu về XPrivacy
```

### Cấu trúc chi tiết thư mục app

```
app/
├── build.gradle           # Cấu hình build của ứng dụng
├── proguard-rules.pro     # Quy tắc ProGuard
├── .gitignore             # Cấu hình Git cho thư mục app
└── src/                   # Mã nguồn
    └── main/              # Mã nguồn chính
        ├── assets/        # Tài nguyên
        ├── java/          # Mã nguồn Java
        ├── res/           # Tài nguyên Android
        ├── AndroidManifest.xml     # Manifest của ứng dụng
        └── các tệp khác...
```

### Cấu trúc chi tiết thư mục java

```
app/src/main/java/
├── org/
│   └── luaj/              # Thư viện Lua cho Java
└── eu/
    └── faircode/
        └── xlua/          # Mã nguồn chính của XPL-EX
            ├── hooks/     # Cơ chế hook cơ bản
            │   ├── LuaHook.java              # Hook cơ bản cho Lua
            │   ├── LuaHookResolver.java      # Bộ xử lý hook Lua
            │   ├── LuaHookWrapper.java       # Wrapper cho hook Lua
            │   ├── LuaLocals.java            # Quản lý biến cục bộ Lua
            │   ├── LuaLog.java               # Ghi log cho Lua
            │   ├── LuaScriptHolder.java      # Giữ mã Lua
            │   ├── XHookUtil.java            # Tiện ích hook
            │   ├── XReport.java              # Báo cáo hook
            │   └── XReporter.java            # Bộ báo cáo hook
            │
            ├── interceptors/                 # Bộ chặn/can thiệp API
            │   ├── shell/                    # Chặn shell
            │   ├── ShellIntercept.java       # Chặn lệnh shell
            │   └── UserContextMaps.java      # Bản đồ ngữ cảnh người dùng
            │
            ├── loggers/                      # Hệ thống ghi nhật ký
            │
            ├── api/                          # API cho Lua
            │
            ├── x/                            # Các module mở rộng
            │   ├── data/                     # Xử lý dữ liệu
            │   │   └── utils/                # Tiện ích xử lý dữ liệu
            │   │       └── random/           # Tạo dữ liệu ngẫu nhiên
            │   │
            │   ├── file/                     # Xử lý tệp
            │   │
            │   ├── hook/                     # Hệ thống hook mở rộng
            │   │   ├── filter/               # Bộ lọc hook
            │   │   ├── inlined/              # Hook nội tuyến
            │   │   │   ├── HashMapHooks.java # Hook cho HashMap
            │   │   │   └── UpTimeHooks.java  # Hook cho thời gian uptime
            │   │   │
            │   │   └── interceptors/         # Bộ chặn API mở rộng
            │   │       ├── build/            # Chặn thông tin build
            │   │       │   └── random/       # Tạo thông tin build ngẫu nhiên
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
            │   │       ├── cell/             # Chặn thông tin di động
            │   │       │   ├── SubInfoServiceHook.java      # Hook thông tin SIM
            │   │       │   └── TelephonyManagerInterceptor.java # Chặn TelephonyManager
            │   │       │
            │   │       ├── devices/          # Chặn thông tin thiết bị
            │   │       │   ├── random/       # Tạo thông tin thiết bị ngẫu nhiên
            │   │       │   ├── InputDeviceType.java
            │   │       │   ├── InputDeviceUtils.java
            │   │       │   ├── MockDevice.java              # Giả lập thiết bị
            │   │       │   └── InputDeviceInterceptor.java  # Chặn thông tin thiết bị đầu vào
            │   │       │
            │   │       ├── hardware/         # Chặn thông tin phần cứng
            │   │       ├── ipc/              # Chặn giao tiếp liên quá trình
            │   │       ├── network/          # Chặn thông tin mạng
            │   │       ├── pkg/              # Chặn thông tin package
            │   │       └── zone/             # Chặn thông tin khu vực
            │   │
            │   ├── network/                  # Module xử lý mạng
            │   │   ├── cell/                 # Xử lý mạng di động
            │   │   │   └── randomizers/      # Tạo thông tin mạng di động ngẫu nhiên
            │   │   │       ├── RandomMCC.java              # Tạo MCC ngẫu nhiên
            │   │   │       └── RandomMNC.java              # Tạo MNC ngẫu nhiên
            │   │   │
            │   │   ├── ipv6/                 # Xử lý IPv6
            │   │   ├── randomizers/          # Tạo thông tin mạng ngẫu nhiên
            │   │   ├── NetInfoGenerator.java # Tạo thông tin mạng
            │   │   ├── NetRandom.java        # Tạo ngẫu nhiên cho mạng
            │   │   └── NetUtils.java         # Tiện ích mạng
            │   │
            │   ├── process/                  # Xử lý quy trình
            │   ├── runtime/                  # Môi trường runtime
            │   ├── ui/                       # Giao diện người dùng
            │   └── xlua/                     # Mở rộng XLua
            │       ├── LibUtil.java          # Tiện ích thư viện
            │       ├── XposedUtility.java    # Tiện ích Xposed
            │       ├── commands/             # Lệnh
            │       ├── hook/                 # Hook
            │       └── settings/             # Cài đặt
            │
            ├── utilities/                    # Các tiện ích
            ├── random/                       # Module tạo ngẫu nhiên
            ├── rootbox/                      # Module root
            ├── tools/                        # Công cụ
            ├── logger/                       # Module ghi nhật ký
            ├── builders/                     # Xây dựng đối tượng
            ├── display/                      # Hiển thị
            ├── ui/                           # Giao diện người dùng
            │
            ├── XLua.java                     # Lớp chính của XLua
            ├── XParam.java                   # Tham số cho hook
            ├── XUtil.java                    # Tiện ích
            ├── XposedUtil.java               # Tiện ích Xposed
            ├── XUiGroup.java                 # Nhóm UI
            ├── XCommandBridgeStatic.java     # Cầu nối lệnh tĩnh
            ├── XDatabaseOld.java             # Cơ sở dữ liệu cũ
            ├── XNotify.java                  # Thông báo
            ├── XPolicy.java                  # Chính sách
            ├── XSecurity.java                # Bảo mật
            ├── XUiConfig.java                # Cấu hình UI
            │
            ├── ActivityBase.java             # Activity cơ sở
            ├── ActivityConfig.java           # Activity cấu hình
            ├── ActivityCpu.java              # Activity CPU
            ├── ActivityDatabase.java         # Activity cơ sở dữ liệu
            ├── ActivityHelp.java             # Activity trợ giúp
            ├── ActivityMain.java             # Activity chính
            ├── ActivityProperties.java       # Activity thuộc tính
            ├── ActivityProps.java            # Activity props
            ├── ActivitySettings.java         # Activity cài đặt
            │
            ├── AdapterApp.java               # Adapter ứng dụng
            ├── AdapterConfig.java            # Adapter cấu hình
            ├── AdapterCpu.java               # Adapter CPU
            ├── AdapterDatabase.java          # Adapter cơ sở dữ liệu
            ├── AdapterGroup.java             # Adapter nhóm
            ├── AdapterGroupHooks.java        # Adapter nhóm hook
            ├── AdapterHook.java              # Adapter hook
            ├── AdapterHookSettings.java      # Adapter cài đặt hook
            ├── AdapterProp.java              # Adapter prop
            ├── AdapterProperties.java        # Adapter thuộc tính
            ├── AdapterPropertiesGroup.java   # Adapter nhóm thuộc tính
            ├── AdapterProperty.java          # Adapter thuộc tính
            ├── AdapterSetting.java           # Adapter cài đặt
            │
            ├── AppGeneric.java               # Ứng dụng chung
            ├── ApplicationEx.java            # Ứng dụng mở rộng
            ├── DebugUtil.java                # Tiện ích gỡ lỗi
            │
            ├── FragmentAppControl.java       # Fragment điều khiển ứng dụng
            ├── FragmentConfig.java           # Fragment cấu hình
            ├── FragmentCpu.java              # Fragment CPU
            ├── FragmentDatabase.java         # Fragment cơ sở dữ liệu
            ├── FragmentMain.java             # Fragment chính
            ├── FragmentProperties.java       # Fragment thuộc tính
            ├── FragmentPropsEx.java          # Fragment props mở rộng
            ├── FragmentSettings.java         # Fragment cài đặt
            │
            ├── GlideHelper.java              # Trợ giúp Glide
            ├── ReceiverPackage.java          # Receiver package
            ├── SettingDefine.java            # Định nghĩa cài đặt
            ├── TextDividerItemDecoration.java# Trang trí phân cách văn bản
            ├── UberCore888.java              # Core Uber
            └── VXP.java                      # Virtual Xposed
```

## Các thành phần spoofing chính

### 1. Spoofing thông tin thiết bị (Build Properties)

XPL-EX sử dụng các lớp như `RandomRomName`, `RandomBuildCodeName`, `RandomDevCodeName` để giả mạo thông tin build. Ví dụ:

```java
public class RandomRomName extends RandomElement {
    public static IRandomizer create() { return new RandomRomName(); }
    public static final String[] ROM_NAMES = {
            // Popular Active ROMs
            "AOSP",         // Android Open Source Project
            "LineageOS",    // Formerly CyanogenMod
            "YAAP",         // Yet Another AOSP Project
            "PixelOS",      // AOSP with Pixel features
            // ...nhiều ROM khác
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

### 2. Spoofing thông tin mạng

XPL-EX sử dụng `NetInfoGenerator` để tạo thông tin mạng giả mạo như IP, DNS, domain, v.v.:

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
    
    // ... phương thức tạo thông tin
}
```

### 3. Spoofing thông tin thiết bị đầu vào

XPL-EX sử dụng `MockDevice` để giả mạo thông tin thiết bị như name, descriptor, vendor ID, product ID:

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
    
    // ... phương thức spoofing
}
```

### 4. Spoofing thông tin SIM/IMEI

XPL-EX có các lớp như `SubInfoServiceHook`, `RandomMCC`, `RandomMNC` để giả mạo thông tin SIM:

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

### 5. Spoofing thông tin thời gian hệ thống (Uptime)

XPL-EX sử dụng `UpTimeHooks` để giả mạo thông tin thời gian hệ thống:

```java
public static class UpTimeHooks extends XHook {
    private static final String LOG_TAG = "XLua.UpTimeHooks";
    
    @Override
    protected void before(XParam param) throws Throwable {
        // Thay đổi giá trị uptime/thời gian hệ thống
        long fakeTime = generateFakeTime();
        param.setResult(fakeTime);
    }
    
    private long generateFakeTime() {
        // Tạo thời gian giả mạo phù hợp
        // ...
    }
}
```

### 6. Spoofing thông tin WiFi Scan

XPL-EX có cơ chế giả mạo kết quả quét WiFi để tránh theo dõi vị trí:

```java
// Phương thức tạo kết quả quét WiFi giả
private void generateFakeWifiScanResults() {
    // Tạo các kết quả quét giả mạo
    FIELD_WIFI_SEEN.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomSeen());
    FIELD_WIFI_UNTRUSTED.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomUntrusted());
    FIELD_WIFI_NUM_USAGE.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomNumUsage(numConnections));
    FIELD_WIFI_FLAGS.trySetValueInstanceEx(emptyResult, WifiRandom.generateRandomFlags());
    FIELD_WIFI_INFORMATION_ELEMENTS.trySetValueInstanceEx(emptyResult, fakeElements);
}
```

## Đề xuất tích hợp vào Android-Spoof

### Chuyển đổi từ Hook sang Sửa đổi Mã nguồn

Dự án Android-Spoof cần chuyển đổi các cơ chế hook từ XPL-EX sang sửa đổi trực tiếp mã nguồn LineageOS:

#### 1. Kiến trúc Service cho thông tin thiết bị

```java
// Thay vì hook Build.java, trực tiếp sửa đổi
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

#### 2. Kiến trúc Service cho thông tin mạng

```java
// Service quản lý thông tin mạng giả
public class SpoofingNetworkService extends SystemService {
    // Quản lý thông tin WiFi
    public ScanResult[] getSpoofedWifiScanResults() {
        // Tạo kết quả quét WiFi giả mạo
        // Dựa trên WifiRandom từ XPL-EX
    }
    
    // Quản lý thông tin mạng
    public NetworkInfo getSpoofedNetworkInfo() {
        // Tạo thông tin mạng giả mạo
        // Dựa trên NetInfoGenerator từ XPL-EX
    }
}
```

#### 3. Kiến trúc Service cho thông tin SIM/IMEI

```java
// Service quản lý thông tin SIM/IMEI
public class SpoofingTelephonyService extends SystemService {
    // Quản lý thông tin IMEI
    public String getSpoofedImei() {
        // Tạo IMEI giả mạo
    }
    
    // Quản lý thông tin SIM
    public String getSpoofedMcc() {
        // Dựa trên RandomMCC từ XPL-EX
    }
    
    public String getSpoofedMnc() {
        // Dựa trên RandomMNC từ XPL-EX
    }
}
```

#### 4. Kiến trúc Service cho thời gian hệ thống

```java
// Service quản lý thời gian giả mạo
public class SpoofingTimeService extends SystemService {
    // Quản lý uptime
    public long getSpoofedUptime() {
        // Dựa trên UpTimeHooks từ XPL-EX
    }
    
    // Quản lý thời gian hệ thống
    public long getSpoofedSystemTime() {
        // Tạo thời gian hệ thống giả mạo
    }
}
```

### Ưu điểm của kiến trúc đề xuất

1. **Quản lý tập trung**: Tất cả thông tin giả mạo được quản lý tập trung thông qua các service hệ thống chuyên biệt

2. **Tính nhất quán**: Đảm bảo tất cả các API trả về thông tin nhất quán

3. **Dễ bảo trì**: Mỗi loại thông tin có một service riêng để dễ bảo trì và mở rộng

4. **Tích hợp với hệ thống**: Hoạt động như một phần của hệ thống LineageOS thay vì một module bên ngoài

### Các bước tích hợp cụ thể

1. **Phân tích và trích xuất logic từ XPL-EX**:
   - Trích xuất các lớp randomizer để tạo thông tin giả
   - Trích xuất cơ chế quản lý cấu hình và tính nhất quán
   - Tách biệt logic spoofing khỏi cơ chế hook

2. **Xây dựng các Service hệ thống trong LineageOS**:
   - Tạo các service như đã đề xuất ở trên
   - Tích hợp vào hệ thống khởi động của LineageOS
   - Thiết lập cơ chế lưu trữ và quản lý cấu hình

3. **Sửa đổi các lớp hệ thống cốt lõi**:
   - Sửa đổi Build.java, TelephonyManager.java, v.v.
   - Thay thế các cuộc gọi API mặc định bằng cuộc gọi đến service giả mạo
   - Đảm bảo tính tương thích với phần còn lại của hệ thống

4. **Xây dựng ứng dụng quản lý**:
   - Tạo giao diện để cấu hình thông tin giả mạo
   - Kết nối với các service hệ thống thông qua AIDL
   - Cho phép người dùng lưu và chọn các profile thiết bị khác nhau

### Các thách thức kỹ thuật

1. **Đảm bảo tính nhất quán**:
   - Thông tin giả mạo phải nhất quán trong tất cả các API
   - Cần cơ chế lưu trữ tập trung và phân phối thông tin

2. **Xử lý vấn đề hiệu suất**:
   - Các phương thức truy xuất thông tin được gọi thường xuyên
   - Cần tối ưu hóa để tránh ảnh hưởng đến hiệu suất

3. **Tính bảo mật**:
   - Cần bảo vệ cơ chế giả mạo khỏi các ứng dụng không đáng tin cậy
   - Quản lý quyền truy cập để chỉ ứng dụng có quyền mới có thể thay đổi cấu hình

## Kết luận

XPL-EX cung cấp một framework mạnh mẽ để giả mạo thông tin thiết bị Android, với các thành phần có thể tích hợp vào dự án Android-Spoof. Bằng cách chuyển đổi từ cơ chế hook sang sửa đổi trực tiếp mã nguồn LineageOS, chúng ta có thể xây dựng một giải pháp spoofing toàn diện, có khả năng kiểm soát mọi thông tin thiết bị để bảo vệ quyền riêng tư của người dùng.

Một ứng dụng hệ thống quản lý cấu hình sẽ giúp người dùng dễ dàng thiết lập thông tin giả mạo, trong khi các service hệ thống bảo đảm tính nhất quán của thông tin trên toàn bộ hệ thống. Kiến trúc module hóa sẽ cho phép dễ dàng mở rộng và bảo trì trong tương lai. 