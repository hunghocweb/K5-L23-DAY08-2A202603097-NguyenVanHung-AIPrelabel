# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 433; đối tượng sau duyệt: 509; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 338 | 338/433 = 78.1% | 433 box máy |
| edited (sửa) | 42 | 42/433 = 9.7% | 433 box máy |
| deleted (xoá) | 53 | 53/433 = 12.2% | 433 box máy |
| added (vẽ thêm) | 129 | 129/509 = 25.3% | 509 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 122.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 216 | 16 | 20 | 25 |
| bicycle | 5 | 0 | 3 | 6 |
| motorbike | 71 | 15 | 5 | 62 |
| car | 36 | 3 | 10 | 21 |
| bus | 2 | 0 | 1 | 2 |
| truck | 5 | 0 | 2 | 6 |
| three_wheeler | 3 | 8 | 12 | 7 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 41 | 25 | 15 | 62.1% (KTC 95%: 50–73%) | 73.2% (KTC 95%: 60–83%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 2/3
  - missing: 3/3
  - wrong_class: 2/3
  - lọt: img_017.jpg (loose), img_018.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
