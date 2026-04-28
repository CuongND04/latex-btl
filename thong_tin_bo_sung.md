# THÔNG TIN CẦN BỔ SUNG VÀO BÁO CÁO

> Tài liệu này liệt kê các nội dung **đã có trong mã nguồn (ArduinoCode + Backend + Frontend)** nhưng **chưa được trình bày** (hoặc trình bày thiếu) trong file PDF/LaTeX hiện tại. Với mỗi mục, đã chỉ rõ:
>
> - **Vị trí cần chèn** (Chương / Section / Subsection trong file `latex-btl/Chuong/...`).
> - **Nội dung cần thêm**.
> - **Ảnh cần chèn** (nếu có) — kèm gợi ý tên file ảnh.

---

## A. TỔNG QUAN CÁC THIẾU SÓT

| #   | Phần thiếu                                                                                                                                                   | Ưu tiên    |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- |
| 1   | Module LCD I2C 16x2 (mã có dùng `LiquidCrystal_I2C` nhưng tài liệu không nhắc)                                                                               | Cao        |
| 2   | Module Relay 2 kênh (chỉ nhắc thoáng qua, chưa có spec & sơ đồ chân)                                                                                         | Cao        |
| 3   | Bảng phân ngưỡng AQI (1200/2000/3000 → GOOD/MOD/BAD/DANG)                                                                                                    | Cao        |
| 4   | Bộ lọc EWMA (Exponential Weighted Moving Average, alpha = 0.2)                                                                                               | Cao        |
| 5   | Cơ chế phát hiện lỗi cảm biến (`MQ_MIN=50`, `MQ_MAX=3800`)                                                                                                   | Cao        |
| 6   | Cơ chế chống nhiễu I2C khi Relay đóng/ngắt (`flagResetLCD` re-init `Wire`+`lcd`)                                                                             | Trung bình |
| 7   | Đồng bộ thời gian NTP (`pool.ntp.org`, `time.nist.gov`, GMT+7)                                                                                               | Trung bình |
| 8   | Kết nối MQTT bảo mật TLS (port 8883, `WiFiClientSecure`, `setInsecure`)                                                                                      | Cao        |
| 9   | Cấu trúc topic MQTT đầy đủ (`air/data/{deviceId}` + legacy `air/data`, `air/control/{deviceId}` + legacy `air/control`)                                      | Cao        |
| 10  | Cấu hình chân GPIO chi tiết (ADC `32`,`33`; Relay `12`,`13`; I2C SDA `26`, SCL `27`)                                                                         | Cao        |
| 11  | **Toàn bộ chương Backend (Node.js + Express + Prisma + MySQL + Socket.IO + MQTT Pool)** chưa có                                                              | Rất cao    |
| 12  | **Toàn bộ chương Frontend (React 19 + Vite + Tailwind + Zustand + Recharts + Socket.IO Client)** chưa có                                                     | Rất cao    |
| 13  | Sơ đồ ERD database (User, Device, UserDevice, MqttConfig, Room, TelemetryData, ActivityLog)                                                                  | Cao        |
| 14  | Danh sách REST API endpoints (Auth, Devices, Rooms, Telemetry, Activity)                                                                                     | Cao        |
| 15  | Cơ chế xác thực JWT (Bearer token, bcrypt hash mật khẩu, expire 24h)                                                                                         | Cao        |
| 16  | Cơ chế Device Claim/Release bằng MAC + PIN                                                                                                                   | Cao        |
| 17  | Cơ chế phát hiện thiết bị OFFLINE bằng telemetry timeout 5 giây                                                                                              | Trung bình |
| 18  | Hệ thống Activity Log (8 loại event, lưu vào DB và push real-time qua Socket.IO)                                                                             | Trung bình |
| 19  | Sơ đồ tuần tự (sequence diagram) cho luồng Telemetry và luồng Control                                                                                        | Cao        |
| 20  | Đặc tả các trang/màn hình Frontend (LoginPage, DashboardPage, OnboardingWizard 4 bước)                                                                       | Cao        |
| 21  | Chỉnh sửa các phần "định hướng" đã ghi trong báo cáo nhưng **chưa được implement** (AI Summary GPT-4o-mini, TLS đầy đủ) → cần điều chỉnh để khớp thực tế     | Rất cao    |
| 22  | **Emergency Mode** với 3 nút bấm vật lý + ISR debounce + Task ưu tiên cao nhất (Prio 6) — **chưa có dòng nào** trong báo cáo                                 | Rất cao    |
| 23  | **Watchdog Timer (WDT)** + **RTC backup memory** để khôi phục trạng thái sau reset                                                                           | Cao        |
| 24  | **OTA Firmware Update** đã được implement đầy đủ (Arduino + Backend + Frontend) — phải MOVE khỏi "Hướng phát triển" sang chương chính                        | Rất cao    |
| 25  | Module **ADS1115 16-bit ADC** (firmware đọc MQ-135 qua I2C, KHÔNG dùng ADC nội ESP32)                                                                        | Rất cao    |
| 26  | **Servo SG90** điều khiển cửa sổ — vai trò trong hệ thống chưa nêu                                                                                           | Cao        |
| 27  | **Còi SFM-27** điều khiển qua transistor — chưa có subsection riêng                                                                                          | Cao        |
| 28  | 3 Mutex chuyên biệt: `dataMutex`, `i2cMutex`, `mqttMutex` (báo cáo chỉ nhắc 1 mutex chung)                                                                   | Cao        |
| 29  | Logic safety lockout: tự động ép AUTO→MANUAL khi mức khí lên DANG                                                                                            | Cao        |
| 30  | WiFi auto-reconnect trong `taskMQTT` + cơ chế MQTT retry 5 giây                                                                                              | Trung bình |
| 31  | **Pin mapping thực tế đã thay đổi**: Relay 32/33 (không phải 12/13), I2C SDA/SCL = 21/22 (không phải 26/27), Servo 26/27, Buzzer 25/4, Nút khẩn cấp 34/35/23 | Rất cao    |

---

## B. CHI TIẾT BỔ SUNG THEO TỪNG SECTION

### B.1. Chương 1 – `Chuong/1_Gioi_thieu.tex`

#### B.1.1. Bổ sung vào Subsection **1.3 Lựa chọn kỹ thuật** (sau đoạn nói về MQTT, trước đoạn nói về OTA)

Thêm 1 ý mới về **kiến trúc 3 lớp**:

> **Kiến trúc 3 lớp (Three-Tier Architecture):** Hệ thống được tách rõ thành 3 tầng độc lập:
>
> - **Tầng Edge (ESP32 + cảm biến + relay + LCD):** thu thập, lọc tín hiệu và điều khiển trực tiếp cơ cấu chấp hành.
> - **Tầng Backend (Node.js + Express + Prisma + MySQL):** đóng vai trò trung gian, lưu trữ dữ liệu thời gian thực, quản lý người dùng/thiết bị/quyền sở hữu, và duy trì một MQTT Pool gồm nhiều client kết nối song song tới HiveMQ Broker.
> - **Tầng Frontend (React 19 + Vite + TailwindCSS):** dashboard giám sát đa thiết bị/đa phòng, đẩy dữ liệu real-time qua Socket.IO mà không cần polling liên tục.

#### B.1.2. Đính chính phần **AI Summary GPT-4o-mini** và **OTA**

Hiện tại Section "Yêu cầu chức năng – Quản trị nâng cao" trong [Chuong/2_Co_so_ly_thuyet.tex](latex-btl/Chuong/2_Co_so_ly_thuyet.tex) đang liệt kê 2 chức năng:

- _AI Summary qua GPT-4o-mini_ — **CHƯA có trong mã nguồn** → đề xuất chuyển sang "Hướng phát triển" hoặc ghi rõ "Đã thiết kế nhưng chưa triển khai".
- _Cập nhật OTA_ — **ĐÃ ĐƯỢC IMPLEMENT ĐẦY ĐỦ** trong firmware mới [Arduino/newestVersion.ino:275-409](Arduino/newestVersion.ino#L275-L409), Backend [firmwareController.js](Backend/src/controllers/firmwareController.js), Frontend [OTAManagement.jsx](Frontend/src/pages/OTAManagement.jsx). → **GIỮ LẠI** ở phần chính, viết hẳn 1 section riêng (xem **B.2.6** bên dưới).

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

##### b-bis) Thêm subsection mới: **2.2.X Module ADC ADS1115 16-bit** _(MỚI — bắt buộc theo firmware mới)_

Firmware [Arduino/newestVersion.ino:10](Arduino/newestVersion.ino#L10) khởi tạo `Adafruit_ADS1115 ads;` và đọc cảm biến qua `ads.readADC_SingleEnded(channel)` — KHÔNG dùng `analogRead()` của ESP32 nữa.

Nội dung cần thêm:

- Lý do chuyển sang ADS1115: ADC nội ESP32 chỉ 12-bit, nhiễu cao ở dải gần 0V do nguồn dùng chung với Relay, không tuyến tính. ADS1115 16-bit có PGA + lọc sigma-delta cho độ chính xác cao hơn nhiều.
- Thông số: 16-bit, 4 kênh single-ended (hoặc 2 kênh vi sai), I2C, dải đo 0-5V (PGA × 2/3), tốc độ 8-860 SPS.
- Cấu hình trong dự án: dùng kênh A0 (Phòng 1) và A1 (Phòng 2), địa chỉ I2C mặc định 0x48, chia chung bus I2C với LCD (do đó cần `i2cMutex`).
- Trong code: kết quả 16-bit được map về thang 12-bit (0-4095) để giữ nguyên các ngưỡng AQI cũ:
  ```cpp
  int16_t adc_val = ads.readADC_SingleEnded(adsChannels[i]);
  rawValues[i] = map(adc_val, 0, 26666, 0, 4095);
  ```

> 📷 **Cần chèn ảnh:** module ADS1115 → `Hinh_ve/ads1115_module.png` (đã có)

##### b-tris) Thêm subsection mới: **2.2.X Servo SG90 (điều khiển cửa sổ)** _(MỚI — bắt buộc theo firmware mới)_

Mỗi phòng có 1 servo SG90 điều khiển góc mở cửa sổ thông gió (0°-180°), kết nối GPIO 26 (R1) và GPIO 27 (R2). Code trong [Arduino/newestVersion.ino:181-191](Arduino/newestVersion.ino#L181-L191) và [line 452](Arduino/newestVersion.ino#L452):

- **Chế độ AUTO:** Góc cửa được map tuyến tính theo nồng độ khí: `map(rooms[i].raw, 400, 3000, 0, 180)`. Nồng độ càng cao → cửa mở càng rộng để tăng tốc thoát khí.
- **Chế độ EMERGENCY:** Cửa mở **toàn bộ 180°** ngay lập tức.
- **Chế độ MANUAL:** Người dùng chỉnh slider 0-100° từ Frontend (xem [RoomCard.jsx](Frontend/src/components/dashboard/RoomCard.jsx)).
- Dùng thư viện `ESP32Servo`, cần allocateTimer 0-3 trong setup, frequency 50Hz, pulse width 500-2400μs.

> 📷 **Cần chèn ảnh:** servo SG90 → `Hinh_ve/servo_sg90.png` (đã có)

##### b-quart) Thêm subsection mới: **2.2.X Còi SFM-27 (cảnh báo âm thanh)** _(MỚI — bắt buộc theo firmware mới)_

Mỗi phòng có 1 còi SFM-27 (active buzzer 3-24V) gắn vào GPIO 25 (R1) và GPIO 4 (R2) thông qua transistor S8050 (vì còi tiêu thụ tới 15mA, vượt khả năng cấp dòng trực tiếp của GPIO ESP32 3.3V).

Vai trò trong hệ thống:

- Phát còi báo động khi mức khí lên DANG (chế độ AUTO).
- Phát còi liên tục khi nút khẩn cấp được kích hoạt (chế độ EMERGENCY).
- MANUAL: người dùng có thể bật/tắt còi từ Dashboard.

> 📷 **Cần chèn ảnh:** còi SFM-27 + sơ đồ driver transistor → `Hinh_ve/buzzer_sfm27.png` (đã có) + cần vẽ thêm schematic driver

##### b-quint) Thêm subsection mới: **2.2.X Hệ thống nút bấm khẩn cấp** _(MỚI — đặc trưng quan trọng nhất của hệ thống)_

Hệ thống có **3 nút bấm vật lý** dành cho vận hành viên trong các tình huống bất thường:

- **Nút R1 (GPIO 34):** kích hoạt khẩn cấp riêng phòng 1.
- **Nút R2 (GPIO 35):** kích hoạt khẩn cấp riêng phòng 2.
- **Nút ALL (GPIO 23):** kích hoạt khẩn cấp đồng thời cả 2 phòng.

Đặc điểm phần cứng:

- GPIO 34, 35 chỉ là input-only trên ESP32, cần điện trở kéo lên 10kΩ bên ngoài.
- Nối FALLING edge (nhấn nút = nối GND).
- Có cơ chế debounce 250ms tại ISR để chống dội phím.

Cấu hình ngắt: `attachInterrupt(digitalPinToInterrupt(BTN_EMG_R1), isrR1, FALLING)` — xem chi tiết logic xử lý ở **B.2.6 (Emergency Mode)**.

##### c) Bổ sung **bảng cấu hình chân (Pin Mapping)** vào trước hoặc sau Subsection 2.2.9 (Sơ đồ tổng quan)

> ⚠️ **CHÚ Ý QUAN TRỌNG:** Pin mapping ban đầu trong tài liệu này (final_code.ino cũ) đã **KHÔNG còn đúng** với firmware mới [Arduino/newestVersion.ino](Arduino/newestVersion.ino). Bảng dưới đây đã được cập nhật theo code thực tế.

| Chức năng                           | Chân ESP32                            | Module/Linh kiện                          |
| ----------------------------------- | ------------------------------------- | ----------------------------------------- |
| ADC Phòng 1 (qua I2C)               | **ADS1115 kênh A0**                   | MQ-135 #1 (A0) — KHÔNG dùng ADC nội ESP32 |
| ADC Phòng 2 (qua I2C)               | **ADS1115 kênh A1**                   | MQ-135 #2 (A0) — KHÔNG dùng ADC nội ESP32 |
| Điều khiển Relay 1                  | **GPIO 32**                           | Relay IN1 → Quạt phòng 1                  |
| Điều khiển Relay 2                  | **GPIO 33**                           | Relay IN2 → Quạt phòng 2                  |
| I2C SDA (chung bus với ADS1115)     | **GPIO 21**                           | LCD I2C 16x2 + ADS1115                    |
| I2C SCL (chung bus với ADS1115)     | **GPIO 22**                           | LCD I2C 16x2 + ADS1115                    |
| Servo cửa sổ phòng 1                | **GPIO 26**                           | SG90 #1                                   |
| Servo cửa sổ phòng 2                | **GPIO 27**                           | SG90 #2                                   |
| Còi SFM-27 phòng 1 (qua transistor) | **GPIO 25**                           | Buzzer R1                                 |
| Còi SFM-27 phòng 2 (qua transistor) | **GPIO 4**                            | Buzzer R2                                 |
| Nút khẩn cấp phòng 1                | **GPIO 34** (input only, FALLING ISR) | Button R1                                 |
| Nút khẩn cấp phòng 2                | **GPIO 35** (input only, FALLING ISR) | Button R2                                 |
| Nút khẩn cấp toàn hệ thống          | **GPIO 23** (FALLING ISR)             | Button ALL                                |
| Nguồn 5V                            | VIN / 5V                              | MQ-135, Relay, LCD, Quạt, Servo, Còi      |
| GND                                 | GND                                   | Toàn hệ thống                             |

Caption: `Bảng cấu hình chân GPIO của ESP32 trong hệ thống`.

> **Lý do dùng ADS1115 thay cho ADC nội ESP32:** ADC tích hợp ESP32 chỉ 12-bit và có sai số đáng kể ở dải gần 0V do nhiễu nguồn. Module ADS1115 16-bit qua I2C cho độ phân giải cao hơn 16 lần, có PGA khuếch đại tín hiệu yếu và tích hợp lọc sigma-delta khử nhiễu nguồn 50/60Hz.

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

| Giá trị raw ADC | Mức (level) | Ý nghĩa    | Hành động ở chế độ AUTO |
| --------------- | ----------- | ---------- | ----------------------- |
| < 1200          | `GOOD`      | Tốt        | Tắt quạt                |
| 1200 – 1999     | `MOD`       | Trung bình | Tắt quạt                |
| 2000 – 2999     | `BAD`       | Kém        | Bật quạt                |
| ≥ 3000          | `DANG`      | Nguy hiểm  | Bật quạt                |

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

##### f) Cập nhật Subsection **2.3.3 Thiết kế có sử dụng RTOS** _(SỬA LỚN — firmware mới có 6 task chứ không phải 4)_

Phần `2.3.3.1 Phân chia Task` hiện liệt kê 4 task. Tuy nhiên code thực tế trong [Arduino/newestVersion.ino:826-831](Arduino/newestVersion.ino#L826-L831) và [line 850](Arduino/newestVersion.ino#L850) dùng **6 task** (kể cả `taskOTA` chạy động):

| Task            | Stack | Priority         | Tạo bằng                               | Chức năng                                                                                                                                                                                                         |
| --------------- | ----- | ---------------- | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `taskEmergency` | 3072  | **6** (cao nhất) | `xTaskCreate` ở `setup()`              | Xử lý ngắt 3 nút bấm vật lý qua `vTaskNotifyGiveFromISR`. Phản ứng tức thời (`portYIELD_FROM_ISR`).                                                                                                               |
| `taskOTA`       | 8192  | **5**            | `xTaskCreate` ở `loop()` khi có cờ OTA | Tải firmware HTTP/HTTPS, ghi flash bằng `Update.writeStream()`, báo cáo kết quả qua MQTT, reboot. Tắt sau khi xong.                                                                                               |
| `taskControl`   | 3072  | **4**            | `xTaskCreate` ở `setup()`              | Bộ não FSM: xử lý 4 mức ưu tiên EMERGENCY > DANG-Lockout > MANUAL > AUTO. Gọi `executeFan/Buzzer/Window`. Backup state vào RTC.                                                                                   |
| `taskSensor`    | 2048  | **3**            | `xTaskCreate` ở `setup()`              | Đọc ADS1115 qua I2C, lọc EWMA (α=0.2), phát hiện lỗi (MQ_MIN/MQ_MAX), phân loại level GOOD/MOD/BAD/DANG. Chu kỳ 1000ms. Tự động đánh thức Control Task khi phát hiện DANG.                                        |
| `taskLCD`       | 2048  | **2**            | `xTaskCreate` ở `setup()`              | Xoay trang hiển thị 2 phòng mỗi 2000ms, xử lý cờ `flagResetI2C` để re-init I2C/LCD/ADS1115. Hiển thị "EMERGENCY ALERT!" nếu cờ khẩn cấp. Hiển thị "Update Firmware / Downloading.../ Successfully!" khi đang OTA. |
| `taskMQTT`      | 8192  | **1**            | `xTaskCreate` ở `setup()`              | Reconnect WiFi tự động, MQTT retry 5s, callback xử lý lệnh control + lệnh OTA, publish telemetry mỗi 3000ms. Stack 8KB do TLS handshake + buffer JSON 1024 byte.                                                  |

**Giải thích chi tiết:**

- **Stack Size:** `taskMQTT` cần 8KB do TLS/SSL handshake và buffer JSON 1024 byte. `taskOTA` cần 8KB do phải buffer firmware download. Các task còn lại 2-3KB là đủ.
- **Priority:** `taskEmergency` ưu tiên cao nhất 6 vì liên quan tới an toàn người vận hành. `taskOTA` ưu tiên 5 cao hơn `taskControl` để OTA không bị gián đoạn nửa chừng. `taskMQTT` ưu tiên thấp nhất (1) vì nó chỉ là kênh giao tiếp, không thời gian thực ngặt nghèo.
- **Cơ chế thông báo task-to-task:** Dùng `xTaskNotifyGive(controlTaskHandle)` từ `taskSensor`, `taskEmergency` và MQTT callback để đánh thức `taskControl` ngay khi có sự kiện, thay cho polling. Trong ISR dùng `vTaskNotifyGiveFromISR` + `portYIELD_FROM_ISR` để chuyển ngữ cảnh CPU LẬP TỨC sang `taskEmergency`.
- **Watchdog:** Tất cả 6 task đều đăng ký với Task WDT bằng `esp_task_wdt_add(NULL)` và gọi `esp_task_wdt_reset()` mỗi vòng lặp. WDT timeout = 30 giây, panic khi quá hạn.

> 📷 **Cần chèn ảnh:** sơ đồ 6 task + ưu tiên + Mutex + RTC backup → cập nhật lại `Hinh_ve/freertos_task_diagram.png`

##### g) Thêm subsection mới: **2.3.X Cơ chế đồng bộ tài nguyên dùng chung** _(SỬA LỚN — firmware mới có 3 Mutex, không phải 1)_

Firmware mới sử dụng **3 Mutex chuyên biệt** ([Arduino/newestVersion.ino:81-83](Arduino/newestVersion.ino#L81-L83)) để chống race condition và deadlock:

| Mutex       | Bảo vệ                                                                                    | Lý do tách riêng                                                                                                            |
| ----------- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `dataMutex` | Struct `rooms[ROOM_COUNT]` (raw, filtered, mode, fan, buzzer, window, level, isEmergency) | Mọi truy cập đọc/ghi từ Sensor/Control/Emergency/MQTT/LCD task đều đi qua đây                                               |
| `i2cMutex`  | Bus I2C (ADS1115 + LCD chia chung GPIO 21/22)                                             | Tách khỏi `dataMutex` để giữ thời gian khóa I2C cực ngắn (chỉ trong khi đọc ADC hoặc cập nhật LCD), tránh chặn Control task |
| `mqttMutex` | Thư viện `PubSubClient`                                                                   | Cả `taskMQTT` và `taskOTA` cùng publish → cần khóa để tránh đụng độ buffer TCP                                              |

**Quy tắc tránh deadlock:**

- KHÔNG BAO GIỜ giữ 2 mutex cùng lúc.
- Nếu cần đọc cảm biến rồi cập nhật `rooms[]`: lấy `i2cMutex` đọc xong → giải phóng → lấy `dataMutex` cập nhật.
- Khóa `dataMutex` luôn ở thời gian ngắn nhất để Emergency task có thể chiếm quyền nhanh.

**Cờ volatile bổ sung:**

- `flagResetI2C` (đổi tên từ `flagResetLCD` cho đúng code) — báo `taskLCD` re-init I2C bus sau khi Relay đóng/ngắt.
- `flagRequestOTA` — báo `loop()` tạo `taskOTA`.
- `flagTriggerR1/R2/All` — báo `taskEmergency` xử lý nút bấm.
- `isUpdatingFirmware` — chặn các task khác can thiệp khi đang OTA.
- `wasResetByWDT` — báo `setup()` khôi phục state từ RTC.

##### h) Thêm subsection mới: **2.3.X Giao thức truyền tin MQTT**

Đây là điểm quan trọng đang **thiếu chi tiết** trong báo cáo. Cần thêm các thông tin sau:

**Cấu hình broker:**

- Server: HiveMQ Cloud (`21b69e31ed5e4e7c86dbc3dc79814eab.s1.eu.hivemq.cloud`).
- Port: **8883 (MQTT over TLS)** — kết nối an toàn từ ESP32 dùng `WiFiClientSecure` với `setInsecure()` (bỏ qua xác thực CA do giới hạn flash của ESP32).
- Username/Password: cấu hình cứng trong firmware ESP32 và lưu trong bảng `mqtt_configs` ở backend (mỗi device có riêng).
- Buffer size client: 1024 byte.
- Client ID: `ESP32_<MAC_ADDRESS>` (đảm bảo duy nhất).
- Backend dùng port **8883 (mqtts://)** trong DB (thực tế đang đặt `mqtts://...:8883`).

**Cấu trúc Topic:** _(CẬP NHẬT — firmware mới có thêm 2 topic OTA)_

| Topic                          | Hướng           | Payload                                                                                                              | Ghi chú                          |
| ------------------------------ | --------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| `air/data/{deviceId}`          | ESP32 → Backend | JSON telemetry (chuẩn mới, dùng UUID device)                                                                         | Backend hỗ trợ multi-tenant      |
| `air/data`                     | ESP32 → Backend | JSON telemetry (legacy)                                                                                              | ESP32 hiện publish ở topic này   |
| `air/control/{deviceId}`       | Backend → ESP32 | JSON command (chuẩn mới)                                                                                             |                                  |
| `air/control`                  | Backend → ESP32 | JSON command (legacy)                                                                                                | ESP32 hiện subscribe ở topic này |
| **`air/updatefirmware`**       | Backend → ESP32 | `{"url":"http://...", "version":"1.0.x"}`                                                                            | **MỚI: lệnh OTA**                |
| **`air/firmwareupdatestatus`** | ESP32 → Backend | `{"mac_address":"...","update":true,"status":"success/failed/rejected","version":"...","reason":"emergency_active"}` | **MỚI: báo cáo kết quả OTA**     |

**Payload Telemetry (ESP32 → Backend), QoS 0, mỗi 3 giây:** _(CẬP NHẬT — thêm `emergency`, `buzzer`, `window`, đổi tên `id`/`value`/`level`)_

```json
{
  "rooms": [
    {
      "id": 1,
      "value": 1319,
      "level": "MOD ",
      "emergency": false,
      "mode": "MANUAL",
      "fan": false,
      "buzzer": false,
      "window": 45
    },
    {
      "id": 2,
      "value": 1758,
      "level": "MOD ",
      "emergency": false,
      "mode": "AUTO",
      "fan": false,
      "buzzer": false,
      "window": 90
    }
  ]
}
```

**Payload Control (Backend → ESP32), QoS 1:** _(CẬP NHẬT — hỗ trợ thêm 2 trường tùy chọn)_

```json
{
  "room": 1,
  "mode": "MANUAL",
  "fan": false,
  "buzzer": false,
  "window": 90
}
```

- `room`: bắt buộc, số nguyên 1 hoặc 2.
- `mode`: bắt buộc, "AUTO" hoặc "MANUAL".
- `fan`, `buzzer`: tùy chọn, boolean. Chỉ có hiệu lực khi `mode == "MANUAL"`.
- `window`: tùy chọn, số nguyên 0-180 (góc servo). Chỉ có hiệu lực khi `mode == "MANUAL"`.
- **Bị từ chối** nếu phòng đang ở trạng thái EMERGENCY (xem Section B.2.6).

##### i) Cập nhật Subsection **2.3.4 Cơ chế điều khiển từ người dùng**

Hiện tại bước 3 ghi `"room": 2, "mode": "manual"`. Tuy nhiên code thực tế:

- Backend gửi `"mode"` viết HOA: `"AUTO"` hoặc `"MANUAL"` (xem `MqttPool.sendCommand()`).
- Payload luôn gửi đủ 3 trường `room`, `mode`, `fan` cùng lúc (không tách thành 2 lệnh riêng như tài liệu mô tả).

→ Cần sửa lại đoạn `"manual"` → `"MANUAL"`, gộp 2 bước (set mode + set fan) thành 1 bước duy nhất với payload đầy đủ.

##### k) **Logic 4 mức ưu tiên trong taskControl** _(MỚI — đặc trưng quan trọng của hệ thống)_

Code [Arduino/newestVersion.ino:429-454](Arduino/newestVersion.ino#L429-L454) implement 4 mức ưu tiên xử lý:

| Mức                           | Điều kiện                                   | Hành động                                                                                                                                                                                    |
| ----------------------------- | ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. EMERGENCY** (tuyệt đối)  | `rooms[i].isEmergency == true` (do nút bấm) | Ép `buzzer=ON`, `fan=ON`, `window=180°`. KHÔNG quan tâm mode.                                                                                                                                |
| **2. DANG-Lockout** (an toàn) | `level == "DANG"` && `mode == AUTO`         | **Ép mode về MANUAL**, `buzzer=ON`, `fan=ON`, `window=180°`. **Không tự rớt về AUTO** kể cả khi nồng độ giảm — bắt buộc người vận hành kiểm tra hiện trường rồi reset thủ công từ Dashboard. |
| **3. AUTO bình thường**       | `mode == AUTO` && level khác DANG           | `buzzer=OFF`, `fan=ON nếu level=BAD`, `window` map từ raw `[400-3000] → [0-180°]`.                                                                                                           |
| **4. MANUAL**                 | `mode == MANUAL` (không EMERGENCY)          | Theo lệnh trực tiếp từ Dashboard/MQTT.                                                                                                                                                       |

**Cần thêm bảng mức ưu tiên này vào Subsection 2.3.4 hoặc tạo subsection riêng trong 2.3.3** để minh hoạ FSM mới.

##### j) Sửa **2.3.2 Thiết kế không sử dụng RTOS**

Lưu ý: code thực tế **chỉ dùng RTOS**. Section 2.3.2 đang trình bày như là một thiết kế song song của nhóm. Cần:

- Hoặc làm rõ ngay từ đầu: "Phần này trình bày phương án thiết kế lý thuyết (không RTOS) để đối chiếu, phương án triển khai thực tế là RTOS được trình bày ở 2.3.3".
- Hoặc lược bớt 2.3.2 nếu muốn tinh giản.

#### B.2.6. Bổ sung Section MỚI: **2.4 Hệ thống Khẩn cấp (Emergency Mode)** _(MỚI — đặc trưng quan trọng nhất của firmware mới, BẮT BUỘC bổ sung)_

> Đặt section này NGAY SAU Section 2.3 (Phần mềm), TRƯỚC Backend. Đây là tính năng nổi bật của hệ thống mà báo cáo hiện đang KHÔNG có 1 dòng nào nói tới.

##### 2.4.1 Tổng quan và động lực

Trong môi trường công nghiệp, ngoài cảnh báo tự động khi cảm biến phát hiện khí, vận hành viên cần **một cơ chế kích hoạt khẩn cấp thủ công** (panic button) để xử lý các tình huống bất thường mà cảm biến chưa kịp phát hiện (ví dụ: nhìn thấy khói, ngửi được khí, nghe tiếng nổ, sự cố cháy nổ ở khu vực bên cạnh...). Hệ thống thiết kế **3 nút bấm vật lý** + cơ chế xử lý ngắt + task ưu tiên cao nhất để đáp ứng yêu cầu này.

##### 2.4.2 Phần cứng nút bấm

| Nút         | GPIO                 | Phạm vi tác động                      |
| ----------- | -------------------- | ------------------------------------- |
| BTN_EMG_R1  | GPIO 34 (input only) | Toggle EMERGENCY của riêng phòng 1    |
| BTN_EMG_R2  | GPIO 35 (input only) | Toggle EMERGENCY của riêng phòng 2    |
| BTN_EMG_ALL | GPIO 23              | Toggle EMERGENCY đồng thời cả 2 phòng |

- Tất cả nút đều cần điện trở kéo lên 10kΩ bên ngoài (do GPIO 34/35 không có internal pull-up).
- Nối FALLING edge: nhấn nút → kéo GND → trigger ngắt.
- Mỗi lần nhấn = TOGGLE (bật ↔ tắt khẩn cấp), nhả ra không thay đổi state.

##### 2.4.3 ISR và Debounce

Code [Arduino/newestVersion.ino:111-156](Arduino/newestVersion.ino#L111-L156):

```cpp
void IRAM_ATTR isrR1() {
  unsigned long interrupt_time = millis();
  if (interrupt_time - last_interrupt_time > DEBOUNCE_TIME && !isUpdatingFirmware) {
    flagTriggerR1 = true;
    if(emergencyTaskHandle) {
      BaseType_t xHigherPriorityTaskWoken = pdFALSE;
      vTaskNotifyGiveFromISR(emergencyTaskHandle, &xHigherPriorityTaskWoken);
      if (xHigherPriorityTaskWoken == pdTRUE) {
        portYIELD_FROM_ISR();   // Ép CPU nhảy sang Task Emergency NGAY LẬP TỨC
      }
    }
    last_interrupt_time = interrupt_time;
  }
}
```

Đặc điểm:

- `IRAM_ATTR`: hàm ISR phải nằm trong RAM nội (không phải flash) để tránh lỗi cache khi flash đang bận.
- Debounce 250ms: chống dội phím cơ học.
- `vTaskNotifyGiveFromISR` + `portYIELD_FROM_ISR`: thay vì set cờ rồi đợi task tự poll, ép CPU **chuyển ngữ cảnh ngay lập tức** sang `taskEmergency` để phản hồi tức thời.
- Bỏ qua khi đang OTA (`isUpdatingFirmware`) — tránh nhiễu trong quá trình ghi flash.

##### 2.4.4 Task xử lý khẩn cấp (Priority 6 - cao nhất)

Code [Arduino/newestVersion.ino:217-272](Arduino/newestVersion.ino#L217-L272):

- Block tại `ulTaskNotifyTake(pdTRUE, portMAX_DELAY)`, không tốn CPU khi không có sự kiện.
- Khi được đánh thức từ ISR: lấy `dataMutex`, toggle `rooms[i].isEmergency`, và **đánh thức `taskControl`** ngay (`xTaskNotifyGive(controlTaskHandle)`) để Control xử lý phần cứng.
- Nút ALL có logic riêng: nếu CÓ phòng nào chưa bật khẩn cấp → bật cả 2; ngược lại tắt cả 2.
- Khi nhả EMERGENCY (`isEmergency = false`) → ép `mode = AUTO` để hệ thống quay lại chế độ tự động.

##### 2.4.5 Logic ưu tiên trong taskControl

Khi `isEmergency == true`:

- `targetBuzzer = true` → còi kêu liên tục.
- `targetFan = true` → quạt chạy hết công suất.
- `targetWindowAngle = 180` → cửa sổ mở tối đa thoát khí.
- **KHÔNG quan tâm `mode`** — ép cứng bất chấp người dùng thiết lập.

##### 2.4.6 Bảo mật khi đang khẩn cấp

Để bảo đảm an toàn, khi phòng đang ở EMERGENCY:

1. **Từ chối lệnh điều khiển từ MQTT** ([Arduino/newestVersion.ino:648-653](Arduino/newestVersion.ino#L648-L653)):

   ```cpp
   if (rooms[idx].isEmergency) {
     Serial.printf("[SECURITY] R%d đang KHẨN CẤP! Từ chối lệnh điều khiển từ xa.\n", room);
     return;
   }
   ```

   → Người ở Dashboard không thể vô tình hoặc cố ý tắt thiết bị khi nhân viên hiện trường đang xử lý sự cố.

2. **Từ chối lệnh OTA firmware** ([Arduino/newestVersion.ino:606-621](Arduino/newestVersion.ino#L606-L621)):
   ```cpp
   if (rooms[0].isEmergency || rooms[1].isEmergency) {
     client.publish("air/firmwareupdatestatus",
       "{\"status\":\"rejected\",\"reason\":\"emergency_active\"}");
     return;
   }
   ```
   → Tránh cập nhật firmware khi hệ thống đang trong tình huống khẩn cấp (reboot có thể làm gián đoạn xử lý sự cố).

##### 2.4.7 Lưu trạng thái vào RTC Memory

`isEmergency` được sao lưu vào RTC RAM bằng `RTC_DATA_ATTR bool savedEmergency[ROOM_COUNT]` ([Arduino/newestVersion.ino:77](Arduino/newestVersion.ino#L77)). Lý do: nếu xảy ra Watchdog reset trong lúc khẩn cấp, hệ thống khởi động lại sẽ **TỰ ĐỘNG khôi phục** trạng thái khẩn cấp thay vì reset về 0 — bảo đảm an toàn liên tục.

> 📷 **Cần chèn ảnh:** sơ đồ luồng từ ISR → Task Emergency → Task Control → phần cứng → `Hinh_ve/emergency_flow.png`
>
> 📷 **Cần chèn ảnh:** ảnh thực tế 3 nút bấm khẩn cấp gắn trên hộp thiết bị → `Hinh_ve/emergency_buttons.png`
>
> 📷 **Cần chèn ảnh:** screenshot LCD hiển thị "EMERGENCY ALERT!" → `Hinh_ve/lcd_emergency.png`

---

#### B.2.7. Bổ sung Section MỚI: **2.5 Watchdog Timer và Khôi phục trạng thái sau Reset** _(MỚI)_

> Đặt section này SAU Section 2.4 (Emergency), TRƯỚC Backend.

##### 2.5.1 Vấn đề độ tin cậy 24/7

Hệ thống công nghiệp cần chạy liên tục 24/7. Tuy nhiên ESP32 có thể bị treo do: nhiễu nguồn, lỗi I2C, lỗi mạng MQTT, deadlock task, tràn stack. Nếu treo mà không tự reset → hệ thống ngừng giám sát → nguy hiểm.

##### 2.5.2 Cơ chế Task Watchdog Timer

Code [Arduino/newestVersion.ino:833-839](Arduino/newestVersion.ino#L833-L839):

```cpp
esp_task_wdt_config_t wdt_config = {
  .timeout_ms = 30000,                                     // 30 giây
  .idle_core_mask = (1 << portNUM_PROCESSORS) - 1,         // Giám sát cả 2 core
  .trigger_panic = true                                    // Panic khi quá hạn
};
esp_task_wdt_init(&wdt_config);
```

Mỗi task khi khởi tạo đăng ký với WDT (`esp_task_wdt_add(NULL)`) và gọi `esp_task_wdt_reset()` mỗi vòng lặp để báo "tôi còn sống". Nếu task bị treo > 30 giây không gọi reset → ESP32 panic → reboot tự động.

Bảng tổng hợp:

| Task            | Đăng ký WDT?                                              | Cơ chế reset                                                    |
| --------------- | --------------------------------------------------------- | --------------------------------------------------------------- |
| `taskEmergency` | Không (luôn block tại `ulTaskNotifyTake`, không thể treo) | —                                                               |
| `taskControl`   | Có                                                        | `esp_task_wdt_reset()` đầu mỗi vòng `for(;;)`                   |
| `taskSensor`    | Có                                                        | `esp_task_wdt_reset()` đầu mỗi vòng                             |
| `taskLCD`       | Có                                                        | `esp_task_wdt_reset()` đầu mỗi vòng                             |
| `taskMQTT`      | Có                                                        | `esp_task_wdt_reset()` đầu mỗi vòng                             |
| `taskOTA`       | Tự **xoá đăng ký** (`esp_task_wdt_delete(NULL)`)          | OTA tải file lâu, không kịp reset → tắt WDT để tránh reboot oan |
| `loop()`        | Có                                                        | `esp_task_wdt_reset()` mỗi 1s                                   |

##### 2.5.3 RTC Backup Memory

ESP32 có 8KB RTC RAM giữ giá trị qua reboot (chỉ mất khi mất nguồn hoàn toàn). Hệ thống dùng vùng này để khôi phục state:

```cpp
RTC_DATA_ATTR Mode savedMode[ROOM_COUNT];
RTC_DATA_ATTR bool savedFan[ROOM_COUNT];
RTC_DATA_ATTR bool savedBuzzer[ROOM_COUNT];
RTC_DATA_ATTR int savedWindowAngle[ROOM_COUNT];
RTC_DATA_ATTR bool savedEmergency[ROOM_COUNT];
RTC_DATA_ATTR bool wasResetByWDT;
```

##### 2.5.4 Logic khôi phục trong setup()

Code [Arduino/newestVersion.ino:806-822](Arduino/newestVersion.ino#L806-L822):

```cpp
esp_reset_reason_t resetReason = esp_reset_reason();
if (resetReason == ESP_RST_TASK_WDT || resetReason == ESP_RST_INT_WDT
    || resetReason == ESP_RST_PANIC) {
  wasResetByWDT = true;
}

// Khi setup actuator:
if (wasResetByWDT) {
  // Khôi phục: mode, fan, buzzer, window angle, emergency
  rooms[i].mode = savedMode[i];
  rooms[i].isEmergency = savedEmergency[i];
  digitalWrite(buzzerPins[i], rooms[i].currentBuzzer ? HIGH : LOW);
  digitalWrite(relayPins[i], rooms[i].currentFan ? HIGH : LOW);
  windowServos[i].write(rooms[i].currentWindowAngle);
} else {
  // Khởi động bình thường: mặc định AUTO, không emergency, tắt hết
}
```

→ Sau WDT reset, hệ thống khôi phục **đúng trạng thái trước khi treo** trong < 5 giây, không cần can thiệp thủ công.

##### 2.5.5 Xử lý mỗi vòng task khi đang OTA

Khi `isUpdatingFirmware == true`, các task khác (Sensor/Control/LCD/MQTT) chuyển sang `vTaskDelay(1000)` thay vì làm việc → tránh đụng độ với taskOTA đang ghi flash.

> 📷 **Cần chèn ảnh:** sơ đồ trạng thái life cycle (boot → check WDT → restore RTC → run tasks) → `Hinh_ve/wdt_restore_flow.png`

---

#### B.2.8. Bổ sung Section MỚI: **2.6 Cơ chế Cập nhật Firmware từ xa (OTA)** _(MỚI — chuyển từ "Hướng phát triển" sang phần chính)_

> Đặt section này SAU Watchdog, TRƯỚC Backend. **Đây là tính năng đã được implement đầy đủ**, không phải hướng phát triển.

##### 2.6.1 Tổng quan kiến trúc OTA 3 lớp

```
[Frontend OTA Page] ──upload .bin──> [Backend storage]
                                          │
                                          ↓ trigger
                          [Backend MQTT publish] ──> [HiveMQ Broker]
                                                          │
                                                          ↓ subscribe
                                                    [ESP32 firmware]
                                                          │
                                                          ↓ HTTP GET
                                  [Backend serves /api/firmware/download/:version]
                                                          │
                                                          ↓ Update.writeStream()
                                                    [ESP32 flash partition]
                                                          │
                                                          ↓ MQTT publish status
                                                    [Backend logs result]
                                                          │
                                                          ↓ Socket.IO emit
                                                    [Frontend updates UI]
```

##### 2.6.2 Phía Frontend ([OTAManagement.jsx](Frontend/src/pages/OTAManagement.jsx))

Chức năng:

- Upload file `.bin` (multipart, max 10MB) qua endpoint `POST /api/firmware/upload`.
- Liệt kê toàn bộ phiên bản firmware đã upload (version, ngày upload, MD5, kích thước, số lần download).
- Sửa metadata (version, release notes) qua `PATCH /api/firmware/:id`.
- Xóa firmware qua `DELETE /api/firmware/:id`.
- Multi-select thiết bị + bấm "Update" → gửi yêu cầu OTA hàng loạt qua `POST /api/firmware/trigger-batch`.
- Polling 500ms để hiện tiến độ realtime.
- Xem log cập nhật cho từng firmware qua `GET /api/firmware/:id/logs`.

> 📷 **Cần chèn ảnh:** screenshot trang OTA Management với danh sách firmware → `Hinh_ve/screen_ota_management.png`
>
> 📷 **Cần chèn ảnh:** screenshot modal "Update progress" hiển thị tiến độ batch update → `Hinh_ve/screen_ota_progress.png`

##### 2.6.3 Phía Backend ([firmwareController.js](Backend/src/controllers/firmwareController.js), [firmwareRoutes.js](Backend/src/routes/firmwareRoutes.js))

Bảng REST API endpoints:

| Method | Endpoint                            | Mô tả                                         |
| ------ | ----------------------------------- | --------------------------------------------- |
| POST   | `/api/firmware/upload`              | Upload .bin (multer multipart, tính MD5 hash) |
| GET    | `/api/firmware`                     | Liệt kê toàn bộ firmware                      |
| GET    | `/api/firmware/latest?current=v1.0` | Kiểm tra phiên bản mới                        |
| GET    | `/api/firmware/status`              | Polling tiến độ OTA                           |
| GET    | `/api/firmware/download/:version`   | Tải file .bin (tăng download_count)           |
| PATCH  | `/api/firmware/:id`                 | Sửa version/release_notes                     |
| DELETE | `/api/firmware/:id`                 | Xoá file + record                             |
| GET    | `/api/firmware/:id/logs`            | Lấy log cập nhật từng device                  |
| POST   | `/api/firmware/trigger-update`      | Đẩy OTA cho 1 device                          |
| POST   | `/api/firmware/trigger-batch`       | Đẩy OTA cho nhiều device song song            |

3 bảng database liên quan:

- `firmwares` — metadata firmware (version unique, file_path, md5_hash, release_notes, uploaded_by, download_count, is_active).
- `firmware_update_logs` — log cập nhật từng device (status: pending/in_progress/success/failed, error_message, started_at, completed_at).
- `ota_update_sessions` — session cập nhật hàng loạt (total_devices, success_count, failed_count, session_status).

##### 2.6.4 Phía Firmware ESP32 ([Arduino/newestVersion.ino:275-409](Arduino/newestVersion.ino#L275-L409))

Quá trình OTA trong `taskOTA` (Priority 5, stack 8KB, tạo động):

1. **Chuẩn bị:** Set `isUpdatingFirmware = true`, ngắt đăng ký WDT, đợi 2 giây cho các task khác idle.
2. **An toàn phần cứng:** Tắt toàn bộ Relay và Buzzer trước khi nạp.
3. **Kiểm tra mạng:** Hủy nếu mất WiFi.
4. **Tải file:** Hỗ trợ cả HTTP và HTTPS (`setInsecure()` với HTTPS).
5. **Ghi flash:** `Update.begin(contentLength)` → `Update.writeStream(*clientPtr)` → `Update.end()`.
6. **Báo cáo thành công:** Publish `air/firmwareupdatestatus` payload `{"mac_address":"...","update":true,"status":"success","version":"..."}`. Lặp `client.loop()` 10 lần trong 1 giây để đảm bảo gói tin đẩy được lên broker. Disconnect MQTT clean.
7. **Reboot:** `vTaskDelay(2000)` → `ESP.restart()`.

Trong khi đang OTA:

- LCD hiển thị `Update Firmware / Downloading...` → khi xong: `Successfully!`.
- Các task khác chuyển sang chế độ chờ (vTaskDelay 1s).
- ISR nút khẩn cấp bị bypass (`!isUpdatingFirmware`) để tránh xung đột.

##### 2.6.5 An toàn khi OTA

- **Kiểm tra phòng đang khẩn cấp:** Nếu `rooms[0].isEmergency || rooms[1].isEmergency == true` → từ chối OTA, gửi `{"status":"rejected","reason":"emergency_active"}`.
- **Kiểm tra MAC address:** Backend chỉ ghi log update cho đúng device gửi báo cáo.
- **HTTPS optional:** Có thể dùng HTTP cho mạng nội bộ tin cậy, HTTPS cho internet.
- **MD5 hash:** Backend lưu hash mỗi file để client có thể verify integrity (chưa enforced).

> 📷 **Cần chèn ảnh:** sơ đồ tuần tự OTA đầy đủ 7 bước → `Hinh_ve/sequence_ota.png`

---

#### B.2.3. Bổ sung Section MỚI: **2.7 Phần Backend của hệ thống** (chèn TRƯỚC Section "Phương pháp thực hiện và kết quả")

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

| Bảng             | Vai trò                                                                       | Khóa chính                          |
| ---------------- | ----------------------------------------------------------------------------- | ----------------------------------- |
| `users`          | Tài khoản người dùng (email, password_hash, full_name, role)                  | `id` (UUID)                         |
| `devices`        | Thiết bị ESP32 (mac_address, claim_pin, device_name, status, last_connected)  | `id` (UUID)                         |
| `user_devices`   | Bảng N-N giữa user và device                                                  | `id`; UNIQUE(user_id, device_id)    |
| `mqtt_configs`   | Thông tin kết nối MQTT của từng device (broker_url, port, username, password) | `id`, UNIQUE `device_id`            |
| `rooms`          | Phòng giám sát (mỗi device có N phòng, hiện = 2)                              | `id`; UNIQUE(device_id, room_index) |
| `telemetry_data` | Dữ liệu time-series từng phòng (aqi_raw, aqi_level, fan_is_on, timestamp)     | `id` (BigInt auto-inc)              |
| `activity_logs`  | Audit log mọi hành động của user và sự kiện hệ thống                          | `id` (BigInt auto-inc)              |

> 📷 **Cần chèn ảnh:** sơ đồ ERD database → `Hinh_ve/database_erd.png`
> (Vẽ bằng dbdiagram.io, MySQL Workbench hoặc Prisma ERD generator)

##### 2.4.3 Hệ thống xác thực JWT

- User đăng ký (`POST /api/auth/register`): mật khẩu được hash bằng `bcryptjs` (salt 10), lưu `password_hash`.
- Đăng nhập (`POST /api/auth/login`): so sánh `bcrypt.compare()`, phát hành JWT có TTL 24h ký bằng `JWT_SECRET`.
- Mọi API ngoài auth đều bảo vệ bằng middleware `authenticate` đọc header `Authorization: Bearer <token>`.

##### 2.4.4 Danh sách REST API endpoints

Bảng tổng hợp (lấy từ các file [Backend/src/routes/\*.js](Backend/src/routes/)):

| Method | Endpoint                              | Mô tả                                  | Auth  |
| ------ | ------------------------------------- | -------------------------------------- | ----- |
| POST   | `/api/auth/register`                  | Đăng ký user                           | Không |
| POST   | `/api/auth/login`                     | Đăng nhập, trả JWT                     | Không |
| POST   | `/api/auth/refresh`                   | Làm mới JWT                            | Có    |
| GET    | `/api/auth/profile`                   | Thông tin user hiện tại                | Có    |
| GET    | `/api/devices`                        | Danh sách device user sở hữu           | Có    |
| GET    | `/api/devices/:id`                    | Chi tiết device                        | Có    |
| POST   | `/api/devices/verify-claim`           | Kiểm tra MAC + PIN có hợp lệ           | Có    |
| POST   | `/api/devices/claim`                  | Claim device về user                   | Có    |
| POST   | `/api/devices/control`                | Gửi lệnh điều khiển (mode, fan)        | Có    |
| POST   | `/api/devices/:id/disconnect`         | Ngắt kết nối MQTT của device           | Có    |
| POST   | `/api/devices/:id/reconnect`          | Kết nối lại MQTT                       | Có    |
| POST   | `/api/devices/:id/release`            | Giải phóng device                      | Có    |
| DELETE | `/api/devices/:id`                    | Xóa quyền sở hữu                       | Có    |
| PUT    | `/api/devices/:id/settings`           | Cập nhật tên device, room, MQTT config | Có    |
| GET    | `/api/devices/:id/telemetry`          | Telemetry theo device                  | Có    |
| GET    | `/api/rooms/:roomId`                  | Chi tiết room                          | Có    |
| PUT    | `/api/rooms/:roomId/mode`             | Đổi mode AUTO/MANUAL                   | Có    |
| PUT    | `/api/rooms/:roomId/fan`              | Bật/tắt quạt (chỉ MANUAL)              | Có    |
| GET    | `/api/rooms/:roomId/telemetry`        | Telemetry theo room                    | Có    |
| GET    | `/api/telemetry/room/:roomId`         | Bản ghi telemetry mới nhất             | Có    |
| GET    | `/api/telemetry/room/:roomId/history` | Lịch sử N giờ + thống kê min/max/avg   | Có    |
| GET    | `/api/activity/user/:userId`          | Activity log của user                  | Có    |
| GET    | `/api/activity/device/:deviceId`      | Activity log của device                | Có    |
| GET    | `/api/activity`                       | Toàn bộ activity log                   | Có    |

##### 2.4.5 MQTT Pool – kết nối đa thiết bị

- Khởi tạo: khi backend start, query toàn bộ `devices` đã được claim (`user_devices` join) cùng `mqtt_config` → tạo 1 MQTT client cho từng device, lưu vào `Map<deviceId, mqttClient>`.
- Subscribe: mỗi client subscribe `air/data/{deviceId}` + topic legacy `air/data`.
- Khi nhận message → parse JSON → ghi vào bảng `telemetry_data` (1 dòng / room) → cập nhật `rooms.current_mode`, `rooms.current_fan_status` → emit Socket.IO event `telemetry_update` cho mọi FE đang kết nối.
- Khi user gửi lệnh: backend `publish()` payload JSON tới cả 2 topic `air/control/{deviceId}` và `air/control` (QoS 1).
- Cơ chế **Telemetry Timeout** 5 giây: nếu không nhận được dữ liệu telemetry trong 5 giây liên tiếp → backend tự động đặt `device.status = OFFLINE` và emit `activity_log` event `DEVICE_OFFLINE`. Khi telemetry quay lại → tự động đánh dấu ONLINE.

##### 2.4.6 Real-time với Socket.IO

Các sự kiện server emit:

| Event                       | Payload                                         | Khi nào emit                                                                                                                                      |
| --------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `telemetry_update`          | `{deviceId, rooms[], timestamp}`                | Mỗi lần ESP32 publish telemetry mới                                                                                                               |
| `telemetry-update` (legacy) | `{deviceId, data, timestamp}`                   | Backward compatibility                                                                                                                            |
| `activity_log`              | `{deviceId, eventType, description, timestamp}` | DEVICE_ONLINE / DEVICE_OFFLINE / MODE_CHANGED / FAN_TOGGLED / CONTROL_SENT / DEVICE_CLAIMED / DEVICE_RELEASED / DEVICE_REMOVED / SETTINGS_UPDATED |

##### 2.4.7 Activity Log – các loại sự kiện

Bảng các `event_type` được hệ thống sinh ra:

| Event Type         | Sinh từ                                | Ý nghĩa                              |
| ------------------ | -------------------------------------- | ------------------------------------ |
| `DEVICE_CLAIMED`   | API claim                              | User đăng ký một device mới          |
| `DEVICE_RELEASED`  | API release                            | Giải phóng device (gỡ liên kết)      |
| `DEVICE_REMOVED`   | API delete                             | Xóa quyền sở hữu device              |
| `DEVICE_ONLINE`    | MQTT connect / telemetry timeout reset | Device lên                           |
| `DEVICE_OFFLINE`   | MQTT offline / telemetry timeout fire  | Device offline                       |
| `MODE_CHANGED`     | API PUT room mode                      | Đổi AUTO ↔ MANUAL                    |
| `FAN_TOGGLED`      | API PUT room fan                       | Bật/tắt quạt MANUAL                  |
| `CONTROL_SENT`     | API POST device control                | Lệnh điều khiển đã gửi               |
| `SETTINGS_UPDATED` | API PUT settings                       | Đổi tên device/room hoặc MQTT config |

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

> 📷 **Cần chèn ảnh:** sơ đồ flow cập nhật real-time từ ESP32 → Frontend (kết hợp luồng telemetry đã nêu ở B.2.3 nhưng nhấn mạnh phía FE) → `Hinh_ve/realtime_flow.png` _(có thể tái dùng `sequence_telemetry.png`)_

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

| Yêu cầu phi chức năng        | Mục tiêu             | Thực tế đo được                                  | Đạt?       |
| ---------------------------- | -------------------- | ------------------------------------------------ | ---------- |
| Độ trễ telemetry → Dashboard | < 3 s                | ~3 s (chu kỳ publish 3s) + < 100ms qua Socket.IO | ✓          |
| Phản hồi kích hoạt quạt      | < 5 s                | < 1.5 s (chu kỳ taskControl 1.5s)                | ✓          |
| Tỷ lệ truyền gói thành công  | ≥ 98%                | (cần đo thực tế và điền)                         | ?          |
| Bảo mật MQTT                 | TLS/SSL              | TLS port 8883 (skip CA verify)                   | ✓ một phần |
| Bảo mật API                  | JWT                  | JWT 24h + bcrypt                                 | ✓          |
| Nguồn dự phòng UPS           | Duy trì khi mất điện | YX850 chuyển sang pin 18650 < 1s                 | ✓          |

---

### B.3. Chương 3 – `Chuong/6_Ket_luan.tex`

#### B.3.1. Cập nhật **Subsection 3.1 Kết luận**

Bổ sung các kết quả nổi bật **đang thiếu**:

- Đã xây dựng được **hệ thống IoT đa thiết bị, đa người dùng** với REST API + WebSocket real-time.
- Hỗ trợ **claim/release device** linh hoạt qua MAC + PIN.
- Có **audit log đầy đủ** mọi hành động và sự kiện hệ thống.
- **Frontend SPA hiện đại** với cập nhật real-time, sparkline, onboarding wizard.

#### B.3.2. Cập nhật **Subsection 3.2 Hướng phát triển** _(SỬA — bỏ OTA vì đã implement, giữ AI Summary)_

Bổ sung:

- Tích hợp **AI Summary tóm tắt mức độ ô nhiễm hàng giờ** bằng GPT-4o-mini hoặc model nội suy thời gian thực (hiện chưa implement, cần phát triển thêm).
- ~~**OTA Update firmware**~~ — **ĐÃ IMPLEMENT** ở Section 2.6, không còn là hướng phát triển nữa.
- Hiện tại MQTT đang dùng `setInsecure()` (ESP32 bỏ qua xác thực CA) — cần **upload CA certificate của HiveMQ** vào ESP32 để bảo mật đầy đủ.
- **Tự động tính ppm** từ raw ADC bằng đường cong calibration R0/Rs của MQ-135.
- Thêm trang **Settings** và **History** ở Frontend (hiện sidebar có nút Settings nhưng chưa có route tương ứng).
- **Sleep Mode tiết kiệm điện** khi chạy bằng pin (hiện firmware luôn chạy 6 task RTOS, chưa có deep sleep).
- **Mở rộng nhiều phòng/thiết bị**: hệ thống hiện hard-code `ROOM_COUNT = 2`, có thể tổng quát hóa thành N phòng cấu hình động.
- **Verify firmware MD5** ở phía ESP32 để chống tấn công man-in-the-middle khi OTA qua HTTP.
- **Push notification** (FCM) khi có sự kiện EMERGENCY hoặc DANG, để cảnh báo qua điện thoại di động kể cả khi không mở Dashboard.

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

| #   | Tên file đề xuất               | Loại ảnh                 | Mục đích                                                                            | Dùng ở section                      |
| --- | ------------------------------ | ------------------------ | ----------------------------------------------------------------------------------- | ----------------------------------- |
| 1   | `lcd_i2c_16x2.png`             | Ảnh sản phẩm linh kiện   | Module LCD I2C 16x2                                                                 | 2.2.X (mới)                         |
| 2   | `lcd_running.png`              | Ảnh chụp thực tế         | LCD đang hiển thị thông tin phòng                                                   | 2.2.X (mới)                         |
| 3   | `relay_2_kenh.png`             | Ảnh sản phẩm linh kiện   | Module Relay 2 kênh 5V                                                              | 2.2.X (mới)                         |
| 4   | `wiring_diagram.png`           | Sơ đồ Fritzing/KiCad     | Sơ đồ đi dây chi tiết toàn hệ thống (chân ESP32 → Relay/MQ-135/LCD)                 | 2.2.9 (bổ sung kèm sơ đồ tổng quan) |
| 5   | `freertos_task_diagram.png`    | Sơ đồ khối               | Quan hệ giữa 4 task RTOS + Mutex + Hardware peripherals                             | 2.3.3                               |
| 6   | `mqtt_topic_flow.png`          | Sơ đồ luồng              | Topic publish/subscribe giữa ESP32 ↔ Backend                                        | 2.3.X (mới — MQTT)                  |
| 7   | `backend_architecture.png`     | Sơ đồ kiến trúc          | 4 khối: ESP32, HiveMQ, Backend (MqttPool/Express/Socket.IO/Prisma), MySQL, Frontend | 2.4.1 (mới)                         |
| 8   | `database_erd.png`             | Sơ đồ ERD                | 7 bảng + quan hệ FK                                                                 | 2.4.2 (mới)                         |
| 9   | `sequence_telemetry.png`       | Sequence diagram         | Luồng telemetry ESP32 → FE                                                          | 2.4.7 (mới)                         |
| 10  | `sequence_control.png`         | Sequence diagram         | Luồng control FE → ESP32                                                            | 2.4.7 (mới)                         |
| 11  | `screen_login.png`             | Screenshot UI            | Trang đăng nhập                                                                     | 2.5.3 (mới)                         |
| 12  | `screen_dashboard.png`         | Screenshot UI            | Dashboard chính                                                                     | 2.5.3 (mới)                         |
| 13  | `screen_room_card.png`         | Screenshot UI zoom       | Một RoomCard kèm chú thích                                                          | 2.5.4 (mới)                         |
| 14  | `screen_onboarding_wizard.png` | Screenshot UI ghép 4 ảnh | 4 bước wizard claim device                                                          | 2.5.4 (mới)                         |
| 15  | `screen_offline.png`           | Screenshot UI            | Device chuyển trạng thái OFFLINE                                                    | 2.6.3 (bổ sung)                     |
| 16  | `screen_sparkline_spike.png`   | Screenshot UI            | Sparkline khi AQI tăng đột biến                                                     | 2.6.3 (bổ sung)                     |
| 17  | `emergency_buttons.png`        | Ảnh thực tế              | 3 nút bấm khẩn cấp gắn trên hộp thiết bị                                            | 2.4.2 (mới)                         |
| 18  | `emergency_flow.png`           | Sơ đồ flow               | ISR → Task Emergency → Task Control → phần cứng                                     | 2.4.3-2.4.5 (mới)                   |
| 19  | `lcd_emergency.png`            | Screenshot LCD thực tế   | LCD hiển thị "EMERGENCY ALERT!"                                                     | 2.4.6 (mới)                         |
| 20  | `wdt_restore_flow.png`         | Sơ đồ trạng thái         | Boot → check WDT → restore RTC → run tasks                                          | 2.5 (mới)                           |
| 21  | `screen_ota_management.png`    | Screenshot UI            | Trang OTA Management với danh sách firmware                                         | 2.6.2 (mới)                         |
| 22  | `screen_ota_progress.png`      | Screenshot UI            | Modal "Update progress" hiển thị tiến độ batch update                               | 2.6.2 (mới)                         |
| 23  | `sequence_ota.png`             | Sequence diagram         | Luồng OTA 7 bước Frontend ↔ Backend ↔ MQTT ↔ ESP32                                  | 2.6 (mới)                           |
| 24  | `priority_logic_diagram.png`   | Sơ đồ khối               | 4 mức ưu tiên EMERGENCY > DANG-Lockout > AUTO > MANUAL                              | 2.3.X (Logic Control)               |
| 25  | `mutex_resource_diagram.png`   | Sơ đồ tài nguyên         | 3 Mutex (data/i2c/mqtt) bảo vệ tài nguyên nào                                       | 2.3.X (Mutex)                       |

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

### Bắt buộc P0 (sai lệch với code thực tế / chức năng cốt lõi)

- [ ] **Sửa pin mapping** — Relay 32/33 (không phải 12/13), I2C SDA/SCL = 21/22 (không phải 26/27), thêm Servo, Buzzer, 3 nút khẩn cấp, ADS1115.
- [ ] **Thêm subsection ADS1115** (firmware đọc qua I2C, không dùng ADC nội).
- [ ] **Thêm subsection Servo SG90** + vai trò điều khiển cửa sổ.
- [ ] **Thêm subsection Còi SFM-27** + driver transistor.
- [ ] **Thêm subsection 3 nút khẩn cấp** (GPIO 34/35/23, FALLING ISR).
- [ ] **Sửa bảng RTOS task** — từ 4 task lên **6 task** (thêm `taskEmergency` Prio 6, `taskOTA` Prio 5).
- [ ] **Sửa subsection Mutex** — từ 1 mutex lên **3 mutex** (data/i2c/mqtt).
- [ ] **Sửa Payload telemetry** — thêm `emergency`, `buzzer`, `window`; đổi tên field cho khớp.
- [ ] **Sửa Payload control** — thêm `buzzer`, `window`.
- [ ] **Sửa bảng MQTT topic** — thêm 2 topic OTA (`air/updatefirmware`, `air/firmwareupdatestatus`).
- [ ] **VIẾT MỚI Section 2.4 — Hệ thống Khẩn cấp (Emergency Mode)** — 7 subsection (B.2.6).
- [ ] **VIẾT MỚI Section 2.5 — Watchdog + RTC Backup** — 5 subsection (B.2.7).
- [ ] **VIẾT MỚI Section 2.6 — OTA Firmware Update** — 5 subsection (B.2.8). MOVE khỏi "Hướng phát triển".
- [ ] **Thêm bảng 4 mức ưu tiên** EMERGENCY > DANG-Lockout > AUTO > MANUAL.

### Bắt buộc P1 (đã có ở thong_tin_bo_sung.md cũ)

- [ ] Thêm subsection LCD I2C 16x2 (B.2.1.a).
- [ ] Thêm subsection Relay 2 kênh (B.2.1.b).
- [ ] Bổ sung NTP, EWMA, AQI threshold, anti-noise LCD, MQTT TLS trong 2.3.
- [ ] **Viết MỚI Section 2.7 (Backend)** — 7 subsection (đổi số từ 2.4 → 2.7).
- [ ] **Viết MỚI Section 2.8 (Frontend)** — 5 subsection (đổi số từ 2.5 → 2.8).
- [ ] Bổ sung bảng so sánh yêu cầu phi chức năng trong Section 2.9 (đổi từ 2.6).
- [ ] Cập nhật Tóm tắt nội dung và Kết luận để khớp kiến trúc 3 tầng + Emergency.
- [ ] AI Summary giữ ở "Hướng phát triển"; OTA chuyển vào phần chính (đã làm ở B.1.2).
- [ ] Ẩn credentials thật trước khi in PDF công khai.

### Chuẩn bị ảnh

- [ ] Chuẩn bị **25 ảnh** ở mục C (gốc 16 + bổ sung 9 ảnh mới cho Emergency / Watchdog / OTA).

### Tổng kết cấu trúc Chương 2 sau khi bổ sung

```
2.1 Xác định bài toán và yêu cầu
2.2 Phần cứng                         (sửa pin mapping + thêm ADS1115/Servo/Buzzer/Nút khẩn cấp)
2.3 Phần mềm Firmware ESP32           (sửa RTOS 6 task + 3 Mutex + 4 mức ưu tiên + payload mới)
2.4 *** Hệ thống Khẩn cấp ***         [SECTION MỚI]
2.5 *** Watchdog + RTC Backup ***     [SECTION MỚI]
2.6 *** OTA Firmware Update ***       [SECTION MỚI]
2.7 *** Backend (Node.js) ***         [SECTION MỚI - dồn từ 2.4 cũ]
2.8 *** Frontend (React) ***          [SECTION MỚI - dồn từ 2.5 cũ]
2.9 Phương pháp thực hiện và kết quả  (đổi số từ 2.6)
```
