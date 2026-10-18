# KIẾN TRÚC HỆ THỐNG LOA PHÁT THANH THÔNG MINH CLOUD IOT
*(Smart Cloud IoT PA Speaker System Architecture)*

## 1. Tổng Quan Kiến Trúc (Architecture Overview)

Hệ thống **Loa Cloud IoT** là giải pháp phát thanh công cộng thế hệ mới, thay thế toàn diện các hệ thống phát thanh truyền thống (FM vô tuyến hoặc truyền thanh có dây kéo dài). Hệ thống hoạt động hoàn toàn độc lập dựa trên hạ tầng mạng viễn thông không dây (4G LTE / Wi-Fi) và nguồn năng lượng tái tạo (Pin năng lượng mặt trời kết hợp lưu trữ ắc quy/Li-ion).

```mermaid
graph TD
    subgraph CLOUD_BROADCAST ["☁️ TRUNG TÂM PHÁT THANH & ĐIỀU HÀNH (CLOUD)"]
        MIC["🎙️ Micro / Bản tin khẩn cấp"] --> OBS["💻 Máy chủ / Trạm phát thanh"]
        AUDIO_FILE["🎵 File âm thanh / MP3 / Playlist"] --> OBS
        OBS --> STREAM_SRV["📡 Streaming Server (Icecast / Shoutcast / RadioKing)"]
        CMS["🖥️ Web Dashboard Quản trị / Node-RED"] --> MQTT_BROKER["📨 MQTT Broker (Port 1883)"]
    end

    subgraph NETWORK_INFRA ["🌐 HẠ TẦNG KẾT NỐI (NETWORK)"]
        STREAM_SRV -->|"TCP Stream (HTTP MP3/AAC)"| ROUTER_4G["📶 4G LTE Gateway / Wi-Fi CPE"]
        MQTT_BROKER <-->|"MQTT pub/sub (QoS 0/1)"| ROUTER_4G
    end

    subgraph IOT_RECEIVER_NODE ["🔊 TRẠM THU LOA CLOUD IOT (TẠI CỘT PHÁT THANH)"]
        ROUTER_4G -->|"Wi-Fi 2.4GHz 802.11b/g/n"| ESP32["🧠 Vi điều khiển ESP32 (Dual Core 240MHz)"]
        
        subgraph EMBEDDED_AUDIO ["Phân Hệ Xử Lý & Giải Mã Âm Thanh"]
            ESP32 -->|"High-Speed SPI Bus"| VS1053["🎛️ IC Giải Mã Phần Cứng VS1053B (MP3/AAC/OGG)"]
            VS1053 -->|"Analog Line-Out L/R"| ISOLATION["⚡ 2x Biến Áp Cách Ly Âm Thanh 600:600Ω"]
            ISOLATION -->|"Clean Analog Audio"| PAM8403["🔊 Mạch Khuếch Đại PAM8403 / Amply Công Suất"]
            PAM8403 --> SPEAKER["📢 Loa Nén Phóng Thanh / Loa Công Cộng"]
        end

        subgraph POWER_SUBSYSTEM ["Phân Hệ Nguồn Độc Lập Ngoài Trời"]
            SOLAR_PANEL["☀️ Tấm Pin Năng Lượng Mặt Trời (18V)"] --> CHARGE_CTRL["⚡ Mạch Sạc Năng Lượng Mặt Trời (PWM/MPPT)"]
            CHARGE_CTRL <--> BATTERY["🔋 Khối Pin Lưu Trữ Li-ion / Ắc quy 12V"]
            CHARGE_CTRL --> BUCK_5V["🔌 Mạch Hạ Áp DC-DC Buck 5V"]
            BUCK_5V -->|"5V VCC (Lọc C2 470uF)"| ESP32
            BUCK_5V --> VS1053
            BUCK_5V --> PAM8403
            BATTERY -->|"Cầu phân áp R1=100k, R2=100k"| ADC_SENSE["📊 GPIO34 (ADC1_CH6 Đo Dung Lượng Pin)"]
        end

        subgraph USER_PERIPHERALS ["Giao Tiếp & Ngoại Vi"]
            ENCODER["🔘 Rotary Encoder (GPIO12, 14, 13)"] --> ESP32
            ESP32 -->|"I2C Bus (GPIO21-SDA, GPIO22-SCL)"| LCD["📟 Màn Hình LCD1602 I2C"]
            LDR["💡 Cảm biến ánh sáng LDR"] --> LIGHT_CTRL["💡 Đèn Chiếu Sáng / Cảnh Báo Ban Đêm"]
        end
    end
```

