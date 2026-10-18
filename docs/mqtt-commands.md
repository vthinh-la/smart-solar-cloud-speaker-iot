# TẬP LỆNH ĐIỀU KHIỂN QUA MQTT (MQTT CONTROL PROTOCOL)
*(Full MQTT Command Dictionary & Integration Guide)*

Hệ thống **Loa Cloud IoT** hỗ trợ giao thức IoT chuẩn công nghiệp **MQTT** (Message Queuing Telemetry Transport), cho phép trung tâm chỉ huy hoặc các nền tảng tự động hóa (Node-RED, Home Assistant, Python script) gửi lệnh điều khiển và giám sát trạng thái thời gian thực.

---

## 1. Cấu Trúc Topic MQTT (Topic Structure)

Cấu hình mặc định trong hệ thống:
- **Broker**: `mqtt.eclipseprojects.io` (hoặc `broker.hivemq.com`)
- **Port**: `1883` (TCP không mã hóa)
- **Prefix**: `LOA CLOUD`

| Loại Dữ Liệu | Cấu Trúc Topic | Chiều Truyền | Chức Năng |
|:---|:---|:---:|:---|
| **Lệnh điều khiển** | `LOA CLOUD/command` | Publisher ➔ ESP32 | Gửi lệnh chuyển kênh, đổi âm lượng, chỉnh âm sắc |
| **Trạng thái hệ thống** | `LOA CLOUD/status` | ESP32 ➔ Subscriber | Nhận chuỗi trạng thái hoạt động, IP, trạm đang phát |

---

## 2. Bảng Lệnh Điều Khiển Chi Tiết (Command Dictionary)

| Cú Pháp Lệnh (Message Payload) | Mô Tả Ý Nghĩa | Ví Dụ Sử Dụng |
|:---|:---|:---|
| `preset=<số>` | Chuyển sang kênh phát thanh đã lưu trong bộ nhớ (0 - 99) | `preset=0` *(Chuyển sang VOV1)* |
| `station=<URL>` | Bắt đầu phát luồng âm thanh trực tuyến ngay lập tức | `station=media.kythuatvov.vn:7001/` |
| `station=<URL>.mp3` | Tải và phát một file thông báo MP3 từ máy chủ | `station=http://myserver.vn/canhbao_bao.mp3` |
| `station=<URL>.m3u` | Đọc danh sách phát từ file playlist định dạng M3U | `station=http://myserver.vn/playlist.m3u` |
| `volume=<0..100>` | Đặt mức âm lượng tuyệt đối (từ 0% đến 100%) | `volume=85` |
| `upvolume=<bước>` | Tăng âm lượng thêm một khoảng phần trăm | `upvolume=5` |
| `downvolume=<bước>` | Giảm âm lượng đi một khoảng phần trăm | `downvolume=5` |
| `mute` | Ngắt tiếng ngay lập tức (tắt ngõ ra âm thanh) | `mute` |
| `unmute` | Khôi phục lại âm thanh về mức âm lượng trước đó | `unmute` |
| `stop` | Dừng phát luồng âm thanh hiện tại | `stop` |
| `resume` | Tiếp tục phát trạm thanh gần nhất | `resume` |
| `toneha=<0..15>` | Điều chỉnh độ lớn dải cao (Treble Gain) | `toneha=8` |
| `tonehf=<0..15>` | Điều chỉnh tần số cắt dải cao (Treble Frequency) | `tonehf=2` |
| `tonela=<0..15>` | Điều chỉnh độ lớn dải trầm (Bass Gain) | `tonela=6` |
| `tonelf=<0..15>` | Điều chỉnh tần số cắt dải trầm (Bass Frequency) | `tonelf=10` |
| `wifi_00=SSID/PASSWORD` | Cập nhật cấu hình mạng Wi-Fi ưu tiên số 0 | `wifi_00=ICTU_IoT/Pass123456` |
| `mqttbroker=<host>` | Đổi địa chỉ máy chủ MQTT Broker | `mqttbroker=broker.hivemq.com` |
| `sleep` | Đưa ESP32 vào chế độ ngủ sâu (Deep Sleep) tiết kiệm pin | `sleep` |
| `status` | Yêu cầu thiết bị phản hồi gói tin trạng thái lên topic status | `status` |

---

## 3. Ví Dụ Điều Khiển Bằng Lệnh Mosquitto (Command Line CLI)

### 3.1. Chuyển kênh phát thanh trực tiếp:
```bash
# Chuyển sang Đài Tiếng nói Việt Nam VOV1
mosquitto_pub -h mqtt.eclipseprojects.io -t "LOA CLOUD/command" -m "station=media.kythuatvov.vn:7001/"

# Chuyển sang kênh phát thanh nội bộ đám mây
mosquitto_pub -h mqtt.eclipseprojects.io -t "LOA CLOUD/command" -m "station=listen.radioking.com/radio/730058/stream/796157"
```

### 3.2. Điều chỉnh âm lượng và ngắt tiếng khẩn cấp:
```bash
# Đặt âm lượng 90%
mosquitto_pub -h mqtt.eclipseprojects.io -t "LOA CLOUD/command" -m "volume=90"

# Mute khẩn cấp
mosquitto_pub -h mqtt.eclipseprojects.io -t "LOA CLOUD/command" -m "mute"
```

### 3.3. Lắng nghe trạng thái thiết bị phản hồi:
```bash
mosquitto_sub -h mqtt.eclipseprojects.io -t "LOA CLOUD/status" -v
```

---

## 4. Tích Hợp Tự Động Hóa Với Node-RED (Node-RED Flow)

Hệ thống có thể dễ dàng ghép nối với Node-RED thông qua các node `mqtt out` và `mqtt in` chuẩn:
1. Tạo một node **Inject** phát xung theo lịch phát thanh cố định (ví dụ 06:00 sáng và 17:00 chiều mỗi ngày).
2. Nối vào node **Function** gán `msg.payload = "preset=0"`.
3. Đưa vào node **MQTT Out** với topic `LOA CLOUD/command`.
4. Loa tại các cụm dân cư sẽ đồng loạt kích hoạt tiếp âm đúng khung giờ hoàn toàn tự động.
