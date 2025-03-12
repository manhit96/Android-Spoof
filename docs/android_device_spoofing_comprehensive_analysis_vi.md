================================================================================
PHÂN TÍCH CÁC DỰ ÁN MÃ NGUỒN MỞ VỀ GIẢ MẠO THÔNG TIN THIẾT BỊ ANDROID
================================================================================

================================================================================
MỤC LỤC
================================================================================

1. GIỚI THIỆU

2. PHÂN TÍCH CÁC DỰ ÁN TIÊU BIỂU
   2.1 XPL-EX – Framework hook thông tin thiết bị (Xposed)
   2.2 PlayIntegrityFix (PIF) – Bypass Google Play Integrity & SafetyNet (module Zygisk)
   2.3 TrickyStore – Giả lập chuỗi chứng chỉ key attestation (Magisk/KernelSU module)
   2.4 Tricky Addon – Update Target List
   2.5 BootloaderSpoofer

3. MODULE MAGISK/KERNELSU LIÊN QUAN ĐẾN GIẢ MẠO THÔNG TIN THIẾT BỊ

4. CHỈNH SỬA TRỰC TIẾP MÃ NGUỒN ANDROID (KHÔNG DÙNG HOOK)

5. GIẢ MẠO KEY ATTESTATION & THÔNG TIN HARDWARE-BACKED KEYSTORE

6. BYPASS SELINUX VÀ NÉ TRÁNH DROIDGUARD

7. TÍCH HỢP GIẢ MẠO VÀO ROM TÙY CHỈNH

8. PHƯƠNG PHÁP REVERSE ENGINEERING ROM ĐỂ THAY ĐỔI THÔNG TIN HỆ THỐNG

9. CÔNG CỤ PHÁT HIỆN VÀ PHÂN TÍCH CƠ CHẾ KIỂM TRA THIẾT BỊ CỦA GOOGLE (2023–2024)

NGUỒN THAM KHẢO CHI TIẾT

KẾT LUẬN

1. GIỚI THIỆU
--------------------------------------------------------------------------------
Các ứng dụng kiểm tra tính toàn vẹn thiết bị Android (SafetyNet, Play Integrity API)
đang ngày càng được Google cải tiến để phát hiện thiết bị đã được root hay chỉnh sửa.
Cộng đồng mã nguồn mở đã phát triển nhiều dự án spoofing nhằm "giả mạo" thông tin thiết bị,
ẩn dấu hiệu root/hook và qua mặt các kiểm tra này. Tài liệu này tổng hợp phân tích các dự án
tiêu biểu (XPL-EX, PlayIntegrityFix, TrickyStore, Tricky-Addon-Update-Target-List, v.v.) và
các giải pháp mới nhất (2023-2024) từ các nguồn như XDA-Developers, GitHub, cũng như các
phương pháp chỉnh sửa ROM, reverse engineering nhằm thay đổi thông tin hệ thống.

================================================================================
2. PHÂN TÍCH CÁC DỰ ÁN TIÊU BIỂU

2.1 XPL-EX – Framework hook thông tin thiết bị (Xposed)
--------------------------------------------------------------------------------
- URL dự án: GitHub – 0bbedCode/XPL-EX

- Mô tả: XPL-EX là một framework hook Xposed mã nguồn mở tập trung vào quyền riêng tư. Ứng dụng này chặn các app khác truy xuất thông tin thật của thiết bị, thay vào đó cung cấp dữ liệu giả (ví dụ: báo cáo RAM 900GB thay vì 8GB, giả mạo model thành iPhone hoặc Pixel). XPL-EX hỗ trợ Android 6.0+ và cho phép người dùng tự viết script Lua để tùy biến hook nhằm che giấu hầu hết thông tin định danh trên thiết bị.

- Ưu điểm: 
  • Hoàn toàn miễn phí, không quảng cáo, không gửi dữ liệu ra ngoài. 
  • Số lượng hook rất phong phú (nhiều hơn các giải pháp cạnh tranh như XPrivacyLua gốc), bao phủ nhiều loại thông tin (ID thiết bị, số serial, cảm biến, vị trí, v.v.). 
  • Hỗ trợ giả lập linh hoạt thông qua script Lua, phù hợp cho cả mục đích bảo mật lẫn tùy biến/hack ứng dụng. 
  • Được cập nhật tích cực (fork từ XPrivacyLua của M66B nhưng đã thay đổi >50k dòng code) và tương thích tốt với LSPosed/Xposed hiện đại.
  • Cung cấp số lượng hook vượt trội so với các công cụ khác.
  • Miễn phí, không thu thập dữ liệu người dùng.
  • Hỗ trợ đa dạng thông tin từ phần mềm (không can thiệp native code).

- Nhược điểm: 
  • Yêu cầu thiết bị đã root và cài Xposed/LSPosed, do đó có rủi ro bị một số app phát hiện (cần kết hợp với công cụ ẩn Xposed như Hide My Applist). 
  • Việc cấu hình nhiều hook phức tạp có thể khó cho người mới. 
  • Chỉ hook ở mức Java (chưa hỗ trợ native hook) – tuy phần lớn kiểm tra thiết bị đều qua API Java, nhưng vẫn có thể lọt một số ít trường hợp dùng API native đặc thù.
  • Chỉ hoạt động ở mức Java, không hỗ trợ hook native.
  • Yêu cầu môi trường Xposed (LSPosed, v.v.) – cần root.

- Mức độ cập nhật: Dự án đang được duy trì tích cực. Tính đến 03/2024, XPL-EX vẫn cập nhật (dev rất năng động – ví dụ bản 26/03/2024). Có cộng đồng Telegram hỗ trợ.

