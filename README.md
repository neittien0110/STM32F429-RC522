# STM32F429 đọc thẻ NFC 13.56Mhz thông qua module RC-522

Board **STM32F429I-DISC1** giao tiếp với module **MFRC522**, cho phép đọc mã định danh (UID) của thẻ RFID/NFC tần số 13.56MHz qua giao thức SPI. Kết quả được gửi về máy tính thông qua UART1 để theo dõi.

---

## Nguyên lý hoạt động

Hệ thống hoạt động dựa trên các bước xử lý chính sau:

* **Khởi tạo ngoại vi:** Vi điều khiển cấu hình bộ SPI4 để truyền nhận dữ liệu với MFRC522 và UART1 (115200 baud) để xuất log.
* **Cấu hình Module (Init):** Module MFRC522 được reset và thiết lập các thông số về bộ định thời, độ lợi anten (48dB) và chế độ tự động bật anten.
* **Quét thẻ (Polling):**
    * `Request`: Gửi tín hiệu tìm kiếm thẻ trong vùng từ trường.
    * `Anticollision`: Thực hiện thuật toán chống xung đột để xác định duy nhất một mã UID thẻ trong trường hợp có nhiều thẻ cùng xuất hiện.
* **Phản hồi:** Sau khi kiểm tra, nếu phát hiện thẻ (trả về `MI_OK`), chương trình sẽ soạn chuỗi thông báo và gửi qua UART.

---

## Sơ đồ đấu nối (Wiring Diagram)

Dựa trên cấu hình trong mã nguồn `main.c` và driver `tm_stm32f4_mfrc522.c`, các chân pin được kết nối như sau:

| Module MFRC522 | STM32F429I-DISC1 | Chức năng | Ghi chú |
| :--- | :--- | :--- | :--- |
| **VCC** | **3V3** | Nguồn cấp | Không dùng 5V để tránh hỏng module |
| **GND** | **GND** | Tiếp địa | |
| **SCK** | **PE2** | SPI4_SCK | Xung nhịp đồng bộ SPI |
| **MISO** | **PE5** | SPI4_MISO | Dữ liệu từ thẻ về MCU |
| **MOSI** | **PE6** | SPI4_MOSI | Dữ liệu từ MCU sang thẻ |
| **SDA (SS/CS)** | **PE4** | GPIO Output | Chân chọn chip (NSS Software) |
| **RST** | **3V3 hoặc GPIO** | Reset | Có thể nối cứng lên 3V3 |
| **UART TX** | **PA9** | USART1_TX | Truyền dữ liệu lên PC |

- Kích hoạt SPI4 trên  __STM32F429zIT__ với các chân pin mặc định.\
  ![image](https://github.com/user-attachments/assets/8acf2ef3-82fd-4ed1-94e7-8c52821fc0a6)

---

## 3. Các hàm quan trọng

### 3.1. Driver MFRC522 (`tm_stm32f4_mfrc522.c`)
* `TM_MFRC522_Init()`: Khởi tạo phần cứng, cấu hình anten và các thanh ghi nội bộ của module.
* `TM_MFRC522_Check(uint8_t* id)`: Hàm chính để kiểm tra sự hiện diện của thẻ và đọc UID 5-byte.
* `TM_MFRC522_ToCard()`: Hàm tầng thấp để truyền các lệnh (PCD_AUTHENT, PCD_TRANSCEIVE) và dữ liệu xuống chip RC522.
* `TM_MFRC522_WriteRegister()` / `TM_MFRC522_ReadRegister()`: Giao tiếp trực tiếp với các thanh ghi của RC522 thông qua thư viện HAL SPI.

### 3.2. Ứng dụng chính (`main.c`)
* `MX_SPI4_Init()`: Thiết lập thông số SPI4 (Master, 8-bit, Prescaler 16, Polar Low, Phase 1 Edge).
* `MX_USART1_UART_Init()`: Thiết lập UART1 ở tốc độ 115200 bps.
* `while (1)`: Thực hiện kiểm tra thẻ định kỳ mỗi 400ms bằng hàm `TM_MFRC522_Check()`.

---

## 4. Lưu ý khi triển khai

* **SPI Configuration:** Chân CS (PE4) được điều khiển bằng phần mềm thông qua các macro `MFRC522_CS_LOW` và `MFRC522_CS_HIGH` trong driver.
* **Timing:** Hàm `HAL_Delay(400)` trong vòng lặp `while` giúp hệ thống ổn định và tránh việc đọc lặp lại một thẻ quá nhanh trong thời gian ngắn.
* **Môi trường:** Đảm bảo sử dụng đúng kit STM32F429I-DISC1 và module RC522 hoạt động ở mức điện áp 3.3V.

## Kết quả

- [Video kết quả](https://youtu.be/1ncJdg9u5jw) \
![image](https://github.com/user-attachments/assets/a86f512e-cacb-46ca-a4ea-f99430c4e4c3)

  
