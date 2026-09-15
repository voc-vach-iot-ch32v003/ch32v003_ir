# 📡 ch32v003_ir

Thư viện thu, phát, giải mã và phân tích tín hiệu hồng ngoại (IR) đa giao thức (NEC, Sony, Samsung, TCL) dành cho vi điều khiển **CH32V003**. Thư viện chạy trên nền framework **ch32v003fun**, hỗ trợ tính năng học lệnh (raw signal), tự động nhận dạng giao thức và gợi ý cấu hình protocol mới qua UART Debug.

---

## 🚀 Tính năng nổi bật

- **Đa giao thức & Linh hoạt:** Hỗ trợ sẵn các chuẩn hồng ngoại phổ biến (`NEC`, `Sony`, `Samsung`, `TCL`) và dễ dàng mở rộng thêm protocol mới qua file cấu hình.
- **Tích hợp tính năng Học lệnh (Raw Signal):** Quét và phát trực tiếp mảng xung thô (`IR_RawData_t`) mà không cần giải mã.
- **Phân tích & Chẩn đoán giao thức (Protocol Finder):** Hàm `irAnalyzeRaw()` tự động đo đạc thời gian xung, tính dung sai, in mảng Byte Hex gợi ý và xuất struct cấu hình cho remote lạ.
- **API kép linh hoạt:** Hỗ trợ API chuẩn chân vật lý (`MCU_PINx`) phong cách Arduino và API truy xuất Port thô tốc độ cao (`GPIOA`, `GPIOC`, `GPIOD`).
- **Hỗ trợ kiểu mã hóa & Thứ tự Bit:** Xử lý linh hoạt cả mã hóa khoảng lặng (`IR_ENCODE_DISTANCE`) lẫn độ rộng xung (`IR_ENCODE_WIDTH`), hỗ trợ cả truyền `LSB-First` và `MSB-First`.

---

## 📑 Cấu trúc hàm (API Reference)

### 1. API Thu / Phát theo chân vật lý (Arduino Style)

| Hàm xử lý                                        | Tham số                                                                            | Giá trị trả về                         | Mô tả                                                                   |
| ------------------------------------------------ | ---------------------------------------------------------------------------------- | -------------------------------------- | ----------------------------------------------------------------------- |
| **`irRead(mcuPin, irData)`**                     | `uint8_t mcuPin`, `IR_Data_t* irData`                                              | `uint8_t` (1: Thành công, 0: Thất bại) | Đọc và giải mã gói tin IR trực tiếp từ chân vật lý chỉ định (Hàm chặn). |
| **`irReadRaw(mcuPin, rawData)`**                 | `uint8_t mcuPin`, `IR_RawData_t* rawData`                                          | `uint8_t` (1: Thành công, 0: Thất bại) | Thu thập chuỗi thời gian xung thô phục vụ tính năng học lệnh.           |
| **`irSend(mcuPin, protocol, address, command)`** | `uint8_t mcuPin`, `IR_Protocol_t protocol`, `uint16_t address`, `uint32_t command` | `void`                                 | Phát mã điều khiển IR chuẩn hóa ra chân vật lý theo giao thức chỉ định. |
| **`irSendRaw(mcuPin, rawData)`**                 | `uint8_t mcuPin`, `const IR_RawData_t* rawData`                                    | `void`                                 | Phát lại tín hiệu hồng ngoại thô đã thu thập trước đó (Phát học lệnh).  |

### 2. API làm việc trực tiếp theo Cổng Port (Tốc độ cao)

| Hàm xử lý                                                      | Tham số                                                                                                      | Giá trị trả về | Mô tả                                                      |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | -------------- | ---------------------------------------------------------- |
| **`irReadPort(GPIOx, pinNumber, irData)`**                     | `const GPIO_TypeDef* GPIOx`, `uint8_t pinNumber`, `IR_Data_t* irData`                                        | `uint8_t`      | Quét và giải mã tín hiệu IR theo Port và vị trí bit (0-7). |
| **`irReadPortRaw(GPIOx, pinNumber, rawData)`**                 | `const GPIO_TypeDef* GPIOx`, `uint8_t pinNumber`, `IR_RawData_t* rawData`                                    | `uint8_t`      | Ghi nhận mảng xung thô dựa vào thanh ghi Port.             |
| **`irSendPort(GPIOx, pinNumber, protocol, address, command)`** | `GPIO_TypeDef* GPIOx`, `uint8_t pinNumber`, `IR_Protocol_t protocol`, `uint16_t address`, `uint32_t command` | `void`         | Phát tín hiệu băm tần số sóng mang ra chân Port.           |
| **`irSendPortRaw(GPIOx, pinNumber, rawData)`**                 | `GPIO_TypeDef* GPIOx`, `uint8_t pinNumber`, `const IR_RawData_t* rawData`                                    | `void`         | Phát lại mảng xung thô trên Port chỉ định.                 |

