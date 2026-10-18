# HƯỚNG DẪN ĐẶT MẠCH IN PCB (PCB MANUFACTURING GUIDE)

Thư mục này chứa file nén Gerber chuẩn công nghiệp của bo mạch **Loa Cloud IoT**, sẵn sàng gửi tới các nhà máy gia công mạch in (JLCPCB, PCBWay, v.v.):

- **File Gerber chính**: [`Gerber_ESP32_Cloud_Speaker_PCB.zip`](Gerber_ESP32_Cloud_Speaker_PCB.zip)

## Thông Số Đặt Mạch Khuyến Nghị (PCB Specifications)

| Thông Số | Giá Trị Khuyến Nghị | Ghi Chú |
|:---|:---|:---|
| **Số lớp mạch (Layers)** | 2 lớp (2 Layers) | Top Layer + Bottom Layer |
| **Vật liệu nền (Material)** | FR-4 Standard Tg 130-140°C | Chuẩn cách điện công nghiệp |
| **Độ dày bo mạch (Thickness)** | 1.6 mm | Đảm bảo độ cứng cáp cơ học |
| **Độ dày lớp đồng (Copper Weight)**| 1 oz (35 µm) | Chịu dòng tải nguồn amply 5V |
| **Màu lớp phủ hàn (Solder Mask)** | Xanh dương (Blue) hoặc Xanh lá (Green) | Phù hợp với màu logo dự án |
| **Màu chữ in lụa (Silkscreen)** | Trắng (White) | Chữ in linh kiện sắc nét |
| **Mạ bề mặt (Surface Finish)** | HASL with lead (hoặc Lead-Free HASL) | Dễ hàn thiếc bằng mỏ hàn tay |

## Các File Trong Gói Gerber
- `Gerber_TopLayer.GTL`: Lớp đồng mặt trên
- `Gerber_BottomLayer.GBL`: Lớp đồng mặt dưới
- `Gerber_TopSilkscreenLayer.GTO`: Lớp in lụa mặt trên
- `Gerber_BottomSilkscreenLayer.GBO`: Lớp in lụa mặt dưới
- `Gerber_TopSolderMaskLayer.GTS`: Lớp phủ hàn mặt trên
- `Gerber_BottomSolderMaskLayer.GBS`: Lớp phủ hàn mặt dưới
- `Gerber_BoardOutlineLayer.GKO`: Đường bao cắt viền mạch
- `Drill_PTH_Through.DRL`: Mũi khoan mạ đồng xuyên lớp
- `Drill_NPTH_Through.DRL`: Mũi khoan lỗ bắt vít cơ khí không mạ