- Tính năng đặc biệt: 
  • Hook động qua Lua – người dùng có thể tự viết script để can thiệp hành vi app. 
  • Cho phép giả mạo hầu hết mọi thông tin thiết bị (từ thông số phần cứng đến danh tính nhà sản xuất, phiên bản hệ điều hành).
  • Đây là bước phát triển tiếp nối XPrivacyLua, mở rộng đáng kể khả năng (hơn 40k dòng code bổ sung).
  • Dự kiến sẽ hỗ trợ hook ở cấp native trong tương lai để trở thành giải pháp "all-in-one" bảo vệ thiết bị và che giấu root/Xposed toàn diện.
  • Cho phép tùy biến thông tin spoof theo script, giả lập thiết bị thành các thương hiệu khác (ví dụ: iPhone, Pixel).

2.2 PlayIntegrityFix (PIF) – Bypass Google Play Integrity & SafetyNet (module Zygisk)
--------------------------------------------------------------------------------
- URL dự án: GitHub – chiteroman/PlayIntegrityFix (module Magisk/KernelSU)

- Mô tả: PlayIntegrityFix là module Zygisk nhằm vá kết quả SafetyNet/Play Integrity API, giúp thiết bị qua được kiểm tra attestation ở mức BASIC và DEVICE (ctsProfileMatch=true, MEETS_DEVICE_INTEGRITY=true). Module can thiệp vào Google Play Services để giả mạo các giá trị cần thiết (như fingerprint, trạng thái bootloader…) nhưng không ẩn root – người dùng cần kết hợp với công cụ ẩn root khác (Shamiko, Zygisk-Assistant) nếu cần. PIF hỗ trợ cả Magisk và KernelSU (hoặc APatch) miễn là có môi trường Zygisk.

- Ưu điểm: 
  • Giải pháp mới, cập nhật thường xuyên để theo kịp cuộc chiến Play Integrity (ra đời cuối 2023 và vẫn cập nhật đến nay). 
  • Dễ sử dụng: chỉ cần flash module, reboot và kiểm tra kết quả bằng YASNAC hoặc app Integrity Checker. 
  • Không đòi hỏi chỉnh sửa thủ công build.prop như trước. 
  • Kết hợp tốt với KernelSU (có ZygiskNext) nên có thể dùng trên các thiết bị root bằng KernelSU. 
  • Sau khi cài, hầu hết thiết bị có thể "MEETS_DEVICE_INTEGRITY ✅" mà không cần khóa bootloader thật (đủ cho Google Wallet, ngân hàng hoạt động).
  • Cập nhật liên tục (nhiều bản phát hành, mới nhất v18.7).
  • Hỗ trợ Android 8 – 14, tương thích với Magisk và KernelSU.
  • Cài đặt đơn giản, chỉ tác động tới GMS mà không ảnh hưởng hệ thống.

- Nhược điểm: 
  • Chưa đạt Strong Integrity: PIF mặc định không vượt qua được MEETS_STRONG_INTEGRITY (❌) do không can thiệp chứng chỉ phần cứng. 
  • Để đạt strongIntegrity, cần kết hợp thêm TrickyStore và keybox (phức tạp, xem bên dưới). 
  • Ngoài ra, PIF không che giấu dấu vết root hay Xposed – những app có cơ chế phát hiện riêng ngoài Play Integrity vẫn có thể nhận biết thiết bị đã can thiệp (PIF lưu ý sẽ không hỗ trợ case app tự phát hiện root). 
  • Module cũng yêu cầu ROM được sign bằng key riêng (không phải test-keys) mới hoạt động – nghĩa là một số ROM tùy chỉnh dùng test-keys có thể cần patch lại. 
  • Việc Google thường xuyên thay đổi API đòi hỏi PIF phải cập nhật liên tục; nếu Google cập nhật bất ngờ, có thể PIF tạm thời mất hiệu lực cho đến bản vá mới.
  • Không giúp đạt "Strong Integrity" (cần key attestation hợp lệ).
  • Không ẩn dấu hiệu root; cần kết hợp với các công cụ ẩn root khác.

- Mức độ cập nhật: Đang được duy trì rất tích cực. PIF có bản cập nhật thường xuyên (thảo luận trên XDA cho thấy dev cập nhật kịp thời khi Google thay đổi) và có cộng đồng Telegram hơn 20k thành viên. Bản mới nhất (vải cuối 2024) bổ sung các tùy chọn vượt cơ chế mới của Play Integrity (ví dụ spoof Android SDK để lách yêu cầu bootloader locked trên Android 13+). Rất tích cực (cộng đồng đông đảo, cập nhật tới năm 2025).

- Tính năng đặc biệt: 
  • PIF tập trung chuyên biệt vào việc "chứng nhận" thiết bị (device certified) mà không ảnh hưởng hệ thống. 
  • Module không can thiệp global mọi app mà chỉ nhằm vào dịch vụ SafetyNet/Integrity trong Google Play Services, nhờ đó tránh gây lỗi cho các phần khác của hệ thống. 
  • Ngoài ra, PIF cho phép cấu hình nâng cao qua file pif.json (ví dụ spoofVendingSdk để giả Android 12 đối với Play Store nếu cần vượt yêu cầu bootloader locked trên Android 13+). 
  • Đây là giải pháp mã nguồn mở tiếp nối ý tưởng của kdrag0n (SafetyNet Fix) và Displax, với nhiều cải tiến phù hợp bối cảnh Play Integrity mới. 
  • PIF cũng cung cấp hướng dẫn rõ ràng về các verdict (Basic/Device/Strong) và cách kiểm tra trạng thái qua app bên thứ ba.
  • Tích hợp script tự động cập nhật fingerprint, cấu hình JSON linh hoạt để spoof thông tin chỉ cho Google Play Services.

PlayIntegrityFix – các fork và bổ trợ liên quan:
- PlayIntegrityNEXT & playcurl: Ban đầu có fork PlayIntegrityFix NEXT nhằm tự động hóa cập nhật fingerprint. Hiện nay fork này được thay thế bằng module "playcurl" – cài song song PIF chính thức để tự động tải fingerprint hợp lệ từ web khi fingerprint cũ bị Google vô hiệu hóa. Cách làm này giúp người dùng không phải tự đổi fingerprint thủ công hay chờ bản PIF mới. Dự án playcurl (trước là PIF NEXT) cũng mã nguồn mở trên GitHub và có ~1.2k sao, cho thấy cộng đồng quan tâm. Ưu: Tự động cập nhật profile thiết bị, tiện lợi cho người dùng cuối; Nhược: Phụ thuộc vào server bên ngoài để lấy dữ liệu fingerprint (có thể rủi ro nếu server ngừng). Hiện playcurl được khuyến nghị cài kèm PIF chính thức thay vì dùng fork PIF riêng.

