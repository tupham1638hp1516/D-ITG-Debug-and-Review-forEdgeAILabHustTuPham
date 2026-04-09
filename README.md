# D-ITG-Debug-and-Review-forEdgeAILabHustTuPham
# D-ITG Debugging and Network Evaluation

## Giới thiệu
Dự án này lưu trữ toàn bộ quá trình xử lý lỗi mã nguồn, biên dịch và thực nghiệm trích xuất số liệu chất lượng mạng (QoS) sử dụng công cụ D-ITG (Distributed Internet Traffic Generator).

## Cấu trúc Repository
* `fixed_ditg.zip`: Chứa toàn bộ mã nguồn D-ITG đã được Refactor, file thực thi và các log file liên quan.
* `ket_qua_chi_tiet.txt`: Dữ liệu thô (Raw Data) đã được giải mã từ định dạng nhị phân, chứa thông tin chi tiết hàng ngàn gói tin (Flow, Seq, txTime, rxTime...).

## Báo cáo Xử lý Lỗi (Troubleshooting & Refactoring)
Mã nguồn gốc (Legacy Code) phát sinh xung đột khi biên dịch trên môi trường C++17 trở lên. Dưới đây là các kỹ thuật đã áp dụng:

### 1. Xử lý xung đột định danh (Name Collision)
* **Lỗi:** Cảnh báo `ambiguous "size"` tại `ITGDecod.cpp`. Trình biên dịch không thể phân biệt giữa biến toàn cục `size` của D-ITG và hàm `std::size` của thư viện chuẩn C++.
* **Giải pháp:** Đổi tên (Rename Symbol) toàn bộ biến `size` thành `itg_size` để tách biệt Namespace.

### 2. Khắc phục lỗi mất liên kết cấu trúc dữ liệu
* **Lỗi:** `struct info has no member named seqNum`.
* **Giải pháp:** Vô hiệu hóa các logic tính toán phụ thuộc vào `seqNum` bị thiếu hụt trong file header lõi. Thay vì biên dịch thủ công từng file, tiến hành Master Build bằng lệnh `make clean` và `make` từ thư mục gốc `src` để đảm bảo hệ thống tự động thực hiện Dynamic Linking giữa các module (`common`, `libITG`).

## Hướng dẫn Thực thi
Để trích xuất dữ liệu từ file log đã bắt, sử dụng các lệnh sau trong Terminal (thư mục `src`):
1. **Lấy dữ liệu tổng quan (Delay, Jitter, Bitrate):**
   `./ITGDec/ITGDec receiver.log`
2. **Xuất dữ liệu chi tiết ra dạng văn bản:**
   `./ITGDec/ITGDec receiver.log -l ket_qua_chi_tiet.txt`
