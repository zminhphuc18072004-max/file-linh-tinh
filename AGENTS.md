# Hướng dẫn cho coding agent

## Đọc trước khi làm

1. Đọc `ANTIGRAVITY_PROMPT.md` để nắm nhu cầu gốc.
2. Đọc `docs/STATUS.md`, `docs/DECISIONS.md`, `docs/ROADMAP.md` để tiếp tục đúng tiến độ.
3. Đọc `docs/ARCHITECTURE.md` và `docs/ACCEPTANCE.md` trước khi thiết kế hoặc đổi pipeline.
4. Kiểm tra file, Git status, manifest và môi trường thực tế; không suy diễn rằng mã nguồn hoặc model đã tồn tại.

## Ưu tiên sản phẩm

- Giữ hình dáng, số thành phần, lỗ rỗng, màu và bố cục gốc là yêu cầu trung tâm.
- Mặc định xử lý cục bộ, bảo toàn bản gốc; không upload ảnh hoặc gọi model bên ngoài ngầm.
- Hai ảnh trong hội thoại chỉ là ví dụ; không giả định có file ảnh đó trong repo.
- Không thay một ca thất bại bằng kiểm thử dễ hơn chỉ để báo xanh.
- Không dùng mock, ảnh hardcode hoặc nút không hoạt động để tuyên bố tính năng đã xong.

## Cách thực hiện

- Mỗi mốc phải có luồng dùng được và bằng chứng kiểm thử trước khi chuyển tiếp.
- Tự xử lý lựa chọn kỹ thuật thông thường. Chỉ hỏi khi thiếu thông tin thật sự cản trở, cần quyền mới hoặc phát sinh chi phí.
- Trình bày quyết định và lý do ngắn gọn; không cần xuất suy luận nội bộ.
- Dùng checkout hiện có. Trong tác vụ cloud đã có môi trường cô lập, không tự tạo Git worktree trừ khi người dùng yêu cầu.
- Bảo toàn thay đổi có sẵn; không force-push, reset hoặc xóa dữ liệu ngoài phạm vi công việc.
- Cập nhật `docs/STATUS.md` sau từng mốc và trước khi kết thúc phiên. Ghi đúng lệnh, hệ điều hành, kết quả và phần chưa chạy.
- Ghi thay đổi quyết định quan trọng vào `docs/DECISIONS.md`. Không tự hạ tiêu chí nghiệm thu; nêu lý do và đánh dấu việc chưa đáp ứng.
- Giữ các tài liệu nhất quán khi mã nguồn thực tế thay đổi. Không sửa nhu cầu gốc chỉ để khớp một implementation yếu hơn.

## Điều kiện bàn giao

- README phải dẫn đến lệnh chạy/build đã kiểm chứng và giới hạn hiện tại.
- Có kiểm thử cho logic ảnh, project/export và luồng GUI quan trọng; kiểm thử Windows được phân biệt với Linux.
- Không commit ảnh cá nhân, secrets, venv, model lớn, cache, build output hay log chứa đường dẫn nhạy cảm.
- Phân biệt rõ: đã viết mã, đã chạy kiểm thử, đã build artifact, đã thử trên Windows, và tính năng AI tùy chọn.