- Universal SafetyNet Fix (USNF): Đây là module Magisk do kdrag0n phát triển, tiền thân của PIF, nhằm qua mặt SafetyNet cũ và một phần Play Integrity. USNF can thiệp chặn hardware attestation và cập nhật logic CTS profile. Yêu cầu thiết bị phải pass basic CTS sẵn (phải chỉnh fingerprint phù hợp). Ưu: Rất hữu ích trên Android 8-12, giúp ép Google dùng Basic Attestation thay vì phần cứng. Module này cũng được tích hợp sẵn trong một số ROM (như ProtonAOSP). Nhược: Đã ngừng cập nhật cho Android 14+ (hỗ trợ đến Android 13); không xử lý được các thay đổi mới trong Play Integrity (vì thế cộng đồng chuyển sang PIF). Hiện USNF chủ yếu còn giá trị lịch sử hoặc cho thiết bị cũ, và kdrag0n đã ngưng duy trì (ProtonAOSP cũng dừng phát triển).

- MagiskHide Props Config: Module Magisk (script shell) đã ngừng phát triển nhưng từng phổ biến để chỉnh sửa build.prop (ro.product.) giả mạo thiết bị khác nhằm pass CTS. Ưu: Đơn giản, cho phép chọn fingerprint của model đạt chứng nhận Google (ví dụ Pixel 5) để fix lỗi CTS profile mismatch. Nhược: Không xử lý attestation hardware – chỉ hiệu quả cho SafetyNet Basic cũ. Dự án ngừng vì không theo kịp Play Integrity mới. Hiện người dùng có thể chỉnh props thủ công hoặc dùng các module mới hơn (như Sensitive Props của Pixel-Props team để auto reset các prop nhạy cảm khi boot).

2.3 TrickyStore – Giả lập chuỗi chứng chỉ key attestation (Magisk/KernelSU module)
--------------------------------------------------------------------------------
- URL dự án: GitHub – 5ec1cff/TrickyStore (module Magisk/KernelSU)

- Mô tả: TrickyStore là module can thiệp chuỗi chứng chỉ khóa bảo mật (key attestation). Cụ thể, nó sửa đổi chuỗi certificate được sinh ra khi Android thực hiện hardware attestation, cho phép giả mạo chứng chỉ khoá phần cứng của thiết bị khác. Mục đích là đánh lừa hệ thống SafetyNet/Play Integrity ở mức sâu hơn – ví dụ cung cấp chuỗi chứng chỉ tương ứng với bootloader locked và thiết bị chưa bị revoke. Hỗ trợ Android 10 trở lên. Cài module xong, người dùng có thể thêm file keybox.xml chứa khóa phần cứng (unrevoked key) vào thư mục module để đạt Device Integrity hoặc thậm chí Strong Integrity tùy key. Ngoài ra, TrickyStore cho phép cấu hình target.txt để chọn ứng dụng nào sẽ áp dụng spoof (tránh ảnh hưởng toàn hệ thống).

- Ưu điểm: 
  • Cấp độ bypass sâu: khác với PIF hay Xposed chỉ can thiệp software, TrickyStore can thiệp vào kết quả attest của TEE/HSM. 
  • Khi kết hợp với một keybox hợp lệ (ví dụ lấy từ thiết bị chưa unlock), TrickyStore có thể giúp MEETS_STRONG_INTEGRITY = true, điều mà các giải pháp khác khó đạt. 
  • Module cũng thông minh nhận biết thiết bị TEE hỏng hoặc không trích xuất được cert – nó sẽ chuyển sang chế độ "tự tạo certificate" thay thế (generate mode) để vẫn cung cấp chuỗi attest (dù chỉ đạt deviceIntegrity). 
  • Có thể áp dụng linh hoạt theo app (chỉ app ngân hàng, GMS, v.v.), tránh động chạm các app khác. 
  • Hỗ trợ tùy chỉnh cả security patch level báo cáo trong attest (giả mạo patch level mới hơn nếu cần).
  • Cho phép đạt "Strong Integrity" nếu có keybox chính xác.
  • Hỗ trợ spoof ở tầng hệ thống, khó bị phát hiện hơn.
  • Tùy biến cấu hình cho từng gói ứng dụng.

- Nhược điểm: 
  • Khó triển khai đầy đủ – để đạt strongIntegrity cần có keybox hợp lệ, mà khóa phần cứng rò rỉ rất hiếm và thường nhanh chóng bị Google thu hồi. 
  • Nói cách khác, ít người dùng có keybox tốt để tận dụng tối đa TrickyStore. 
  • Thực tế hầu hết chỉ đạt mức deviceIntegrity với keybox AOSP công khai (Google cung cấp key AOSP cho thiết bị không có TEE). 
  • Ngoài ra, TrickyStore từ phiên bản 1.1.0 đã đóng mã nguồn (closed-source) do dev không hài lòng việc bị lạm dụng và thiếu đóng góp – bản 1.0 open-source cuối cùng vẫn có trên GitHub nhưng các bản mới thì phân phối dạng binary. 
  • Điều này có thể gây e ngại về tính minh bạch ở các bản sau. 
  • Cuối cùng, cài đặt/thiết lập TrickyStore phức tạp hơn (phải hiểu khái niệm keybox, certificate chain). 
  • Không ẩn được root, nên vẫn cần kết hợp Shamiko/denylist nếu app kiểm tra root ngoài attestation.
  • Yêu cầu keybox hợp lệ (thường khó có với người dùng thông thường).
  • Cài đặt phức tạp, rủi ro cao (có thể gây bootloop nếu cấu hình sai).
  • Mã nguồn mở ngừng từ v1.0.x, sau đó chuyển sang phát triển đóng.

