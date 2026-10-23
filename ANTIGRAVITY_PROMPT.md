# Prompt chính cho Antigravity — xây ứng dụng “Giữ Nét”

Bạn là coding agent phụ trách kiến trúc, triển khai, kiểm thử và đóng gói ứng dụng desktop Windows 11. Hãy biến đặc tả này thành sản phẩm chạy được. Đây là yêu cầu thực hiện dự án, không phải yêu cầu viết thêm một bản kế hoạch rồi dừng lại.

## 1. Hiểu điều tôi thật sự cần

Tôi có những ảnh chất lượng thấp, thường là ảnh bị nén, nhỏ, răng cưa hoặc đã cắt thành các đối tượng riêng. Tôi muốn tách chúng để sử dụng lại và phóng to cho rõ hơn, nhưng vẫn nhận ra đó là đúng đối tượng gốc.

Ví dụ có thể là mũi tên cong, mũi tên đứt nét, ký hiệu, hình minh họa, thuyền hoặc đối tượng nhiều màu. **Hai hình tôi từng gửi chỉ minh họa nhu cầu.** Không giới hạn ứng dụng vào bản đồ, màu đỏ/xanh, một bộ hình cố định hay một kích thước ảnh. Không coi chúng là file đầu vào đã có sẵn.

Điều tôi không hài lòng ở AI tạo ảnh là kết quả trông đẹp hơn nhưng đã đổi dáng, thêm nét, nối những đoạn tách rời hoặc tự bịa chi tiết. Hãy tối ưu khả năng giữ hình và khả năng kiểm soát của người dùng. Đừng hứa khôi phục được chính xác dữ liệu vốn không còn trong ảnh.

Mục tiêu ưu tiên theo thứ tự: **giữ hình → người dùng kiểm soát được → xuất file dùng được → thao tác dễ → tốc độ → AI tùy chọn**.

## 2. Cách làm việc tôi mong đợi

- Kiểm tra repository, quy tắc agent, môi trường và phần mềm sẵn có trước khi viết mã.
- Đọc các tài liệu ở cuối prompt. Nếu chúng chưa có, triển khai theo prompt này và tạo tài liệu tương ứng; không giả vờ đã đọc file không tồn tại.
- Nêu kế hoạch ngắn, chọn stack phù hợp rồi bắt đầu triển khai. Không hỏi tôi xác nhận từng bước nhỏ.
- Làm theo từng mốc có chức năng thật. Tự chẩn đoán lỗi cài đặt, build và test; tiếp tục phần độc lập khi gặp chặn.
- Nếu cần model, license, quyền tải hoặc tài khoản chưa có, nêu chính xác thứ còn thiếu. Không bịa API, model, checksum, đường dẫn hoặc kết quả kiểm thử.
- Không yêu cầu tôi gửi secrets trong chat. Không thêm dịch vụ trả phí hoặc upload ảnh mặc định.
- Ghi bộ nhớ dự án trong `docs/DECISIONS.md` và `docs/STATUS.md` để phiên sau tiếp tục được.
- Khi báo cáo, ghi rõ điều đã thực hiện, bằng chứng, giới hạn và việc kế tiếp; không coi skeleton hay mockup là sản phẩm hoàn thành.

## 3. Nền tảng và phạm vi

- Ứng dụng desktop cho **Windows 11 x64**, ưu tiên máy chỉ có CPU; GPU là tăng tốc tùy chọn.
- Giao diện tiếng Việt rõ ràng, hỗ trợ tên file và đường dẫn tiếng Việt, khoảng trắng, màn hình HiDPI.
- Hướng mặc định: Python 3.12 + PySide6 + OpenCV + Pillow + NumPy, pytest và PyInstaller. Kiểm tra phiên bản tương thích rồi pin dependency/lockfile phù hợp. Chỉ đổi stack khi có lý do kỹ thuật cụ thể được ghi lại.
- Không cần tài khoản, server, trình duyệt hay Antigravity chạy nền để sử dụng các chức năng cơ bản.
- Antigravity là công cụ giúp xây ứng dụng này; không tự thiết kế việc điều khiển giao diện Antigravity bên trong app hoặc phụ thuộc vào một API Antigravity chưa được xác minh.
- Trước tiên giao core tách đối tượng + vector/raster + so sánh + export hoạt động thật. Sau đó bổ sung AI local tùy chọn theo đúng nguồn lực thực tế.
- Không mở rộng sang trình sửa ảnh tổng hợp, tạo ảnh từ prompt, OCR, phục chế khuôn mặt, video, plugin marketplace hoặc hệ thống tài khoản trong phiên bản đầu.