### 3. API Chẩn đoán & Debug

| Hàm xử lý                   | Tham số                       | Giá trị trả về | Mô tả                                                                                              |
| --------------------------- | ----------------------------- | -------------- | -------------------------------------------------------------------------------------------------- |
| **`irAnalyzeRaw(rawData)`** | `const IR_RawData_t* rawData` | `void`         | Phân tích mảng xung thô, kiểm tra mức độ trùng khớp protocol và in gợi ý Struct cấu hình lên UART. |
| **`irPrintResult(irData)`** | `const IR_Data_t* irData`     | `void`         | In chi tiết kết quả giải mã (Protocol, Bits, Raw Hex, Address, Command, Checksum) qua UART.        |

---

## ⚙️ Cấu hình hệ thống & Biên dịch

Thư viện phụ thuộc vào `ch32v003_gpio`, `ch32v003_delay`, `ch32v003_timer`, và `ch32v003_debug`. Bạn có thể tinh chỉnh các thông số hệ thống trong file `sys_config.h` hoặc `platformio.ini`:

```c
// Bật/Tắt các giao thức cần dùng để tiết kiệm dung lượng Flash
#define IR_SUPPORT_NEC          1
#define IR_SUPPORT_SAMSUNG      1
#define IR_SUPPORT_SONY         1
#define IR_SUPPORT_TCL          1

// Tham số thời gian và dung lượng bộ đệm
#define IR_TIMEOUT_US           15000 // Thời gian chờ tối đa chặn treo chip (15ms)
#define IR_TOLERANCE_PERCENT    25    // Dung sai cho phép khi so khớp xung (25%)
#define IR_RAW_BUFFER_SIZE      256   // Số lượng sườn xung tối đa trong 1 khung
#define IR_MAX_SUPPORTED_BITS   80    // Số bit dữ liệu tối đa hỗ trợ giải mã

```

---

## 📝 Code mẫu sử dụng

### 1. Đọc và giải mã tín hiệu IR (IR Receiver)

```c
#include "ch32fun.h"
#include <ch32v003_gpio.h>
#include <ch32v003_delay.h>
#include <ch32v003_debug.h>
#include <ch32v003_ir.h>

#define IR_RX_PIN MCU_PIN3 // Chân nhận IR kết nối mắt thu (VD: VS1838B)

void setup() {
    pinMode(IR_RX_PIN, INPUT_PULLUP);
}

void loop() {
    IR_Data_t irData;

    // Quét và giải mã gói tin
    if (irRead(IR_RX_PIN, &irData)) {
        // In chi tiết kết quả giải mã ra UART Debug
        irPrintResult(&irData);
    }
}

int main() {
    SystemInit();
    SetupDebugPrintf(); // Khởi tạo UART Debug Printf
    delayMs(1000);

    setup();
    while (1) {
        loop();
    }
}

```

### 2. Phân tích remote lạ & Học lệnh (IR Analyzer & Raw Learn)

```c
#include "ch32fun.h"
#include <ch32v003_gpio.h>
#include <ch32v003_delay.h>
#include <ch32v003_debug.h>
#include <ch32v003_ir.h>

#define IR_RX_PIN MCU_PIN3
#define IR_TX_PIN MCU_PIN4

void setup() {
    pinMode(IR_RX_PIN, INPUT_PULLUP);
    pinMode(IR_TX_PIN, OUTPUT);
}

void loop() {
    IR_RawData_t rawData;

    // Thu thập dữ liệu xung thô từ remote
    if (irReadRaw(IR_RX_PIN, &rawData)) {
        // Phân tích dữ liệu xung thô và in gợi ý Struct lên màn hình
        irAnalyzeRaw(&rawData);

        // Trễ ngắn và phát lại chính xác xung thô vừa học (Phát học lệnh)
        delayMs(2000);
        irSendRaw(IR_TX_PIN, &rawData);
    }
}

int main() {
    SystemInit();
    SetupDebugPrintf();
    delayMs(1000);

    setup();
    while (1) {
        loop();
    }
}

```

---