- Mức độ cập nhật: Dự án vẫn được duy trì (dù đóng mã nguồn). Dev 5ec1cff rất tích cực – TrickyStore có hàng nghìn sao (2.3k+) và bản mới liên tục. Tuy mã nguồn đóng nhưng cộng đồng vẫn theo dõi qua changelog. Dự kiến module sẽ cần thích ứng với thay đổi Play Integrity (Google đã thông báo tăng yêu cầu cho deviceIntegrity với bootloader trên Android 13+ vào 2025, và các công cụ như TrickyStore sẽ là chìa khóa để lách). Phiên bản mã nguồn mở ngừng từ 2022; addon hỗ trợ tiếp tục phát triển.

- Tính năng đặc biệt: 
  • TrickyStore thực hiện "hack leaf certificate" – tức chỉnh sửa trường mở rộng của certificate do TEE cấp (ví dụ xóa cờ Unlocked trong chứng chỉ) hoặc tạo hẳn certificate mới nếu cần. 
  • Nó cũng có tùy chọn "! / ? mode" cho từng app trong target.txt để ép dùng chế độ generate certificate hoặc leaf hack tùy tình huống. 
  • Điểm độc đáo là khả năng nhúng keybox phần cứng tùy ý: nếu người dùng có file keybox từ OEM (rò rỉ), module sẽ chèn vào quá trình attest như thể đó là khóa của thiết bị. 
  • Đây là hướng đi sáng tạo để đánh lừa cả hệ thống hardware attestation – vốn được coi là "bất khả xâm phạm" nếu không có exploit. 
  • TrickyStore, kết hợp với BootloaderSpoofer (dưới đây), tạo thành bộ đôi mạnh mẽ để qua mặt DroidGuard/TEE ở cả cấp hệ thống và ứng dụng.
  • Độc nhất trong việc giả mạo hardware-backed attestation thông qua keybox, giúp spoof chứng chỉ attestation cấp cao.

2.4 Tricky Addon – Update Target List
--------------------------------------------------------------------------------
- URL dự án: GitHub – KOWX712/Tricky-Addon-Update-Target-List

- Mô tả: Đây là module phụ trợ giao diện WebUI cho TrickyStore, giúp người dùng dễ dàng cấu hình danh sách app target.txt và một số thiết lập spoofing. Thay vì phải sửa file thủ công, module cung cấp giao diện trực quan (trên trình duyệt) để chọn app cần giả mạo và thiết lập các chế độ đặc biệt. Hoạt động trên KernelSU (qua KSU WebUI) và cả Magisk (qua nút mở WebUI trong app Magisk). Addon cho TrickyStore cung cấp giao diện WebUI để cấu hình danh sách các ứng dụng sẽ bị spoof, hỗ trợ thêm tùy chọn ép buộc hay mặc định cho từng app.

- Ưu điểm: 
  • Thân thiện người dùng: liệt kê app theo tên (dễ chọn), cho phép chạm giữ để chọn chế độ ! hoặc ? cho từng app (chuyển đổi giữa hack leaf và generate mode). 
  • Có thể lấy danh sách từ Magisk DenyList sẵn (tiện nếu trước đó đã cấu hình denylist cho app nhạy cảm). 
  • Tính năng lọc app không cần thiết để tránh thêm thừa. 
  • Hỗ trợ cài nhanh keybox AOSP chỉ bằng một nút (dành cho ai không có keybox riêng), và cho phép tải lên keybox tùy chỉnh từ bộ nhớ máy. 
  • Tự động cấu hình giá trị verifiedBootHash và security patch phù hợp trong attest (có thể tùy chỉnh nếu cần). 
  • Nói chung, addon này giúp tự động hóa nhiều bước cấu hình phức tạp của TrickyStore.
  • Giao diện web trực quan, đơn giản hóa việc cấu hình.
  • Tích hợp với KernelSU WebUI và Magisk DenyList.
  • Hỗ trợ import keybox và tùy chọn whitelist cho Shamiko.

- Nhược điểm: 
  • Bản thân addon không mở rộng khả năng spoof mới, chỉ là công cụ hỗ trợ UI. 
  • Nếu người dùng không cài TrickyStore thì module này vô nghĩa. 
  • Cũng không "đảm bảo keybox hợp lệ" – chỉ hỗ trợ nhập, chứ không phép màu tạo keybox mới. 
  • Ngoài ra, cần KernelSU (để có web server) hoặc Magisk + plugin WebUI, nghĩa là môi trường root phức tạp hơn một chút so với thông thường.
  • Là addon bổ sung, không được hỗ trợ chính thức bởi dự án TrickyStore.
  • Cài đặt yêu cầu quyền root, mở cổng WebUI có thể tiềm ẩn rủi ro.
  • Phức tạp với nhiều tùy chọn, không phù hợp với người dùng không chuyên.

- Mức độ cập nhật: Đang được duy trì rất tốt. Addon có ~300 sao, 30 bản phát hành, bản mới nhất v3.6 (tháng 3/2025). Dev KOWX712 cũng đóng góp code cho PIF (action.sh), cho thấy dự án đáng tin cậy và bám sát cập nhật từ TrickyStore/PIF. Cập nhật thường xuyên (đến năm 2024).

- Tính năng đặc biệt: 
  • Tricky-Addon gần như là bảng điều khiển tất-cả-trong-một cho spoofing attestation. 
  • Nó tích hợp cả việc cấu hình Shamiko whitelist (dự kiến, mặc dù hiện tại tạm tắt), và có ý tưởng thêm tính năng định kỳ thêm mọi app vào target (chưa bật) – điều này gợi ý tương lai có thể tự động bảo vệ toàn bộ app nếu người dùng muốn. 
  • Điểm hay là addon hỗ trợ cả Magisk lẫn KernelSU, tận dụng KSU WebUI template – do đó trên ROM không Magisk vẫn có cách cấu hình tiện lợi. 
  • Nó cũng góp phần mang spoofing attestation vào UI người dùng cuối, mở đường cho việc tích hợp sâu hơn vào ROM tùy chỉnh (có thể hình dung ROM tương lai tích hợp sẵn giao diện quản lý key attestation giả mạo, tương tự như addon này làm).
  • Cho phép cấu hình trực quan thông qua WebUI và tự động xử lý danh sách spoof dựa trên Magisk DenyList.

