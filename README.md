# 🛒 Edge-Enabled Smart Shopping
**An AI-Powered Cart System for Real-Time Product Recommendation and Indoor Localization**

---

## 📝 Các điểm cải thiện sau lần trình bày 1

| Vấn đề ban đầu                            | Giải pháp cập nhật                                                 |
|------------------------------------------|--------------------------------------------------------------------|
| ❌ Chưa có sơ đồ tổng quát               | ✅ Đã bổ sung sơ đồ hệ thống ở Slide 5                             |
| ❌ Chưa nêu rõ nguồn dữ liệu              | ✅ Mô tả chi tiết từng nguồn dữ liệu (tự thu hoặc doanh nghiệp)     |
| ❌ Chức năng nút nhấn chưa rõ ràng       | ✅ Là nút bật/tắt chế độ Smart                                     |
| ❌ Vai trò Wi-Fi module chưa rõ          | ✅ Ghi log người dùng, OTA, kết nối hệ thống                       |
| ❌ Chưa rõ thuật toán BLE & KWS          | ✅ BLE dùng MLP, KWS dùng MobileNetV2 đã quantized INT8            |
| ❌ Có nút “Out-of-Stock” thừa trong User Flow | ✅ Đã loại bỏ khỏi sơ đồ luồng người dùng                         |

---

## 📊 Dataset

| Loại dữ liệu        | Số lượng    | Nguồn thu thập                         |
|---------------------|-------------|----------------------------------------|
| Giọng nói (KWS)     | 1350 mẫu    | Tự thu thập bằng micro thực tế         |
| RSSI BLE            | 1165 mẫu    | Tự thu từ 3 beacon tại 6 vị trí khác nhau |
| Dữ liệu sản phẩm    | >50 sản phẩm| Doanh nghiệp cung cấp (giá, tên, ID...) |

---

## 🧠 Mô hình AI sử dụng

| Tác vụ               | Mô hình             | Chi tiết kỹ thuật                                     |
|----------------------|---------------------|--------------------------------------------------------|
| Keyword Spotting     | MobileNetV2 (0.35x) | Nhẹ, hiệu quả, quantized INT8 để chạy trên MCU        |
| BLE Localization     | MLP (3 input → 6 output) | Fully Connected NN phân loại vị trí theo RSSI      |

> ✅ Cả hai mô hình đã được convert và quantized (INT8) để tương thích với CMSIS-NN và TFLite Micro cho vi điều khiển EFR32MG24 (Cortex-M33, 78 MHz).

Mô hình Keyword Spotting:

![image](https://github.com/user-attachments/assets/236229ac-9eaf-4d05-b535-cfda9748d42d)

Mô hình BLE Localization:

![image](https://github.com/user-attachments/assets/5d473b38-46b4-4dfe-9518-3722990882cd)


---

## 📉 So sánh tài nguyên hệ thống

| Tiêu chí                      | KWS (TFLite)              | BLE (MLP)                    | MG24 MCU                      |
|-------------------------------|---------------------------|------------------------------|-------------------------------|
| Kích thước mô hình            | ~733.5 KB                 | ~26.6 KB                     | 1536 KB Flash                 |
| Tensor Arena RAM              | ~216.3 KB                 | ~1.7 KB                      | 256 KB RAM                    |
| Inference time                | ~408 ms                   | ~1 ms                        | Cortex-M33 @ 78 MHz           |
| Tổng bộ nhớ sử dụng           | ~949.8 KB (Flash + RAM)   | ~28.3 KB (Flash + RAM)       | 1630 KB tổng (Flash + RAM)   |
| Ước lượng điện năng tiêu thụ  | ~3.5 mJ / inference       | ~0.0085 mJ / inference       | ~2.6 mA @ 3.3V                |

---

## 🧪 Live Demo: 2 chức năng song song
### 1. Cách thu thập dữ liệu:
#### 1.1 Giọng nói
- Sử dụng ứng dụng ghi âm (Voice Recorder) trên điện thoại/laptop.

- Ghi lại từng từ khóa, mỗi từ lặp lại ~30–50 lần (từ nhiều người, nhiều môi trường).

- Lưu dưới định dạng `.wav`, tần số lấy mẫu 16kHz, mono
#### 1.2 RSSI
- 3 BLE Beacon phát tín hiệu (ESP32) -> Ghi lại RSSI từ 3 beacon tại từng vị trí (khoảng 50 mẫu mỗi vị trí)

- 1 thiết bị thu RSSI: ESP32 -> Quét BLE

- Lưu lại thành file `.csv` để huấn luyện:
```
rssi1,rssi2,rssi3,label
-89,-76,-85,entrance
-88,-74,-84,entrance
...
-65,-90,-70,checkout
```

### 1. Web Dashboard (Edge Impulse)
- Hiển thị trực quan mô hình BLE và KWS.
- Test mô hình trong tab **Model Testing**.

### 2. Local Inference (Ubuntu Terminal)
- Mô phỏng quá trình inference trên thiết bị biên (MCU-like).
- Các bước thực hiện:
```bash
cd demo
nano main.cpp   # Chỉnh sửa raw features của input
sh build.sh     # Biên dịch chương trình
./build/app     # Chạy chương trình mô phỏng inference
```

### 3. 
