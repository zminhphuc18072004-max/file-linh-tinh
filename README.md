# Giữ Nét — đặc tả ứng dụng Windows 11

Bộ prompt và tài liệu giao việc cho **Antigravity** xây dựng ứng dụng tách đối tượng, làm sạch biên và phóng ảnh, ưu tiên giữ sát hình gốc.

**Trạng thái:** mới có đặc tả, chưa có ứng dụng, model AI hay bộ cài. Hai hình đã trao đổi trong chat chỉ minh họa nhu cầu; không phải dữ liệu kiểm thử và không có trong repository. File Word cũ đã được xóa theo yêu cầu.

## Bắt đầu với Antigravity

Mở repository này trong Antigravity và gửi nguyên đoạn sau:

```text
Hãy đọc AGENTS.md, ANTIGRAVITY_PROMPT.md và toàn bộ tài liệu được dẫn trong đó.
Thực hiện dự án Giữ Nét theo đặc tả, bắt đầu bằng việc kiểm tra repository và môi trường thực tế.
Tôi muốn bạn triển khai ứng dụng Windows 11 hoạt động thật, không dừng ở kế hoạch hoặc mockup.
Ưu tiên giữ hình gốc, xử lý cục bộ, dễ sử dụng; AI chỉ là tùy chọn để tôi chủ động thử.
Tự quyết định các chi tiết kỹ thuật thông thường và ghi lại quyết định quan trọng.
Triển khai theo từng mốc, chạy kiểm thử phù hợp và cập nhật docs/STATUS.md bằng bằng chứng thực tế.
Không tuyên bố đã chạy trên Windows hoặc đã có bộ cài nếu bạn chưa kiểm chứng.
```

Nếu chỉ muốn sao chép một tài liệu sang phiên làm việc khác, dùng toàn bộ [ANTIGRAVITY_PROMPT.md](ANTIGRAVITY_PROMPT.md); các tài liệu còn lại bổ sung tiêu chí kỹ thuật và nghiệm thu.

## Bản đồ “não bộ” dự án

| Tài liệu | Vai trò |
| --- | --- |
| [ANTIGRAVITY_PROMPT.md](ANTIGRAVITY_PROMPT.md) | Prompt chính: nhu cầu, phạm vi, cách làm việc và kết quả phải giao |
| [AGENTS.md](AGENTS.md) | Quy tắc làm việc cho coding agent trong repository |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Kiến trúc, dữ liệu và nguyên tắc thuật toán |
| [docs/ACCEPTANCE.md](docs/ACCEPTANCE.md) | Ca kiểm thử, ngưỡng nghiệm thu và giới hạn của phép đo |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Thứ tự triển khai và điều kiện kết thúc từng mốc |
| [docs/DECISIONS.md](docs/DECISIONS.md) | Bộ nhớ quyết định, lý do và hệ quả |
| [docs/STATUS.md](docs/STATUS.md) | Tiến độ thực tế, bằng chứng và việc tiếp theo |

## Kết quả mong muốn

Người dùng mở ảnh hoặc cả thư mục → chọn một hay nhiều đối tượng → chỉnh vùng tách → chọn xử lý vector hoặc raster → so sánh với gốc → xuất PNG trong suốt hoặc SVG thực sự có đường vector.

Mũi tên, biểu tượng ít màu phù hợp với vector hóa có kiểm soát. Ảnh nhiều chi tiết cần xử lý raster thận trọng. Mọi kết quả AI phải được ghi rõ và cho phép đối chiếu; không cam kết tái tạo chính xác chi tiết vốn đã mất.

Đây là tài liệu bàn giao phát triển, không chứa ảnh cá nhân, khóa API hoặc hướng dẫn cài app đã được kiểm chứng. Hướng dẫn chạy/build sẽ được coding agent bổ sung khi có mã nguồn thực tế.
