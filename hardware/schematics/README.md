# TÀI LIỆU SƠ ĐỒ NGUYÊN LÝ MẠCH (SCHEMATIC OVERVIEW)

Thư mục này chứa sơ đồ nguyên lý mạch điện hoàn chỉnh của hệ thống **Loa Cloud IoT**:

- **File ảnh sơ đồ độ phân giải cao**: [`Schematic_ESP32_Cloud_Speaker.png`](Schematic_ESP32_Cloud_Speaker.png)

## Các Khối Chức Năng Trên Sơ Đồ:
1. **Khối Vi Điều Khiển U1 (ESP32-DEVKITV1)**: Trung tâm điều phối truyền thông mạng, phân phối luồng dữ liệu SPI, xử lý giao tiếp I2C và đo đạc ADC.
2. **Khối Giải Mã Phần Cứng U3 (VS1053B MP3 Module)**: Chuyển đổi dòng dữ liệu nén thành tín hiệu âm thanh Stereo độ trung thực cao.
3. **Khối Cách Ly Âm Thanh T1 & T2 (Audio Isolation Transformers 600:600Ω)**: Cách ly điện hoàn toàn giữa đường mass analog của VS1053 và đường mass công suất của PAM8403, triệt tiêu tiếng rít Wi-Fi.
4. **Khối Khuếch Đại Công Suất U6 (PAM8403)**: Khuếch đại âm thanh hiệu suất cao Class-D xuất ra cọc loa H3.
5. **Khối Giao Tiếp Cục Bộ U5 (Rotary Encoder) & Header I2C**: Hỗ trợ chiết áp xoay xung số và màn hình LCD1602.
6. **Khối Nguồn & Giám Sát Điện Áp (CN1, C2, R1, R2)**: Ngõ vào nguồn 5V DC kèm tụ lọc Low-ESR 470µF và cầu phân áp 100k/100k kết nối vào chân GPIO34 để giám sát điện áp pin lưu trữ theo thời gian thực.
