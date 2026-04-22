# THÔNG TIN CẦN BỔ SUNG VÀO BÁO CÁO

> Tài liệu này liệt kê các nội dung **đã có trong mã nguồn (ArduinoCode + Backend + Frontend)** nhưng **chưa được trình bày** (hoặc trình bày thiếu) trong file PDF/LaTeX hiện tại. Với mỗi mục, đã chỉ rõ:
> - **Vị trí cần chèn** (Chương / Section / Subsection trong file `latex-btl/Chuong/...`).
> - **Nội dung cần thêm**.
> - **Ảnh cần chèn** (nếu có) — kèm gợi ý tên file ảnh.

---

## A. TỔNG QUAN CÁC THIẾU SÓT

| # | Phần thiếu | Ưu tiên |
|---|-----------|---------|
| 1 | Module LCD I2C 16x2 (mã có dùng `LiquidCrystal_I2C` nhưng tài liệu không nhắc) | Cao |
| 2 | Module Relay 2 kênh (chỉ nhắc thoáng qua, chưa có spec & sơ đồ chân) | Cao |
| 3 | Bảng phân ngưỡng AQI (1200/2000/3000 → GOOD/MOD/BAD/DANG) | Cao |
| 4 | Bộ lọc EWMA (Exponential Weighted Moving Average, alpha = 0.2) | Cao |
| 5 | Cơ chế phát hiện lỗi cảm biến (`MQ_MIN=50`, `MQ_MAX=3800`) | Cao |
| 6 | Cơ chế chống nhiễu I2C khi Relay đóng/ngắt (`flagResetLCD` re-init `Wire`+`lcd`) | Trung bình |
| 7 | Đồng bộ thời gian NTP (`pool.ntp.org`, `time.nist.gov`, GMT+7) | Trung bình |
| 8 | Kết nối MQTT bảo mật TLS (port 8883, `WiFiClientSecure`, `setInsecure`) | Cao |
| 9 | Cấu trúc topic MQTT đầy đủ (`air/data/{deviceId}` + legacy `air/data`, `air/control/{deviceId}` + legacy `air/control`) | Cao |
| 10 | Cấu hình chân GPIO chi tiết (ADC `32`,`33`; Relay `12`,`13`; I2C SDA `26`, SCL `27`) | Cao |
| 11 | **Toàn bộ chương Backend (Node.js + Express + Prisma + MySQL + Socket.IO + MQTT Pool)** chưa có | Rất cao |
| 12 | **Toàn bộ chương Frontend (React 19 + Vite + Tailwind + Zustand + Recharts + Socket.IO Client)** chưa có | Rất cao |
| 13 | Sơ đồ ERD database (User, Device, UserDevice, MqttConfig, Room, TelemetryData, ActivityLog) | Cao |
| 14 | Danh sách REST API endpoints (Auth, Devices, Rooms, Telemetry, Activity) | Cao |
| 15 | Cơ chế xác thực JWT (Bearer token, bcrypt hash mật khẩu, expire 24h) | Cao |
| 16 | Cơ chế Device Claim/Release bằng MAC + PIN | Cao |
| 17 | Cơ chế phát hiện thiết bị OFFLINE bằng telemetry timeout 5 giây | Trung bình |
| 18 | Hệ thống Activity Log (8 loại event, lưu vào DB và push real-time qua Socket.IO) | Trung bình |
| 19 | Sơ đồ tuần tự (sequence diagram) cho luồng Telemetry và luồng Control | Cao |
| 20 | Đặc tả các trang/màn hình Frontend (LoginPage, DashboardPage, OnboardingWizard 4 bước) | Cao |
| 21 | Chỉnh sửa các phần "định hướng" đã ghi trong báo cáo nhưng **chưa được implement** (AI Summary GPT-4o-mini, OTA Update, TLS đầy đủ) → cần điều chỉnh để khớp thực tế | Rất cao |

---

## B. CHI TIẾT BỔ SUNG THEO TỪNG SECTION

### B.1. Chương 1 – `Chuong/1_Gioi_thieu.tex`

#### B.1.1. Bổ sung vào Subsection **1.3 Lựa chọn kỹ thuật** (sau đoạn nói về MQTT, trước đoạn nói về OTA)

Thêm 1 ý mới về **kiến trúc 3 lớp**:

> **Kiến trúc 3 lớp (Three-Tier Architecture):** Hệ thống được tách rõ thành 3 tầng độc lập:
> - **Tầng Edge (ESP32 + cảm biến + relay + LCD):** thu thập, lọc tín hiệu và điều khiển trực tiếp cơ cấu chấp hành.
> - **Tầng Backend (Node.js + Express + Prisma + MySQL):** đóng vai trò trung gian, lưu trữ dữ liệu thời gian thực, quản lý người dùng/thiết bị/quyền sở hữu, và duy trì một MQTT Pool gồm nhiều client kết nối song song tới HiveMQ Broker.
> - **Tầng Frontend (React 19 + Vite + TailwindCSS):** dashboard giám sát đa thiết bị/đa phòng, đẩy dữ liệu real-time qua Socket.IO mà không cần polling liên tục.

#### B.1.2. Đính chính phần **AI Summary GPT-4o-mini** và **OTA**

Hiện tại Section "Yêu cầu chức năng – Quản trị nâng cao" trong [Chuong/2_Co_so_ly_thuyet.tex](latex-btl/Chuong/2_Co_so_ly_thuyet.tex) đang liệt kê 2 chức năng **chưa hề có trong mã nguồn**:
- *AI Summary qua GPT-4o-mini*
- *Cập nhật OTA*

→ **Cần một trong hai phương án:**
1. Chuyển 2 mục này sang phần **"Hướng phát triển"** trong [Chuong/6_Ket_luan.tex](latex-btl/Chuong/6_Ket_luan.tex) (đề xuất).
2. Hoặc giữ lại nhưng ghi rõ "Đã thiết kế nhưng chưa triển khai trong phạm vi BTL".