2.5 BootloaderSpoofer
--------------------------------------------------------------------------------
- URL dự án: GitHub – chiteroman/BootloaderSpoofer (module LSPosed)

- Mô tả: BootloaderSpoofer là một module Xposed/LSPosed nhẹ nhằm giả mạo trạng thái bootloader locked trong các chứng chỉ attestation. Nó can thiệp vào quá trình local attestation (khi một ứng dụng trên máy yêu cầu chứng chỉ từ KeyStore/TEE) và sửa trường mở rộng của certificate để xoá dấu hiệu bootloader mở khóa. Mục tiêu chính là những app tự kiểm tra khóa bootloader cục bộ (offline) thay vì dựa vào kết quả server của Google. Nếu app gửi chứng chỉ lên server Google để kiểm tra, BootloaderSpoofer sẽ không giúp được (dev nói thẳng "nếu app gửi certificate lên server bảo mật thì coi như chịu"). Module cho phép chọn cụ thể ứng dụng cần hook (khuyến cáo chỉ chọn app kiểm tra bootloader, không hook vào Google Play Services hoặc framework để tránh tự gây fail SafetyNet/Integrity).

- Ưu điểm: 
  • Đây là giải pháp rất chuyên dụng nhưng hữu ích cho các trường hợp khó: một số ứng dụng ngân hàng, ví điện tử hoặc trò chơi có thể kiểm tra trực tiếp chứng chỉ SafetyNet để xem bootloader có mở không, thay vì dùng API Google. 
  • BootloaderSpoofer sẽ đánh lừa các app này nghĩ rằng bootloader đang locked (bằng cách chỉnh bit trong certificate). 
  • Nhẹ và đơn giản: cài như module LSPosed và chỉ cấu hình app mục tiêu, không đòi hỏi thay đổi hệ thống hay keybox phức tạp. 
  • Tương thích tốt với TrickyStore/PIF – ta có thể dùng BootloaderSpoofer cho vài app đặc thù mà không ảnh hưởng cơ chế toàn cục của PIF. 
  • Dự án mã nguồn mở do cùng tác giả PIF, thường cập nhật song song (bản mới v3.8 phát hành 01/2024).

- Nhược điểm: 
  • Giới hạn phạm vi: chỉ có tác dụng với local TEE attestation. Nếu app hoàn toàn phụ thuộc server Google (đa số trường hợp Play Integrity hiện nay) thì module này không giúp gì. 
  • Thêm nữa, do là Xposed module nên cần môi trường LSPosed (root, Zygisk…), và một số app có thể phát hiện LSPosed. 
  • Tuy nhiên, vì BootloaderSpoofer chỉ hook vào ứng dụng mục tiêu, ta có thể hạn chế nguy cơ bị phát hiện (nhưng vẫn nên dùng Shamiko/Hide My Applist để giấu LSPosed). 
  • Ngoài ra, nếu lạm dụng hook bừa (vd hook cả GMS) sẽ gây fail SafetyNet như cảnh báo của dev – nên người dùng cần thận trọng cấu hình đúng app.

- Mức độ cập nhật: Được duy trì tốt. Chiteroman thường xuyên cập nhật BootloaderSpoofer cùng với PIF. Cộng đồng Xposed repo xác nhận module hoạt động trên Android 13, 14.

- Tính năng đặc biệt: 
  • BootloaderSpoofer tập trung vào certificate extension – đây là phần chứa thông tin như trạng thái khóa bootloader, trạng thái thiết bị (Certified/Unlocked). 
  • Module sẽ chỉnh sửa trực tiếp certificate do TEE trả về trước khi app nhận được, sao cho cờ "unlocked" biến mất. 
  • Về cơ bản, nó bypass cơ chế hardware-backed attestation tại client. 
  • Đây là một miếng ghép quan trọng: kết hợp BootloaderSpoofer (giấu bootloader mở) + TrickyStore (cung cấp key hợp lệ) có thể cho phép qua mặt cả attestation phần cứng lẫn phần mềm. 
  • BootloaderSpoofer cũng minh họa cách dùng Xposed để can thiệp mục tiêu hẹp (chỉ vài app), giúp giảm nguy cơ bị phát hiện so với patch hệ thống toàn cục.

================================================================================
3. MODULE MAGISK/KERNELSU LIÊN QUAN ĐẾN GIẢ MẠO THÔNG TIN THIẾT BỊ
--------------------------------------------------------------------------------
- Universal SafetyNet Fix (kdrag0n):
   • URL dự kiến: https://github.com/kdrag0n/universal-safetynet-fix
   • Chức năng: Can thiệp attestation phần cứng, hỗ trợ spoof thông tin SafetyNet.

- MagiskHide Props Config:
   • URL: https://forum.xda-developers.com/t/tool-magiskhide-props-config.3654247/
   • Chức năng: Cho phép thay đổi các prop như build fingerprint, model… nhằm spoof thông tin thiết bị.

- Shamiko:
   • URL dự kiến: https://github.com/LSPosed/Shamiko
   • Chức năng: Ẩn hoàn toàn dấu hiệu root/hook khỏi các tiến trình nhạy cảm (vd: Google Play Services).

- Magisk & KernelSU:
   • Magisk URL: https://github.com/topjohnwu/Magisk
   • KernelSU (với ZygiskNext): Hỗ trợ chạy module Zygisk trên môi trường KernelSU.

