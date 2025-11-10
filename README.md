# Wireless Sensor Network

# Mạng cảm biến không dây - Lập trình mạng cảm biến hình sao nhận kết quả nhiệt độ, độ ẩm từ Sensor Node để gửi lên host

Ý tưởng triển khai: Trong mô hình, các Sensor nodes kết nối trực tiếp với một cổng kết nối trung tâm (Gateway). Gateway sẽ là trung tâm thu thập, xử lý và truyền dữ liệu giúp hệ thống dễ dàng được quản lý, và đảm bảo yêu cầu về độ trễ, thời gian hoạt động và độ chính xác

Từ GW, dữ liệu được gửi đến máy chủ (Server Thingsboard).


1. Yêu cầu dự án:

1.1. Dải đo từ -10 - 70 độ C. với độ chính xác 1 độ C, độ phân giải hiển thị là 0.1.
   
   --> Đo nhiệt độ tại các điểm đo, mỗi điểm đo cách nhau ít nhất 20meters
   
1.2. Với nguồn pin và thời gian hoạt động lên đến 8h
   
   --> Sử dụng Pin Lithium, sạc trực tiếp trên máy, hoặc tháo ra ngoài.
   
1.3. Thời gian đo 1 mẫu ~120seconds.
   
1.4. Khoảng các truyền trong phạm vi 20meters từ hệ thống đo đến trạm thu RF có nối nguồn và mạng.
   
   Với độ trễ thu thập thông tin nhiệt độ, độ ẩm < 10seconds.
   
   --> Sử dụng công nghệ BLE: Thông tin của các node được đưa về GW qua BLE và đẩy lên server bằng Wi-fi.
   
   Gateway là Wifi kết nối với máy tính bằng wifi.
   
1.5. Quản lý tối thiểu 11 thiết bị đo.

   --> Đưa ra định tuyến và gia nhập mạng. Đưa ra thuật toán truyền dữ liệu, phát hiện được thiết bị lỗi và thiết bị thêm vào mạng.
1.6. Phần mềm máy tính: Truyền tải dữ liệu lên Thingsboard.
   
1.7. Có đèn LED báo ngưỡng nhiệt độ (3 LED), độ ẩm. Các ngưỡng nhiệt độ, độ ẩm và chu kỳ đo được cập nhật lên máy tính.

   --> 15-40 độ C, có nút bấm bắt đầu đo.
   
Chưa update được firmware OTA.


2. Lý do lựa chọn giải pháp.

2.1. Chọn mạng Star. --> Dễ thiết lập và quản lý, khắc phục sự cố, hỏng của 1 thiết bị ngoài trung tâm không làm ảnh hưởng tới phần còn lại, và dễ thêm, bớt thiết bị.

ĐN: Mạng Star là một cấu trúc mạng trong đó mọi thiết bị trong mạng đều kết nối với một thiết bị trung tâm (giống như hub, switch, router) để quản lý dữ liệu, điều phối liên lạc giữa các thiết bị còn lại.

*** Nhược điểm: Ảnh hưởng thiết bị trung tâm sẽ ảnh hưởng tới toàn hệ thống, tức là sẽ phải phụ thuộc vào khả năng xử lý của thiết bị trung tâm.

Trong đề tài: Cần node trung tâm là node thu thập dữ liệu từ node cảm biến.

2.2. Lựa chọn công nghệ mạng (BLE).

Do có yêu cầu về tiết kiệm năng lượng tại các Sensor Nodes, với công nghệ phổ biến, khoảng cách truyền tốt (~100m - đã được kiểm chứng) --> Đáp ứng nhu cầu ~20m là đơn giản trong yêu cầu thiết kế. 

--> Cho phép các thiết bị cảm biến đặt xa nhau mà vẫn giữ được khả năng kết nối ổn định.

BTW, tốc độ truyền của BLE lên tới 1Mbps, đủ nhanh để truyền tải dữ liệu như cảm biến nhiệt độ, độ ẩm. 

Đáp ứng cả yêu cầu về giá. 


