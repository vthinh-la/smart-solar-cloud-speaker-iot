# HƯỚNG DẪN QUY TRÌNH LẮP RÁP HỆ THỐNG (ASSEMBLY GUIDE)
*(Step-by-Step Mechanical & Electronic Assembly Guide)*

Tài liệu này hướng dẫn chi tiết quy trình lắp ráp phần cứng, hàn bo mạch, tích hợp hộp kỹ thuật chống nước ngoài trời và lắp đặt hệ thống loa trên cột phát thanh thực tế.

---

## 1. Quy Trình Gia Công & Hàn Bo Mạch Điện Tử (PCB Assembly)

### 1.1. Thứ tự hàn linh kiện (Soldering Order)

Để đảm bảo các mối hàn phẳng đẹp, không bị cấn chân và dễ thao tác, hãy tuân thủ thứ tự lắp từ thấp đến cao:
1. **Linh kiện thụ động thấp**:
   - Hàn 2 điện trở cầu phân áp ADC $R1, R2 = 100\text{ k}\Omega$.
   - Hàn cọc cắm chân nguồn $CN1$ (XH2.54 2-pin).
2. **Biến áp cách ly âm thanh**:
   - Gắn 2 biến áp cách ly $T1, T2$ (600:600Ω) đúng chiều chân theo ký hiệu sơ đồ trên bo mạch.
   - Chú ý không uốn gãy chân chân biến áp; hàn ngấu thiếc vào pad đồng.
3. **Các hàng rào chân cắm (Female Header Sockets)**:
   - Hàn hàng rào 15-pin đôi cho socket cắm ESP32 DEVKIT V1 (sử dụng socket giúp tháo lắp vi điều khiển dễ dàng khi cần thay thế).
   - Hàn hàng rào cắm cho module giải mã VS1053B và module PAM8403.
   - Hàn cọc pin header 11-pin ($CN2$) và header I2C cho màn hình LCD.
4. **Tụ lọc nguồn**:
   - Hàn tụ hóa $C2 = 470\text{ µF} / 16\text{V}$.
   - **Lưu ý cực tính**: Vạch trắng trên thân tụ là cực âm (-), cắm đúng chiều chân âm vào pad có ký hiệu sọc trên PCB.

### 1.2. Kiểm tra không tải trước khi cắm module (Pre-Power Check)

> [!CAUTION]
> **Không cắm ngay ESP32 và VS1053B lên bo mạch** khi mới hàn xong. Hãy thực hiện kiểm tra an toàn điện:
> 1. Dùng đồng hồ VOM đo thông mạch giữa đường $5\text{V}$ và $GND$ trên bo mạch. Điện trở phải tăng dần (do nạp tụ C2) rồi tiến về vô cùng. Nếu kêu còi báo thông mạch (0Ω), phải dò tìm và xử lý chập thiếc ngay lập tức.
> 2. Cấp nguồn $5.0\text{V}$ vào cọc $CN1$. Dùng VOM đo điện áp tại các chân socket cấp cho ESP32 ($VIN = 5.0\text{V} \pm 0.1\text{V}$) và module VS1053 ($5\text{V} = 5.0\text{V}$).
> 3. Sau khi xác nhận điện áp hoàn toàn chính xác, ngắt nguồn rồi mới tiến hành cắm ESP32, VS1053B và PAM8403 vào các socket.

---

## 2. Lắp Ráp Cơ Khí & Tích Hợp Vỏ Tủ Ngoài Trời (Enclosure Integration)

Hệ thống loa Cloud IoT hoạt động 24/7 trong môi trường thời tiết khắc nghiệt ngoài trời (mưa bão, nắng gắt, độ ẩm cao). Do đó, cấu trúc vỏ tủ bảo vệ cần đáp ứng tiêu chuẩn kỹ thuật:

```text
    ┌────────────────────────────────────────────────────────┐
    │     HỘP KỸ THUẬT NGOÀI TRỜI CHUẨN IP65 (ABS CHỐNG NẮNG) │
    │                                                        │
    │  ┌─────────────────────────┐  ┌──────────────────────┐ │
    │  │  Màn Hình LCD 1602 I2C  │  │ Rotary Encoder (Núm) │ │
    │  │  (Mặt trước có mica)    │  │ (Có gioăng cao su)   │ │
    │  └─────────────────────────┘  └──────────────────────┘ │
    │                                                        │
    │  ┌──────────────────────────────────────────────────┐  │
    │  │ Bo Mạch Điều Khiển Trung Tâm (ESP32 + VS1053B)   │  │
    │  │ Đặt trên trụ đồng cách điện M3 x 10mm            │  │
    │  └──────────────────────────────────────────────────┘  │
    │                                                        │
    │  ┌─────────────────────────┐  ┌──────────────────────┐ │
    │  │ Mạch Sạc Solar + Buck 5V│  │ Khối Pin Lưu Trữ     │ │
    │  │ Có tấm nhôm tản nhiệt   │  │ Đệm mút chống sốc    │ │
    │  └─────────────────────────┘  └──────────────────────┘ │
    │                                                        │
    │  O  O  O  (Các ốc siết cáp Cable Gland PG7 / PG9 chống nước ở đáy hộp)
    └────────────────────────────────────────────────────────┘
```

1. **Vật liệu vỏ hộp**: Sử dụng hộp nhựa kỹ thuật ABS chống tia UV hoặc tủ composite/sơn tĩnh điện, có gioăng cao su làm kín nắp.
2. **Vị trí các ngõ cáp**: Toàn bộ lỗ luồn dây cáp nguồn mặt trời, dây tín hiệu loa và ăng-ten phải được bố trí **ở đáy dưới của hộp kỹ thuật** kết hợp ốc siết cáp chống nước (Cable Gland PG7/PG9). Không bao giờ khoét lỗ ở nắp trên hoặc hai bên hông để tránh nước mưa chảy men theo dây vào trong.
3. **Cố định bo mạch**: Bo mạch PCB được bắt vít cố định trên các trụ đồng cách điện M3 cao 10mm, tạo khoảng cách thông thoáng với đáy hộp.
4. **Tản nhiệt và thông khí**: Mạch hạ áp Buck DC-DC và IC PAM8403 cần được gắn tản nhiệt nhôm mini. Nếu bố trí quạt tản nhiệt hoặc lỗ thở, phải có lưới chắn côn trùng và mái che giọt bắn.

---

## 3. Lắp Đặt Trên Cột & Định Hướng Năng Lượng Mặt Trời (Pole & Solar Installation)

1. **Định hướng tấm pin mặt trời**:
   - Tại Việt Nam (Bắc bán cầu), tấm pin năng lượng mặt trời phải được hướng về **chính Nam** để hứng được lượng bức xạ mặt trời tối đa trong ngày.
   - Góc nghiêng khuyến nghị: **15° - 20°** so với mặt phẳng nằm ngang. Góc nghiêng này giúp nước mưa tự rửa trôi bụi bẩn và lá cây bám trên bề mặt kính.
2. **Lắp đặt loa nén phóng thanh**:
   - Loa nén được lắp trên đỉnh cột ở độ cao **4m - 6m** so với mặt đất.
   - Miệng loa hơi chếch xuống dưới một góc **10° - 15°** về phía khu dân cư cần phát thanh để tối ưu hóa góc phủ âm thanh và tránh nước mưa đọng trong vành loa.
3. **Tiếp địa & Chống sét lan truyền**:
   - Vỏ hộp kim loại hoặc khung giá đỡ tấm pin mặt trời cần được nối tiếp địa vào cọc đồng tiếp đất sâu tối thiểu 1.5m dưới lòng đất nhằm giảm thiểu rủi ro sét đánh cảm ứng trong mùa mưa bão.
