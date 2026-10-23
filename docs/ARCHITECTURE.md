# Kiến trúc triển khai

Đây là thiết kế khởi điểm cần hiện thực và kiểm chứng, không phải mô tả mã nguồn đã tồn tại. Thay đổi thiết kế được phép nếu giữ yêu cầu sản phẩm và ghi lý do vào `DECISIONS.md`.

## 1. Ranh giới module

| Khối | Trách nhiệm | Không được phụ thuộc |
| --- | --- | --- |
| `domain` | Source, mask, đối tượng, biến thể, transforms, project version | Widget, model AI cụ thể |
| `engine/io` | Decode, EXIF orientation, profile màu, PNG/SVG/project export | GUI |
| `engine/masking` | ROI, chọn màu, alpha, cọ, thành phần liên thông | GUI |
| `engine/vector` | Contour hierarchy, đơn giản hóa có giới hạn, path, rasterize để đo | Widget |
| `engine/raster` | Resize alpha đúng, sharpen, bộ lọc tường minh | Model sinh ảnh |
| `engine/metrics` | Topology, IoU, sai lệch biên, định nghĩa phép đo | Phán đoán “ảnh thật” |
| `backends` | Giao diện và adapter AI tùy chọn | UI cụ thể |
| `services` | Jobs, cancellation, cache, project persistence, batch | Widget |
| `ui` | Canvas, controls, object list, so sánh, thông báo | Chi tiết nội bộ model |

Cấu trúc `src/giu_net/`, `tests/`, `scripts/`, `docs/` là gợi ý. Không tạo tầng trừu tượng vô ích chỉ để đủ thư mục. Engine phải import/test được mà không tạo QApplication hoặc tải model.

## 2. Dữ liệu và hệ tọa độ

- `ImageSource`: ID, bản sao byte nguồn hoặc tham chiếu được quản lý, hash, kích thước sau EXIF orientation, metadata decode, trạng thái profile màu.
- `ObjectAsset`: ID, tên, source ID, ROI, transform local→source, mask gốc đã xác nhận, mask hiện tại và danh sách biến thể.
- `VectorAsset`: các path đóng/mở thích hợp, nhóm màu, hierarchy/lỗ, bounds và transform. Nét đứt có thể là nhiều path trong cùng object.
- `Variant`: mode, tham số, phiên bản thuật toán/model, source/mask revision, scale, dữ liệu kết quả thực (PNG/path SVG) và thông tin đo được. Biến thể đã chấp nhận phải được lưu cùng project, không chỉ lưu một recipe phụ thuộc model.
- `Project`: schema version, assets, object order/visibility, settings, lịch sử cần thiết hoặc snapshot trạng thái hiện tại.

Quy ước rõ pixel center, giới hạn crop, alpha và đơn vị. Tất cả tọa độ hình học gốc dùng pixel của ảnh đã chuẩn hóa orientation. Canvas zoom chỉ đổi phép hiển thị. Không lưu kết quả theo tọa độ màn hình.

Mỗi job nhận snapshot bất biến gồm source/mask/parameter revision. Chỉ áp dụng kết quả nếu revision còn hợp lệ. Preview, chấp nhận kết quả và export dùng cùng implementation; preview giảm độ phân giải phải được ghi rõ và export tính lại từ nguồn.

## 3. Luồng xử lý

```text
Decode + orientation + chính sách màu
  -> chọn ROI / alpha / mask màu / cọ
  -> người dùng kiểm tra mask và nhóm các phần rời
  -> snapshot nguồn + mask + tham số
  -> vector | raster | AI tùy chọn
  -> đối chiếu cùng hệ tọa độ
  -> chấp nhận biến thể
  -> export atomic + metadata/project
```

Đọc file Unicode an toàn trên Windows; nếu API OpenCV dùng đường dẫn không phù hợp, đọc bytes bằng pathlib rồi decode. Ảnh không được EXIF-rotate hai lần. Dùng một chính sách sRGB có tài liệu; chế độ no-op phải tránh chuyển màu không cần thiết. Công bố ảnh/profiles chưa hỗ trợ, không bỏ qua âm thầm.

## 4. Tách đối tượng

- Alpha có sẵn là một nguồn mask, không mặc định threshold làm mất mọi mép bán trong suốt.
- Chọn màu bằng khoảng cách màu có định nghĩa, lấy mẫu và tolerance; không khóa vào đỏ/xanh. Nếu dùng HSV, xử lý hue wrap-around.
- Tách nền từ vùng biên hoặc điểm mẫu có thể giữ chi tiết trắng bên trong tốt hơn xóa toàn bộ màu trắng. Cho phép xem/sửa kết quả ở mọi trường hợp.
- Connected components chỉ gợi ý vùng. Người dùng có thể giữ nhiều component thành một object hoặc tách lại.
- Mặc định không morphological closing/opening, lấp lỗ hoặc loại component nhỏ. Tác vụ sửa cấu trúc phải tường minh và có undo.
- Cọ thêm/xóa làm việc trên mask, không phá pixel source. Mask mềm và mask nhị phân phục vụ vector có chính sách chuyển đổi riêng, có preview.

## 5. Vector hóa

