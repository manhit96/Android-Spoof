> **Lưu ý quan trọng**: Tài liệu này tập trung vào phân tích chi tiết về PlayIntegrityFix và hệ sinh thái liên quan. Để có cái nhìn tổng quan về toàn bộ các giải pháp giả mạo thông tin thiết bị Android, vui lòng tham khảo thêm tài liệu "[Phân tích các dự án mã nguồn mở về giả mạo thông tin thiết bị Android](android_device_spoofing_comprehensive_analysis.md)". Tài liệu tổng quan cung cấp thông tin về nhiều dự án khác nhau, các phương pháp thay thế, và xu hướng mới nhất trong lĩnh vực này.

# Phân tích PlayIntegrityFix và Hệ sinh thái liên quan

## Tóm tắt ứng dụng cho dự án Android-Spoof

**Giá trị cốt lõi cho dự án Android-Spoof:**
- Cung cấp cơ chế giả mạo thông tin bootloader và vượt qua xác thực Play Integrity
- Công nghệ giả mạo key attestation để đạt được MEETS_STRONG_INTEGRITY
- Mô hình kiến trúc module hóa có thể áp dụng cho dự án
- Cơ chế cập nhật tự động thông tin thiết bị từ thiết bị Pixel mới nhất

**Các thành phần chính cần tích hợp:**
1. Giả mạo thông tin bootloader và thuộc tính hệ thống 
2. Giả mạo chữ ký package và key attestation
3. Cơ chế tự động cập nhật fingerprint
4. Quản lý keybox và security patch level

## Tổng quan

PlayIntegrityFix là một module Zygisk được thiết kế để vượt qua các cơ chế kiểm tra tính toàn vẹn của Google Play Integrity và SafetyNet trên thiết bị Android đã root. Module này đặc biệt quan trọng cho dự án Android-Spoof vì nó giải quyết một trong những mục tiêu chính: "Vượt qua các cơ chế kiểm tra như Google Play Integrity, SafetyNet".

## Mục tiêu của PlayIntegrityFix

- Giả mạo trạng thái bootloader (để báo cáo là "locked")
- Đạt được các mức độ xác thực:
  - MEETS_BASIC_INTEGRITY ✅
  - MEETS_DEVICE_INTEGRITY ✅
  - MEETS_STRONG_INTEGRITY ❌ (cần thêm keybox từ TrickyStore)
- Trong SafetyNet:
  - basicIntegrity: true
  - ctsProfileMatch: true
  - evaluationType: BASIC

## Hệ sinh thái PlayIntegrityFix

PlayIntegrityFix hoạt động cùng với các module khác để đạt được mức độ xác thực toàn diện nhất:

1. **PlayIntegrityFix (PIF)**: Module cốt lõi giúp đạt MEETS_BASIC_INTEGRITY và MEETS_DEVICE_INTEGRITY
2. **TrickyStore**: Module bổ sung giúp đạt MEETS_STRONG_INTEGRITY
3. **Tricky-Addon-Update-Target-List**: Công cụ quản lý cấu hình TrickyStore
4. **playcurlNEXT**: Bổ sung cho PIF, tự động cập nhật fingerprint 
5. **BootloaderSpoofer**: Bổ sung cho PIF, giả mạo trạng thái bootloader locked

### 1. PlayIntegrityFix (PIF)

PlayIntegrityFix là thành phần cốt lõi của hệ sinh thái, được phát triển bởi chiteroman. Đây là một module Zygisk nhằm vá kết quả SafetyNet/Play Integrity API, giúp thiết bị qua được kiểm tra attestation ở mức BASIC và DEVICE.

#### Cấu trúc dự án

Dự án PlayIntegrityFix có cấu trúc như sau:

