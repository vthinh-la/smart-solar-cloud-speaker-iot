# DANH MỤC LINH KIỆN & VẬT TƯ (BILL OF MATERIALS - BOM)
*(Hardware Components & Specifications for Cloud IoT Speaker System)*

Dưới đây là bảng thống kê toàn bộ linh kiện điện tử, module chức năng và vật tư cơ khí chế tạo được sử dụng trong hệ thống **Loa Cloud IoT**, được đối soát trực tiếp từ sơ đồ nguyên lý mạch EasyEDA, file thiết kế PCB Gerber và cấu hình thực tế đã vận hành.

---

## 1. Bảng Linh Kiện Bo Mạch Xử Lý & Giải Mã (Main Controller Board BOM)

| STT | Ký Hiệu Mạch | Tên Linh Kiện / Module | Thông Số Kỹ Thuật | Đóng Gói (Footprint) | Số Lượng | Chức Năng Trong Hệ Thống |
|:---:|:---|:---|:---|:---|:---:|:---|
| 1 | **U1** | ESP32 DOIT DevKit V1 | 30-pin, ESP32-WROOM-32D, 240MHz, 4MB Flash | DIP-30 (Socket 2.54mm) | 1 | Vi điều khiển trung tâm, xử lý Wi-Fi, MQTT, Web Server, FreeRTOS |
| 2 | **U3** | Module VS1053B Codec | VLSI Solution VS1053B, giao tiếp SPI, DAC 18-bit | Module DIP-10 | 1 | Giải mã phần cứng MP3, AAC, OGG Vorbis, WMA, FLAC |
| 3 | **U6** | Module PAM8403 | IC PAM8403 Class D stereo 2x3W, hiệu suất >90% | Module SMD / Mini board | 1 | Khuếch đại âm thanh ra loa kiểm âm hoặc xuất line ra amply ngoài |
| 4 | **T1, T2** | Biến Áp Cách Ly Âm Thanh | Tỉ lệ 1:1 (600Ω : 600Ω), lõi EI14 permalloy | DIP-5 Through-hole | 2 | Khử nhiễu mass (Ground Loop) và triệt tiêu tiếng rít cao tần RF từ Wi-Fi/4G |
| 5 | **U5** | Rotary Encoder EC11 | Chiết áp xung số 20 xung/vòng + Switch nhấn | Through-hole 5-pin | 1 | Điều chỉnh âm lượng phần cứng, chuyển kênh phát thanh tại chỗ |
| 6 | **LCD1** | Màn Hình LCD1602 + I2C | Màn LCD 16x2 ký tự, kèm mạch chuyển đổi PCF8574 | Header 4-pin (I2C) | 1 | Hiển thị trạm phát, tên bài, âm lượng, địa chỉ IP và trạng thái mạng |
| 7 | **R1, R2** | Điện trở chính xác | 100 kΩ ±1%, công suất 1/4W | Through-hole / 0805 | 2 | Cầu phân áp 1:2 đưa điện áp bình về ngưỡng đo ADC an toàn (0 - 3.3V) |
| 8 | **C2** | Tụ hóa nhôm (Electrolytic) | 470 µF / 16V (hoặc 25V) Low-ESR | Radial Through-hole | 1 | Lọc phẳng nguồn 5V, bù dòng xung tức thời khi ESP32 phát xạ Wi-Fi |
| 9 | **CN1** | Giắc nguồn DC XH-2AW | Chuẩn XH2.54 2-pin đực góc thẳng | XH2.54-2P | 1 | Kết nối nguồn 5V DC từ mạch hạ áp năng lượng mặt trời |
| 10 | **CN2** | Pin Header 11-pin | Hàng rào đơn đực khoảng cách chân 2.54mm | SIP-11 (2.54mm) | 1 | Cổng mở rộng GPIO (D25, D33, RX2, TX2, D21, D22, D39, D34, D35, VCC, GND) |
| 11 | **H2** | Header 3-pin | Chuẩn 2.54mm (L - GND - R) | SIP-3 | 1 | Ngõ vào âm thanh kiểm âm từ jack VS1053 |
| 12 | **H3** | Terminal / Header 4-pin | Chuẩn 2.54mm / Terminal kẹp dây | SIP-4 | 1 | Ngõ ra âm thanh công suất (L+, L-, R+, R-) đấu ra củ loa |
| 13 | **PCB** | Bo Mạch In 2 Lớp | FR4 chuẩn công nghiệp, mạ thiếc, phủ xanh, 2 lớp | Custom PCB (EasyEDA) | 1 | Bo mạch tích hợp toàn bộ module và linh kiện hệ thống |

---

## 2. Bảng Thiết Bị Phụ Trợ & Nguồn Năng Lượng Mặt Trời (Power & Peripherals BOM)

| STT | Thiết Bị / Linh Kiện | Thông Số Kỹ Thuật Khuyến Nghị | Số Lượng | Mục Đích Ứng Dụng |
|:---:|:---|:---|:---:|:---|
| 1 | **Tấm Pin Mặt Trời (Solar Panel)** | Monocrystalline 18V / 30W - 50W (chuẩn ngoài trời IP65) | 1 tấm | Cung cấp nguồn năng lượng tái tạo độc lập |
| 2 | **Bộ Điều Khiển Sạc Solar** | PWM / MPPT 12V 10A có màn hình LCD hiển thị và bảo vệ quá áp/quá xả | 1 bộ | Quản lý dòng nạp xả thông minh cho ắc quy/pin lưu trữ |
| 3 | **Khối Pin Lưu Trữ** | Khối pin Lithium Li-ion 3S (11.1V - 12.6V, 12Ah - 20Ah) hoặc ắc quy Gel 12V 12Ah | 1 khối | Duy trì hệ thống hoạt động liên tục 24 - 48 giờ khi trời mưa hoặc ban đêm |
| 4 | **Mạch Hạ Áp Buck DC-DC** | Mạch hạ áp XL4015 hoặc LM2596, ngõ vào 8-35V, ngõ ra 5.0V / 3A - 5A | 1 module | Chuyển đổi điện áp bình 12V thành nguồn 5V ổn định cho bo mạch |
| 5 | **Bộ Phát 4G LTE Wi-Fi (CPE)** | Router 4G LTE công nghiệp hỗ trợ SIM Nano (Viettel/Vinaphone), cấp nguồn 12V/5V | 1 bộ | Thu sóng 4G di động và phát Wi-Fi cho ESP32 kết nối Internet |
| 6 | **Loa Phóng Thanh (Horn Speaker)** | Loa nén 15W - 30W trở kháng 8Ω/16Ω hoặc dàn loa truyền thanh 100V line | 1 - 2 loa | Phát âm thanh thông báo vang xa bán kính 500m - 1km trong khu dân cư |
| 7 | **Hộp Kỹ Thuật Bảo Vệ Ngoài Trời** | Hộp nhựa ABS hoặc tủ sắt sơn tĩnh điện chuẩn chống nước IP65 có gioăng cao su | 1 hộp | Chống nước mưa, bụi bẩn, bảo vệ bo mạch và bình pin ngoài cột |
| 8 | **Module Cảm Biến Ánh Sáng (LDR)** | Mạch quang trở LM393 có rơ-le / ngõ ra kích transistor | 1 bộ | Tự động kích hoạt đèn LED chiếu sáng / cảnh báo khi trời tối |
