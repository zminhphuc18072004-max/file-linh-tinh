# Bộ nhớ quyết định

Các quyết định dưới đây xuất phát từ nhu cầu người dùng và đặc tả khởi điểm. Chưa có benchmark hay implementation xác nhận một thư viện/model cụ thể.

| ID | Quyết định | Lý do và hệ quả |
| --- | --- | --- |
| D001 | Hai hình trong chat chỉ là ví dụ | App phải xử lý đầu vào người dùng chọn; không có fixture ảnh thật được cung cấp trong repo |
| D002 | Giữ hình là ưu tiên cao nhất | Mọi bước nối nét, làm mượt, bỏ component hoặc thêm chi tiết phải có kiểm soát và đối chiếu |
| D003 | Tách luồng vector, raster và AI | Ký hiệu đơn giản khác ảnh nhiều chi tiết; không có một thuật toán giữ đúng mọi loại ảnh |
| D004 | Core cục bộ, CPU-only và offline | Không phụ thuộc tài khoản, GPU hoặc tải model để dùng chức năng chính |
| D005 | Python 3.12 + PySide6 là stack khởi điểm | Phù hợp engine ảnh và desktop; cần kiểm tra dependency, license, đóng gói Windows thực tế |
| D006 | Antigravity là tác nhân phát triển | Không giả định có API Antigravity phục chế ảnh hoặc cần chạy Antigravity khi dùng app |
| D007 | Giữ source và nhóm nhiều vùng thành một object | Phục vụ nét đứt, phục hồi thao tác và đối chiếu; không phá dữ liệu nguồn |
| D008 | SVG phải có path thật | Một raster nhúng trong SVG không giải quyết nhu cầu phóng hình vector |
| D009 | Bộ tài liệu là “não bộ”, chưa phải app | Coding agent phải tạo mã, kiểm thử, artifact và cập nhật tiến độ bằng bằng chứng |
| D010 | AI optional sau core, chưa chốt model | Tránh tải nặng/giấy phép không rõ; AI vẫn có thể suy đoán chi tiết ngay cả model upscale |

Khi thêm quyết định: ghi ID, bối cảnh, lựa chọn, lý do, hệ quả và bằng chứng nếu có. Nếu thay thế một quyết định, giữ lịch sử và dẫn ID mới; không xóa lý do cũ để giả vờ chưa có thay đổi.