- **module/**: Chứa các script và file cấu hình của module Magisk/KernelSU
  - `action.sh`: Script tự động cập nhật fingerprint
  - `common_func.sh`: Các hàm tiện ích chung
  - `customize.sh`: Script cài đặt module
  - `module.prop`: Thông tin module
  - `pif.json`: File cấu hình thông tin giả mạo
  - `post-fs-data.sh`: Script chạy sau khi filesystem được mount
  - `service.sh`: Script chạy khi hệ thống khởi động
  - `sepolicy.rule`: Quy tắc SELinux
  - `uninstall.sh`: Script gỡ cài đặt

- **app/src/main/java/es/chiteroman/playintegrityfix/**: Mã nguồn Java
  - `EntryPoint.java`: Điểm vào chính của module
  - `CustomKeyStoreSpi.java`: Lớp giả mạo KeyStore
  - `CustomPackageInfoCreator.java`: Lớp giả mạo thông tin package
  - `CustomProvider.java`: Lớp giả mạo provider

- **app/src/main/cpp/**: Mã nguồn C++
  - `main.cpp`: Triển khai module Zygisk
  - `zygisk.hpp`: API Zygisk
  - `json.hpp`: Thư viện xử lý JSON

#### Cơ chế hoạt động

PlayIntegrityFix hoạt động thông qua các cơ chế sau:

1. **Giả mạo thông tin thiết bị**: Module sử dụng file `pif.json` để cấu hình thông tin giả mạo như:
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

2. **Hook vào quá trình khởi tạo ứng dụng**: Sử dụng framework Zygisk để can thiệp vào quá trình khởi tạo của các ứng dụng Google như:
   - `com.google.android.gms` (Google Play Services)
   - `com.android.vending` (Google Play Store)
   - `com.google.android.gms.unstable` (DroidGuard)

3. **Giả mạo chữ ký và provider**: Module thay thế chữ ký package bằng chữ ký Google hợp lệ và hook vào AndroidKeyStore provider để vượt qua xác thực.

4. **Sửa đổi thuộc tính hệ thống**: Thông qua `post-fs-data.sh` và `service.sh`, module thay đổi nhiều thuộc tính hệ thống:
```
# Bootloader và trạng thái xác minh
ro.boot.flash.locked 1
ro.boot.verifiedbootstate green
ro.boot.veritymode enforcing
ro.boot.vbmeta.device_state locked
vendor.boot.verifiedbootstate green
vendor.boot.vbmeta.device_state locked

# Các thuộc tính liên quan đến bảo mật
ro.boot.warranty_bit 0
ro.vendor.boot.warranty_bit 0
ro.vendor.warranty_bit 0
ro.warranty_bit 0
ro.debuggable 0
ro.secure 1
ro.adb.secure 1
sys.oem_unlock_allowed 0
```

5. **Hook vào system_property_read_callback**: Module hook vào hàm đọc thuộc tính hệ thống để điều chỉnh các giá trị được trả về.

### 2. TrickyStore

TrickyStore là module Zygisk bổ sung cho PlayIntegrityFix, tập trung vào việc sửa đổi chuỗi chứng chỉ được tạo ra cho key attestation của Android. Đây là một phương pháp nâng cao để đạt mức MEETS_STRONG_INTEGRITY.

#### Cách TrickyStore hoạt động

1. **Sửa đổi chuỗi chứng chỉ**: TrickyStore sửa đổi chuỗi chứng chỉ được tạo ra bởi KeyStore Android để vượt qua xác thực mạnh.

2. **Sử dụng keybox.xml**: Module yêu cầu file keybox.xml hợp lệ (không bị thu hồi) được đặt tại `/data/adb/tricky_store/keybox.xml`.

3. **Tùy chỉnh ứng dụng mục tiêu**: Người dùng có thể chỉ định các ứng dụng cần áp dụng trick thông qua file `/data/adb/tricky_store/target.txt`.

4. **Hai phương pháp hoạt động**:
   - **Leaf Hack Mode**: Hack chứng chỉ gốc (mặc định)
   - **Certificate Generating Mode**: Tạo mới chứng chỉ (cho thiết bị có TEE bị hỏng)

5. **Tùy chỉnh security patch level**: Có thể tùy chỉnh mức bản vá bảo mật thông qua file `/data/adb/tricky_store/security_patch.txt`.

#### Định dạng keybox.xml

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

Đây là một module phụ trợ để cấu hình TrickyStore thông qua giao diện Web UI của KernelSU.

#### Tính năng

- Cấu hình target.txt với tên hiển thị của ứng dụng
- Chọn chế độ `!` hoặc `?` cho từng ứng dụng
- Chọn ứng dụng từ Magisk DenyList
- Bỏ chọn các ứng dụng không cần thiết
- Thiết lập verifiedBootHash
- Tự động cấu hình security patch
- Cung cấp AOSP Keybox hoặc nhập Keybox tùy chỉnh

### 4. playcurlNEXT

playcurlNEXT là một module bổ sung cho PlayIntegrityFix, được phát triển bởi daboynb. Module này tự động cập nhật fingerprint từ thiết bị Pixel mới nhất để đảm bảo PlayIntegrityFix luôn có thông tin fingerprint hợp lệ.

#### Cấu trúc dự án

Dự án playcurlNEXT có cấu trúc đơn giản hơn:

- `service.sh`: Script chính, chạy khi hệ thống khởi động
- `minutes.txt`: File cấu hình khoảng thời gian cập nhật
- `module.prop`: Thông tin module
- `customize.sh`: Script cài đặt module
- `update.json`: Thông tin cập nhật module

#### Cơ chế hoạt động

playcurlNEXT hoạt động như sau:

1. **Kiểm tra môi trường**: Script kiểm tra xem PlayIntegrityFix đã được cài đặt chưa và tìm đường dẫn đến busybox.

2. **Sao chép và thiết lập script cron**: Script sao chép thư mục PlayIntegrityFix vào thư mục tạm thời và chuẩn bị script `action.sh` để chạy định kỳ.

3. **Đọc cấu hình thời gian**: Script đọc giá trị từ file `minutes.txt` để xác định khoảng thời gian cập nhật (mặc định là 60 phút).

4. **Thiết lập cron job**: Script thiết lập một cron job để chạy `action.sh` theo khoảng thời gian đã cấu hình.

5. **Khởi tạo và chạy script**: Script chạy `action.sh` một lần khi khởi động và sau đó chạy định kỳ theo cron job.

### 5. BootloaderSpoofer

BootloaderSpoofer là một module Xposed được phát triển bởi chiteroman, nhằm giả mạo trạng thái bootloader locked trong các chứng chỉ attestation. Module này hoạt động ở cấp độ ứng dụng, can thiệp vào quá trình local attestation khi một ứng dụng yêu cầu chứng chỉ từ KeyStore/TEE.

#### Cấu trúc dự án

Dự án BootloaderSpoofer chủ yếu bao gồm một file Java chính:

- `app/src/main/java/es/chiteroman/bootloaderspoofer/Xposed.java`: Mã nguồn chính của module

#### Cơ chế hoạt động

BootloaderSpoofer hoạt động thông qua các cơ chế sau:

1. **Khởi tạo KeyPair và Certificate**: Module khởi tạo các cặp khóa EC và RSA, cùng với các chứng chỉ tương ứng.

2. **Hook vào quá trình attestation**: Module hook vào các phương thức liên quan đến attestation trong KeyStore để can thiệp vào quá trình tạo chứng chỉ.

3. **Sửa đổi Extension trong chứng chỉ**: Module sửa đổi các extension trong chứng chỉ để xóa dấu hiệu bootloader mở khóa.

4. **Tạo chứng chỉ mới hoặc hack chứng chỉ hiện có**: Module có thể tạo một chứng chỉ mới hoàn toàn hoặc sửa đổi chứng chỉ hiện có để đánh lừa ứng dụng nghĩ rằng bootloader đang locked.

5. **Áp dụng cho từng ứng dụng cụ thể**: Module cho phép chọn cụ thể ứng dụng cần hook, tránh ảnh hưởng đến toàn bộ hệ thống.

## Đạt mức MEETS_STRONG_INTEGRITY

Để đạt mức MEETS_STRONG_INTEGRITY, cần phải tuân theo các bước sau:

1. **Cài đặt cả PlayIntegrityFix và TrickyStore**:
   - PlayIntegrityFix giải quyết vấn đề cơ bản về bootloader và tính toàn vẹn thiết bị
   - TrickyStore cung cấp cơ chế giả mạo key attestation để đạt STRONG_INTEGRITY

2. **Sử dụng keybox hợp lệ**:
   - Tìm kiếm keybox chưa bị thu hồi (đây là phần khó nhất)
   - Đặt keybox vào `/data/adb/tricky_store/keybox.xml`

3. **Cấu hình TrickyStore**:
   - Sử dụng Tricky-Addon-Update-Target-List để cấu hình dễ dàng hơn
   - Thêm các ứng dụng cần xác thực vào target.txt
   - Cấu hình security patch để phù hợp

4. **Lưu ý về thời gian hiệu lực**:
   - Các keybox thường bị Google phát hiện và thu hồi nhanh chóng
   - Khi keybox bị thu hồi, bạn sẽ chỉ đạt MEETS_DEVICE_INTEGRITY
   - Thường xuyên cập nhật hoặc tìm kiếm keybox mới

## Tích hợp giữa các thành phần

Các thành phần trong hệ sinh thái PlayIntegrityFix hoạt động cùng nhau như sau:

1. **PlayIntegrityFix (PIF)**: Thành phần cốt lõi, giúp đạt MEETS_BASIC_INTEGRITY và MEETS_DEVICE_INTEGRITY bằng cách giả mạo thông tin thiết bị và hook vào quá trình attestation.

2. **playcurlNEXT**: Bổ sung cho PIF, tự động cập nhật fingerprint để đảm bảo PIF luôn có thông tin fingerprint hợp lệ mới nhất.

3. **BootloaderSpoofer**: Bổ sung cho PIF, giả mạo trạng thái bootloader locked trong các chứng chỉ attestation ở cấp độ ứng dụng.

4. **TrickyStore**: Bổ sung cho PIF, can thiệp vào chuỗi chứng chỉ khóa bảo mật để đạt MEETS_STRONG_INTEGRITY.

5. **Tricky-Addon-Update-Target-List**: Công cụ quản lý để cấu hình TrickyStore dễ dàng hơn.

## Ứng dụng cho dự án Android-Spoof

### Xác định các API cần sửa đổi

- `android.os.Build`: Để thay đổi thông tin thiết bị (MANUFACTURER, MODEL, v.v.)
- `android.security.keystore`: Để giả mạo KeyStore và các chứng chỉ
- `android.content.pm.PackageManager`: Để thay đổi thông tin chữ ký package

### Các kỹ thuật cần tích hợp vào LineageOS

1. **Giả mạo thông tin thiết bị**: Tích hợp kỹ thuật giả mạo thông tin thiết bị từ PlayIntegrityFix vào framework Android.

2. **Tự động cập nhật fingerprint**: Tích hợp cơ chế tự động cập nhật fingerprint từ playcurlNEXT vào hệ thống.

3. **Giả mạo trạng thái bootloader**: Tích hợp kỹ thuật giả mạo trạng thái bootloader từ BootloaderSpoofer vào framework Android.

4. **Giả mạo chuỗi chứng chỉ khóa bảo mật**: Tích hợp kỹ thuật giả mạo chuỗi chứng chỉ từ TrickyStore vào framework Android.

### Kiến trúc tích hợp đề xuất

1. **Thay vì sử dụng hook Zygisk**:
   - Sửa đổi trực tiếp các lớp trong framework Android
   - Thêm các service hệ thống mới để quản lý thông tin giả mạo
   - Tạo cơ chế lưu trữ và quản lý thông tin giả mạo nhất quán

2. **Phân tách chức năng**:
   - Mỗi thành phần có một nhiệm vụ cụ thể, giúp dễ dàng bảo trì và cập nhật
   - Thiết kế các service chuyên biệt cho từng loại thông tin cần giả mạo

3. **Tự động hóa**:
   - Tích hợp cơ chế tự động cập nhật thông tin thiết bị mới
   - Giảm sự can thiệp của người dùng

4. **Linh hoạt trong cấu hình**:
   - Cung cấp các tùy chọn cấu hình linh hoạt
   - Cho phép người dùng điều chỉnh theo nhu cầu

### Các thách thức cần giải quyết

- Đảm bảo tính nhất quán giữa các API
- Vượt qua cơ chế phát hiện root
- Duy trì cập nhật theo thay đổi của Google Play Integrity
- Quản lý và cập nhật keybox hiệu quả
- Đảm bảo tất cả các thành phần hoạt động nhất quán
- Xây dựng cơ chế cập nhật liên tục

## Kết luận

Hệ sinh thái PlayIntegrityFix cung cấp một giải pháp toàn diện để vượt qua các cơ chế kiểm tra tính toàn vẹn của Google Play Integrity và SafetyNet. Bằng cách tích hợp các kỹ thuật từ PlayIntegrityFix, TrickyStore, Tricky-Addon-Update-Target-List, playcurlNEXT và BootloaderSpoofer vào dự án Android-Spoof, chúng ta có thể xây dựng một giải pháp giả mạo thiết bị toàn diện, mạnh mẽ và bền vững hơn, có khả năng vượt qua mức xác thực cao nhất của Google Play Integrity.

Thông qua phân tích sâu các thành phần này, chúng ta có thể tạo ra một giải pháp tích hợp trực tiếp vào LineageOS, đảm bảo hiệu suất, ổn định và khả năng chống phát hiện cao hơn so với các phương pháp dựa trên hook hiện tại. 