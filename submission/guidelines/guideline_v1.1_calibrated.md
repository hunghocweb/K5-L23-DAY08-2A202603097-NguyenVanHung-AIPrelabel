# Label contract — Phố châu Á — BỔ SUNG v1.1 — 202603097 - Nguyễn Văn Hùng

> Tôi hiệu chỉnh guideline AI-assisted dựa trên ảnh mẫu và lỗi QC đã ghi. Tôi chưa xác nhận các ca đánh dấu tạm thời, nên chưa coi đây là bản guideline đã ký duyệt.

> Lưu ý: CAL-06–CAL-08 và các ID box trong đó thuộc snapshot nhãn trước. Export CVAT mới đã có thay đổi ở các ảnh liên quan; các ví dụ này giữ vai trò lịch sử hiệu chỉnh, không phải kết luận rằng lỗi vẫn còn trong snapshot mới.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Xe máy gần nhau | Từng xe còn phân biệt được thì gán riêng; chỉ dùng `ignore_region` khi thực sự không tách được hoặc vật dưới ngưỡng v1.0. | img_006 | CAL-09 |
| 3 | Hàng hóa nhô ra | Giữ ngưỡng 20% của v1.0; hàng nhô quá ngưỡng không thuộc box xe, mask chỉ theo phần xe. | img_009 | CAL-01 |
| 4 | Người ngồi dưới mui | Gán `person` riêng nếu thấy ít nhất 20%; ca sát ngưỡng gắn `can_review`; không nhập người vào mask xe. | img_013 | CAL-02 |
| 5 | Kính/cửa sổ | Chỉ mask phần vật liệu xe; loại kính trong/khoảng hở khi vùng đó chiếm từ 5% diện tích vật, và tách người nhìn thấy. | img_017 | CAL-03 |
| 6 | Box chồng nhau | Không gộp xe riêng thành một instance; một box cho một xe, chồng lấn chỉ được giữ khi phản ánh che khuất thực tế. | img_023 | CAL-04 |
| 7 | Ngưỡng conf | Không có ngưỡng conf đủ để tự động chấp nhận; luôn kiểm lớp và biên trên ảnh thật. | img_018 | CAL-05 |

**Tôi chưa chốt câu 1 (xe đạp điện/xe máy điện):** mẫu QC hiện không có ví dụ đủ rõ để quyết định nhất quán; tôi cần xem ảnh có cấu tạo động cơ/pedal nhìn được rồi chốt quy tắc. Câu hỏi này khiến quyết định phát hành phải giữ `HOLD`.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì | Quyết định hiệu chỉnh | Quy tắc mới |
|---|---|---|---|---|
| img_014 | Xe bán hàng đẩy tay bị coi là phương tiện ba bánh. | Các box #141/#142 gán `three_wheeler`; tôi chưa xác nhận được bánh xe vì hình bị che một phần. | Không nhận xe đẩy/quầy hàng là `three_wheeler` nếu không thấy cấu trúc xe ba bánh, tuk-tuk hoặc xích lô; chuyển ca mờ sang `can_review`. | Mục 2; CAL-06 |
| img_018 | Minibus trắng ở giao lộ. | Box #190 (50x35 px, conf=0.53) gán `three_wheeler`. | Theo taxonomy, minibus chở khách là `bus`; không để conf thay cho nhận diện loại xe. | Mục 2; CAL-07 |
| img_023 | Nhiều tuk-tuk đỗ nối tiếp. | Box #248 (585x534 px) phủ nhiều xe và chồng lên các box xe riêng. | Tách mỗi xe thành một instance; sửa box bao trùm nhiều phương tiện trước khi release. | Mục 4.1; CAL-08 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt cách xử lý hàng nhô, người dưới mui, vùng kính, box chồng lấn, conf; bổ sung ví dụ xe đẩy tay và minibus. Câu xe điện vẫn mở. | Ảnh QC `img_006`, `img_009`, `img_013`, `img_014`, `img_017`, `img_018`, `img_023`; CAL-01–CAL-09 |