---

### B.2. Chương 2 – `Chuong/2_Co_so_ly_thuyet.tex`

#### B.2.1. Bổ sung vào Section **2.2 Phần cứng của hệ thống**

##### a) Thêm subsection mới: **2.2.X Module LCD I2C 16x2**

Hiện tại tài liệu **hoàn toàn không** đề cập tới LCD, nhưng ArduinoCode dùng `LiquidCrystal_I2C lcd(0x27, 16, 2)` với SDA=26, SCL=27.

Nội dung cần thêm:
- Mục đích: hiển thị tại chỗ (offline display) thông tin nồng độ khí, mức cảnh báo, trạng thái quạt và chế độ AUTO/MANUAL của từng phòng → giúp vận hành viên đứng tại trạm có thể kiểm tra mà không cần mở điện thoại/laptop.
- Thông số:
  - Loại: LCD ký tự 16x2 dùng module chuyển đổi I2C PCF8574.
  - Địa chỉ I2C: `0x27`.
  - Điện áp: 5V.
  - Giao tiếp: I2C 2 dây (SDA, SCL) → tiết kiệm chân GPIO so với LCD song song 16 chân.
- Cấu hình chân trong dự án: SDA → GPIO 26, SCL → GPIO 27 (đã `Wire.begin(SDA_PIN, SCL_PIN)` trong `setup()`).
- Logic hiển thị xoay vòng (paging) giữa Phòng 1 / Phòng 2 mỗi 2 giây trong `taskLCD`.

> 📷 **Cần chèn ảnh:** ảnh thực tế module LCD I2C 16x2 → `Hinh_ve/lcd_i2c_16x2.png`
>
> 📷 **Cần chèn ảnh:** ảnh chụp màn hình LCD đang hoạt động trong sản phẩm thật (chụp khi LCD đang hiển thị `R1:1234 MOD` ở dòng 1 và `F:OFF M:A` ở dòng 2) → `Hinh_ve/lcd_running.png`

##### b) Thêm subsection mới: **2.2.X Module Relay 2 kênh 5V**

Báo cáo có nhắc Relay vài chỗ trong Section 2.4 nhưng không có subsection riêng mô tả linh kiện. Cần bổ sung:
- Loại: Relay 2 kênh, kích hoạt mức thấp (Active LOW) hoặc cao tùy module.
- Thông số: cuộn dây 5V, tiếp điểm chịu tải tới 10A/250V AC hoặc 10A/30V DC, opto-coupler cách ly tín hiệu.
- Vai trò trong hệ thống: đóng/ngắt nguồn cấp tới 2 quạt 5V, chuyển trạng thái khi nồng độ khí vượt ngưỡng (chế độ AUTO) hoặc theo lệnh người dùng (chế độ MANUAL).
- Cấu hình chân trong dự án: IN1 → GPIO 12 (Phòng 1), IN2 → GPIO 13 (Phòng 2). Khởi tạo `pinMode(OUTPUT)` và mặc định `LOW` trong `setup()`.

> 📷 **Cần chèn ảnh:** ảnh module Relay 2 kênh → `Hinh_ve/relay_2_kenh.png`

##### c) Bổ sung **bảng cấu hình chân (Pin Mapping)** vào trước hoặc sau Subsection 2.2.9 (Sơ đồ tổng quan)

Thêm 1 bảng tổng hợp toàn bộ chân ESP32 đang dùng – hiện tài liệu chỉ vẽ block diagram chứ không liệt kê chân:

| Chức năng | Chân ESP32 | Module/Linh kiện |
|----------|-----------|------------------|
| ADC Phòng 1 | GPIO 32 | MQ-135 #1 (A0) |
| ADC Phòng 2 | GPIO 33 | MQ-135 #2 (A0) |
| Điều khiển Relay 1 | GPIO 12 | Relay IN1 → Quạt 1 |
| Điều khiển Relay 2 | GPIO 13 | Relay IN2 → Quạt 2 |
| I2C SDA | GPIO 26 | LCD I2C 16x2 |
| I2C SCL | GPIO 27 | LCD I2C 16x2 |
| Nguồn 5V | VIN / 5V | MQ-135, Relay, LCD, Quạt |
| GND | GND | Toàn hệ thống |

Caption: `Bảng cấu hình chân GPIO của ESP32 trong hệ thống`.

#### B.2.2. Bổ sung vào Section **2.3 Phần mềm của hệ thống**

##### a) Bổ sung vào Subsection **2.3.1 Tổng quan chức năng phần mềm**

Hiện đoạn này chỉ liệt kê 5 chức năng chung. Cần thêm:
- Hiển thị thông tin tại chỗ trên **LCD I2C 16x2** (xoay trang 2 phòng mỗi 2 giây).
- Đồng bộ thời gian qua **NTP** (`pool.ntp.org`, `time.nist.gov`, múi giờ GMT+7) trước khi bắt đầu publish dữ liệu.
- Lọc nhiễu cảm biến bằng **EWMA** (Exponential Weighted Moving Average) với hệ số `α = 0.2`.
- Phát hiện lỗi cảm biến qua dải hợp lệ `MQ_MIN ≤ raw ≤ MQ_MAX` (50–3800).
- Cơ chế chống nhiễu I2C khi Relay đóng/ngắt: re-init `Wire` và `lcd` ngay khi có sự kiện chuyển trạng thái quạt.

##### b) Thêm subsection mới: **2.3.X Phân ngưỡng chất lượng không khí (AQI Classification)**

Báo cáo chưa đề cập cụ thể ngưỡng phân loại. Thực tế trong code (`getLevel()`):

| Giá trị raw ADC | Mức (level) | Ý nghĩa | Hành động ở chế độ AUTO |
|----------------|-------------|---------|-------------------------|
| < 1200 | `GOOD` | Tốt | Tắt quạt |
| 1200 – 1999 | `MOD` | Trung bình | Tắt quạt |
| 2000 – 2999 | `BAD` | Kém | Bật quạt |
| ≥ 3000 | `DANG` | Nguy hiểm | Bật quạt |