## 4. Luồng sử dụng bắt buộc

1. Kéo thả ảnh, mở nhiều file hoặc chọn thư mục. Nhận PNG, JPEG, BMP và TIFF một trang; định dạng/phân trang chưa hỗ trợ phải có thông báo.
2. Xem ảnh với zoom/pan và nền caro biểu thị trong suốt. Hiển thị kích thước pixel, không gán một con số “chất lượng” giả.
3. Với ảnh lớn: người dùng chọn crop, vùng chữ nhật/đa giác hoặc lấy màu để tạo mask. Với ảnh đã cắt: giữ alpha có sẵn làm điểm xuất phát.
4. Cho phép sửa mask bằng cọ thêm/xóa; xem overlay, undo/redo. Không bắt người dùng phụ thuộc hoàn toàn vào tách tự động.
5. Tạo một đối tượng từ nhiều vùng rời nhau để giữ nguyên mũi tên đứt nét. Đặt tên, đổi thứ tự, ẩn/hiện, chọn lại từng đối tượng.
6. Chọn **Vector giữ hình**, **Phóng ảnh thận trọng**, hoặc **AI thử nghiệm** nếu model đã sẵn sàng. Gợi ý chế độ được phép, nhưng người dùng có quyền đổi.
7. Xem trước kết quả cạnh ảnh gốc, kéo thanh so sánh và chồng ảnh với độ mờ. Zoom/pan đồng bộ, cùng hệ tọa độ.
8. Chấp nhận một kết quả cụ thể rồi xuất PNG RGBA hoặc SVG vector. Không xuất nhầm một preview cũ sau khi người dùng đã thay thông số.
9. Lưu project để mở lại mask, đối tượng và thông số; không ghi đè ảnh gốc.
10. Với nhiều ảnh: có hàng đợi, tiến độ, hủy, thông báo lỗi từng file và quy tắc đặt tên tránh ghi đè.

## 5. Chế độ Vector giữ hình

Dành cho ký hiệu và hình ít màu, đặc biệt khi đường biên quan trọng hơn texture.

- Tách mask theo alpha, màu chọn hoặc vùng chọn. Các khoảng đứt nét, lỗ rỗng và thành phần nhỏ phải được giữ mặc định.
- Không tự nối đoạn, lấp lỗ, bỏ đốm nhỏ, làm dày/mỏng hoặc thay bằng một mẫu mũi tên có sẵn.
- Ngưỡng màu, lọc thành phần, sửa topology và làm mượt là các điều khiển riêng, mặc định thận trọng; mọi thay đổi phải xem trước được.
- Trace contour gồm cả lỗ, duy trì đúng vị trí tương đối. Bản đầu có thể dùng polygon chính xác; chỉ fit Bézier khi đã có kiểm soát sai lệch.
- Tham số độ lệch phải dùng đơn vị **pixel ảnh nguồn**, không phụ thuộc mức zoom. Budget là sai lệch biên lớn nhất hai chiều, không phải trung bình hay p95. Tăng làm mượt không được lặng lẽ vượt ngưỡng đã đặt, đặc biệt ở đầu nhọn; nếu không giữ được thì giảm đơn giản hóa hoặc giữ contour gốc.
- Mũi tên nhọn không được biến thành đầu tròn; đường cong không bị thay bằng đường thẳng; thành phần nhỏ và khe hở không bị gộp ngầm.
- Cho phép chỉnh điểm/nút, undo/redo và xem đường gốc chồng phía dưới. Bắt đầu bằng di chuyển, thêm/xóa điểm polygon; thêm tay nắm Bézier nếu đã triển khai đường cong.
- SVG xuất ra phải chứa path/vector thực, `viewBox` đúng, nền trong suốt và màu có chủ đích. Không bọc một ảnh PNG trong thẻ SVG rồi gọi đó là vector hóa.
- Đơn sắc và nhiều màu ít lớp có phạm vi riêng: hoàn thành đơn sắc trước, sau đó cho người dùng chọn số màu và xem bản xem trước khi giảm bảng màu. Không hứa vector hóa đẹp mọi ảnh nhiều chi tiết.

