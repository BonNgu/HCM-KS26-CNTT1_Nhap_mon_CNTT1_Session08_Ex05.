🚀 Smart-Farm: Hệ thống tưới tự động cho trang trại Lan

Sơ đồ phối hợp phần cứng Smart-Farm: cảm biến → ESP32 → relay → máy bơm, kèm Wi-Fi gửi thông báo về điện thoại

Hình 1. Sơ đồ phối hợp phần cứng của hệ thống Smart-Farm.

📖 Mô tả

Smart-Farm là hệ thống IoT giúp tự động đo nhiệt độ, độ ẩm đất và tự bật/tắt máy bơm đúng lúc cây cần nước. Bạn không cần ra vườn kiểm tra nhiều lần mỗi ngày.

Hệ thống được thiết kế để dễ dùng, tiết kiệm nước và an toàn: nếu có sự cố, máy bơm luôn tự tắt.

👩‍🌾 Dành cho: chị Lan và những người làm nông không cần biết lập trình.

🔍 Phân tích hệ thống (Input / Output)

Hệ thống hoạt động theo mô hình IPO:

📥 Input (Đầu vào)	⚙️ Process (Xử lý)	📤 Output (Đầu ra)
Nhiệt độ không khí (cảm biến DHT22)	Vi điều khiển ESP32 so sánh với ngưỡng đã cài	Relay đóng, máy bơm chạy
Độ ẩm đất (cảm biến độ ẩm đất)	Ra quyết định: tưới hay không, tưới bao lâu	Relay ngắt, máy bơm dừng
Nút bấm tưới tay	Ghi nhật ký hoạt động	Đèn LED báo trạng thái
Giờ hệ thống	Kiểm tra an toàn (giới hạn thời gian bơm)	Thông báo về điện thoại qua Wi-Fi
🧩 SoC (System on Chip) là gì trong dự án này?

ESP32 là một SoC: một con chip duy nhất đã gồm sẵn CPU + RAM + bộ nhớ Flash + Wi-Fi. Nhờ vậy mạch gọn, rẻ, ít hỏng vặt, rất hợp với trang trại.

Thành phần trong SoC	Vai trò trong Smart-Farm
CPU	"Bộ não" đọc cảm biến và ra lệnh
RAM	Giữ dữ liệu tạm thời khi đang chạy (mất khi mất điện)
Flash	Lưu chương trình và cài đặt lâu dài
Wi-Fi	Gửi thông báo về điện thoại
🛠️ Hướng dẫn cài đặt
Bước 1. Chuẩn bị phần cứng
1 mạch ESP32
1 cảm biến nhiệt độ DHT22
1 cảm biến độ ẩm đất
1 module relay + 1 máy bơm nước nhỏ
Nguồn điện phù hợp cho mạch và bơm
Bước 2. Nạp chương trình (người kỹ thuật thực hiện)
bash
# 1. Tải mã nguồn về máy (thay đường dẫn bằng kho mã thực tế của dự án)
git clone https://github.com/<ten-tai-khoan>/smart-farm.git
cd smart-farm

# 2. Nạp chương trình vào ESP32 (dùng PlatformIO)
pio run --target upload

# 3. Mở màn hình theo dõi để kiểm tra
pio device monitor
Bước 3. Cài đặt tại vườn
Cắm cảm biến độ ẩm vào đất gần gốc cây, sâu khoảng 5–10 cm.
Đặt cảm biến nhiệt độ ở nơi thoáng, có bóng râm.
Nối ống nước với máy bơm, kiểm tra không rò nước gần mạch điện.
Cấp điện và chờ đèn LED xanh sáng là hệ thống sẵn sàng.
✅ Tính năng
 Đo nhiệt độ và độ ẩm đất liên tục
 Tự động bật bơm khi đất khô
 Tự động tắt bơm khi đất đủ ẩm
 Nút bấm tưới tay khi cần
 Đèn LED báo trạng thái (xanh: bình thường, đỏ: cần chú ý)
 Ghi nhật ký hoạt động
 Thông báo về điện thoại qua Wi-Fi (đang hoàn thiện)
 Hẹn giờ tưới theo mùa (dự kiến)
📊 Bảng thông số

ℹ️ Các giá trị dưới đây là mặc định đề xuất, có thể chỉnh theo loại cây và thời tiết thực tế.

