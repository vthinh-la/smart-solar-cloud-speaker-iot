# KẾT QUẢ KIỂM THỬ THỰC NGHIỆM (TESTING & VERIFICATION)
*(Empirical Test Procedures & Experimental Results)*

Tài liệu này ghi nhận đầy đủ các bài kiểm thử kỹ thuật đã được thực hiện trên hệ thống **Loa Cloud IoT**, phân định rõ ràng giữa **kết quả đã xác minh có bằng chứng** và **các khuyến nghị kiểm tra mở rộng**.

---

## 1. Bảng Tổng Hợp Kết Quả Kiểm Thử (Verification Matrix)

| Hạng Mục Kiểm Thử | Phương Pháp & Thiết Bị Kiểm Tra | Tiêu Chuẩn Đạt | Kết Quả Thực Tế | Tình Trạng |
|:---|:---|:---|:---|:---:|
| **1. An toàn nguồn tĩnh** | VOM đo thông mạch và điện áp tại socket cắm module | $V_{CC} = 5.0\text{V} \pm 5\%$, không ngắn mạch $V_{CC}-GND$ | Điện áp đo được $5.04\text{V}$ ổn định | ✅ ĐẠT |
| **2. Cách ly mass âm thanh** | Đo điện trở DC giữa mass VS1053 (`AGND`) và mass amply (`GND`) | Cách ly hoàn toàn ($R > 10\text{ M}\Omega$) | Đồng hồ đo vô cùng (Open Loop), triệt tiêu tiếng rít Wi-Fi | ✅ ĐẠT |
| **3. Biên dịch Firmware** | PlatformIO Core v6.1.19 / Espressif 32 v6.13.0 | Biên dịch thành công 0 lỗi, Flash < 80% | Flash: 62.2% (1.22MB / 1.96MB), RAM: 14.5% (47.6KB / 327KB) | ✅ ĐẠT |
| **4. Kết nối mạng Wi-Fi/4G** | Kết nối tới Wi-Fi từ router 4G CPE ngoài | Nhận địa chỉ IP DHCP trong vòng < 5 giây | Kết nối thành công, tự động cấp IP cục bộ | ✅ ĐẠT |
| **5. Giải mã luồng MP3/AAC** | Kết nối luồng VOV1/VOV2/VOV3 và stream cá nhân RadioKing | Âm thanh giải mã liên tục, không vấp tiếng | VS1053B giải mã mượt mà bitrate 128k - 192k | ✅ ĐẠT |
| **6. Điều khiển từ xa MQTT** | Gửi lệnh qua Broker `mqtt.eclipseprojects.io:1883` | Đáp ứng lệnh đổi kênh, âm lượng độ trễ < 1 giây | Nhận lệnh `volume`, `station`, `preset` tức thời | ✅ ĐẠT |
| **7. Đo dung lượng pin ADC** | Đo điện áp khối pin qua cầu phân áp $100\text{k}/100\text{k}$ trên GPIO34 | Điện áp ADC trong dải an toàn $0 - 2.5\text{V}$ | Giá trị ADC hiệu chuẩn chuẩn xác theo cấu hình | ✅ ĐẠT |
| **8. Vận hành thực địa** | Chạy thử nghiệm thực tế ngoài trời | Hệ thống phát thanh ổn định liên tục | Có video thực tế minh chứng `0425.mp4` (2 phút 3 giây) | ✅ ĐẠT |

---

## 2. Chi Tiết Các Quy Trình Kiểm Thử Đã Thực Hiện

### 2.1. Kiểm thử biên dịch Firmware PlatformIO (Firmware Compilation Test)
- **Môi trường**: PlatformIO Core 6.1.19, framework `arduino-espressif32`, board `esp32doit-devkit-v1`.
- **Kết quả build**:
  ```text
  HARDWARE: ESP32 240MHz, 320KB RAM, 4MB Flash
  RAM:   [=         ]  14.5% (used 47648 bytes from 327680 bytes)
  Flash: [======    ]  62.2% (used 1222377 bytes from 1966080 bytes)
  Building .pio\build\esp32\firmware.bin
  esptool.py v4.11.0: Merged 2 ELF sections
  Successfully created esp32 image.
  ========================= [SUCCESS] Took 16.59 seconds =========================
  ```
- **Phát hiện kỹ thuật quan trọng**: Công cụ biên dịch `xtensa-esp32-elf-g++` trên hệ điều hành Windows yêu cầu đường dẫn thư mục làm việc hoàn toàn là ký tự **ASCII không dấu**. Nếu thư mục cha chứa ký tự tiếng Việt có dấu, bộ tiền xử lý sẽ báo lỗi `Invalid argument`. Khi clone repository, cần lưu tại thư mục ASCII (ví dụ: `C:/Projects/smart-cloud-speaker-iot`).

### 2.2. Kiểm thử chất lượng âm thanh & Biến áp cách ly (Audio Isolation Test)
- **Hiện tượng trước khi có biến áp cách ly**: Khi cắm trực tiếp ngõ ra 3.5mm của VS1053 vào PAM8403 và dùng chung nguồn 5V, mỗi khi module Wi-Fi ESP32 truyền gói tin TCP/IP, loa phát ra tiếng "xè xè" và tiếng rít xung cao tần rất khó chịu.
- **Sau khi lắp 2 biến áp cách ly 600:600Ω**: Tín hiệu âm thanh truyền qua bằng cảm ứng từ trường; đường mass analog của VS1053 hoàn toàn độc lập với đường mass công suất của amply. Nhiễu xung RF và nhiễu vòng lặp mass giảm triệt để, nền âm thanh tĩnh và trong trẻo.

### 2.3. Kiểm thử luồng phát thanh trực tuyến (Online Audio Stream Test)
Các địa chỉ luồng phát thanh đã được kiểm thử thực tế và cấu hình trong file `MAC DINH.txt`:
1. `media.kythuatvov.vn:7001/` : Đài Tiếng nói Việt Nam VOV1 (Kênh Thời sự - Chính trị).
2. `media.kythuatvov.vn:7003/` : VOV2 (Kênh Văn hóa - Xã hội).
3. `media.kythuatvov.vn:7005/` : VOV3 (Kênh Âm nhạc - Thông tin giải trí).
4. `listen.radioking.com/radio/730058/stream/796157` : Kênh truyền thanh phát thanh nội bộ đám mây (Cloud Private Broadcasting).

---

## 3. Video Thực Nghiệm Vận Hành (Empirical Video Evidence)

- Hệ thống lưu trữ video thực nghiệm quay trực tiếp thiết bị hoạt động tại hiện trường:
  - **Tên file gốc**: `0425.mp4`
  - **Thời lượng**: 00:02:03 (2 phút 03 giây)
  - **Dung lượng**: ~195 MB (được cấu hình trong `.gitignore` do vượt quá giới hạn 100MB của GitHub).
  - **Nội dung video**: Ghi lại quá trình thiết bị khởi động, hiển thị thông tin lên màn hình LCD1602, kết nối mạng Wi-Fi 4G, bắt luồng phát thanh từ trạm Cloud và phát âm thanh rõ nét ra loa.
