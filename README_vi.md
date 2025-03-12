# Android-Spoof

Giải pháp toàn diện giả mạo thông tin thiết bị Android tích hợp trực tiếp vào mã nguồn LineageOS thay vì sử dụng các hook như Xposed.

[English](README.md) | [Tiếng Việt](README_vi.md)

## Tổng quan

Android-Spoof phát triển một giải pháp để giả mạo thông tin thiết bị Android thông qua việc tích hợp trực tiếp vào mã nguồn LineageOS, tạo ra một hệ thống ổn định hơn và nhất quán hơn so với các giải pháp dựa trên hook hiện có.

## Mục tiêu

- Phát triển giải pháp giả mạo thông tin thiết bị Android toàn diện
- Tích hợp trực tiếp vào mã nguồn LineageOS thay vì sử dụng hook (Xposed)
- Tạo hệ thống có khả năng giả mạo hoàn toàn một thiết bị mới (brand, model, IMEI, MAC, v.v.)
- Đảm bảo tính nhất quán của tất cả thông tin để tạo fingerprint hợp lệ
- Vượt qua các cơ chế kiểm tra như Google Play Integrity và SafetyNet
- Nghiên cứu các cơ chế bảo mật và phương pháp xác minh tính toàn vẹn hệ thống

## Trạng thái dự án

Dự án hiện đang trong **giai đoạn nghiên cứu**. Chúng tôi đang tập trung vào phân tích các dự án hiện có và nghiên cứu mã nguồn LineageOS để xác định cách tích hợp tối ưu.

## Cấu trúc dự án

```
Android-Spoof/
├── docs/                     # Tài liệu phân tích và thiết kế
├── lineageos_integration/    # Mã nguồn tích hợp LineageOS (sẽ phát triển)
└── memory-bank/              # Tài liệu theo dõi dự án
```

## Tài liệu dự án

Để hiểu và đóng góp vào dự án này, chúng tôi khuyến nghị làm quen với tài liệu của chúng tôi theo thứ tự sau:

### Tài liệu Memory Bank (Thứ tự đọc được khuyến nghị)
Memory Bank chứa thông tin và ngữ cảnh dự án cập nhật nhất:

1. [Tóm tắt dự án](memory-bank/projectbrief.md) - Tổng quan và mục tiêu cốt lõi của dự án
2. [Ngữ cảnh sản phẩm](memory-bank/productContext.md) - Mô tả vấn đề và tầm nhìn trải nghiệm người dùng
3. [Mô hình hệ thống](memory-bank/systemPatterns.md) - Kiến trúc hệ thống và các thành phần chính
4. [Ngữ cảnh kỹ thuật](memory-bank/techContext.md) - Chi tiết kỹ thuật và môi trường phát triển
5. [Ngữ cảnh hiện tại](memory-bank/activeContext.md) - Trọng tâm công việc hiện tại và các quyết định đang diễn ra
6. [Tiến độ](memory-bank/progress.md) - Trạng thái tiến độ dự án và các mốc sắp tới

### Tài liệu phân tích kỹ thuật

Thư mục `docs/` chứa các phân tích chuyên sâu về các dự án và công nghệ liên quan:

- [Phân tích toàn diện về giả mạo thiết bị Android](docs/android_device_spoofing_comprehensive_analysis_vi.md) - Tổng quan về công nghệ giả mạo thiết bị
- [Phân tích XPL-EX](docs/XPL-EX_Analysis_vi.md) - Phân tích kiến trúc và phương pháp của XPL-EX
- [Phân tích PlayIntegrityFix](docs/PlayIntegrityFix_Analysis_vi.md) - Phân tích cách tiếp cận của PlayIntegrityFix
- [Chi tiết kỹ thuật](docs/technical_details_vi.md) - Thông tin kỹ thuật chi tiết về cách triển khai của chúng tôi

### Cách sử dụng tài liệu này

1. **Người đóng góp mới**: Bắt đầu với Memory Bank, đặc biệt là Tóm tắt dự án và Ngữ cảnh hiện tại
2. **Hiểu biết kỹ thuật**: Xem xét các tài liệu phân tích để hiểu cơ chế cơ bản
3. **Công việc hiện tại**: Kiểm tra tài liệu Ngữ cảnh hiện tại và Tiến độ để biết trạng thái phát triển mới nhất
4. **Chi tiết triển khai**: Chi tiết kỹ thuật cung cấp các cách tiếp cận triển khai cụ thể

## Cộng đồng & Đóng góp

Chúng tôi chào đón đóng góp từ cộng đồng! Dự án này nhằm mục đích thúc đẩy:

1. **Nhận thức về quyền riêng tư**: Giúp người dùng hiểu và kiểm soát thông tin mà thiết bị của họ chia sẻ
2. **Mục đích giáo dục**: Tìm hiểu về hệ thống nội bộ Android và cơ chế bảo mật
3. **Nghiên cứu và phát triển**: Nâng cao hiểu biết của chúng ta về bảo mật và quyền riêng tư trên thiết bị di động

### Cách đóng góp

- Fork repository và gửi pull request
- Báo cáo lỗi và đề xuất tính năng thông qua issues
- Cải thiện tài liệu và chia sẻ cách triển khai của bạn
- Tham gia thảo luận về quyền riêng tư và bảo mật thiết bị

### Trường hợp sử dụng hợp pháp

Dự án tập trung vào các trường hợp sử dụng hợp pháp sau:
- Bảo vệ quyền riêng tư và ngăn chặn theo dõi thiết bị
- Nghiên cứu về cơ chế bảo mật Android
- Kiểm tra tính tương thích của ứng dụng trên các cấu hình thiết bị khác nhau
- Mục đích giáo dục để hiểu hoạt động cấp hệ thống của Android

## Tham khảo

Dự án này tham khảo và phân tích các dự án sau:
- XPL-EX
- PlayIntegrityFix
- TrickyStore

## Giấy phép

Dự án này được cấp phép theo Giấy phép MIT - xem tệp [LICENSE](LICENSE) để biết chi tiết.

Giấy phép MIT có nghĩa là bạn có thể tự do sử dụng, sửa đổi, phân phối và bán các sản phẩm của riêng bạn có tích hợp mã của chúng tôi, miễn là bạn bao gồm thông báo bản quyền và giấy phép gốc trong bất kỳ bản sao nào của phần mềm/mã nguồn. 