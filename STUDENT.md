# Thông tin học viên

- Họ và tên: Nguyễn Văn Hùng
- MSHV: 202603097
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: hunghocweb
- Repo bài nộp (URL): https://github.com/hunghocweb/K5-L23-DAY08-2A202603097-NguyenVanHung-AIPrelabel
- Commit chốt bài (hash): d9bef9c
- Máy: Windows; PyTorch CPU-only (CUDA không khả dụng)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 338 / 42 / 53 (mẫu số: 433 box máy); added: 129 (mẫu số: 509 đối tượng sau duyệt)
- So với gold (4 ảnh, 56 đối tượng gán tay): precision / recall của **máy** 62.1% (41/66) / 73.2% (41/56)
- Bắt bẫy: 10/12 = 83.3% (theo script, KTC 95%: 55–95%); script ghi lọt img_017 (loose; nhãn cuối đã tách thành ba box three_wheeler) và img_018 (wrong_class; tôi sửa nhãn cuối thành person theo CVAT, khác lớp gốc) — xem REPORT mục 3
- Thời gian: gán tay 25 phút / 4 ảnh (ước tính, chưa bấm giờ); batch review 120 phút / 24 ảnh (theo ghi nhận)
- Lỗi QC: theo ảnh 0/8, theo đối tượng 0/205 (mẫu 8 ảnh, 3 ảnh là ca khó chọn có chủ ý); cận trên Wilson 95% của ảnh lỗi: 32%, nên chưa chứng minh được ngưỡng 2%
- **Quyết định:** Hold — quy tắc xe đạp điện/xe máy điện chưa chốt trong v1.1, mẫu QC chỉ 8 ảnh, bẫy 83.3% chỉ sát ngưỡng 80%
