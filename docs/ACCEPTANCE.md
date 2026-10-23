# Tiêu chí nghiệm thu và bằng chứng

Đây là các ca **cần triển khai và chạy**, chưa phải kết quả kiểm thử hiện có. Các ngưỡng đo dưới đây là mục tiêu cho fixture kiểm soát được, không phải lời bảo đảm cho mọi ảnh thật.

## A. Quy ước phép đo

- Dữ liệu tổng hợp phải độc lập với implementation được kiểm thử. Không tạo expected bằng chính hàm đang test.
- Có fixture sạch và fixture giảm chất lượng: mũi tên thẳng/cong, đầu nhọn, nét đứt, vòng có lỗ, ký hiệu nhiều phần, màu bất kỳ, alpha mềm và vùng trắng bên trong.
- Reference của fidelity vector là **mask đầu vào đã được xác nhận**, không phải một ảnh kết quả tự chọn cho dễ đạt.
- Khi đo: rasterize SVG về cùng kích thước, bounds và tọa độ nguồn; threshold alpha >= 128/255; không tự co giãn/căn chỉnh tối ưu để giấu sai vị trí.
- Topology dùng foreground 8-connectivity và background 4-connectivity, padding bên ngoài khi đếm lỗ. Ghi rõ quy ước trong code/test.
- IoU = intersection/union foreground; ảnh foreground rỗng không được coi là ca giữ hình thành công. Đo thêm khoảng cách biên hai chiều: gộp khoảng cách từ mỗi biên sang biên kia, lấy phân vị 95, đơn vị pixel nguồn.
- Budget đơn giản hóa là **sai lệch biên lớn nhất hai chiều**, kiểm tra bằng Hausdorff hoặc xấp xỉ bảo thủ có cách lấy mẫu/sai số được công bố. Không dùng p95 làm phép kiểm tra budget vì có thể che đầu nhọn bị lệch cục bộ. Mặc định đặt budget không quá 1 pixel nguồn; khi chưa chứng minh được thì giữ contour chính xác hơn. Kiểm thử đo đạc phải có một sai lệch cục bộ cố ý để chứng minh bộ kiểm tra phát hiện được nó.
- Mục tiêu fixture sạch đủ lớn (ít nhất 1.000 pixel foreground, khe hở/chi tiết quan trọng rộng ít nhất 3 pixel): IoU >= 0,98 và sai lệch biên p95 <= 1 pixel nguồn với preset giữ hình. Cần báo giá trị thực và các ca không đạt.
- Với fixture rất nhỏ hoặc chi tiết 1 pixel, các tỷ lệ có thể không ổn định: kiểm tra topology và biên riêng, công bố kết quả. Không tự xóa các fixture khó hoặc dùng chúng làm lý do bỏ kiểm tra cấu trúc.
- Đạt điểm với mask không chứng minh mask đã tách đúng đối tượng trong ảnh gốc. So sánh bằng mắt và sửa mask vẫn cần thiết.
- SSIM/PSNR nếu dùng chỉ là thông tin bổ sung với reference hợp lệ, không gọi chúng là “độ chính xác phục hồi” hay “giống 98%”.

## B. Core bắt buộc

