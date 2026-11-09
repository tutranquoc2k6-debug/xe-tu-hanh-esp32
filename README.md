# Xe tự hành ESP32 – né vật cản, điều khiển và đi theo hình vẽ qua web

Xe 4 bánh dùng **ESP32 DevKit V1**, cảm biến siêu âm **HC-SR04** gắn trên servo **MG995** và driver **L298N**.
ESP32 tự phát WiFi và chạy web server, nên điện thoại chỉ cần mở trình duyệt là giám sát và điều khiển được xe. Không cần cài ứng dụng, không cần Internet.

> Đồ án môn học – Trường Đại học Công nghệ, ĐHQG Hà Nội (UET), 2026.

<p align="center"><img src="docs/hinh/xe.jpg" width="420" alt="Ảnh xe tự hành"></p>

## Nhóm thực hiện

| Họ và tên | MSSV |
|---|---|
| Trần Quốc Anh Tú | 24022227 |
| Nguyễn Khắc Tùng | 240222231 |
| Lê Anh Đức | 24022099 |

## 4 chế độ hoạt động

| Chế độ | Hoạt động | An toàn |
|---|---|---|
| **Tự động** (mặc định khi bật nguồn) | Chạy, giảm tốc, lượn tránh vật. Bị chặn thì dừng cách vật ≈ 10 cm, lùi ≈ 5 cm, quét 2 bên rồi rẽ sang hướng trống. Ngõ cụt thì lùi ≈ 15 cm, quét lại, quay đầu 180°. | Có chống kẹt góc phòng |
| **Lái tay** | Giữ nút ▲ ▼ ◀ ▶ trên web để chạy, thả nút là dừng; chỉnh được tốc độ | Mất lệnh 0,4 s thì tự dừng; phía trước ≤ 15 cm thì không cho tiến |
| **Đi theo hình vẽ** | Vẽ một nét trên điện thoại, web đổi thành các đoạn "quay X°, chạy Y cm", xe đi theo | Gặp vật ≤ 18 cm thì phanh, dừng |
| **Dừng** | Dừng ngay, hủy mọi thao tác đang làm dở | – |

Trang web còn có: radar 0°–180°, khoảng cách trái/trước/phải, tốc độ 2 bánh, trạng thái, nhật ký, nút quét radar và mục hiệu chỉnh quãng đường/góc quay (lưu vào Flash).

<p align="center"><img src="docs/hinh/web.png" width="640" alt="Giao diện web (số liệu minh họa)"></p>

## Phần cứng

| Linh kiện | SL |
|---|---|
| ESP32 DevKit V1 (30 chân) + đế mở rộng G-V-S | 1 |
| Driver L298N | 1 |
| Động cơ DC giảm tốc + bánh | 4 |
| HC-SR04 | 1 |
| Servo MG995 (180°) | 1 |
| Pin 18650 (3S) + hộp pin + công tắc | 3 |
| Mạch hạ áp LM2596 (chỉnh 5,0 V) | 1 |

### Đấu nối

| Ngoại vi | Chân | ESP32 |
|---|---|---|
| L298N (bên trái) | ENA / IN1 / IN2 | D25 / D26 / D27 |
| L298N (bên phải) | ENB / IN3 / IN4 | D14 / D33 / D32 |
| HC-SR04 | Trig / Echo | D18 / D19 (VCC 5 V) |
| Servo MG995 | Tín hiệu | D13 (nguồn 5 V đủ dòng ≥ 3 A) |

<p align="center"><img src="docs/hinh/daynoi.png" width="640" alt="Sơ đồ đấu nối"></p>

## Cài đặt và nạp code

1. Cài **Arduino IDE 2.x**, rồi vào Boards Manager cài gói **esp32 (Espressif) bản 3.x**.
2. Mở `firmware/xe_v7_ve_hinh/xe_v7_ve_hinh.ino`. Không cần thêm thư viện, vì `WiFi`, `WebServer` và `Preferences` có sẵn trong gói esp32.
3. Chọn board **ESP32 Dev Module**, chọn cổng COM, bấm Upload. Khi hiện `Connecting...` thì giữ nút **BOOT**.
   - Cắm cáp vào cổng micro-USB **trên chính ESP32**, không cắm vào đế mở rộng.