- Các module nhỏ khác:
  • Android Faker (Xposed): Module Xposed hỗ trợ Android 8.1 – 13, cho phép giả mạo nhiều thông tin thiết bị (IMEI, Android ID, email, v.v.) trên cơ sở từng ứng dụng. Tương tự XPrivacy nhưng tập trung vào thay đổi danh tính thiết bị. Ưu: dễ dùng, có giao diện chọn các giá trị fake, giúp vượt qua kiểm tra cơ bản ở một số app; Nhược: không đủ cho SafetyNet nâng cao (không động tới attestation). Dự án này hữu ích cho mục đích giả lập thiết bị khác (như Device ID Masker cũng tương tự) hơn là vượt Play Integrity, nên ít được nhắc đến trong bối cảnh SafetyNet mới.

  • ih8sn (I hate SafetyNet): Add-on dạng script cho ROM AOSP, do vài dev LineageOS phát triển. Nó chạy sớm khi boot để spoof các prop nhạy cảm ở cấp hệ thống (giống MagiskHide Props nhưng thực hiện trong init của ROM). Kết quả là một ROM tùy chỉnh có thể qua basicIntegrity/CTS mà không cần Magisk. Ưu: Phù hợp cho ai dùng ROM thuần không muốn cài Magisk (ví dụ LineageOS + MicroG) – chỉ cần flash ih8sn và config file cho đúng model. Không phụ thuộc dịch vụ chạy nền nên khó bị phát hiện. Nhược: Yêu cầu thiết lập cấu hình cho từng model (phải có đúng fingerprint, model, patch level… trong file config). Không linh hoạt theo app – tác động trên toàn hệ thống. Hiện LineageOS chưa tích hợp chính thức (vì lý do pháp lý), người dùng phải tự thêm. Dự án được duy trì trên GitHub bởi cộng đồng Lineage, nhưng hiệu quả chủ yếu dừng ở mức SafetyNet cơ bản; với Play Integrity nâng cao, vẫn cần kết hợp phương pháp khác.

  • selinux_permissive_hide, BypassRootCheckPro, zygisk-maphide (ẩn dấu vết file Magisk trên /proc), Zygisk-Assistant (thay Shamiko cho KernelSU/APatch).

================================================================================
4. CHỈNH SỬA TRỰC TIẾP MÃ NGUỒN ANDROID (KHÔNG DÙNG HOOK)
--------------------------------------------------------------------------------
- ProtonAOSP (by kdrag0n):
   • URL: Thường được thảo luận trên XDA; bạn có thể tham khảo ví dụ tại:
     https://forum.xda-developers.com/t/protonaosp-rom-tinh-nang-safetynet.4214223/
   • Chức năng: ROM tùy chỉnh tích hợp patch spoof bootloader, fingerprint, và chặn key attestation.

- CalyxOS:
   • URL: https://calyxos.org/
   • Chức năng: ROM chú trọng bảo mật, tích hợp patch đổi model nhằm chặn attestation phần cứng.

- ih8sn:
   • URL tham khảo (XDA): https://forum.xda-developers.com/t/ih8sn-safetynet-fix.4214298/
   • Chức năng: Script/shim thay đổi các prop hệ thống để pass SafetyNet trên ROM như LineageOS.

- microG:
   • URL: https://microg.org/
   • Chức năng: Cung cấp bản SafetyNet API giả lập cho các thiết bị không dùng GMS chính thống.

- Tích hợp vào ROM tùy chỉnh: Một số ROM như ProtonAOSP trước đây đã tích hợp sẵn patch SafetyNet: họ set các thuộc tính giả ngay trong source (ví dụ báo bootloader locked bằng cách override prop kernel khi khởi động, chỉ gửi fingerprint "chuẩn" tới dịch vụ Google Play, loại bỏ hậu tố lineage_ trong device name, chặn luôn việc sử dụng hardware attestation trong keystore service). Những thay đổi đó cho phép ROM pass SafetyNet out-of-the-box mà không cần Magisk. Ưu: người dùng cuối trải nghiệm liền mạch, không cần cài thêm gì; Nhược: các dự án ROM lớn (LineageOS, etc.) khó làm vậy do chính sách – chỉ các ROM custom nhỏ hoặc build cá nhân mới dám tích hợp (ProtonAOSP đã dừng vì Google siết dần và dev chuyển hướng). Tuy nhiên, xu hướng hiện nay là các bản port Pixel Experience hoặc MIUI EU thường cố gắng pass deviceIntegrity mặc định (sign ROM bằng key riêng, chỉnh fingerprint đúng thiết bị…) để tiện cho người dùng. Cộng đồng cũng thảo luận việc tích hợp sẵn spoofing framework vào ROM (ví dụ đưa chức năng signature spoofing & SafetyNet passing vào LineageOS for microG), nhưng nhìn chung các dự án chính thống e dè do nguy cơ pháp lý và mèo vờn chuột với Google.

================================================================================
5. GIẢ MẠO KEY ATTESTATION & THÔNG TIN HARDWARE-BACKED KEYSTORE
--------------------------------------------------------------------------------
Các giải pháp liên quan đến key attestation chủ yếu bao gồm:
- Key injection qua keybox (như trong TrickyStore).
- Chặn key attestation để buộc hệ thống dùng basic attestation.
- Điều chỉnh thông tin keystore hardware qua prop (ví dụ: ro.product.first_api_level).
- Exploit phần cứng: Chưa có dự án mã nguồn mở nào thành công do Google có cơ chế revoke khóa.
  
Lưu ý: Các phương pháp spoof key attestation thường dựa vào việc sử dụng keybox hợp lệ;
nếu khóa này bị lộ hoặc bị thu hồi, sẽ gây rủi ro nghiêm trọng.

================================================================================
6. BYPASS SELINUX VÀ NÉ TRÁNH DROIDGUARD
--------------------------------------------------------------------------------
- SELinux bypass:
   • Ví dụ: Module selinux_permissive_hide (thường được chia sẻ trên XDA) giúp ẩn trạng thái
     SELinux permissive khỏi các kiểm tra của SafetyNet.