Thông số	Giá trị mặc định	Ghi chú
Ngưỡng độ ẩm đất để tưới	< 40%	Dưới mức này, đất bị coi là khô
Ngưỡng độ ẩm đất để dừng	≥ 60%	Đủ nước, bơm tắt
Ngưỡng nhiệt độ cao	> 35 °C	Tăng mức ưu tiên tưới
Thời gian bơm tối đa mỗi lần	5 phút	Chống bơm chạy mãi khi lỗi
Chu kỳ đọc cảm biến	30 giây	Đo lại sau mỗi 30 giây
Điện áp cấp cho mạch	5 V	Dùng nguồn đúng chuẩn
🧠 Cách hệ thống "suy nghĩ"
1. Chu kỳ Fetch–Decode–Execute khi nhận lệnh tưới

Khi hệ thống nhận lệnh "TƯỚI", CPU của ESP32 xử lý lệnh qua 3 bước:

① Fetch (Nạp lệnh) CPU lấy lệnh "TƯỚI" từ bộ nhớ (RAM/Flash) về thanh ghi để chuẩn bị xử lý.

② Decode (Giải mã) CPU "đọc hiểu" lệnh: "Cần đóng relay chân số mấy, trong bao lâu?"

③ Execute (Thực thi) CPU gửi tín hiệu điện đến relay, máy bơm chạy. Xong việc, CPU quay lại bước ① để nhận lệnh tiếp theo.

2. Từ dữ liệu đến hành động: kim tự tháp DIKW
text
                 /\
                /  \         🌟 WISDOM (Trí tuệ)
               /    \        Bật bơm 5 phút, tưới vào lúc mát
              /------\
             /        \      🧠 KNOWLEDGE (Tri thức)
            /          \     Nóng + đất khô → cây mất nước nhanh
           /------------\
          /              \   📋 INFORMATION (Thông tin)
         /                \  Nhiệt độ cao hơn ngưỡng, đất đang khô
        /------------------\
       /                    \ 🔢 DATA (Dữ liệu)
      /                      \ 36,2 °C và độ ẩm đất 28%
     /________________________\
Tầng	Ví dụ trong Smart-Farm
🔢 Data (Dữ liệu)	Cảm biến đo được 36,2 °C và độ ẩm đất 28%
📋 Information (Thông tin)	36,2 °C cao hơn ngưỡng 35 °C; 28% thấp hơn ngưỡng 40%
🧠 Knowledge (Tri thức)	Trời nóng và đất khô thì cây mất nước rất nhanh
🌟 Wisdom (Trí tuệ / Hành động)	Bật bơm tối đa 5 phút, rồi tắt để tiết kiệm nước
⚠️ Lưu ý
An toàn khi sử dụng
🔌 Không rút điện đột ngột khi bơm đang chạy.
💧 Giữ mạch điện khô ráo, tránh nước bắn vào.
🛑 Máy bơm được cài tự tắt khi mất điện hoặc hệ thống khởi động lại.
💾 Khi bộ nhớ RAM bị đầy do dữ liệu nhật ký

Dấu hiệu nhận biết: đèn LED đỏ nhấp nháy, hệ thống phản hồi chậm hoặc tự khởi động lại.

Nguyên nhân: RAM của ESP32 rất nhỏ (cỡ vài trăm KB) và mất sạch khi mất điện. Nếu nhật ký ghi quá nhiều vào RAM thì RAM đầy, hệ thống có thể bị chậm hoặc treo.

Cách xử lý an toàn (dành cho người dùng):

Giữ bình tĩnh. Máy bơm vẫn có cơ chế tự tắt, cây không bị ngập.
Kiểm tra bơm đang tắt trước khi làm bất kỳ việc gì khác.
Bấm nút RESET một lần (không rút điện đột ngột).
Chờ khoảng 30 giây cho đèn LED chuyển xanh.
Nếu đèn đỏ vẫn nhấp nháy, liên hệ kỹ thuật viên, đừng tự tháo mạch.

Cách phòng tránh (dành cho kỹ thuật viên):

text
- Giới hạn bộ đệm nhật ký trong RAM (vòng tròn - ring buffer), ví dụ tối đa 100 dòng.
- Khi RAM trống dưới 20%: ghi nhật ký cũ sang Flash/thẻ nhớ rồi xóa khỏi RAM.
- Chỉ ghi sự kiện quan trọng (bật/tắt bơm, lỗi); không ghi mọi lần đọc cảm biến.
- Lệnh an toàn (tắt bơm) luôn được ưu tiên hơn việc ghi nhật ký.
- Định kỳ xóa nhật ký quá 30 ngày để tránh đầy bộ nhớ lâu dài.
📞 Hỗ trợ

Nếu gặp sự cố, vui lòng liên hệ kỹ thuật viên dự án Smart-Farm (thông tin liên hệ sẽ được bàn giao cùng thiết bị).