4. Mở Serial Monitor ở tốc độ 115200 để xem nhật ký.

## Sử dụng web

1. Bật xe, rồi trên điện thoại kết nối WiFi **`XE_ESP32`**, mật khẩu **`12345678`**.
2. Nếu điện thoại báo "không có Internet" thì chọn giữ kết nối và **tắt dữ liệu di động**.
3. Mở trình duyệt, gõ **`http://192.168.4.1`** (gõ đủ cả `http://`).

### Hiệu chỉnh (nên làm trước khi đi theo hình vẽ)
Mở mục **Hiệu chỉnh** dưới khung vẽ:
- Bấm **Chạy thử 1 m**, đo quãng đường xe đi được thật rồi nhập vào và bấm Lưu.
- Bấm **Quay thử 1 vòng**, nhập số độ xe quay được thật rồi bấm Lưu.

Làm lại 2–3 lần. Số đã lưu vẫn còn sau khi tắt nguồn.

### Tỉ lệ khung vẽ
Cả khung vẽ bằng kích thước chọn ở ô **Khung** (1 / 2 / 3 / 5 m). Ví dụ khung 2 m thì mỗi ô lưới là 50 cm.
Xe đặt ở chấm xanh, mũi xe hướng lên trên màn hình.

## Lưu đồ thuật toán

| Tổng quát (chọn chế độ) | Chế độ Tự động | Đi theo hình vẽ |
|---|---|---|
| <img src="docs/hinh/luudo_tong.png" width="300"> | <img src="docs/hinh/luudo_tudong.png" width="300"> | <img src="docs/hinh/luudo_hinh.png" width="160"> |

## Cấu trúc thư mục

```
firmware/
  xe_v7_ve_hinh/        code chính (bản mới nhất)
  phien_ban_cu/         v4 (quay mượt), v5 (sửa thoát kẹt), v6 (thêm web)
docs/
  BaoCao_XeTuHanh_ESP32_v7.docx   báo cáo
  hinh/                 sơ đồ khối, kiến trúc, đấu nối, lưu đồ, ảnh giao diện
```

## Các tham số hay chỉnh (đầu file `.ino`)

| Tham số | Mặc định | Ý nghĩa |
|---|---|---|
| `DUNG_CM` / `DU_PHANH` | 10 / 10 cm | Khoảng cách dừng / phanh sớm thêm |
| `SPEED_CRUISE` / `SPEED_NEAR` | 160 / 145 | PWM khi chạy thẳng / khi gần vật |
| `TI_LE_XA` / `TI_LE_GAN` | 0.85 / 0.75 | Tỉ lệ chạy xung (nhỏ hơn thì chậm hơn) |
| `SPEED_TURN` | 210 | PWM khi quay tại chỗ |
| `LECH_TRAI_PHAI` | 0 | Bù khi đi thẳng bị lệch (lệch phải: 10, lệch trái: -10) |
| `TU_CHAY_KHI_BAT` | true | `false` thì bật nguồn xe đứng chờ lệnh từ web |
| `MODE` | 0 | 1: thử động cơ, 2: thử cảm biến, 3: thử quay |

## Giới hạn và hướng phát triển

- Pin 18650 bị sụt áp khi servo và 4 động cơ chạy cùng lúc, có thể làm ESP32 khởi động lại. Cách khắc phục: sạc đầy pin, dùng pin xả dòng cao, hoặc tách nguồn riêng cho servo.
- Xe không có encoder nên quãng đường và góc quay chỉ ước lượng theo thời gian, vì vậy đi theo hình vẽ chỉ đúng tương đối. Hướng cải tiến: thêm encoder và MPU6050.
- L298N sụt khoảng 2 V. Có thể thay bằng TB6612FNG.
- Hướng mở rộng: dùng camera và YOLO để nhận diện, bám mục tiêu.
