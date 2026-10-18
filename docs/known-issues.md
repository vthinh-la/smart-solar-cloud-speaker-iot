# CÁC VẤN ĐỀ ĐÃ BIẾT & GIỚI HẠN THIẾT KẾ (KNOWN ISSUES & CONSTRAINTS)
*(Known Technical Issues, Hardware Constraints & Mitigations)*

Tài liệu này tổng hợp toàn bộ các lưu ý kỹ thuật, giới hạn phần cứng và kinh nghiệm thực tế trong quá trình phát triển và vận hành hệ thống **Loa Cloud IoT**.

---

## 1. Giới Hạn Công Cụ Biên Dịch Trên Windows (Toolchain Non-ASCII Path)

- **Mô tả hiện tượng**: Bộ tiền xử lý và trình biên dịch `xtensa-esp32-elf-g++` trên hệ điều hành Windows không hỗ trợ đường dẫn thư mục có chứa ký tự tiếng Việt có dấu (ví dụ: `HỆ THỐNG LOA CLOUD IOT`). Khi chạy `pio run`, trình biên dịch sẽ vấp lỗi `fatal error: no input files` hoặc `tmp...tmp: Invalid argument`.
- **Nguyên nhân**: Toolchain GNU Xtensa GCC trên Windows phân tích chuỗi đường dẫn theo bảng mã ASCII truyền thống.
- **Giải pháp xử lý (Mitigation)**:
  - Khi clone repository về máy, hãy đặt trong thư mục có tên thuần ASCII không dấu và không chứa khoảng trắng (ví dụ: `C:/Projects/esp32-cloud-speaker` hoặc `D:/smart-cloud-speaker-iot`).
  - Nếu bắt buộc phải biên dịch từ thư mục tiếng Việt, có thể ánh xạ tạm thời một ổ đĩa ảo bằng lệnh Windows:
    ```cmd
    subst X: "c:\Users\THINHLA\Desktop\HỆ THỐNG LOA CLOUD IOT"
    cd /d X:\ESP32RadioV2
    pio run
    subst X: /D
    ```

---

## 2. Nhiễu Xung RF Wi-Fi & Vòng Lặp Mass (RF Buzz & Ground Loop)

- **Mô tả hiện tượng**: Khi module Wi-Fi của ESP32 phát gói tin ở công suất phát tối đa (+19.5 dBm), dòng tiêu thụ nhảy vọt tức thời gây sụt áp vi mô trên đường mass chung, truyền vào đầu vào có độ nhạy cao của IC khuếch đại PAM8403 tạo tiếng rít tần số cao ở loa.
- **Giải pháp xử lý**:
  - Bắt buộc phải gắn **2 biến áp cách ly âm thanh 600:600Ω (T1, T2)** ngăn cách giữa ngõ ra Line-Out của VS1053 và ngõ vào của amply.
  - Tách đường mass Analog (`AGND`) của chip giải mã âm thanh độc lập hoàn toàn với đường mass công suất (`PGND`) của mạch khuếch đại.
  - Lắp tụ hóa $470\text{ µF} / 16\text{V}$ (Low-ESR) đặt song song ngay sát chân nguồn 5V của ESP32 để hấp thụ xung dòng.

---

## 3. Hiện Tượng Sụt Áp & Kích Hoạt Bộ Bảo Vệ Brownout (Brownout Detector)

- **Mô tả hiện tượng**: ESP32 bị reset bất ngờ và xuất hiện dòng thông báo `Brownout detector was triggered` trên Serial Monitor khi bắt đầu kết nối Wi-Fi hoặc khi amply đẩy âm lượng lớn đột ngột.
- **Nguyên nhân**: Dây dẫn nguồn từ mạch sạc / bộ hạ áp Buck 5V quá dài, tiết diện nhỏ làm tăng nội trở dây dẫn, hoặc mạch Buck 5V không đủ công suất dòng đỉnh (> 2A).
- **Giải pháp xử lý**:
  - Sử dụng dây dẫn nguồn có tiết diện tối thiểu $0.75\text{ mm}^2$.
  - Mạch hạ áp Buck DC-DC phải đạt công suất liên tục từ $3\text{A}$ trở lên (khuyến nghị module XL4015).
  - Trong firmware, lệnh tắt hoặc nới lỏng giới hạn brownout đã được cấu hình trong `src/main.cpp` để tăng độ ổn định khi chạy pin năng lượng mặt trời.

---

## 4. Quản Lý File Dung Lượng Lớn Trên GitHub (Git Large File Storage)

- **Mô tả hiện tượng**: File video thực nghiệm `0425.mp4` có dung lượng thực tế là **~195 MB**, vượt quá giới hạn tải lên tối đa **100 MB** cho mỗi file đơn lẻ của GitHub.
- **Giải pháp xử lý**:
  - File `*.mp4` đã được đưa vào cấu hình `.gitignore` để tránh gây nghẽn quá trình đẩy mã nguồn (push rejection).
  - Video thực nghiệm được lưu trữ trên Google Drive / YouTube dưới dạng unlisted, và đính kèm đường dẫn truy cập trong tài liệu kiểm thử.

---

## 5. Kết Nối Mạng 4G LTE

- **Mô tả**: Phiên bản hiện tại sử dụng **Router 4G LTE CPE phát Wi-Fi** độc lập ngoài cột loa, sau đó ESP32 kết nối vào mạng Wi-Fi này để lấy dữ liệu Internet.
- **Ưu điểm**: Giữ cho firmware ESP32 gọn nhẹ, ổn định cao, không tốn tài nguyên quản lý modem USB/UART qua tập lệnh AT command, đồng thời cho phép bảo trì hoặc thay thế loại SIM/nhà mạng độc lập.
- **Giới hạn**: Cần thêm nguồn cấp 12V cho bộ phát 4G CPE.
- **Lộ trình tương lai**: Nghiên cứu tích hợp module modem 4G LTE (như SIM7600 hoặc A7670C qua giao tiếp UART/PPP) trực tiếp trên một bo mạch duy nhất.