# Phần cứng: 
1. Sensor Nodes:

  1.1. Input: Pin Lithium (2 Cell pin có điện áp 7.4Volt) 
  
  1.2. Output: Tín hiệu cảm biến nhiệt độ, độ ẩm.
  
  1.3. Sensor: DHT22
  
  1.4. Ổn áp 5V: IC IC-TPS5430DDAR (Hạ áp 5V)

# Chương trình triển khai GW, SN

1. Chương trình tại GW

Bước 1: Thực hiện kết nối WIFI, nếu kết nối thành công thì cấu hình BLE  
Bước 2: Gateway thực hiện scan các node xung quanh và nếu phát hiện thực node có địa  
chỉ mac nằm trong bảng lookup table thì sẽ thực hiện kết nối  
Bước 3: Khi thực hiện kết nối thành công, gateway sẽ đợi để nhận bản tin từ node cảm 
biến  
Bước 4: Sau khi nhận được bản tin từ node cảm biến, gateway sẽ publish bản tin lên 
thingsboard và tiếp tục scan.  

2. Chương trình tại Sensor Node.

Bước 1: Cấu hình ngoại vi và các chế độ cho ESP32  
Bước 2: Khởi tạo mạng BLE, nếu không khởi tạo thành công thử lại và cảnh báo ra 
thông qua LED  
Bước 3: Quảng bá bản tin để yêu cầu tìm kiếm gateway (Advertising-ADV)  
Bước 4: Tìm kiếm gateway  
Node cảm biến kiểm tra xem có tìm thấy gateway không. Bằng việc quảng bá liên tục 
mỗi 20ms, nếu trong 20 giây không thể kết nối gateway sẽ cảnh báo thông qua LED và 
lặp lại. Nếu tìm thấy thành công sẽ gửi thông số cấu hình (MAC – ADDR) cho gateway 
nếu phù hợp với yêu cầu được tạo trong gateway sẽ thiết lập kết nối.  
Bước 5: Thiết lập kết nối:  
Sau khi kết nối được thiết lập. Node sẽ đọc giá trị từ cảm biến DHT22 với các giá trị 
được callback là temperature và humdity.  
Bước 6: Gửi dữ liệu và nhận response  

Node cảm biến sẽ bật LED báo ngưỡng trong 3s và gửi giá trị temperature đo được cho 
Gateway. Node sau đó chờ phản hồi từ gateway. Trong lúc này sẽ đi vào trạng thái deep 
sleep để tiết kiệm năng lượng. Nếu không nhận được phản hồi trong thời gian chờ đợi 
với số lượng < 10 lần sẽ gửi lại bản tin, ngược lại sẽ ngủ sâu và báo LED cảnh cáo sau 
đó quay lại trạng thái khởi động.  
Chương trình hoạt động theo chu kỳ với các bước kiểm tra và phản hồi để đảm bảo việc 
truyền tải dữ liệu nhiệt độ một cách chính xác và hiệu quả, đồng thời sử dụng các cơ chế 
cảnh báo khi có lỗi xảy ra.  

3. Quá trình kết nối.

Khi bắt đầu khởi động, node cảm biến gửi bản tin tới gateway, trên kênh 37,38, 39. 
Tuy nhiên các kênh này có thể fix khác đi. Bản tin gửi bao gồm yêu cầu kết nối, độ 
dài tham số , kiểu địa chỉ , địa chỉ MAC, tín hiệu RSSI, độ dài dữ liệu, công suất 
truyền , UUID và tên  
Sau khi node cảm biến gửi yêu cầu kết nối, gateway thực hiện scan thành công dựa 
trên bảng lookup table của nó, gateway sẽ gửi một bản tin yêu cầu kết nối tới sensor 
node  
Khi mà sensor node nhận được bản tin này, nó bắt đầu phản hồi lại gateway để xác 
thực kết nối  
Bản tin phản hồi sẽ là địa chỉ của sensor node cũng như trạng thái kết nối  
Sau khi thực hiện kết nối thành công, gateway gửi bản tin để đăng ký nhận bản tin từ 
sensor node.  
Từ đây, sensor node sẽ chủ động bắn được bản tin cho gateway.  