## 6. Chế độ Phóng ảnh thận trọng

Dành cho ảnh nhiều màu/chi tiết hoặc khi vector hóa làm mất hình.

- Có phóng 2×, 4× và kích thước tùy chỉnh, mặc định giữ tỷ lệ. Dùng nội suy phù hợp như Lanczos và tăng nét nhẹ có thể chỉnh/tắt.
- Không dùng diffusion, inpainting, face restoration, tái tạo chữ hoặc thêm texture trong chế độ này.
- Alpha được xử lý đúng khi resize để tránh viền đen/trắng. Mask và màu phải cùng hệ tọa độ; không nhân alpha hai lần.
- Tách nền không được xóa tất cả pixel trắng một cách mù quáng, vì bên trong đối tượng có thể có vùng trắng hợp lệ.
- Khử nhiễu, khử quầng, làm sạch nền hoặc sửa màu là tùy chọn tường minh, có preview; không tự bật ở mức mạnh.
- “Không xử lý” phải là đường đi thực sự giữ dữ liệu pixel đã decode khi xuất PNG, không âm thầm thêm sharpen, crop hay đổi màu.
- Nêu rõ phóng to giúp hiển thị mượt hơn, không đảm bảo tìm lại chi tiết gốc đã mất.

## 7. AI là tùy chọn có kiểm soát

- App core phải mở, chạy và xuất được khi chưa cài model hoặc đang offline.
- Chỉ tích hợp một model super-resolution local có nguồn, license, cách tải và runtime được kiểm chứng. Model có tính sinh chi tiết phải được ghi rõ, dù không dùng diffusion.
- Cho người dùng bật AI riêng, xem model, hệ số hỗ trợ, CPU/GPU, kích thước/tài nguyên ước lượng và mức ảnh hưởng nếu có blend đã triển khai.
- Không cam kết một thanh “giữ nguyên 100%” có thể ngăn model bịa chi tiết. Không dùng prompt “giữ nguyên” như bằng chứng về độ chính xác.
- Không mặc định bật face enhancer hoặc generative restoration. Không yêu cầu prompt văn bản cho việc phóng ảnh cơ bản.
- Tải model theo thao tác rõ ràng, hiển thị dung lượng, lưu cache riêng, giữ xác minh TLS và kiểm tra toàn vẹn từ nguồn tin cậy khi có. Không tải trong lúc mở app một cách ngầm định.
- Giữ kết quả AI thành một biến thể riêng, không ghi đè source/mask. Cho phép loại bỏ hoàn toàn và quay lại pipeline thường.
- Nếu chưa đủ điều kiện triển khai AI thực, ẩn/vô hiệu hóa với lý do cụ thể và ghi trạng thái chưa hoàn thành; không trả ảnh nội suy rồi gắn nhãn AI.

## 8. Giao diện phải giúp tôi kiểm soát kết quả

- Bố cục: bên trái danh sách ảnh/đối tượng; giữa canvas lớn; bên phải thông số; hàng công cụ và thanh tiến độ dễ thấy.
- Dùng ngôn ngữ theo tác vụ: “Chọn đối tượng”, “Sửa vùng tách”, “Giữ hình”, “So sánh”, “Xuất”. Thông số kỹ thuật nâng cao có giải thích ngắn.
- Những thay đổi ảnh hưởng hình phải thấy được trước khi áp dụng. Không tự bắt đầu xử lý nặng sau từng lần rê slider; debounce và nút Áp dụng/Xem trước khi cần.
- Phím tắt: mở ảnh, lưu project, undo/redo, zoom vừa màn hình; tránh xung đột với ô nhập văn bản.
- Job nặng chạy ngoài GUI thread. Có hủy hợp tác, trạng thái rõ, lỗi dễ hiểu; đóng cửa sổ không để worker ghi file dở dang.
- Hiển thị minh bạch chế độ đang dùng, kích thước nguồn/đích và kết quả nào đã được chấp nhận. Các điểm số fidelity nằm ở phần nâng cao, không gọi đó là xác suất đúng ảnh gốc.

## 9. Dữ liệu, độ bền và riêng tư

