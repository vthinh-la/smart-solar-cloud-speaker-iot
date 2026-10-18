# HƯỚNG DẪN ĐẤU DÂY & SƠ ĐỒ CHÂN KẾT NỐI (WIRING & PINOUT GUIDE)
*(Complete Electrical Interconnections for ESP32 Cloud IoT Speaker)*

Tài liệu này cung cấp bảng ánh xạ chân (Pinout Mapping) chi tiết và hướng dẫn đấu nối chính xác giữa vi điều khiển ESP32 và toàn bộ các module ngoại vi, mạch giải mã, biến áp cách ly và mạch khuếch đại công suất.

---

## 1. Bảng Ánh Xạ Sơ Đồ Chân (Pinout Mapping Table)

| Chân ESP32 | Tên Tín Hiệu | Module Kết Nối | Chân Trên Module | Chức Năng Chi Tiết |
|:---:|:---:|:---|:---:|:---|
| **GPIO5** | `VS_CS` | VS1053B Codec | `XCS` | Chọn chip điều khiển lệnh SPI (Control Chip Select) |
| **GPIO16** | `VS_DCS` | VS1053B Codec | `XDCS` | Chọn dữ liệu âm thanh SPI (Data Chip Select) |
| **GPIO4** | `VS_DREQ` | VS1053B Codec | `DREQ` | Ngắt yêu cầu dữ liệu (Data Request - active HIGH) |
| **GPIO18** | `SPI_SCK` | VS1053B Codec | `SCK` | Xung nhịp bus SPI (SPI Clock) |
| **GPIO19** | `SPI_MISO` | VS1053B Codec | `MISO` | Đường dữ liệu SPI từ ngoại vi về ESP32 |
| **GPIO23** | `SPI_MOSI` | VS1053B Codec | `MOSI` | Đường dữ liệu SPI từ ESP32 sang ngoại vi |
| **EN / 3V3** | `VS_XRST` | VS1053B Codec | `XRST` | Reset module giải mã (nối mức cao 3.3V hoặc EN) |
| **GPIO21** | `I2C_SDA` | LCD1602 I2C | `SDA` | Đường dữ liệu I2C cho màn hình LCD |
| **GPIO22** | `I2C_SCL` | LCD1602 I2C | `SCL` | Đường xung nhịp I2C cho màn hình LCD |
| **GPIO12** | `ENC_CLK` | Rotary Encoder | `CLK` / `A` | Kênh xung pha A của chiết áp xung |
| **GPIO14** | `ENC_DT` | Rotary Encoder | `DT` / `B` | Kênh xung pha B của chiết áp xung |
| **GPIO13** | `ENC_SW` | Rotary Encoder | `SW` | Nút bấm tích hợp trên trục chiết áp |
| **GPIO34** | `ADC_BAT` | Cầu phân áp pin | Giữa R1 & R2 | Đo điện áp khối pin/ắc quy (Kênh ADC1_CH6, Input-only) |
| **GPIO25** | `VOL_UP` | Nút bấm mở rộng | Nút bấm 1 | Tăng âm lượng nhanh tại chỗ (Pull-up) |
| **GPIO33** | `VOL_DOWN` | Nút bấm mở rộng | Nút bấm 2 | Giảm âm lượng nhanh tại chỗ (Pull-up) |
| **5V (VIN)** | `VCC_5V` | Toàn hệ thống | `5V / VCC` | Nguồn dương 5.0V DC từ mạch nguồn Buck NLMT |
| **GND** | `GND` | Toàn hệ thống | `GND` | Đường mass chung của hệ thống |

---

## 2. Sơ Đồ Đấu Nối Tín Hiệu Âm Thanh & Biến Áp Cách Ly (Audio Wiring)

Khâu quan trọng nhất trong việc loại trừ tiếng rít Wi-Fi và nhiễu mass là lắp đặt **Biến áp cách ly âm thanh 600:600Ω (T1, T2)** đúng kỹ thuật:

```text
       [ Module VS1053B ]                       [ Biến Áp Cách Ly ]                      [ Module PAM8403 ]
       
   Line-Out L ────────────────────────────── Pin 1 (Cuộn Sơ Cấp 1)
                                                 ││ (Cách ly từ)
                                              Pin 3 (Cuộn Thứ Cấp 1) ────────────── IN_L (Kênh Trái)
   AGND (Mass VS1053) ────────────────────── Pin 2 (Cuộn Sơ Cấp 1)
                                                 ││
                                              Pin 4 (Cuộn Thứ Cấp 1) ────────────── GND (Mass PAM8403)
                                              
   Line-Out R ────────────────────────────── Pin 1 (Cuộn Sơ Cấp 2)
                                                 ││ (Cách ly từ)
                                              Pin 3 (Cuộn Thứ Cấp 2) ────────────── IN_R (Kênh Phải)
   AGND (Mass VS1053) ────────────────────── Pin 2 (Cuộn Sơ Cấp 2)
                                                 ││
                                              Pin 4 (Cuộn Thứ Cấp 2) ────────────── GND (Mass PAM8403)
```

> [!IMPORTANT]
> **Quy tắc vàng về Mass (GND)**: Tuyệt đối không nối tắt chân `AGND` của module VS1053 vào chân `GND` của module PAM8403 bằng dây đồng. Hai đường mass này phải được **cách ly điện một chiều 100%** và chỉ truyền tín hiệu âm thanh thông qua cảm ứng từ trường của biến áp.

---

## 3. Sơ Đồ Đấu Dây Cầu Phân Áp Đo Điện Áp Pin (Battery ADC Sense)

Vì chân GPIO của ESP32 chỉ chịu được điện áp tối đa **3.3V**, trong khi khối pin 3S / ắc quy 12V có điện áp dao động từ **9.6V đến 12.6V**, cầu phân áp phải được đấu nối theo đúng sơ đồ sau:

```text
  [ Cực Dương Pin/Ắc Quy: 10V - 13V ]
                 │
                 ├──[ R1: 100 kΩ ±1% ]──┐
                                        │
                                        ├───> Nối vào chân ESP32: GPIO34 (ADC1_CH6)
                                        │
                 ├──[ R2: 100 kΩ ±1% ]──┘
                 │
  [ Cực Âm Pin/Ắc Quy (GND) ]
```

- **Hệ số phân áp**: $V_{\text{ADC}} = V_{\text{BAT}} \times \frac{100\text{k}}{100\text{k} + 100\text{k}} = \frac{V_{\text{BAT}}}{2}$.
- Với $V_{\text{BAT}} = 12.6\text{ V}$ (pin đầy), $V_{\text{ADC}} = 6.3\text{ V}$ (vẫn vượt 3.3V nếu cấp thẳng 12V).
- **Lưu ý kiểm chứng thực tế**: Nếu mạch đo trực tiếp pin 1S (3.7V - 4.2V), cầu phân áp 100k/100k đưa điện áp về $1.85\text{V} - 2.1\text{V}$, nằm hoàn hảo trong dải tuyến tính tối ưu của bộ chuyển đổi ADC ESP32 ($0.1\text{V} - 2.5\text{V}$). Nếu đo bình 12V, cần tăng tỉ lệ $R1 = 300\text{ k}\Omega, R2 = 100\text{ k}\Omega$ (tỉ lệ 1:4).

---

## 4. Sơ Đồ Đấu Nối Khối Nguồn Năng Lượng Mặt Trời (Power Subsystem Wiring)

```text
  [ Tấm Pin NLMT 18V ]
          │ (Cáp DC Solar 2.5mm²)
          ▼
  [ Bộ Điều Khiển Sạc Solar PWM/MPPT ] <═══> [ Khối Pin Lưu Trữ Li-ion / Ắc quy 12V ]
          │ (Ngõ ra tải LOAD 12V)
          ▼
  [ Mạch Hạ Áp Buck DC-DC XL4015 ]
          │ (Chỉnh biến trở xuất đúng 5.05V DC)
          ▼
     ┌────┴──────────────────────────┐
     │ C2: Tụ hóa 470µF/16V Low-ESR   │ (Hấp thụ xung dòng Wi-Fi)
     └────┬──────────────────────────┘
          ├─────────> Chân 5V (VIN) Bo Mạch ESP32
          ├─────────> Chân 5V Module VS1053B
          ├─────────> Chân 5V Module PAM8403
          ├─────────> Chân 5V Màn Hình LCD1602
          └─────────> Cấp nguồn cho Router 4G LTE CPE
```