---

## 2. Phân Hệ Chi Tiết (Subsystems Breakdown)

### 2.1. Phân Hệ Xử Lý Trung Tâm & Giải Mã Âm Thanh (Core & Codec)

- **Vi điều khiển ESP32-WROOM-32D**:
  - Vi xử lý lõi kép Tensilica Xtensa 32-bit LX6, xung nhịp 240 MHz.
  - Tích hợp kết nối Wi-Fi 802.11 b/g/n (2.4 GHz) và Bluetooth v4.2 BR/EDR & BLE.
  - Chạy hệ điều hành thời gian thực FreeRTOS: phân chia luồng nhận stream mạng (Network Task) và luồng đẩy dữ liệu vào bộ giải mã (Play Task) độc lập để ngăn chặn hiện tượng nghẽn bộ đệm.
- **Module Giải Mã Phần Cứng VS1053B**:
  - Giao tiếp với ESP32 thông qua chuẩn SPI tốc độ cao.
  - Giải mã phần cứng hoàn toàn cho các định dạng: MP3, AAC, WMA, OGG Vorbis, FLAC, WAV, MIDI.
  - Tích hợp bộ giải mã DAC 18-bit với độ méo hài tổng THD cực thấp (< 0.1%).
  - Chân tín hiệu điều khiển:
    - `VS_CS` (Chip Select) -> **GPIO5**
    - `VS_DCS` (Data Chip Select) -> **GPIO16** (hoặc GPIO32 trên bản mở rộng)
    - `VS_DREQ` (Data Request Interrupt) -> **GPIO4**
    - Bus SPI dùng chung: `SCK` (**GPIO18**), `MISO` (**GPIO19**), `MOSI` (**GPIO23**).

### 2.2. Phân Hệ Khử Nhiễu & Khuếch Đại Âm Thanh (Noise Isolation & PA)

Trong các thiết bị IoT tích hợp Wi-Fi/4G và bộ khuếch đại Class-D trên cùng một mạch nguồn, hiện tượng **nhiễu vòng lặp mass (Ground Loop)** và **nhiễu RF xung cao tần từ anten** thường tạo ra tiếng rít khó chịu ở loa.

```mermaid
graph LR
    VS_OUT["Ngõ ra VS1053 (L/R/GND)"] -->|"Tín hiệu vi sai"| PRI["Cuộn Sơ Cấp (600Ω)"]
    PRI -.->|"Cách ly điện 100% bằng từ trường"| SEC["Cuộn Thứ Cấp (600Ω)"]
    SEC -->|"Tín hiệu sạch không lẫn nhiễu mass"| PAM_IN["Ngõ vào PAM8403 (IN_L/IN_R)"]
```

- **2x Biến áp âm thanh cách ly 600:600Ω (T1, T2)**: Cách ly hoàn toàn đường mass tín hiệu âm thanh của module VS1053 khỏi đường mass công suất của PAM8403, triệt tiêu 100% tiếng ù xì tần số 50/100Hz và tiếng lách tách do phát xạ Wi-Fi.
- **Mạch khuếch đại PAM8403**:
  - Mạch công suất Class D hiệu suất cao (> 90%), hạn chế sinh nhiệt.
  - Công suất stereo 2 x 3W @ 4Ω (dùng phát thử nghiệm hoặc kiểm âm nội bộ).
  - Có thể đưa tín hiệu từ sau biến áp cách ly ra cổng Jack 3.5mm / Terminal để truyền vào Amply truyền thanh công suất lớn (60W - 120W - 250W) để kéo dàn loa nén ngoài trời.

### 2.3. Phân Hệ Nguồn Điện Năng Lượng Mặt Trời (Solar Power Subsystem)

- **Nguyên lý hoạt động**:
  - Tấm pin mặt trời hấp thụ quang năng và tạo điện áp 18V nạp cho pin lưu trữ thông qua mạch sạc năng lượng mặt trời chuyên dụng.
  - Mạch hạ áp Buck DC-DC hiệu suất cao chuyển đổi điện áp bình về chuẩn ổn định 5.0V / 3A cung cấp cho hệ thống vi mạch.
  - Tụ hóa lọc nguồn dung lượng lớn **C2 (470µF / 16V)** đặt sát chân nguồn vào nhằm hấp thụ xung dòng tức thời khi module Wi-Fi phát công suất đỉnh (Peak Current lên tới ~500mA).