- Source là bất biến. Mỗi thao tác thay đổi mask hoặc tham số tạo trạng thái mới có revision để undo/redo và tránh preview cũ ghi đè kết quả mới.
- Project có phiên bản schema, ảnh nguồn hoặc bản sao nguồn, mask, đối tượng, thông số và dữ liệu PNG/path SVG thực của các biến thể đã chấp nhận. Di chuyển project sang máy khác vẫn mở được mà không phụ thuộc một đường dẫn tuyệt đối bí mật. Thiếu model không được ngăn xem/xuất lại kết quả AI đã lưu; chỉ xử lý AI mới cần model.
- Lưu và export theo cách atomic khi filesystem hỗ trợ. Không để file xuất dang dở mang tên kết quả thành công; không ghi đè khi chưa có lựa chọn rõ ràng.
- Tên file tiếng Việt, trùng tên, đường dẫn có khoảng trắng, thư mục chỉ đọc, ảnh hỏng và hết dung lượng phải được xử lý.
- PNG hỗ trợ trong suốt; JPEG không có alpha. Nếu xuất JPEG sau này, phải cho chọn màu nền.
- Mặc định không telemetry, không upload, không gọi API ngoài. Log tránh chứa ảnh, nội dung riêng tư, secrets hoặc đường dẫn đầy đủ không cần thiết.
- Giới hạn kích thước ảnh và tổng pixel xuất theo tài nguyên; đưa thông báo hữu ích thay vì làm treo máy. Hủy job không phá source hoặc project đang mở.

## 10. Kiến trúc, kiểm thử và bàn giao

- Tách engine ảnh khỏi UI để test không cần mở desktop. Có model dữ liệu rõ cho source, object, mask, vector, biến thể và thông số.
- Tách vectorizer, raster resizer và AI backend thành module với đầu vào/đầu ra rõ ràng; không gọi widget từ engine.
- Tạo fixture tổng hợp bằng code cho mũi tên, nét đứt, lỗ rỗng, nhiều thành phần, alpha mềm và cạnh sắc. Không lấy ảnh ví dụ trong chat làm fixture giả định.
- Kiểm thử bằng so sánh dữ liệu ảnh, topology, export round-trip và hành vi GUI phù hợp. Chỉ kiểm tra có file/PID không chứng minh luồng xử lý đúng.
- Không dùng một điểm SSIM để kết luận giữ nguyên hình, và không lấy ảnh low-res làm bằng chứng đã tìm lại high-res thật. Đọc chi tiết phép đo trong `docs/ACCEPTANCE.md`.
- Có smoke test Windows 11 hoặc Windows runner phù hợp; phân biệt CI trên Windows Server với thử thủ công trên Windows 11.
- Build bộ portable Windows x64 bằng công cụ chạy trên Windows. Không tuyên bố PyInstaller trên Linux đã tạo EXE Windows hợp lệ.
- Hướng dẫn build, chạy dev, test, cài model tùy chọn, license và giới hạn phải khớp mã thực tế. Kiểm tra license của Qt/PySide6, dependency/model trước khi phân phối; không giả định model code và weights cùng license.
- Không hardcode API key hoặc tải model lớn vào Git. Release artifact/model cache tách khỏi source control.

## 11. Trình tự và điều kiện hoàn thành

Thực hiện theo các mốc trong `docs/ROADMAP.md`: nền tảng → luồng raster đầu cuối → vector chỉnh được → độ bền/batch/đóng gói → AI tùy chọn.

Tại mỗi mốc, trả lời ngắn: đã có chức năng gì chạy được; đã chạy lệnh/ca kiểm thử nào và kết quả; chưa kiểm chứng điều gì; mốc tiếp theo là gì. Nếu task đủ thời gian và môi trường, tiếp tục thực hiện chứ không dừng sau báo cáo.

Phiên bản core chỉ hoàn thành khi tôi có thể: mở ảnh thật → tách/sửa mask → giữ nhiều đoạn thành một đối tượng → chọn vector/raster → so sánh → xuất file đúng → lưu/mở lại project → dùng bản Windows mà không phải cài Python. Không dùng trạng thái AI tùy chọn để che phần core còn thiếu, hoặc chặn việc giao core đã hoàn thiện.

**Tài liệu đi kèm phải đọc:** `AGENTS.md`, `docs/ARCHITECTURE.md`, `docs/ACCEPTANCE.md`, `docs/ROADMAP.md`, `docs/DECISIONS.md`, `docs/STATUS.md`.
