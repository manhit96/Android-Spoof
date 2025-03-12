# Android-Spoof: Chi tiết kỹ thuật

Tài liệu này cung cấp thông tin kỹ thuật toàn diện về dự án Android-Spoof. Nó bao gồm phân tích chi tiết các dự án hiện có, phương pháp tích hợp ROM, kiến trúc đề xuất và lộ trình triển khai.

## Mục lục
- [Phân tích các dự án hiện có](#phân-tích-các-dự-án-hiện-có)
- [Phương pháp tích hợp vào ROM](#phương-pháp-tích-hợp-vào-rom)
- [Kiến trúc đề xuất](#kiến-trúc-đề-xuất)
- [Giải pháp cho các thách thức chính](#giải-pháp-cho-các-thách-thức-chính)
- [Lộ trình triển khai](#lộ-trình-triển-khai)

## Phân tích các dự án hiện có

Chúng tôi đã phân tích sâu nhiều dự án hiện có để học hỏi các kỹ thuật và phương pháp tiếp cận tốt nhất.

### Phát hiện chính từ phân tích

1. **XPL-EX**: Cung cấp giải pháp toàn diện để giả mạo hầu hết thông tin thiết bị Android, nhưng dựa vào hook Xposed
   - Kiến trúc module hóa với các interceptor và randomizer
   - Khả năng giả mạo thông tin build, mạng, thiết bị đầu vào, SIM/IMEI

2. **PlayIntegrityFix và Hệ sinh thái liên quan**: Tập trung vào việc giả mạo tình trạng bootloader và vượt qua Google Play Integrity
   - Thay đổi thuộc tính hệ thống liên quan đến bootloader và tính bảo mật
   - Hook vào API để thay đổi chữ ký và provider

3. **TrickyStore**: Bổ sung khả năng giả mạo key attestation để đạt MEETS_STRONG_INTEGRITY
   - Sửa đổi chuỗi chứng chỉ từ KeyStore Android
   - Sử dụng keybox.xml chưa bị thu hồi
   - Hai phương pháp hoạt động khác nhau:
     - **Leaf Hack Mode**: Sửa đổi chứng chỉ gốc, phù hợp với hầu hết thiết bị
     - **Certificate Generating Mode**: Tạo mới toàn bộ chuỗi chứng chỉ, dành cho thiết bị có TEE bị hỏng

4. **BootloaderSpoofer**: Module Xposed bổ sung giúp giả mạo trạng thái bootloader trong chứng chỉ attestation
   - Tập trung vào giả mạo trạng thái bootloader trong quá trình attestation
   - Hoạt động ở cấp độ ứng dụng, can thiệp vào quá trình attestation cục bộ

5. **playcurlNEXT**: Module bổ sung tự động cập nhật fingerprint
   - Tự động cập nhật thông tin thiết bị từ thiết bị Pixel mới nhất
   - Cấu hình thời gian cập nhật thông qua file minutes.txt

Để tìm hiểu thêm phân tích chi tiết của các dự án này, xem:
- [Phân tích toàn diện về giả mạo thiết bị Android](android_device_spoofing_comprehensive_analysis_vi.md)
- [Phân tích XPL-EX](XPL-EX_Analysis_vi.md)
- [Phân tích PlayIntegrityFix](PlayIntegrityFix_Analysis_vi.md)

## Phương pháp tích hợp vào ROM

Dự án đề xuất hai phương pháp tích hợp chính: tích hợp vào LineageOS và tích hợp trực tiếp vào ROM gốc thông qua reverse engineering.

### Tích hợp vào LineageOS

Phương pháp này tận dụng mã nguồn mở của LineageOS để tích hợp trực tiếp giải pháp spoofing vào hệ thống:

1. Sửa đổi các lớp hệ thống chính trong framework Android
2. Tạo service framework quản lý thông tin giả mạo tập trung
3. Sửa đổi KeyStore để hỗ trợ giả mạo key attestation
4. Tạo ứng dụng quản lý cấu hình spoofing

Ví dụ sửa đổi android.os.Build:
```java
// Ví dụ sửa đổi android.os.Build
public class Build {
    private static final DeviceSpoofService sDeviceSpoofService = 
            DeviceSpoofServiceManager.getService();
    
    public static final String BOARD = sDeviceSpoofService.getProperty("ro.product.board");
    public static final String BRAND = sDeviceSpoofService.getProperty("ro.product.brand");
    public static final String MODEL = sDeviceSpoofService.getProperty("ro.product.model");
    // ... các trường khác
}
```

### Phương pháp tích hợp trực tiếp vào ROM gốc

Phương pháp này sử dụng kỹ thuật reverse engineering để tích hợp vào ROM gốc của nhà sản xuất:

1. Trích xuất và phân tích cấu trúc ROM
2. Decompile và sửa đổi các thành phần hệ thống
3. Tạo các component mới và recompile
4. Đóng gói và cài đặt thông qua các phương pháp như Magisk Module

#### Công cụ cần thiết:
- ApkTool: Giải nén và đóng gói APK
- Dex2jar: Chuyển đổi file .dex sang .jar
- JADX: Decompile APK thành mã Java
- Smali/Baksmali: Sửa đổi mã bytecode Dalvik
- Magisk/KernelSU: Tích hợp thay đổi ở cấp hệ thống

#### Phương pháp "Hybrid Integration"
Kết hợp cả reverse engineering và module hóa:
1. **Tạo APK hệ thống** chứa logic spoofing
2. **Sửa đổi nhỏ framework** để chuyển hướng cuộc gọi
3. **Tạo boot script** đảm bảo service khởi động sớm
4. **Xây dựng bridge API** để truy cập từ các framework khác

## Kiến trúc đề xuất

Kiến trúc đề xuất là một hệ thống module hóa với các thành phần chuyên biệt, tích hợp sâu vào hệ thống Android nhưng vẫn dễ bảo trì và mở rộng.

### 1. Thiết kế tổng thể hệ thống

Hệ thống modular bao gồm:

1. **Core Service Framework**: Quản lý và cung cấp thông tin giả mạo nhất quán
2. **System Properties Modifier**: Sửa đổi các lớp hệ thống chính và chuyển hướng API
3. **Key Attestation Provider**: Tích hợp giải pháp giả mạo attestation
4. **Device Spoof Manager**: Ứng dụng giao diện người dùng quản lý cấu hình

Cấu trúc thư mục đề xuất:
```
LineageOS/
├── frameworks/
│   ├── base/
│   │   ├── core/java/android/
│   │   │   ├── os/
│   │   │   │   ├── Build.java (Đã sửa đổi)
│   │   │   │   └── SystemClock.java (Đã sửa đổi)
│   │   │   ├── net/
│   │   │   │   └── wifi/
│   │   │   │       └── ScanResult.java (Đã sửa đổi)
│   │   │   └── ...
│   │   └── services/
│   │       └── java/com/android/server/
│   │           └── devicespoof/
│   │               ├── DeviceSpoofService.java
│   │               ├── DeviceInfoManager.java
│   │               └── ...
│   └── spoofing/
│       ├── api/
│       │   └── ...
│       └── core/
│           └── java/com/android/spoofing/
│               ├── profiles/
│               ├── randomizers/
│               ├── keybox/
│               └── security/
├── packages/
│   └── apps/
│       └── DeviceSpoofManager/
└── system/
    └── etc/
        └── security/
            ├── default_keybox.xml
            └── keybox_rotation/
```

### 2. Quản lý cấu hình thông qua profiles

Hệ thống sử dụng profiles thiết bị dưới dạng JSON để quản lý tất cả thông tin:

```json
{
  "profile_name": "Pixel 6",
  "device": {
    "brand": "google",
    "model": "Pixel 6",
    "manufacturer": "Google",
    "fingerprint": "google/oriole/oriole:12/SQ1D.220205.003/8069835:user/release-keys",
    "security_patch": "2022-02-05"
  },
  "network": {
    "wifi_mac": "randomize",
    "bluetooth_mac": "randomize",
    "wifi_scan_results": {
      "network_count": "random(3,8)",
      "include_common_networks": true
    }
  },
  "telephony": {
    "imei": "randomize",
    "imsi": "randomize",
    "mcc": "310",
    "mnc": "260"
  },
  "keystore": {
    "keybox_path": "/system/etc/security/default_keybox.xml",
    "attestation_mode": "auto",
    "rotate_keybox": true,
    "rotation_interval_days": 7
  },
  "security": {
    "patch_level": {
      "system": "latest_pixel",
      "boot": "latest_pixel",
      "vendor": "latest_pixel"
    },
    "selinux_mode": "enforcing"
  }
}
```

## Giải pháp cho các thách thức chính

1. **Đảm bảo tính nhất quán của thông tin giả mạo**:
   - Lưu trữ tập trung thông tin trong DeviceSpoofService
   - Sử dụng profile thiết bị để quản lý tất cả thông tin liên quan

2. **Vượt qua cơ chế phát hiện root và sửa đổi**:
   - Tích hợp trực tiếp vào mã nguồn thay vì hook
   - Tích hợp giải pháp key attestation và bootloader spoofing
   - Quản lý SELinux và xử lý DroidGuard

3. **Quản lý Keybox và Cập nhật**:
   - Cung cấp keybox mặc định và hỗ trợ xoay vòng
   - Tự động cập nhật thông tin từ thiết bị Pixel mới nhất

## Lộ trình triển khai

Dự án được chia thành 5 giai đoạn phát triển rõ ràng, từ nghiên cứu đến triển khai và tối ưu hóa.

1. **Giai đoạn 1: Nghiên cứu và Prototype**
   - Phân tích sâu hơn mã nguồn LineageOS
   - Tạo POC cho các module chính

2. **Giai đoạn 2: Phát triển Core Services**
   - Phát triển DeviceSpoofService
   - Xây dựng cơ chế profile và cấu hình

3. **Giai đoạn 3: Sửa đổi API Hệ thống**
   - Sửa đổi các lớp trong framework Android
   - Triển khai KeyStore và key attestation spoofer

4. **Giai đoạn 4: Phát triển Ứng dụng Quản lý**
   - Xây dựng giao diện quản lý
   - Tạo cơ chế nhập/xuất profile

5. **Giai đoạn 5: Kiểm thử và Tối ưu**
   - Kiểm thử với các ứng dụng phổ biến
   - Tối ưu hiệu suất 