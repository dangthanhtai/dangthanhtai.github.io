---
title: "Tổng quan về AUTOSAR - Tiêu chuẩn 'vàng' của ngành Automotive"
date: 2026-01-03T15:00:00+07:00
draft: false
summary: "Giải mã kiến trúc AUTOSAR (Classic Platform), tại sao nó lại quan trọng và cách nó thay đổi tư duy lập trình nhúng truyền thống."
author: "Dang Thanh Tai"
tags: ["Embedded", "Automotive", "AUTOSAR"]
cover:
    image: "" # Nếu có ảnh sơ đồ AUTOSAR Layer thì chèn vào đây
    alt: "AUTOSAR Architecture"
    relative: false
---

Nếu như trong lập trình web có mô hình MVC, thì trong lập trình ô tô (Automotive), chúng ta có **AUTOSAR**.

Khi mới bắt đầu tìm hiểu về Automotive, mình đã bị "ngợp" bởi hàng trăm trang tài liệu về chuẩn này. Hôm nay, mình sẽ tóm tắt lại những ý cốt lõi nhất về AUTOSAR (Classic Platform) để các bạn có cái nhìn tổng quan trước khi đi sâu vào chi tiết.

## 1. Tại sao lại cần AUTOSAR?

Hãy tưởng tượng một chiếc xe hơi hiện đại có tới hơn 100 hộp ECU (Electronic Control Unit).
* ECU động cơ do Bosch làm.
* ECU phanh ABS do Continental làm.
* ECU điều hòa do Denso làm.

Nếu không có một chuẩn chung, việc tích hợp các thiết bị này lại với nhau sẽ là một cơn ác mộng. Code viết cho chip của hãng này sẽ không chạy được trên chip của hãng kia.

**AUTOSAR (AUTomotive Open System ARchitecture)** ra đời để giải quyết vấn đề đó với phương châm:
> *"Cooperate on standards, compete on implementation."*
> (Hợp tác trên các tiêu chuẩn, cạnh tranh trên việc thực thi.)

## 2. Kiến trúc phân tầng (Layered Architecture)

Điểm đặc biệt nhất của AUTOSAR là nó chia phần mềm thành các lớp rõ ràng. Điều này giúp tách biệt phần cứng (Hardware) và phần mềm ứng dụng (Application).

Một cách dễ hiểu, nó giống như chiếc bánh Hamburger 3 lớp:

### 2.1. Application Layer (Lớp ứng dụng - "Bánh mì trên")
Đây là nơi chứa logic điều khiển của xe (ví dụ: thuật toán điều khiển phun xăng, thuật toán bật đèn tự động...).
Các khối phần mềm ở đây được gọi là **SWC (Software Component)**. Điều thú vị là các SWC này hoàn toàn không biết gì về phần cứng bên dưới.

### 2.2. RTE (Runtime Environment - "Lớp phô mai/rau")
Đây là lớp trung gian, đóng vai trò như một "tổng đài điện thoại".
* Nếu SWC muốn gửi dữ liệu đi đâu, nó phải gọi qua RTE.
* RTE sẽ quyết định xem dữ liệu đó cần gửi cho một SWC khác nằm cùng chip, hay gửi ra ngoài qua mạng CAN/LIN.

### 2.3. Basic Software (BSW - "Lớp thịt/Bánh mì dưới")
Đây là phần gần phần cứng nhất, bao gồm:
* **Services Layer:** Hệ điều hành (OS), quản lý bộ nhớ, quản lý mạng.
* **ECU Abstraction Layer:** Trừu tượng hóa các chân I/O.
* **MCAL (Microcontroller Abstraction Layer):** Đây là lớp quan trọng nhất với anh em lập trình nhúng. Nó chứa các driver điều khiển trực tiếp thanh ghi (Register) của vi điều khiển (như GPIO, ADC, PWM...).

## 3. Sự khác biệt trong cách Code

Trong lập trình nhúng truyền thống (ví dụ trên STM32 dùng HAL Library), để đọc một nút nhấn, ta viết:

```c
// Lập trình truyền thống
void Read_Button() {
    if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_SET) {
        // Xử lý...
    }
}