- **Giám Sát Điện Áp Nguồn Bằng Cầu Phân Áp ADC**:
  - Cầu phân áp gồm 2 điện trở $R1 = 100\text{ k}\Omega$ và $R2 = 100\text{ k}\Omega$ mắc nối tiếp.
  - Hệ số phân áp: $V_{\text{ADC}} = V_{\text{BAT}} \times \frac{R2}{R1 + R2} = \frac{V_{\text{BAT}}}{2}$.
  - Ngõ ra nối vào chân **GPIO34** (kênh ADC1_CH6 không bị xung đột với Wi-Fi).
  - Mức cài đặt cấu hình trong firmware:
    - `batt0 = 2600`: Giá trị ADC tương ứng mức cạn pin (cần ngắt tải hoặc cảnh báo).
    - `batt100 = 2950`: Giá trị ADC tương ứng mức pin đầy (100%).

---

## 3. Kiến Trúc Luồng Dữ Liệu & Điều Khiển (Data & Control Flow)

```mermaid
sequenceDiagram
    autonumber
    actor Admin as 👨‍💼 Quản Trị Viên / Trực Ban
    participant WebUI as 🖥️ Dashboard / MQTT Client
    participant Broker as 📨 MQTT Broker (Port 1883)
    participant ESP as 🧠 ESP32 Controller
    participant AudioSrc as 📻 Audio Stream Server (VOV/Icecast)
    participant VS as 🎛️ VS1053B Codec
    participant Speaker as 📢 Loa Nén Công Cộng

    Note over Admin,Broker: LỆNH PHÁT THÔNG BÁO KHẨN CẤP
    Admin->>WebUI: Nhập lệnh: station=media.kythuatvov.vn:7001/
    WebUI->>Broker: Publish: Topic "LOA CLOUD/command", Msg: "station=..."
    Broker->>ESP: Nhận lệnh MQTT qua kết nối Wi-Fi/4G
    ESP->>AudioSrc: Thiết lập HTTP GET Stream Request
    AudioSrc-->>ESP: Trả về HTTP 200 OK + Audio Stream Header
    
    loop Truyền & Giải Mã Dữ Liệu Âm Thanh
        AudioSrc->>ESP: Gói tin dữ liệu MP3/AAC
        ESP->>ESP: Nạp vào FreeRTOS RingBuffer
        ESP->>VS: Truyền gói qua SPI khi DREQ = HIGH
        VS->>Speaker: Xuất tín hiệu Analog sau lọc biến áp
    end

    Note over ESP,WebUI: BÁO CÁO TRẠNG THÁI ĐỊNH KỲ
    ESP->>Broker: Publish: Topic "LOA CLOUD/status", Msg: "IP: 192.168.1.50 | Vol: 80% | Batt: 92%"
    Broker->>WebUI: Cập nhật chỉ số trực quan cho ban chỉ huy
```

---

## 4. Đặc Điểm Thiết Kế Chuẩn Kỹ Sư (Engineering Highlights)

1. **Khả năng hoạt động không phụ thuộc hạ tầng cáp**: Lắp đặt dễ dàng tại vùng sâu vùng xa, biên giới, hải đảo, khu vực công trình thi công nhờ kết nối 4G và năng lượng mặt trời.
2. **Kháng nhiễu chuyên nghiệp**: Thiết kế biến áp cách ly galvanic 600:600Ω loại trừ triệt để nhiễu ground loop và nhiễu sóng vô tuyến từ bộ thu phát 4G/Wi-Fi.
3. **Quản trị kép (Dual Control Management)**:
   - Điều khiển tập trung từ xa thông qua giao thức **MQTT** chuẩn IoT công nghiệp.
   - Điều khiển và cấu hình tại chỗ qua giao diện **Web UI nhúng** (ESPAsyncWebServer trên Port 80) trực tiếp từ điện thoại thông minh qua mạng SoftAP/Local LAN.
4. **Hỗ trợ cập nhật từ xa (OTA - Over-The-Air)**: Nâng cấp firmware không dây, giảm thiểu chi phí bảo trì tại hiện trường.
