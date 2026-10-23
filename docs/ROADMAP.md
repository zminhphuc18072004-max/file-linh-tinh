# Lộ trình thực hiện

Không cần chờ người dùng chọn từng mốc. Tiếp tục mốc kế tiếp khi phần phụ thuộc đã được kiểm chứng; ghi chặn thực tế thay vì bỏ qua. Tài liệu hiện tại không phải bằng chứng mốc nào đã hoàn thành.

## M0 — Nền tảng có thể chạy và test

- Kiểm tra checkout, quyền, OS, tài nguyên, dependency và mục tiêu Windows x64.
- Tạo cấu trúc nhỏ gọn, dependency pin thực tế, entry point, logging sạch, fixture tổng hợp và test harness.
- Thiết lập domain/source bất biến và canvas mở ảnh/zoom/pan.
- **Cổng:** mở được ảnh tổng hợp với path Unicode; engine test không cần GUI/model; lệnh setup/run/test trong README đã chạy trong môi trường hiện có.

## M1 — Luồng raster đầu cuối

- ROI, alpha/chọn màu, cọ sửa mask, nhóm nhiều phần, undo/redo cơ bản.
- Resize thận trọng, alpha đúng, preview so sánh, chấp nhận biến thể và xuất PNG.
- Job ngoài UI thread, revision tránh kết quả cũ, hủy cơ bản.
- **Cổng:** mở → tách/sửa → resize → so sánh → xuất hoạt động thật; C01–C10 và phần C16/C18 liên quan có kiểm thử phù hợp. Source không bị ghi đè.

## M2 — Vector giữ hình

- Contour/hierarchy, polygon baseline, tolerance và đo fidelity; SVG path thật.
- Chỉnh điểm cơ bản và undo; giữ nét đứt/lỗ; export khớp tọa độ.
- Sau đơn sắc, thêm ít màu với palette preview. Bézier nâng cao chỉ thêm khi kiểm soát được sai lệch.
- **Cổng:** C11–C13 và các ca C trong ACCEPTANCE có kết quả thực; sửa được SVG trong công cụ độc lập hoặc ghi đúng phần nghiệm thu còn thiếu.

## M3 — Dùng hằng ngày và giao core Windows

- Project portable/schema, batch, tên tránh đè, lỗi file/hết tài nguyên, HiDPI/phím tắt và polish tác vụ.
- Hoàn tất kiểm thử core, build portable bằng Windows runner, artifact và license notices; README khớp implementation.
- Thử trên Windows 11 không Python khi có khả năng truy cập; nếu thiếu, giữ trạng thái nghiệm thu Windows 11 đang chờ với hướng dẫn cụ thể.
- **Cổng:** toàn bộ luồng core từ import đến project/export hoạt động, các ca B/E có bằng chứng phân loại rõ. Không coi build Linux là artifact Windows.

## M4 — AI local tùy chọn

- Chọn một model/runtime sau khi kiểm tra license, nguồn tải, CPU/GPU và tài nguyên.
- Tải có chủ đích, cache, integrity phù hợp, adapter thật, preview/biến thể riêng, fallback core minh bạch.
- Integration run model thực và ghi benchmark; không bật generative restoration mặc định.
- **Cổng:** phần D dành cho AI có bằng chứng. Nếu model/license/tài nguyên chưa có, ghi chặn và giữ core dùng được; không gắn nhãn AI hoàn thành.

## Ngoài phạm vi ban đầu

Trình biên tập vector đầy đủ, diffusion vẽ lại ảnh, tự sửa mặt/chữ, server cloud, điều khiển Antigravity khi chạy app, tài khoản, OCR/video và marketplace. Không thêm các phần này để trì hoãn luồng chính.