- DroidGuard evasion:
   • Sử dụng các công cụ như Magisk DenyList, Shamiko, XposedHide để ẩn dấu hiệu hook/root
     khỏi DroidGuard của Google Play Services.
   • Một số bài phân tích về DroidGuard:
     https://forum.xda-developers.com/t/droidguard-deep-dive.4214234/

- Bypass DroidGuard & SELinux:
  • Về SELinux: Đa số giải pháp hiện đại giữ SELinux ở chế độ Enforcing rồi giả lập môi trường "sạch" thay vì tắt SELinux (vì SELinux permissive sẽ bị SafetyNet phát hiện ngay). Ví dụ, MagiskHide/Shamiko giấu các dịch vụ root dưới Enforcing, ProtonAOSP thì override prop ro.boot.verifiedbootstate thành "GREEN". Không có module cụ thể "giả SELinux" công khai, thường chỉ là chỉnh props và ẩn tiến trình.
  
  • Về DroidGuard (thành phần kiểm tra ẩn của SafetyNet): các nghiên cứu kỹ thuật (BlackHat 2019, SSTIC 2021) mô tả cách nó thu thập dữ liệu hệ thống. Giải pháp bypass chủ yếu là tạo môi trường "tin cậy" để DroidGuard không phát hiện bất thường. Do đó các module như SafetyNet Fix/PIF thực chất vô hiệu hóa một số check của DroidGuard bằng cách chặn attestation phần cứng, sửa prop thiết bị, ẩn root… (tránh để DroidGuard thấy dấu hiệu bất thường). Trực tiếp "patch" DroidGuard là rất khó vì nó chạy mã native do Google tải về. Dự án microG có từng giả lập một phần SafetyNet (chỉ trả về kết quả basic OK giả lập), nhưng đã bỏ do vấn đề pháp lý và không đủ tin cậy. Tóm lại, không có công cụ công khai để bypass DroidGuard 100% – thay vào đó, người dùng kết hợp hide root, hide Xposed, spoof prop, chặn key attestation như các dự án trên để DroidGuard nghĩ rằng thiết bị hợp lệ. 
  
  • Một số module hỗ trợ gián tiếp: ví dụ zygisk-maphide (ẩn dấu vết file Magisk trên /proc), Zygisk-Assistant (thay Shamiko cho KernelSU/APatch)… Chúng không spoof info nhưng giúp môi trường sạch hơn. Những kỹ thuật sandbox evasion (né phát hiện môi trường ảo) được nghiên cứu nhiều trong tài liệu học thuật và cộng đồng đang áp dụng dần vào các module thực tế.

================================================================================
7. TÍCH HỢP GIẢ MẠO VÀO ROM TÙY CHỈNH
--------------------------------------------------------------------------------
Các giải pháp tích hợp trực tiếp vào ROM nhằm pass SafetyNet mà không cần flash module riêng:
- LineageOS & ih8sn:
   • Tham khảo thread: https://forum.xda-developers.com/t/lineageos-pass-safetynet-with-ih8sn.4214300/
- CalyxOS:
   • URL: https://calyxos.org/
- PixelExperience, crDroid:
   • PixelExperience URL: https://download.pixelexperience.org/
   • crDroid URL: https://crdroid.net/
- /e/OS:
   • URL: https://e.foundation/
  
Ưu điểm: Giảm rủi ro flash module riêng, OTA update không làm mất tính năng spoof.
Nhược điểm: Ít linh hoạt với người dùng muốn root sâu, phải chờ cập nhật ROM khi Google thay đổi.