| ID | Tình huống | Kết quả phải kiểm chứng |
| --- | --- | --- |
| C01 | Mở PNG RGBA, JPEG có EXIF orientation, BMP và TIFF một trang | Nội dung/kích thước/orientation đúng; ảnh hỏng/không hỗ trợ có lỗi hữu ích |
| C02 | File/thư mục có dấu tiếng Việt và khoảng trắng | Mở, lưu project, export thành công trên Windows |
| C03 | Ảnh đã có alpha | Alpha được dùng làm mask ban đầu; mép mềm không bị mất ngầm |
| C04 | Tách nền gần trắng, đối tượng có chi tiết trắng bên trong | Có cách giữ lại chi tiết trắng bằng ROI/mask; không xóa trắng toàn cục ngầm |
| C05 | Một mũi tên đứt thành nhiều phần | Người dùng nhóm được thành một object, giữ vị trí tương đối và khoảng trống |
| C06 | Cọ thêm/xóa, sửa contour và đổi tham số | Undo/redo khôi phục đúng dữ liệu, không chỉ đổi giao diện |
| C07 | Crop có offset rồi zoom/pan và export | Mask/vector/raster không lệch tọa độ; viewBox và kích thước đúng |
| C08 | Thay tham số khi job preview cũ chưa xong | Kết quả revision cũ không ghi đè trạng thái mới; export theo biến thể đã chấp nhận |
| C09 | No-op 1× PNG, không crop, không đổi profile | RGBA sau decode của file xuất bằng RGBA chuẩn hóa đầu vào; source bytes/hash không đổi |
| C10 | Resize 2×/4× ảnh có alpha mềm và màu sáng/tối | Đúng kích thước/tỷ lệ; không viền đen/trắng ngoài chủ đích, kiểm tra composite trên nền trắng/đen/caro |
| C11 | Vector hóa mũi tên, vòng rỗng, nét đứt | Giữ component/lỗ, đạt phép đo A trên fixture thuộc phạm vi; path có thể sửa được |
| C12 | Hình ít màu sau bước đơn sắc | Người dùng kiểm soát số màu; lớp/vùng không trôi vị trí, preview cho thấy thay đổi bảng màu |
| C13 | PNG và SVG export | PNG giữ alpha; SVG hợp lệ chứa path thật, không có ảnh nhúng thay thế, không external resource/script |
| C14 | Lưu rồi mở lại project sau khi chuyển thư mục | Nguồn/mask/đối tượng/thông số và dữ liệu biến thể đã chấp nhận được giữ; không cần đường dẫn tuyệt đối cũ; xem/xuất lại kết quả đã lưu kể cả thiếu model |
| C15 | Mở nhiều ảnh, trùng basename | Batch hoạt động, tên xuất không đè nhau, lỗi một file không mất kết quả file khác |
| C16 | Hủy job, lỗi ghi file, thư mục chỉ đọc | UI còn phản hồi, source nguyên vẹn, không báo file dở dang là thành công |
| C17 | Ảnh vượt giới hạn hoặc kích thước xuất quá lớn | Báo giới hạn/tài nguyên và cách giảm; không treo GUI hay cố cấp RAM vô hạn |
| C18 | Không mạng, không model, CPU-only | Core mở ảnh, sửa mask, vector/raster, so sánh, project và export vẫn hoạt động |
| C19 | Đóng/mở app, bàn phím và HiDPI | Trạng thái lưu rõ ràng, không mất thay đổi đã lưu; controls dùng được, phím tắt không phá nhập text |

Đối chiếu export bằng phần mềm độc lập khi khả dụng: xem PNG trong trình xem ảnh Windows và mở/chỉnh path SVG bằng Inkscape hoặc trình chỉnh SVG tương đương. Automated rasterization là bổ sung, không gọi đó là đã thử Inkscape nếu chưa chạy.

## C. Đo chất lượng vector cụ thể

1. Fixture cong có đầu nhọn: kiểm tra đầu mũi tên còn nguyên, không biến thành mũi tên mẫu đối xứng; báo IoU/p95.
2. Fixture gồm 5 phần rời: giữ đúng 5 phần với default, không tự đóng khoảng trống. Topology so với mask xác nhận phải chính xác.
3. Vòng có một lỗ và vùng rời nhỏ: giữ lỗ và vùng nhỏ; export rồi rasterize không lấp lỗ.
4. Tăng slider làm mượt: kết quả không vượt budget âm thầm. Đo sai lệch lớn nhất ở đầu nhọn/góc ngoài p95; fixture có một đầu bị kéo lệch phải bị phát hiện dù IoU/p95 còn tốt. Khi không thể đạt, dùng contour chính xác hơn hoặc báo rõ, không xuất nhãn “giữ hình” sai.
5. Crop ở tọa độ khác gốc: vector overlay khớp vị trí; không chỉ đạt hình dáng sau khi tự căn chỉnh.
6. Fixture nhân đôi kích thước: cùng tolerance pixel nguồn phải có nghĩa nhất quán; zoom UI không làm đổi kết quả xử lý.

