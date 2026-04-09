# D-ITG-Debug-and-Review-forEdgeAILabHustTuPham
# D-ITG Debugging and Network Evaluation

## Giới thiệu
Dự án gốc thuộc sở hữu của tác giả Javier Búcar, tuy nhiên do được code từ năm 2013 nên khi được biên dịch ở môi trường hiện đại gặp phải nhiều vấn đề.
Dự án này lưu trữ toàn bộ quá trình xử lý lỗi mã nguồn, biên dịch và thực nghiệm trích xuất số liệu chất lượng mạng (QoS) sử dụng công cụ D-ITG (Distributed Internet Traffic Generator).

## Cấu trúc Repository
* `docs/`: Tài liệu tham khảo.
* `results/`: Bao gồm các bài kiểm thử và nhận xét đánh giá QoS.
* `src/`: Mã nguồn chạy D-ITG đã được chỉnh sửa.
* `tools/`: Các công cụ hỗ trợ thực nghiệm.

## Báo cáo Xử lý Lỗi (Troubleshooting & Refactoring)
Mã nguồn gốc D-ITG (Legacy Code từ năm 2013) phát sinh xung đột nghiêm trọng khi được biên dịch trên môi trường C++ hiện đại (chuẩn C++17 trở lên). Dưới đây là các kỹ thuật can thiệp mã nguồn đã được áp dụng để khôi phục khả năng thực thi:

### Xử lý xung đột định danh (Name Collision)
Vấn đề: Trình biên dịch báo lỗi ambiguous "size" hàng loạt tại file ITGDecod.cpp. Nguyên nhân là do khai báo using namespace std; khiến trình biên dịch không thể phân biệt giữa biến toàn cục size của phần mềm và hàm std::size mới được bổ sung vào thư viện chuẩn C++.
### Giải pháp: 
Đổi tên toàn bộ biến size thành itg_size trong file ITGDecod.cpp để đóng gói hoàn toàn Namespace, loại bỏ xung đột.

### Xử lý lỗi cấu trúc dữ liệu và Tái thiết lập quy trình Build (Master Build)
Vấn đề: Trình biên dịch báo lỗi struct info has no member named seqNum. Nguyên nhân xuất phát từ sự thiếu đồng bộ của file header lõi (ITG.h) ở phiên bản cũ. Bên cạnh đó, việc biên dịch độc lập từng file (như ITGDecod.cpp) làm phá vỡ kiến trúc liên kết tĩnh của dự án.
### Giải pháp:
Can thiệp mã nguồn: Vô hiệu hóa (comment out) khối lệnh tính toán totalpktloss phụ thuộc vào trường (*infos).seqNum bị thiếu hụt (nằm tại khu vực dòng 1484-1485 của file ITGDecod.cpp).

### Quy trình biên dịch: 
Loại bỏ việc dùng lệnh g++ thủ công. Tiến hành Master Build bằng chuỗi lệnh make clean và make trực tiếp từ thư mục gốc src. Điều này buộc hệ thống tự động xử lý Dynamic Linking, liên kết chính xác các module dùng chung (common, libITG) để tạo ra file thực thi ITGDec hoàn chỉnh.

## Hướng dẫn Thực thi
Để trích xuất dữ liệu từ file log đã bắt, sử dụng các lệnh sau trong Terminal (thư mục `src`):
1. **Lấy dữ liệu tổng quan (Delay, Jitter, Bitrate):**
   `./ITGDec/ITGDec receiver.log`
2. **Xuất dữ liệu chi tiết ra dạng văn bản:**
   `./ITGDec/ITGDec receiver.log -l ket_qua_chi_tiet.txt`
