**Chức năng:**
D-ITG (Distributed Internet Traffic Generator) là một phần mềm chuyên dụng để tạo, mô phỏng và đo lường lưu lượng mạng ở cấp độ gói tin.

Chức năng cốt lõi của D-ITG bao gồm 3 điểm chính:

* **Phát sinh tải mạng (Traffic Generation):** Tạo ra các luồng dữ liệu trên nhiều giao thức khác nhau (TCP, UDP, ICMP, VoIP, DNS, Telnet...) từ một thiết bị này sang một thiết bị khác.
* **Mô phỏng thống kê (Statistical Modeling):** Nó có khả năng tùy chỉnh kích thước gói tin và khoảng thời gian giữa các gói tin theo các hàm phân phối toán học (Constant, Uniform, Poisson, Pareto, Normal...). Việc này giúp giả lập chính xác các kiểu dữ liệu Internet đa dạng trong thực tế, thay vì chỉ gửi dữ liệu tĩnh, đều đặn.
* **Đo lường chất lượng dịch vụ (QoS Measurement):** Ghi nhận và tính toán chi tiết các chỉ số hiệu năng mạng ở đầu nhận, bao gồm:
    * **Throughput:** Băng thông truyền tải.
    * **One-way Delay:** Trễ một chiều của từng gói tin.
    * **Jitter:** Độ biến thiên của trễ.
    * **Packet Loss:** Tỷ lệ rớt gói.

---

### 1. Cấu trúc lệnh của D-ITG
Cấu trúc lệnh của D-ITG gồm 3 loại lệnh khác nhau, và điều đó phụ thuộc vào bản chất kiến trúc 3 thành phần của nó. D-ITG không được gộp chung vào một giao diện duy nhất mà tách làm 3 module riêng biệt chạy trên Terminal để tối ưu hóa hiệu năng.

**1.1 ITGRecv (Máy Thu):**
Mở cổng (mặc định 8999) để lắng nghe lưu lượng. Nó mở cổng và đứng chờ dữ liệu tới. Khi đang chạy, nó không in kết quả ra màn hình hay lưu thành file văn bản (Text). Nó chỉ làm một việc duy nhất là ghi dữ liệu thô dưới dạng nhị phân trực tiếp xuống ổ cứng với tốc độ cực nhanh. Việc này giúp CPU không bị quá tải khi phải hứng hàng ngàn gói tin mỗi giây, đảm bảo hiện tượng rớt gói (Packet drop) không xảy ra do phần mềm xử lý chậm (I/O Bottleneck).

**1.2 ITGSend (Máy Phát):**
Đọc lệnh cấu hình và tiến hành bơm dữ liệu sang máy thu để tạo tải mạng.

**1.3 ITGDec (Bộ Giải Mã):**
Hoạt động offline (tức là sau khi quá trình bắn dữ liệu đã kết thúc hoàn toàn). Từ file nhị phân mà ITGRecv vừa lưu lên, nó thực hiện các phép toán để tính ra Trễ (Delay), Biến thiên trễ (Jitter), Suy hao (Loss) và dịch ra chữ.

---

### 2. Quản lý Không gian làm việc (Workspace Management)
D-ITG không chạy ngầm (Daemonize), do đó việc quản lý luồng Terminal là bắt buộc để tránh xung đột tiến trình và lỗi đường dẫn:

* **Phân luồng Terminal:** Sử dụng ít nhất 2 phiên làm việc riêng biệt.
    * **Terminal 1** (Tại thư mục src): Khởi chạy và treo cố định tiến trình `./ITGRecv/ITGRecv`.
    * **Terminal 2** (Tại thư mục src): Thực thi các lệnh bơm tải `./ITGSend` và giải mã `./ITGDec`.
    * **Terminal 3** (Tại thư mục gốc): Dùng cho các tác vụ quản lý mã nguồn như Git, tránh tình trạng gõ `git add .` bên trong src dẫn đến việc bỏ sót các thư mục kết quả nằm ở phân vùng khác (như results). // Không nhất thiết phải một terminal riêng thứ 3
* **Khởi tạo cấu trúc lưu trữ:** Bộ giải mã sẽ báo lỗi văng (Crash) nếu thư mục đích không tồn tại. Chú ý dùng `mkdir -p` (ví dụ: `mkdir -p ../results/Burst_Traffic`) trước khi dùng toán tử điều hướng `>` để xuất file log.

---

### 3. Quy trình thực chiến và Cú pháp chi tiết (Trình tự 3 bước bắt buộc)
Quy trình sử dụng D-ITG luôn cố định theo thứ tự: **Bật ITGRecv -> Bắn ITGSend -> Dịch ITGDec.**

**Bước 1: Bật máy thu**
Mở Terminal(Terminal 1) tại máy đích và gõ lệnh: `./ITGRecv/ITGRecv`.
Lúc này màn hình sẽ hiện thông báo chờ kết nối và đứng im. Lúc này thì ta không cần làm gì cả.

**Bước 2: Cấu hình và Bắn gói tin (Tại máy phát)**
Mở một Terminal thứ 2. Một lệnh ITGSend luôn có cấu trúc: *Địa chỉ đích (-a) -> Giao thức (-T) -> Hàm định hình gói tin -> Quản lý thời gian (-t) và Lưu trữ (-x).*

Dưới đây là cách dùng 4 kịch bản bắn cốt lõi nhất:

* **Cú pháp 1 - Bắn tải đều đặn (Constant Traffic):** Gói tin bay ra liên tục, đều đặn, dùng để test băng thông lý tưởng.
    * Lệnh: `./ITGSend/ITGSend -a 127.0.0.1 -T UDP -C 100 -c 500 -t 20000 -x test.log`.
    * Giải thích: Bắn vào IP đích 127.0.0.1 (`-a`), dùng giao thức UDP (`-T`), tốc độ bắn cố định là 100 gói/giây (`-C 100`), kích thước mỗi gói nặng 500 bytes (`-c 500`), chạy liên tục trong 20.000 mili-giây tức 20 giây (`-t 20000`), và ra lệnh cho máy nhận phải lưu nhật ký vào file nhị phân tên là test.log (`-x test.log`).

* **Cú pháp 2 - Bắn tải ngẫu nhiên (Poisson / Normal):** Gói tin bay lúc ít lúc nhiều theo hàm toán học, giả lập rất hành vi của người dùng thật lướt web (lúc click, lúc nghỉ).
    * Lệnh: `./ITGSend/ITGSend -a 127.0.0.1 -T UDP -O 50 -c 500 -t 20000 -x test.log`.
    * Giải thích: Thay vì dùng chữ C cố định, ta dùng cờ `-O 50` để yêu cầu bắn với tốc độ trung bình 50 gói/giây nhưng biến thiên ngẫu nhiên theo hàm Poisson. Nếu muốn biến thiên theo hàm Gauss/Normal thì dùng cờ `-N`.

* **Cú pháp 3 - Bắn tải đột biến (Burst Traffic):** Ép mạng chạy theo chu kỳ Bật (ON) xối xả rồi Tắt (OFF) nghỉ ngơi liên tục, dùng để xem thiết bị mạng có bị sốc khi lượng lớn dữ liệu ập đến hay không.
    * Lệnh: `./ITGSend/ITGSend -a 127.0.0.1 -C 100 -c 500 -t 30000 -x burst.log -B C 2000 C 500`.
    * Giải thích: Điểm mấu chốt nằm ở cụm `-B C 2000 C 500`. Cú pháp này bắt buộc phải có chữ C (Constant) để phân tách. Nó mang ý nghĩa: Cố định Bật bắn liên tục trong 2000ms (2 giây), sau đó Cố định Tắt nghỉ 500ms (0.5 giây) rồi lặp lại cho đến hết 30 giây. Lưu ý bắt buộc phải có dấu cách giữa `-B`, `C` và các con số, không được viết dính liền.

* **Cú pháp 4 - Bắn đa luồng (Multi-flow) bằng File kịch bản:**
    Thay vì gõ lệnh trực tiếp, ta tạo một file văn bản (ví dụ tên là `script.txt`), ghi mỗi dòng là một luồng mạng khác nhau (VD: dòng 1 tải file TCP, dòng 2 xem video UDP).
    * Lệnh: `./ITGSend/ITGSend script.txt -l sender.log -x receiver.log`.
    * Lưu ý: Bộ phân tích lệnh của D-ITG đọc theo từng dòng, và nó chỉ nhận diện lệnh đã viết xong bằng ký tự xuống dòng. Ta bắt buộc phải gõ phím Enter để tạo một dòng trống ngay sau dòng lệnh cuối cùng trong file script. Nếu không có dấu Enter này ở cuối file, D-ITG sẽ bỏ qua hoàn toàn luồng dữ liệu của dòng cuối cùng đó.

**Bước 3: Giải mã kết quả:**
Sau khi Terminal 2 báo đã gửi xong gói tin, ta sử dụng ITGDec để dịch cái file log nhị phân thành chữ. Có 2 cách giải mã phục vụ 2 mục đích khác nhau:

* **Cách 1 - Lấy Bảng tổng kết trung bình (Averages):** Cung cấp các thông số tổng quan như Average Delay (Trễ trung bình), Average Jitter (Độ biến thiên trễ trung bình), Packet Loss (Tỉ lệ rớt gói).
    * Lệnh: `./ITGDec/ITGDec receiver.log`.
* **Cách 2 - Trích xuất dữ liệu siêu chi tiết (Deep Analysis):** Liệt kê hàng ngàn gói tin, cho biết chính xác thời gian đi, thời gian đến và độ trễ của từng gói một.
    * Lệnh: `./ITGDec/ITGDec receiver.log -pd > ket_qua_chi_tiet.txt`.

Sử dụng cờ `-x` (Receiver log) để đảm bảo tính trễ chính xác. Lưu ý về cờ giải mã: Để trích xuất chi tiết độ trễ từng gói tin, ta bắt buộc phải dùng cờ `-pd` (Packet Delay). Tuyệt đối không được dùng cờ `-d`. Trong D-ITG, cờ `-d` dùng để thiết lập độ trễ cố định (Delay offset) đồng bộ đồng hồ, nếu đưa đường dẫn file vào sau cờ `-d` nó sẽ báo Error 2 ngay lập tức. Việc kết hợp `-pd` với dấu `>` sẽ giúp "hứng" toàn bộ dữ liệu đang tuôn ra và cất gọn gàng vào file văn bản `ket_qua_chi_tiet.txt` để ta xem.

---

### 4. Ràng buộc Môi trường Thực tế (Physical Deployment Constraints)
Khi không kiểm thử trên Loopback (127.0.0.1) mà triển khai trên mạng vật lý giữa 2 node độc lập:

* **Đồng bộ thời gian:** Thuật toán tính Delay của D-ITG phụ thuộc hoàn toàn vào hệ thống Timestamps của hệ điều hành. Nếu hai thiết bị lệch giờ (dù chỉ vài mili-giây), kết quả trễ sẽ bị âm hoặc sai lệch nghiêm trọng. Bắt buộc phải cấu hình đồng bộ qua giao thức NTP hoặc PTP trước khi phát tải.