## D. Raster và AI tùy chọn

- Resize alpha: có test số trên ảnh RGBA tổng hợp ngoài kiểm tra thị giác. RGB dưới alpha=0 không đại diện màu hiển thị; đo composite và vùng alpha có nghĩa.
- Với ảnh high-res tổng hợp hoặc được cấp quyền: tạo low-res theo phép giảm mẫu có tài liệu, xử lý, rồi so với high-res giữ lại. Không suy rộng kết quả một benchmark thành phục chế đúng mọi ảnh thật.
- Kiểm tra preview đã chấp nhận và export đúng tham số/kích thước; bản preview downsample không phải ground truth của file xuất.
- AI adapter unit test bằng fake backend chỉ chứng minh wiring/cancellation. Phải có một integration run với model thực để tuyên bố AI đã hoạt động.
- Ghi model/version/license/runtime/device, nguồn weights, input/output dimensions và thời gian thực tế. Không nêu FPS hoặc yêu cầu GPU chưa đo.
- Không model hoặc model lỗi: không tải ngầm, không chặn core, không đổi sang nội suy rồi ghi là AI.
- Nếu model chỉ hỗ trợ một hệ số, UI không được hứa hệ số tùy ý không có pipeline rõ ràng. Tiling phải kiểm tra seam.
- Kết quả AI luôn là biến thể riêng, có cảnh báo khả năng suy đoán chi tiết và so sánh; không tự thay nguồn/biến thể đã chấp nhận.

## E. Windows, hiệu năng và đóng gói

- Build core Windows x64 trên Windows. Ghi OS/build, Python/Qt/runtime và command đã dùng; CI Windows Server khác nghiệm thu Windows 11.
- Thử portable trên Windows 11 không cài Python, CPU-only, offline. Có thể cần người dùng chạy nếu môi trường agent không có Windows 11; ghi rõ đây là phần chưa kiểm chứng.
- Smoke flow trên bản build: mở ảnh có tên tiếng Việt → chọn/sửa mask → raster/vector → so sánh → xuất → lưu/mở lại project.
- Chạy thao tác lớn trong worker, kiểm tra người dùng vẫn pan/zoom/hủy được. Đo thời gian và peak RAM trên input/máy ghi rõ, không đặt SLA tùy tiện trước khi benchmark.
- Gói phân phối kèm license notices và không chứa source ảnh cá nhân/secrets/cache/model không được phép phân phối.
- Không tuyên bố code signing hoặc installer đã hoàn thiện khi chỉ có unsigned portable build. Không đề nghị tắt hệ thống bảo vệ Windows để che lỗi đóng gói.

## F. Cách ghi kết quả

Mỗi mốc cập nhật `STATUS.md` với commit/mốc, lệnh hoặc thao tác, OS, dữ liệu đầu vào, kết quả pass/fail/skip/unrun và giới hạn. Kiểm thử 0 ca hoặc skip vì thiếu model không phải bằng chứng tính năng đó đã chạy. Phân biệt defect ứng dụng với thiếu môi trường hoặc thiếu quyền.

Core hoàn thành khi các ca B và nghiệm thu đóng gói tương ứng đã được kiểm chứng, hoặc phần Windows 11 còn chặn được nêu chính xác để nghiệm thu tiếp. Trường hợp còn chặn không được gọi là đã hoàn thành toàn bộ sản phẩm. AI có cổng nghiệm thu riêng, không làm thay đổi nghĩa “core hoạt động offline”.