Caption bảng: `Bảng phân ngưỡng chất lượng không khí và hành động điều khiển tự động`.

> Lưu ý: giá trị này là raw ADC 12-bit (0–4095) chứ chưa quy đổi sang ppm. Lý do: việc quy đổi MQ-135 sang ppm yêu cầu Calibration đường cong R0/Rs phức tạp và vượt quá phạm vi BTL — nhóm dùng raw ADC làm chỉ số tương đối để so ngưỡng.

##### c) Thêm subsection mới: **2.3.X Bộ lọc EWMA & Phát hiện lỗi cảm biến**

Nội dung:
- Công thức:
  ```
  filtered_n = α · raw_n + (1 - α) · filtered_{n-1}     với α = 0.2
  ```
- Mục đích: làm mượt dao động ngẫu nhiên của ADC (do nhiễu nguồn 5V dùng chung với Relay/Motor), tránh quạt bị "trip" liên tục khi giá trị dao động quanh ngưỡng.
- Cơ chế phát hiện lỗi: nếu giá trị nằm ngoài `[MQ_MIN, MQ_MAX] = [50, 3800]` → đánh dấu trường `error = true` và payload MQTT sẽ gắn `"sensor": "ERR"`.

##### d) Thêm subsection mới: **2.3.X Cơ chế chống nhiễu LCD do Relay**

Nội dung:
- Vấn đề thực tế: khi Relay đóng/ngắt cuộn cảm của quạt, dòng cảm ứng ngược (back-EMF) gây nhiễu lan ra bus I2C, khiến LCD bị "treo" hoặc hiển thị ký tự rác.
- Giải pháp: trong hàm `executeFanControl()`, ngay sau khi `digitalWrite()` thay đổi trạng thái Relay, đặt cờ `flagResetLCD = true`. Task LCD ở vòng lặp tiếp theo sẽ:
  ```
  Wire.end();
  delay(50);
  Wire.begin(SDA_PIN, SCL_PIN);
  lcd.init();
  lcd.backlight();
  ```
- Đây là điểm kỹ thuật đáng nhấn mạnh — minh chứng cho việc nhóm đã thực sự chạy thử và xử lý lỗi thực tế chứ không chỉ chạy mô phỏng.

##### e) Thêm subsection mới: **2.3.X Đồng bộ thời gian NTP**

Trong `setup()`, ESP32 gọi `configTime(7*3600, 0, "pool.ntp.org", "time.nist.gov")` và đợi tối đa 10 giây để lấy giờ thật trước khi bắt đầu publish — bảo đảm timestamp dữ liệu telemetry chính xác cho việc lưu vào DB và vẽ biểu đồ lịch sử.

##### f) Cập nhật Subsection **2.3.3 Thiết kế có sử dụng RTOS**

Phần `2.3.3.1 Phân chia Task` hiện liệt kê 4 task (Sensor / MQTT / Control / Processing). Tuy nhiên code thực tế dùng **4 task khác nhau** (không có Processing Task tách riêng):

| Task | Stack Size | Priority | Period | Chức năng |
|------|-----------|----------|--------|-----------|
| `taskSensor` | 4096 | 3 | 1000 ms | Đọc ADC, lọc EWMA, phát hiện lỗi, phân loại level |
| `taskControl` | 4096 | 2 | 1500 ms | FSM AUTO/MANUAL, xuất GPIO Relay |
| `taskLCD` | 4096 | 1 | 2000 ms | Xoay trang hiển thị 2 phòng, xử lý reset I2C |
| `taskMQTT` | 8192 | 1 | 50 ms (loop) + publish 3000 ms | Reconnect, callback, publish JSON |

Cần **chỉnh lại bảng** theo đúng code thực tế (tên hàm, stack, priority, chu kỳ) và giải thích lý do `taskMQTT` cần stack lớn hơn (8 KB do TLS/SSL handshake và buffer JSON 1024 byte).

##### g) Thêm subsection mới: **2.3.X Cơ chế đồng bộ tài nguyên dùng chung**

Cần làm rõ trong code đã dùng:
- `SemaphoreHandle_t dataMutex = xSemaphoreCreateMutex();` — bảo vệ struct `rooms[]` khỏi race condition giữa 4 task.
- `volatile bool flagResetLCD` — biến cờ giao tiếp giữa `executeFanControl()` (gọi từ taskControl/MQTT callback) và `taskLCD`.
- Mọi truy cập `rooms[i].xxx` đều được bao bởi `xSemaphoreTake(dataMutex, portMAX_DELAY)` / `xSemaphoreGive(dataMutex)`.

##### h) Thêm subsection mới: **2.3.X Giao thức truyền tin MQTT**

Đây là điểm quan trọng đang **thiếu chi tiết** trong báo cáo. Cần thêm các thông tin sau:

**Cấu hình broker:**
- Server: HiveMQ Cloud (`21b69e31ed5e4e7c86dbc3dc79814eab.s1.eu.hivemq.cloud`).
- Port: **8883 (MQTT over TLS)** — kết nối an toàn từ ESP32 dùng `WiFiClientSecure` với `setInsecure()` (bỏ qua xác thực CA do giới hạn flash của ESP32).
- Username/Password: cấu hình cứng trong firmware ESP32 và lưu trong bảng `mqtt_configs` ở backend (mỗi device có riêng).
- Buffer size client: 1024 byte.
- Client ID: `ESP32_<MAC_ADDRESS>` (đảm bảo duy nhất).
- Backend dùng port **8883 (mqtts://)** trong DB (thực tế đang đặt `mqtts://...:8883`).

**Cấu trúc Topic:**

| Topic | Hướng | Payload |
|-------|-------|---------|
| `air/data/{deviceId}` | ESP32 → Backend | JSON telemetry (chuẩn mới, dùng UUID device) |
| `air/data` | ESP32 → Backend | JSON telemetry (legacy, ESP32 hiện publish ở topic này) |
| `air/control/{deviceId}` | Backend → ESP32 | JSON command (chuẩn mới) |
| `air/control` | Backend → ESP32 | JSON command (legacy, ESP32 hiện subscribe ở topic này) |

**Payload Telemetry (ESP32 → Backend), QoS 0, mỗi 3 giây:**

```json
{
  "rooms": [
    {"id": 1, "value": 1319, "level": "MOD ", "fan": false, "mode": "MANUAL", "sensor": "OK"},
    {"id": 2, "value": 1758, "level": "MOD ", "fan": false, "mode": "MANUAL", "sensor": "OK"}
  ]
}
```

**Payload Control (Backend → ESP32), QoS 1:**

```json
{
  "room": 1,
  "mode": "MANUAL",
  "fan": false
}
```

##### i) Cập nhật Subsection **2.3.4 Cơ chế điều khiển từ người dùng**

Hiện tại bước 3 ghi `"room": 2, "mode": "manual"`. Tuy nhiên code thực tế:
- Backend gửi `"mode"` viết HOA: `"AUTO"` hoặc `"MANUAL"` (xem `MqttPool.sendCommand()`).
- Payload luôn gửi đủ 3 trường `room`, `mode`, `fan` cùng lúc (không tách thành 2 lệnh riêng như tài liệu mô tả).

→ Cần sửa lại đoạn `"manual"` → `"MANUAL"`, gộp 2 bước (set mode + set fan) thành 1 bước duy nhất với payload đầy đủ.

##### j) Sửa **2.3.2 Thiết kế không sử dụng RTOS**

Lưu ý: code thực tế **chỉ dùng RTOS**. Section 2.3.2 đang trình bày như là một thiết kế song song của nhóm. Cần:
- Hoặc làm rõ ngay từ đầu: "Phần này trình bày phương án thiết kế lý thuyết (không RTOS) để đối chiếu, phương án triển khai thực tế là RTOS được trình bày ở 2.3.3".
- Hoặc lược bớt 2.3.2 nếu muốn tinh giản.

#### B.2.3. Bổ sung Section MỚI: **2.4 Phần Backend của hệ thống** (chèn TRƯỚC Section "Phương pháp thực hiện và kết quả")

Đây là phần **lớn và quan trọng nhất** đang **hoàn toàn thiếu** trong báo cáo. Đề xuất nội dung gồm các subsection:

##### 2.4.1 Tổng quan kiến trúc Backend

- Stack công nghệ: **Node.js + Express 5 + Prisma ORM + MySQL 8 + Socket.IO + MQTT.js**.
- Vai trò: cầu nối giữa thiết bị Edge (ESP32) và giao diện người dùng (Frontend).
- 3 nhiệm vụ chính:
  1. **Maintain MQTT Pool** — duy trì N kết nối MQTT đồng thời (1 kết nối / 1 device đã được claim) tới HiveMQ Broker, lắng nghe telemetry và relay command.
  2. **Persist & Query Data** — lưu telemetry, log hoạt động, quản lý user/device/room/MQTT config qua Prisma + MySQL.
  3. **Real-time Push** — đẩy telemetry và activity log tới Frontend qua Socket.IO, không bắt FE phải polling.

> 📷 **Cần chèn ảnh:** sơ đồ kiến trúc Backend → `Hinh_ve/backend_architecture.png`
> (gồm 4 khối: ESP32 ↔ MQTT Broker ↔ Backend (với MqttPool, Express REST, Socket.IO, Prisma) ↔ MySQL ↔ Frontend)

##### 2.4.2 Sơ đồ ERD cơ sở dữ liệu

Lược đồ 7 bảng (theo file [Backend/prisma/schema.prisma](Backend/prisma/schema.prisma)):

| Bảng | Vai trò | Khóa chính |
|------|---------|-----------|
| `users` | Tài khoản người dùng (email, password_hash, full_name, role) | `id` (UUID) |
| `devices` | Thiết bị ESP32 (mac_address, claim_pin, device_name, status, last_connected) | `id` (UUID) |
| `user_devices` | Bảng N-N giữa user và device | `id`; UNIQUE(user_id, device_id) |
| `mqtt_configs` | Thông tin kết nối MQTT của từng device (broker_url, port, username, password) | `id`, UNIQUE `device_id` |
| `rooms` | Phòng giám sát (mỗi device có N phòng, hiện = 2) | `id`; UNIQUE(device_id, room_index) |
| `telemetry_data` | Dữ liệu time-series từng phòng (aqi_raw, aqi_level, fan_is_on, timestamp) | `id` (BigInt auto-inc) |
| `activity_logs` | Audit log mọi hành động của user và sự kiện hệ thống | `id` (BigInt auto-inc) |

> 📷 **Cần chèn ảnh:** sơ đồ ERD database → `Hinh_ve/database_erd.png`
> (Vẽ bằng dbdiagram.io, MySQL Workbench hoặc Prisma ERD generator)

##### 2.4.3 Hệ thống xác thực JWT

- User đăng ký (`POST /api/auth/register`): mật khẩu được hash bằng `bcryptjs` (salt 10), lưu `password_hash`.
- Đăng nhập (`POST /api/auth/login`): so sánh `bcrypt.compare()`, phát hành JWT có TTL 24h ký bằng `JWT_SECRET`.
- Mọi API ngoài auth đều bảo vệ bằng middleware `authenticate` đọc header `Authorization: Bearer <token>`.

##### 2.4.4 Danh sách REST API endpoints

Bảng tổng hợp (lấy từ các file [Backend/src/routes/*.js](Backend/src/routes/)):

| Method | Endpoint | Mô tả | Auth |
|--------|----------|-------|------|
| POST | `/api/auth/register` | Đăng ký user | Không |
| POST | `/api/auth/login` | Đăng nhập, trả JWT | Không |
| POST | `/api/auth/refresh` | Làm mới JWT | Có |
| GET | `/api/auth/profile` | Thông tin user hiện tại | Có |
| GET | `/api/devices` | Danh sách device user sở hữu | Có |
| GET | `/api/devices/:id` | Chi tiết device | Có |
| POST | `/api/devices/verify-claim` | Kiểm tra MAC + PIN có hợp lệ | Có |
| POST | `/api/devices/claim` | Claim device về user | Có |
| POST | `/api/devices/control` | Gửi lệnh điều khiển (mode, fan) | Có |
| POST | `/api/devices/:id/disconnect` | Ngắt kết nối MQTT của device | Có |
| POST | `/api/devices/:id/reconnect` | Kết nối lại MQTT | Có |
| POST | `/api/devices/:id/release` | Giải phóng device | Có |
| DELETE | `/api/devices/:id` | Xóa quyền sở hữu | Có |
| PUT | `/api/devices/:id/settings` | Cập nhật tên device, room, MQTT config | Có |
| GET | `/api/devices/:id/telemetry` | Telemetry theo device | Có |
| GET | `/api/rooms/:roomId` | Chi tiết room | Có |
| PUT | `/api/rooms/:roomId/mode` | Đổi mode AUTO/MANUAL | Có |
| PUT | `/api/rooms/:roomId/fan` | Bật/tắt quạt (chỉ MANUAL) | Có |
| GET | `/api/rooms/:roomId/telemetry` | Telemetry theo room | Có |
| GET | `/api/telemetry/room/:roomId` | Bản ghi telemetry mới nhất | Có |
| GET | `/api/telemetry/room/:roomId/history` | Lịch sử N giờ + thống kê min/max/avg | Có |
| GET | `/api/activity/user/:userId` | Activity log của user | Có |
| GET | `/api/activity/device/:deviceId` | Activity log của device | Có |
| GET | `/api/activity` | Toàn bộ activity log | Có |

##### 2.4.5 MQTT Pool – kết nối đa thiết bị

- Khởi tạo: khi backend start, query toàn bộ `devices` đã được claim (`user_devices` join) cùng `mqtt_config` → tạo 1 MQTT client cho từng device, lưu vào `Map<deviceId, mqttClient>`.
- Subscribe: mỗi client subscribe `air/data/{deviceId}` + topic legacy `air/data`.
- Khi nhận message → parse JSON → ghi vào bảng `telemetry_data` (1 dòng / room) → cập nhật `rooms.current_mode`, `rooms.current_fan_status` → emit Socket.IO event `telemetry_update` cho mọi FE đang kết nối.
- Khi user gửi lệnh: backend `publish()` payload JSON tới cả 2 topic `air/control/{deviceId}` và `air/control` (QoS 1).
- Cơ chế **Telemetry Timeout** 5 giây: nếu không nhận được dữ liệu telemetry trong 5 giây liên tiếp → backend tự động đặt `device.status = OFFLINE` và emit `activity_log` event `DEVICE_OFFLINE`. Khi telemetry quay lại → tự động đánh dấu ONLINE.

##### 2.4.6 Real-time với Socket.IO

Các sự kiện server emit:

| Event | Payload | Khi nào emit |
|-------|---------|--------------|
| `telemetry_update` | `{deviceId, rooms[], timestamp}` | Mỗi lần ESP32 publish telemetry mới |
| `telemetry-update` (legacy) | `{deviceId, data, timestamp}` | Backward compatibility |
| `activity_log` | `{deviceId, eventType, description, timestamp}` | DEVICE_ONLINE / DEVICE_OFFLINE / MODE_CHANGED / FAN_TOGGLED / CONTROL_SENT / DEVICE_CLAIMED / DEVICE_RELEASED / DEVICE_REMOVED / SETTINGS_UPDATED |

##### 2.4.7 Activity Log – các loại sự kiện

Bảng các `event_type` được hệ thống sinh ra:

| Event Type | Sinh từ | Ý nghĩa |
|------------|---------|---------|
| `DEVICE_CLAIMED` | API claim | User đăng ký một device mới |
| `DEVICE_RELEASED` | API release | Giải phóng device (gỡ liên kết) |
| `DEVICE_REMOVED` | API delete | Xóa quyền sở hữu device |
| `DEVICE_ONLINE` | MQTT connect / telemetry timeout reset | Device lên |
| `DEVICE_OFFLINE` | MQTT offline / telemetry timeout fire | Device offline |
| `MODE_CHANGED` | API PUT room mode | Đổi AUTO ↔ MANUAL |
| `FAN_TOGGLED` | API PUT room fan | Bật/tắt quạt MANUAL |
| `CONTROL_SENT` | API POST device control | Lệnh điều khiển đã gửi |
| `SETTINGS_UPDATED` | API PUT settings | Đổi tên device/room hoặc MQTT config |

> 📷 **Cần chèn ảnh:** sơ đồ tuần tự (Sequence Diagram) cho **Luồng Telemetry**: `ESP32 → HiveMQ → MqttPool → Prisma (insert) → Socket.IO emit → Frontend cập nhật UI` → `Hinh_ve/sequence_telemetry.png`
>
> 📷 **Cần chèn ảnh:** sơ đồ tuần tự (Sequence Diagram) cho **Luồng Control**: `Frontend (click button) → POST /api/devices/control → DeviceController → MqttPool.publish → HiveMQ → ESP32 callback → executeFanControl → digitalWrite Relay` → `Hinh_ve/sequence_control.png`

#### B.2.4. Bổ sung Section MỚI: **2.5 Phần Frontend của hệ thống** (chèn sau Section Backend, trước "Phương pháp thực hiện và kết quả")

##### 2.5.1 Tổng quan kiến trúc Frontend

- Stack: **React 19 + Vite 5 + TailwindCSS 4 + Zustand 5 (state) + React Router 7 + Recharts 3 (biểu đồ) + Socket.IO Client 4 + Axios 1 + Framer Motion + Sonner (toast) + Lucide Icons**.
- Mô hình SPA, routing client-side, không SSR.
- 2 route chính:
  - `/login` — trang đăng nhập / đăng ký.
  - `/` — Dashboard (yêu cầu authenticated, được bao bởi `ProtectedRoute`).

##### 2.5.2 Quản lý state với Zustand

- `useAuthStore` — lưu `user`, `token` (persist localStorage qua middleware `persist`), action `login`, `register`, `logout`.
- `useDeviceStore` — lưu `devices[]`, `activities[]`, action `fetchDevices`, `fetchActivities`, `verifyClaim`, `claimDevice`, `setRoomMode`, `setRoomFan`, `applyTelemetryUpdate` (optimistic update từ Socket.IO event), `appendActivity`, `updateDeviceStatus`.

##### 2.5.3 Các trang/màn hình

**LoginPage** ([Frontend/src/pages/LoginPage.jsx](Frontend/src/pages/LoginPage.jsx)):
- Form đăng nhập / đăng ký (toggle giữa 2 chế độ).
- Validate email/password tối thiểu, hiển thị toast khi thành công/thất bại.
- Sau khi login, redirect tới `/`.

> 📷 **Cần chèn ảnh:** screenshot trang Login → `Hinh_ve/screen_login.png`

**DashboardPage** ([Frontend/src/pages/DashboardPage.jsx](Frontend/src/pages/DashboardPage.jsx)):
- Bao gồm `Navbar` (logo + email user + nút Logout) và `Sidebar` (Dashboard / Settings / Add Device).
- Khi mount: gọi `fetchDevices()` + `fetchActivities()`, sau đó tạo `socket = createSocketClient()` và đăng ký 5 listener (`connect`, `disconnect`, `connect_error`, `telemetry_update`, `activity_log`).
- Có cơ chế **fallback polling 5 giây** khi socket disconnect: tự động `fetchDevices()` lại để đồng bộ trạng thái.
- Render danh sách `DeviceSection` (mỗi device 1 thẻ) — nếu rỗng thì hiện "No devices yet" + nút Add Device.

> 📷 **Cần chèn ảnh:** screenshot Dashboard với 1 device + 2 room đang chạy → `Hinh_ve/screen_dashboard.png`

##### 2.5.4 Các component nghiệp vụ chính

**DeviceSection** ([Frontend/src/components/dashboard/DeviceSection.jsx](Frontend/src/components/dashboard/DeviceSection.jsx)):
- Header: tên device, MAC address, status badge ONLINE/OFFLINE.
- 5 nút action: Edit Device Name / Edit Room Names / Disconnect (nếu ONLINE) hoặc Reconnect (nếu OFFLINE) / Delete Device.
- Grid 2 cột chứa các `RoomCard`.

**RoomCard** ([Frontend/src/components/dashboard/RoomCard.jsx](Frontend/src/components/dashboard/RoomCard.jsx)):
- Hiển thị: tên phòng, giá trị raw AQI, badge mức (GOOD/MOD/BAD/DANG với 4 màu khác nhau), chấm tròn ON/OFF.
- Sparkline mini-chart (Recharts) thể hiện xu hướng 20 điểm gần nhất, đổi màu theo level.
- `SegmentedControl` chọn mode AUTO ↔ MANUAL.
- `Toggle` bật/tắt quạt (disabled khi đang ở AUTO).
- Hiển thị trạng thái sensor OK/ERR.
- Có `React.memo` với custom comparator để tránh re-render thừa.

> 📷 **Cần chèn ảnh:** screenshot 1 RoomCard zoom-in + chú thích các thành phần → `Hinh_ve/screen_room_card.png`

**OnboardingWizardModal** ([Frontend/src/components/dashboard/OnboardingWizardModal.jsx](Frontend/src/components/dashboard/OnboardingWizardModal.jsx)):
- Wizard **4 bước**:
  1. **Verify** — nhập MAC + PIN, gọi `POST /api/devices/verify-claim`.
  2. **Connect** — spinner chờ backend xác nhận.
  3. **Configure** — hiển thị tên device và danh sách room có sẵn từ DB, cho phép sửa tên device.
  4. **Complete** — hiện thông báo thành công và đóng modal sau 500ms.

> 📷 **Cần chèn ảnh:** screenshot 4 bước onboarding (có thể ghép 4 ảnh trong 1 figure subcaption) → `Hinh_ve/screen_onboarding_wizard.png`

**ActivityFeed** ([Frontend/src/components/layout/ActivityFeed.jsx](Frontend/src/components/layout/ActivityFeed.jsx)):
- Danh sách event log real-time, mỗi loại event có icon và màu border riêng (xanh = ONLINE, đỏ = OFFLINE, cyan = FAN, ...).
- Hiện đang **disabled** trong DashboardPage (xem comment `{/* Temporarily disabled activity feed */}`). → Trong báo cáo có thể nhắc đây là tính năng đã có nhưng tạm ẩn để tinh giản UI.

##### 2.5.5 Cập nhật real-time qua Socket.IO

- Khởi tạo `createSocketClient()` ([Frontend/src/utils/socket.js](Frontend/src/utils/socket.js)) — ép `transports: ["polling"]` để tránh lỗi WebSocket upgrade ở dev local, có reconnection vô hạn với backoff 1–5s.
- Listener `telemetry_update`: gọi `applyTelemetryUpdate(payload)` → store cập nhật `value`, `level`, `fan`, `mode`, `sensor`, đẩy điểm mới vào `trend` (giới hạn 20 điểm).
- Listener `activity_log`: append vào activity feed; nếu là DEVICE_ONLINE/OFFLINE → cũng cập nhật trực tiếp `device.status` mà không cần re-fetch.

> 📷 **Cần chèn ảnh:** sơ đồ flow cập nhật real-time từ ESP32 → Frontend (kết hợp luồng telemetry đã nêu ở B.2.3 nhưng nhấn mạnh phía FE) → `Hinh_ve/realtime_flow.png` *(có thể tái dùng `sequence_telemetry.png`)*

#### B.2.5. Bổ sung vào Section **2.6 Phương pháp thực hiện và kết quả** (đổi số từ 2.4 → 2.6 sau khi thêm Backend & Frontend)

##### a) Thêm subsection **2.6.1.X Triển khai Backend**

- Cài đặt MySQL 8, tạo DB `air_quality_db`.
- Chạy `npm install`, copy `.env.example → .env`, cấu hình `DATABASE_URL`, `JWT_SECRET`.
- Chạy `npm run prisma:migrate` để apply migration.
- (Tùy chọn) Chạy script `init-db.sql` để seed 5 device mẫu + 2 user test (xem [Backend/init-db.sql](Backend/init-db.sql)).
- Khởi động `npm run dev` (nodemon) — server chạy ở port 5000.

##### b) Thêm subsection **2.6.1.X Triển khai Frontend**

- Cài đặt Node 18+, vào thư mục `Frontend`, `npm install`.
- Cấu hình file `.env` chứa `VITE_API_URL=http://localhost:5000/api`.
- Chạy `npm run dev` — Vite dev server mặc định port 5173.

##### c) Bổ sung vào **2.6.3 Thử nghiệm khả năng vận hành**

Thêm các kịch bản test:
- **Test multi-tenancy:** 2 user khác nhau cùng đăng ký và claim 2 device khác nhau → mỗi user chỉ thấy device của mình.
- **Test multi-device:** 1 user claim cùng lúc 2+ device → MqttPool tạo 2+ kết nối song song, dashboard hiện 2+ DeviceSection.
- **Test telemetry timeout:** rút nguồn ESP32 → sau 5 giây, FE tự động đổi badge sang OFFLINE, activity log xuất hiện sự kiện DEVICE_OFFLINE.
- **Test mất kết nối FE → BE:** kill backend → FE tự fallback polling và hiện toast "Realtime channel unstable", khi BE chạy lại → tự reconnect socket.
- **Test phân quyền Mode:** ở chế độ AUTO, click toggle quạt → backend trả lỗi 400 "Cannot toggle fan while mode is AUTO", FE hiển thị toast cảnh báo (cũng có chặn phía FE trong `setRoomFan`).

> 📷 **Cần chèn ảnh:** screenshot dashboard khi 1 device chuyển sang OFFLINE (badge đỏ) → `Hinh_ve/screen_offline.png`
>
> 📷 **Cần chèn ảnh:** screenshot biểu đồ Sparkline khi nồng độ tăng đột biến (đường gấp khúc đi lên) → `Hinh_ve/screen_sparkline_spike.png`

##### d) Thêm subsection **2.6.4 So sánh với yêu cầu phi chức năng**

Bảng tổng hợp đối chiếu yêu cầu (đã ghi ở 2.1.3) với thực tế đo được:

| Yêu cầu phi chức năng | Mục tiêu | Thực tế đo được | Đạt? |
|----------------------|----------|-----------------|------|
| Độ trễ telemetry → Dashboard | < 3 s | ~3 s (chu kỳ publish 3s) + < 100ms qua Socket.IO | ✓ |
| Phản hồi kích hoạt quạt | < 5 s | < 1.5 s (chu kỳ taskControl 1.5s) | ✓ |
| Tỷ lệ truyền gói thành công | ≥ 98% | (cần đo thực tế và điền) | ? |
| Bảo mật MQTT | TLS/SSL | TLS port 8883 (skip CA verify) | ✓ một phần |
| Bảo mật API | JWT | JWT 24h + bcrypt | ✓ |
| Nguồn dự phòng UPS | Duy trì khi mất điện | YX850 chuyển sang pin 18650 < 1s | ✓ |

---

### B.3. Chương 3 – `Chuong/6_Ket_luan.tex`

#### B.3.1. Cập nhật **Subsection 3.1 Kết luận**

Bổ sung các kết quả nổi bật **đang thiếu**:
- Đã xây dựng được **hệ thống IoT đa thiết bị, đa người dùng** với REST API + WebSocket real-time.
- Hỗ trợ **claim/release device** linh hoạt qua MAC + PIN.
- Có **audit log đầy đủ** mọi hành động và sự kiện hệ thống.
- **Frontend SPA hiện đại** với cập nhật real-time, sparkline, onboarding wizard.

#### B.3.2. Cập nhật **Subsection 3.2 Hướng phát triển**

Bổ sung (chuyển 2 ý từ Chương 2 sang đây):
- Tích hợp **AI Summary tóm tắt mức độ ô nhiễm hàng giờ** bằng GPT-4o-mini hoặc model nội suy thời gian thực.
- Hỗ trợ **OTA Update firmware** từ xa qua HTTPS, giúp hiệu chỉnh ngưỡng cảm biến mà không cần tháo lắp.
- Hiện tại MQTT đang dùng `setInsecure()` (ESP32 bỏ qua xác thực CA) — cần **upload CA certificate của HiveMQ** vào ESP32 để bảo mật đầy đủ.
- **Tự động tính ppm** từ raw ADC bằng đường cong calibration R0/Rs của MQ-135.
- Thêm trang **Settings** và **History** ở Frontend (hiện sidebar có nút Settings nhưng chưa có route tương ứng).

---

### B.4. Tóm tắt nội dung – `Chuong/0_3_Tom_tat_noi_dung.tex`

Cần bổ sung 1 đoạn nói rõ về **3 tầng kiến trúc** vì hiện tại tóm tắt chỉ nhắc tới ESP32 + MQTT + Dashboard, chưa nói tới Backend Node.js, MySQL, Socket.IO. Đề xuất thêm:

> Hệ thống được tổ chức theo kiến trúc 3 tầng: **tầng Edge** sử dụng ESP32 với cảm biến MQ-135 và LCD I2C; **tầng Backend** xây dựng trên Node.js/Express kết hợp Prisma ORM với cơ sở dữ liệu MySQL, đóng vai trò vận hành MQTT Pool đa thiết bị và đẩy dữ liệu real-time qua Socket.IO; **tầng Frontend** là Single Page Application viết bằng React 19 + TailwindCSS hỗ trợ giám sát đa phòng đa thiết bị, biểu đồ xu hướng và wizard claim thiết bị.

---

## C. CÁC ẢNH CẦN CHUẨN BỊ MỚI (TỔNG HỢP)

> Toàn bộ ảnh nên đặt vào thư mục `latex-btl/Hinh_ve/`. Khi chèn dùng cú pháp đã quen:
>
> ```latex
> \begin{figure}[H]
> \centering
> \includegraphics[width=0.85\linewidth]{Hinh_ve/<ten_anh>.png}
> \caption{<chú thích>}
> \label{fig:<label>}
> \end{figure}
> ```

| # | Tên file đề xuất | Loại ảnh | Mục đích | Dùng ở section |
|---|-----------------|----------|----------|----------------|
| 1 | `lcd_i2c_16x2.png` | Ảnh sản phẩm linh kiện | Module LCD I2C 16x2 | 2.2.X (mới) |
| 2 | `lcd_running.png` | Ảnh chụp thực tế | LCD đang hiển thị thông tin phòng | 2.2.X (mới) |
| 3 | `relay_2_kenh.png` | Ảnh sản phẩm linh kiện | Module Relay 2 kênh 5V | 2.2.X (mới) |
| 4 | `wiring_diagram.png` | Sơ đồ Fritzing/KiCad | Sơ đồ đi dây chi tiết toàn hệ thống (chân ESP32 → Relay/MQ-135/LCD) | 2.2.9 (bổ sung kèm sơ đồ tổng quan) |
| 5 | `freertos_task_diagram.png` | Sơ đồ khối | Quan hệ giữa 4 task RTOS + Mutex + Hardware peripherals | 2.3.3 |
| 6 | `mqtt_topic_flow.png` | Sơ đồ luồng | Topic publish/subscribe giữa ESP32 ↔ Backend | 2.3.X (mới — MQTT) |
| 7 | `backend_architecture.png` | Sơ đồ kiến trúc | 4 khối: ESP32, HiveMQ, Backend (MqttPool/Express/Socket.IO/Prisma), MySQL, Frontend | 2.4.1 (mới) |
| 8 | `database_erd.png` | Sơ đồ ERD | 7 bảng + quan hệ FK | 2.4.2 (mới) |
| 9 | `sequence_telemetry.png` | Sequence diagram | Luồng telemetry ESP32 → FE | 2.4.7 (mới) |
| 10 | `sequence_control.png` | Sequence diagram | Luồng control FE → ESP32 | 2.4.7 (mới) |
| 11 | `screen_login.png` | Screenshot UI | Trang đăng nhập | 2.5.3 (mới) |
| 12 | `screen_dashboard.png` | Screenshot UI | Dashboard chính | 2.5.3 (mới) |
| 13 | `screen_room_card.png` | Screenshot UI zoom | Một RoomCard kèm chú thích | 2.5.4 (mới) |
| 14 | `screen_onboarding_wizard.png` | Screenshot UI ghép 4 ảnh | 4 bước wizard claim device | 2.5.4 (mới) |
| 15 | `screen_offline.png` | Screenshot UI | Device chuyển trạng thái OFFLINE | 2.6.3 (bổ sung) |
| 16 | `screen_sparkline_spike.png` | Screenshot UI | Sparkline khi AQI tăng đột biến | 2.6.3 (bổ sung) |

---

## D. GHI CHÚ KỸ THUẬT KHÁC (NẾU MUỐN BỔ SUNG ĐẦY ĐỦ)

1. **Đặt mật khẩu/credentials vào báo cáo:** WiFi password và MQTT password thực tế đang để cứng trong `final_code.ino` và `init-db.sql`. **Khi viết báo cáo nên ẩn/thay bằng `<wifi_password>`, `<mqtt_password>`** để tránh lộ credential trong PDF công khai.
2. **Phiên bản thư viện cụ thể** (nếu muốn liệt kê dependency) đã có sẵn trong [Backend/package.json](Backend/package.json) và [Frontend/package.json](Frontend/package.json) — có thể bổ sung 1 bảng cuối phụ lục.
3. **Phụ lục mã nguồn:** hiện đã có file `Phu_luc_A.pdf` rời. Nếu muốn đầy đủ hơn, có thể tổ chức:
   - Phụ lục A: Code Arduino (`final_code.ino`).
   - Phụ lục B: Schema Prisma + danh sách REST API.
   - Phụ lục C: Cấu trúc thư mục Frontend + các component chính.
4. **Có sai lệch tên giữa "ROOM" và "PHÒNG":** trong báo cáo dùng "phòng" còn DB và FE dùng "room/zone" — có thể bổ sung 1 chú thích nhỏ ở đầu Section 2.4 để thống nhất thuật ngữ.

---

## E. CHECKLIST NHANH KHI VIẾT BỔ SUNG

- [ ] Chuẩn bị 16 ảnh ở mục C.
- [ ] Thêm 4 subsection mới trong Section 2.2 (LCD, Relay, Pin Mapping bảng).
- [ ] Bổ sung/sửa 6+ subsection trong Section 2.3 (NTP, EWMA, AQI threshold, anti-noise LCD, MQTT TLS, sửa bảng RTOS task).
- [ ] **Viết MỚI hoàn toàn Section 2.4 (Backend) — 7 subsection.**
- [ ] **Viết MỚI hoàn toàn Section 2.5 (Frontend) — 5 subsection.**
- [ ] Bổ sung bảng so sánh yêu cầu phi chức năng trong Section 2.6.
- [ ] Cập nhật Tóm tắt nội dung và Kết luận để khớp kiến trúc 3 tầng.
- [ ] Quyết định: giữ AI Summary + OTA ở "Yêu cầu chức năng" hay chuyển sang "Hướng phát triển".
- [ ] Ẩn credentials thật trước khi in PDF công khai.