================================================================================
8. PHƯƠNG PHÁP REVERSE ENGINEERING ROM ĐỂ THAY ĐỔI THÔNG TIN HỆ THỐNG
--------------------------------------------------------------------------------
Các phương pháp thay đổi thông tin hệ thống bao gồm:
- Chỉnh sửa file build.prop, default.prop:
   • Công cụ: MagiskBoot (https://github.com/topjohnwu/MagiskBoot)
- Smali patching framework:
   • Công cụ: APKTool (https://ibotpeaches.github.io/Apktool/)
- Hex edit các file binary:
   • Công cụ: HxD (https://mh-nexus.de/en/hxd/)
- Hook native bằng Frida/Xposed:
   • Frida URL: https://frida.re/
- Phân tích cập nhật Google Play Services:
   • Công cụ: JADX (https://github.com/skylot/jadx), Ghidra (https://ghidra-sre.org/)
- Tham khảo thêm các bài viết chuyên sâu:
   • Ví dụ: "Inside SafetyNet Attestation" (các bài báo tại Black Hat, SSTIC).

================================================================================
9. CÔNG CỤ PHÁT HIỆN VÀ PHÂN TÍCH CƠ CHẾ KIỂM TRA THIẾT BỊ CỦA GOOGLE (2023–2024)
--------------------------------------------------------------------------------
- YASNAC (Yet Another SafetyNet Attestation Checker):
   • URL: https://github.com/LSPosed/YASNAC
- RootBeer & các bộ test root:
   • RootBeer URL: https://github.com/scottyab/rootbeer
- Reverse engineering tools:
   • APKTool: https://ibotpeaches.github.io/Apktool/
   • JADX: https://github.com/skylot/jadx
   • Ghidra: https://ghidra-sre.org/
   • Frida: https://frida.re/
- JD-GUI (decompiler Java):
   • URL: http://java-decompiler.github.io/
- MagiskBoot:
   • URL: https://github.com/topjohnwu/MagiskBoot
- Google Play Services Info:
   • Ví dụ: https://play.google.com/store/apps/details?id=com.googlesource.android.playservicesinfo

- Công cụ phân tích cơ chế kiểm tra: 
  • Người dùng có thể dùng app YASNAC (Yet Another SafetyNet Attestation Checker) để kiểm tra trạng thái SafetyNet/Integrity của máy. YASNAC mã nguồn mở do Rikka phát triển, cho phép xem chi tiết JSON response (ví dụ xem evaluationType = BASIC/HARDWARE để biết máy đang dùng attestation loại nào). 
  • Ngoài ra có app Play Integrity API Checker trên Play Store (cũng open-source) hiển thị các verdict: MEETS_BASIC/DEVICE/STRONG_INTEGRITY. 
  • Công cụ Key Attestation Demo của vvb2060 cũng hữu ích: nó kiểm tra khóa phần cứng của thiết bị, báo xem key còn hiệu lực hay bị revoke – hỗ trợ người dùng xác định keybox (nếu dùng) đã bị Google vô hiệu hay chưa. 
  • Về phân tích sâu, dev có thể dùng logcat với tag Attestation hoặc thậm chí hook các hàm keystore để xem quá trình attest. 
  • Các dự án mã nguồn mở như SafetyNet Helper (cũ) hoặc các bản phân tích của ROMain Thomas, Nikos (SSTIC, NDSS) cung cấp cái nhìn chi tiết về cơ chế. 
  • Trong năm 2023-2024, cộng đồng XDA và Telegram chính là "công cụ" mạnh – mọi thay đổi nhỏ của Google đều được phát hiện nhanh chóng qua hàng loạt báo cáo người dùng, từ đó dev module sẽ debug bằng cách so sánh log, dump JSON và reverse engineer binary mới của Play Services. 
  • Không có nhiều tool tự động nào hơn ngoài các app kiểm tra nói trên và phương pháp thủ công (đọc log, phân tích mã smali của Google Play Services cập nhật). 
  • Tuy nhiên, với sự ra đời của các dự án như PIF và TrickyStore, chúng ta thấy rằng kiến thức cộng đồng về Play Integrity đang rất nâng cao – ví dụ PIF đã tích hợp sẵn cả chế độ debug mà dev có thể bật để log quá trình attestation. 
  • Người dùng nâng cao có thể tham khảo mã nguồn các dự án này để hiểu Google kiểm tra những gì và cách bypass.

================================================================================
NGUỒN THAM KHẢO CHI TIẾT:
--------------------------------------------------------------------------------
1. XPL-EX: https://github.com/0bbedCode/XPL-EX
2. PlayIntegrityFix: https://github.com/chiteroman/PlayIntegrityFix
3. TrickyStore: https://github.com/5ec1cff/TrickyStore
4. Tricky-Addon-Update-Target-List: https://github.com/KOWX712/Tricky-Addon-Update-Target-List
5. Universal SafetyNet Fix (kdrag0n): https://github.com/kdrag0n/universal-safetynet-fix
6. ProtonAOSP (XDA thread): https://forum.xda-developers.com/t/protonaosp-rom-tinh-nang-safetynet.4214223/
7. ih8sn (XDA thread): https://forum.xda-developers.com/t/ih8sn-safetynet-fix.4214298/
8. CalyxOS: https://calyxos.org/
9. LineageOS: https://lineageos.org/
10. PixelExperience: https://download.pixelexperience.org/
11. crDroid: https://crdroid.net/
12. /e/OS: https://e.foundation/
13. Magisk: https://github.com/topjohnwu/Magisk
14. MagiskHide Props Config: https://forum.xda-developers.com/t/tool-magiskhide-props-config.3654247/
15. Shamiko: https://github.com/LSPosed/Shamiko
16. APKTool: https://ibotpeaches.github.io/Apktool/
17. JADX: https://github.com/skylot/jadx
18. Ghidra: https://ghidra-sre.org/
19. Frida: https://frida.re/
20. JD-GUI: http://java-decompiler.github.io/
21. MagiskBoot: https://github.com/topjohnwu/MagiskBoot
22. RootBeer: https://github.com/scottyab/rootbeer
23. YASNAC: https://github.com/LSPosed/YASNAC
24. Google Play Services Info (ví dụ): https://play.google.com/store/apps/details?id=com.googlesource.android.playservicesinfo
25. XDA-Developers Forum: https://forum.xda-developers.com/

================================================================================
KẾT LUẬN:
--------------------------------------------------------------------------------
Hệ sinh thái giả mạo thông tin thiết bị trên Android rất sôi động, từ các Xposed module như XPL-EX, Android Faker (thiên về giả thông tin chung) đến các Magisk/KernelSU module như SafetyNet Fix, PlayIntegrityFix, TrickyStore (chuyên bypass attestation), cùng những công cụ bổ trợ như BootloaderSpoofer, ih8sn, hay addon WebUI. Mỗi dự án có ưu/nhược riêng và thường được kết hợp để đạt hiệu quả cao nhất. Các giải pháp mới nhất tập trung vào Play Integrity API (thay thế SafetyNet) – ví dụ PIF và playcurl để duy trì deviceIntegrity, và TrickyStore để nhắm tới strongIntegrity. Bên cạnh đó, việc tích hợp ở mức ROM và các thủ thuật kỹ thuật (patch prop, chặn keystore) vẫn là nền tảng cho mọi phương pháp.

Cộng đồng mã nguồn mở luôn không ngừng phát triển các giải pháp spoof nhằm vượt qua các kiểm tra an ninh của Google (SafetyNet, Play Integrity API). Dù mỗi giải pháp đều có ưu, nhược điểm riêng, nhưng sự phối hợp giữa module Magisk/KSU, chỉnh sửa mã nguồn ROM và reverse engineering đã tạo ra hệ sinh thái đa dạng, đáp ứng nhu cầu của người dùng từ cơ bản đến cao cấp.

Cuộc chiến với cơ chế kiểm tra của Google mang tính chất mèo đuổi chuột, nhưng nhờ mã nguồn mở và cộng đồng (XDA, GitHub), người dùng nâng cao luôn có lựa chọn để giành lại quyền kiểm soát thiết bị của mình trong giới hạn cho phép. Các dự án kể trên đại diện cho tinh thần đó – tận dụng kiến thức tập thể để vượt qua rào cản và tiếp tục tùy biến Android theo ý muốn.

Tuy nhiên, Google cũng không ngừng cập nhật, vì vậy các giải pháp hiện tại có thể chỉ là tạm thời, cần theo dõi và cập nhật liên tục.

================================================================================
(End of Document)
