---
title: "Mobile Robot Project"
date: 2026-01-03
draft: false
summary: "Quá trình thiết kế và lập trình Mobile Robot tự hành sử dụng vi điều khiển STM32F103, giải thuật PID"
author: "Dang Thanh Tai"
tags: ["STM32", "Robotics"]
---
Chào mừng các bạn đến với dự án đầu tiên trên blog **Tai no Biboroku**.

Là một sinh viên Cơ điện tử, việc tự tay chế tạo một chú Mobile Robot (Robot di động) luôn là một cột mốc đáng nhớ. Trong bài viết này, mình sẽ chia sẻ lại quá trình xây dựng một Mobile Robot cơ bản sử dụng dòng chip STM32, tập trung vào việc xử lý cân bằng tốc độ động cơ bằng giải thuật PID.

## 1. Tại sao lại chọn STM32?

Ban đầu mình định dùng Arduino Uno vì sự đơn giản. Tuy nhiên, với mục tiêu học sâu về Embedded Systems và chuẩn bị cho các dự án Automotive sau này, mình quyết định chọn **STM32F103C8T6 (Blue Pill)**.

Lý do rất đơn giản:
* **Tốc độ xử lý:** 72MHz (so với 16MHz của Arduino).
* **Nhiều ngoại vi:** Nhiều kênh PWM, ADC tốc độ cao và đặc biệt là Hardware Timer hỗ trợ tốt cho Encoder.
* **Môi trường:** Làm quen với KeilC và thư viện HAL là bước đệm tốt cho công việc kỹ sư nhúng.

## 2. Phần cứng (Hardware Design)

Để robot hoạt động ổn định, mình đã lựa chọn các linh kiện sau:

* **Vi điều khiển:** STM32F103C8T6.
* **Động cơ:** 2 Motor DC giảm tốc GA25-370 (có gắn sẵn Encoder để đo tốc độ).
* **Driver điều khiển động cơ:** Module TB6612FNG (nhỏ gọn và hiệu suất cao hơn L298N).
* **Cảm biến:** HC-SR04 (đo khoảng cách vật cản) và MPU6050 (đo góc nghiêng/gia tốc).
* **Nguồn:** Pin Li-ion 18650 2S (7.4V).

Sơ đồ nguyên lý được thiết kế trên Altium Designer, sau đó mình tự ủi mạch và hàn linh kiện để tối ưu kích thước.

## 3. Giải thuật điều khiển (PID Control)

Thách thức lớn nhất của các robot 2 bánh là làm sao để nó đi thẳng. Do sai số cơ khí, hai động cơ dù cấp cùng một điện áp nhưng tốc độ thực tế sẽ khác nhau, dẫn đến robot bị lệch hướng.

Giải pháp ở đây là **Bộ điều khiển PID**.
* **Input:** Tốc độ thực tế đọc từ Encoder.
* **Setpoint:** Tốc độ mong muốn.
* **Output:** Xung PWM cấp cho Driver.

Dưới đây là đoạn code C mình triển khai PID trên STM32:

```c
/* Cấu trúc dữ liệu PID */
typedef struct {
    float Kp, Ki, Kd;
    float error, pre_error;
    float integral;
    float output;
} PID_Config;

/* Hàm tính toán PID - Gọi trong ngắt Timer mỗi 10ms */
float PID_Calculate(PID_Config *pid, float setpoint, float current_speed) {
    // 1. Tính sai số
    pid->error = setpoint - current_speed;

    // 2. Tính thành phần P, I, D
    float P_term = pid->Kp * pid->error;
    pid->integral += pid->error;
    float I_term = pid->Ki * pid->integral;
    float D_term = pid->Kd * (pid->error - pid->pre_error);

    // 3. Tính ngõ ra
    pid->output = P_term + I_term + D_term;

    // 4. Lưu sai số cho lần tính sau
    pid->pre_error = pid->error;

    // Giới hạn xung PWM (Clamping)
    if (pid->output > 1000) pid->output = 1000;
    if (pid->output < 0) pid->output = 0;

    return pid->output;
}