1. Lấy snapshot mask đã xác nhận; tách theo màu chỉ khi người dùng chọn chế độ ít màu.
2. Trích contour với hierarchy đầy đủ, dùng quy ước biên pixel rõ ràng và padding cần thiết.
3. Tạo path bao ngoài và lỗ với `fill-rule` phù hợp; không đảo lỗ thành vùng đặc.
4. Đơn giản hóa theo độ lệch pixel nguồn, kiểm tra topology trước/sau. Budget là khoảng cách biên lớn nhất hai chiều (Hausdorff), không dùng p95 để thay thế. Định nghĩa phép đo/lấy mẫu và sai số số học; lấy mẫu thích ứng đủ dày ở đầu nhọn, góc và đoạn cong. Nếu vượt budget hoặc không đủ bằng chứng giữ budget, giảm đơn giản hóa hoặc giữ contour gốc và thông báo.
5. Polygon chính xác là baseline; fit đường cong là cải tiến riêng có đo sai lệch. Kiểm tra cạnh giữa các điểm fit, không chỉ khoảng cách ở control point.
6. Chỉnh nút ở tọa độ nguồn, hỗ trợ undo. Không đẩy phạm vi thành một trình vẽ vector đầy đủ.
7. Export viewBox, bounds/margin theo lựa chọn, group và path thật. Escape dữ liệu tên đối tượng đúng XML; không tạo script, external resource hoặc nhúng PNG thay cho path.

Mức màu mặc định phải tôn trọng đầu vào. Khi quantize, cho xem bảng màu thay đổi và lưu thiết lập. Ảnh nhiều texture nên được gợi ý dùng raster, không tự đổi chế độ và bỏ chi tiết.

## 6. Raster và alpha

- Khi resize RGBA, xử lý màu ở dạng premultiplied alpha trong phép nội suy rồi unpremultiply an toàn; chọn và ghi rõ không gian màu dùng cho phép tính.
- Bảo vệ alpha bằng 0, không tạo NaN, không nhân mask/alpha hai lần. Pixel RGB dưới alpha bằng 0 cần chính sách nhất quán, không dùng để kết luận màu hiển thị sai.
- Sharpen và denoise có thể làm đổi cạnh/màu; tắt được, không bật mạnh mặc định. Tránh halo, clipping và viền tối quanh đối tượng.
- No-op PNG phải giữ RGBA đã decode khi không có crop/filter/color conversion. File bytes/metadata không bắt buộc giống nguyên bản; nội dung pixel phải đáp ứng ca tương ứng.
- Ghi rõ kích thước xuất, scale, margins và transform. “Khít đối tượng” là tùy chọn; mặc định giữ bố cục đã chọn.

## 7. Jobs, lưu trữ và giới hạn

- Qt signals chuyển kết quả từ worker; UI thread chỉ cập nhật widget. Hủy dùng token/checkpoint; backend không hỗ trợ hủy ngay phải có trạng thái trung thực.
- Ước lượng RAM gồm nguồn, mask, float buffers và đích. Giới hạn theo tổng pixel đích: 4× chiều rộng và cao là 16× số pixel. Không sao chép toàn bộ ảnh cho mỗi bước undo nhỏ nếu có thể dùng delta.
- CPU là baseline. Tiling chỉ dùng khi cần và phải kiểm tra đường nối, overlap và alpha; không thêm sớm khi chưa có bằng chứng cần thiết.
- Project portable có thể là ZIP gồm JSON manifest, source, mask và PNG/path SVG của các biến thể đã chấp nhận. Preview/cache có thể tái tạo; dữ liệu kết quả đã lưu phải xem/xuất lại được khi offline, kể cả thiếu model từng tạo ra nó. Validate schema, kích thước, đường dẫn entry; chống path traversal/ZIP bomb khi mở. Không thực thi pickle hoặc code từ project.
- Lưu ra file tạm cùng thư mục rồi replace khi thành công; kiểm tra overwrite trước đó. Batch không ghi đè file cùng basename từ thư mục khác.
- Không commit ảnh thật, model/cache và output vào Git. Không ghi log dữ liệu ảnh hoặc secrets.

## 8. AI adapter

Giao diện tối thiểu: thông tin model/license, kiểm tra availability/capabilities, ước lượng tài nguyên, xử lý có progress/cancellation, và lỗi có cấu trúc. Adapter thực chỉ được thêm sau khi xác minh model/runtime; không tạo “API AI” tưởng tượng.

Tải weights riêng với manifest nguồn/version/license và checksum tin cậy nếu có. Không dùng `torch.load` không an toàn cho weights không đáng tin; ưu tiên định dạng/runtime an toàn được upstream hỗ trợ. Không tự chấp nhận pickle từ người dùng như project/model thông thường.

Mỗi kết quả lưu tên/version model và tham số. Thiếu GPU không được chặn core; thiếu model không được kích hoạt tải ngầm. Backend chưa khả dụng không được giả lập AI bằng resize thông thường.

## 9. Build và phân phối

- Pin dependency thực tế sau khi thử cài, không bịa hash. Chuẩn bị môi trường dev lặp lại được và CI thích hợp.
- Build PyInstaller trên Windows, ưu tiên `onedir` portable để kiểm tra dependency/license dễ hơn; installer là mốc sau nếu cần.
- Core package không bundle weights lớn. Portable core phải mở và xử lý được trên máy Windows không có Python và không có mạng.
- Giao license notices cho Python packages, Qt/PySide6, codecs và model nếu phân phối. License của app cần chủ repo quyết định; không tự cấp license cho nội dung bên thứ ba.
- CI build thành công không chứng minh toàn bộ GUI hoạt động trên Windows 11. Bằng chứng CI và nghiệm thu thủ công được ghi riêng.
