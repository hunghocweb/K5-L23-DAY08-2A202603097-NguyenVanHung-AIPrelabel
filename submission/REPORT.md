# Báo cáo Day 08 — 202603097 - Nguyễn Văn Hùng

> Mọi con số đều kèm **mẫu số** và lấy từ `submission/qc/*` sau lần chạy lại ngày 2026-11-18 (gold = bản export CVAT gán tay, 56 đối tượng). Phần nào tôi chưa tự kiểm bằng mắt đều ghi rõ.

## 1. Model và recipe (Part 2–3)

| Lớp                                              | Model + cỡ                                      | Ngưỡng                                                                              | Vì sao (dẫn `run_id` trong `model_log.csv`)                                                                                                                                                                                                                                                                                         |
| ------------------------------------------------ | ----------------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`--conf 0.25` khi cần recall)          | Sweep trên gold: motorbike=0.35; person, bicycle, car, bus, truck dùng mặc định 0.3 | **r01 vs r02:** conf 0.25 → 215 box, conf 0.5 → 139 box (nhanh hơn: 5.28 so với 11.35 giây/ảnh nhưng img_007 rơi từ 26 xuống 11 box, mất người và xe máy nhỏ ở xa). Chọn ngưỡng thấp để giữ recall, rồi rà tay. **r05:** cache toàn 36 ảnh ở conf 0.05 (1669 box, 355 box lớp ngoài taxonomy bị loại) làm nguồn cho sweep và merge. |
| three_wheeler                                    | YOLOE-26s, prompt chữ `--classes three_wheeler` | Sweep: 0.15 (recall 100%, precision 78%, 7 vật gold)                                | **r03:** bắt tuk-tuk xanh ở img_002 (0.93) mà YOLO26 (r01) bỏ sót hoàn toàn; chỉ 8 box trên 8 ảnh pilot nên recall nền xa chưa xác nhận. **r06:** chạy toàn tập ở conf 0.05 (125 box); sau review chỉ 3/23 box three_wheeler của máy được giữ nguyên, 12 bị xoá, 8 phải sửa.                                                        |
| Mask                                             | SAM 2.1-b (`sam2.1_b.pt`)                       | —                                                                                   | **r04, r07:** xem mục 5.                                                                                                                                                                                                                                                                                                            |

**Kết quả sweep trên 4 ảnh gold mới (56 đối tượng)** — `qc/threshold_sweep.md`:

- motorbike 0.35: recall 90%, precision 75% (10 vật).
- three_wheeler 0.15: recall 100%, precision 78% (7 vật).
- person: recall tối đa 88% (25 vật), car: 88% (8 vật), bicycle: 60% (5 vật): không ngưỡng ≤ 0.5 đạt 90% nên dùng mặc định.
- bus có 0 vật, truck có 1 vật trong gold: quá ít, không chọn ngưỡng riêng.

**Ngưỡng dùng thật ở Part 4 khác với kết quả sweep.** Lệnh merge đã chạy là:
`python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
Bộ `person=0.35, motorbike=0.5, car=0.35` trùng với ví dụ trong đề, không phải đầu ra của sweep trên gold của tôi. Tôi không dựng lại pre-label và không review lại 24 ảnh theo ngưỡng của sweep, nên mọi số ở mục 2–3 phản ánh ngưỡng merge thực tế, không phải ngưỡng sweep. Hệ quả có thể thấy trong số liệu: với motorbike, máy chỉ giữ 71 box còn tôi phải vẽ thêm 62 (mục 2). Cả hai lần sweep (gold cũ gợi ý 0.2, gold mới gợi ý 0.35) đều thấp hơn 0.5 đã dùng, nên ngưỡng 0.5 nhiều khả năng làm máy bỏ sót xe máy.

**Đối chiếu ngược trên gold (56 đối tượng, in-sample, đọc từ `qc/threshold_sweep.csv`):** tôi không dựng lại pre-label hay review lại 24 ảnh theo sweep, vì làm vậy sẽ làm số đo review (accepted/edited/deleted/added và bắt bẫy) không còn khớp với nhãn tôi đã duyệt. Thay vào đó tôi so hai bộ ngưỡng ngay trên gold. Lớp không có trong chuỗi `--thr` (three_wheeler) dùng `--conf 0.3` mặc định.

| Lớp           | Ngưỡng đã dùng ở Part 4 | Recall / precision trên gold | Ngưỡng sweep   | Recall / precision trên gold |
| ------------- | ----------------------- | ---------------------------- | -------------- | ---------------------------- |
| person        | 0.35                    | 76% (19/25) / 61.3%          | 0.3 (mặc định) | 84% (21/25) / 60.0%          |
| motorbike     | 0.5                     | 80% (8/10) / 72.7%           | 0.35           | 90% (9/10) / 75.0%           |
| car           | 0.35                    | 87.5% (7/8) / 50.0%          | 0.3 (mặc định) | 87.5% (7/8) / 46.7%          |
| three_wheeler | 0.3 (mặc định)          | 71.4% (5/7) / 83.3%          | 0.15           | 100% (7/7) / 77.8%           |

Đọc bảng: ngưỡng sweep cho recall cao hơn ở motorbike (+10 điểm), three_wheeler (+29 điểm) và person (+8 điểm), precision gần như không đổi. Điều này khớp với việc tôi phải vẽ thêm 62 xe máy khi review. Mẫu rất nhỏ (5–25 vật mỗi lớp) nên chỉ là chiều hướng, không phải kết luận.

**Lỗi không sửa được bằng ngưỡng (tuk-tuk → car/truck):** ở r01/r02, đổi conf từ 0.25 lên 0.5 không đổi nhãn: img_011 tuk-tuk vẫn bị gọi là `truck`; ở r04, img_004 và img_013 tuk-tuk vẫn mang nhãn `car` vì box đầu vào từ detector đã sai lớp (SAM chỉ vẽ mask, không đổi lớp). Đây là giới hạn của detector 80 lớp COCO. Chỉ open-vocabulary (YOLOE) hoặc sửa tay mới xử lý được.

**Phiên bản SAM đã chọn:** SAM 2.1-b (`models/sam2.1_b.pt`) vì trọng số có sẵn trên máy (PyTorch CPU-only, 41.59 giây/ảnh ở r04). MobileSAM không có trọng số cục bộ nên chưa có đối chứng trực tiếp.

**Giới hạn:** chọn ngưỡng và chấm precision/recall trên cùng 4 ảnh gold nên kết quả là in-sample, lạc quan; các lớp có 1–8 vật thì kết luận yếu.

## 2. AI error profile (từ `qc/review_stats.md`)

Mẫu: 24 ảnh batch; 433 box máy sau khi loại 24 đối tượng quanh các bẫy (lỗi nhân tạo không phản ánh chất lượng máy); 509 đối tượng sau duyệt.

| Trạng thái | Số  | Tỉ lệ           | Mẫu số                  |
| ---------- | --- | --------------- | ----------------------- |
| accepted   | 338 | 338/433 = 78.1% | 433 box máy             |
| edited     | 42  | 42/433 = 9.7%   | 433 box máy             |
| deleted    | 53  | 53/433 = 12.2%  | 433 box máy             |
| added      | 129 | 129/509 = 25.3% | 509 đối tượng sau duyệt |

- Trong deleted + added có 7 trường hợp là **đổi lớp** (box đúng chỗ, sai lớp): không tính là máy bỏ sót. Số vật máy sót thật ≈ 129 − 7 = 122.
- Theo lớp (accepted / edited / deleted / added): motorbike 71/15/5/62; person 216/16/20/25; car 36/3/10/21; **three_wheeler 3/8/12/7**; bicycle 5/0/3/6; truck 5/0/2/6; bus 2/0/1/2. Lớp yếu nhất của máy là `three_wheeler` (trong 23 box máy chỉ 3 box giữ nguyên, 12 bị xoá) và `motorbike` (bị sót nhiều).
- **So với gold (4 ảnh, 56 đối tượng gán tay):** máy đạt precision 62.1% (41/66, KTC 95%: 50–73%) và recall 73.2% (41/56, KTC 95%: 60–83%). 25 box FP chưa được tôi phân loại theo từng ảnh; có thể một phần là vật nhỏ dưới 20 px mà tôi không vẽ (đúng theo contract, vật như vậy gom vào `ignore_region`), nhưng tôi chưa kiểm nên không khẳng định. Đây là kết quả in-sample, không phải ước lượng độc lập.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được **10/12 = 83.3%** bẫy (KTC 95%: 55–95%; ngưỡng đạt 80%, tức sát mức tối thiểu). Theo loại: extra 3/3, missing 3/3, **wrong_class 2/3**, **loose 2/3**. Script ghi lọt img_017 (loose) và img_018 (wrong_class).
- **Bẫy script ghi lọt: `img_017.jpg` (loose)** — bẫy là box `three_wheeler` bị nới lệch (box gốc 586×293 px, box bẫy 565×322 px). Khi đối chiếu với `batch_reviewed.json`: không có box cuối nào giống box bẫy (IoU lớn nhất 0.37), tức là tôi không chấp nhận nguyên box lỏng. Nhãn cuối của ảnh có ba box `three_wheeler` riêng ([466, 325, 421×306], [202, 329, 287×235] và [71, 326, 146×170] cho xe mép trái bị che một phần); hai box đầu mỗi box có IoU khoảng 0.4 với box gốc. Script chỉ tính là bắt được khi box cuối khớp lại box gốc, nên ca tách thành nhiều box như thế này bị ghi là lọt. Script vẫn tính ca này là lọt; đây là giới hạn của cách chấm, không phải bằng chứng tôi bỏ bước kiểm 4 cạnh. Đối chiếu bằng ảnh (Claude mở ảnh và so với nhãn ngày 2026-11-18; tôi chưa tự nhìn lại): ảnh có ba tuk-tuk xếp hàng, box gốc rộng 586 px lớn hơn xe lớn nhất (421 px) nên phải bao ít nhất hai xe, và ba box cuối khớp ba xe. Vậy ca này thực tế đã xử lý đúng, chỉ script không nhận.
- `accepted` 78.1% không chứng minh máy tốt: cùng lúc máy **sót 122 vật** (added) và recall trên gold chỉ 73.2%. Điểm accepted cao chỉ nói về những box máy đã vẽ, không nói về những vật máy không vẽ.
- **img_018 (wrong_class):** bẫy đổi box #169 (11×17 px, gốc `bicycle`) thành `motorbike`. Sau khi đối chiếu với CVAT (2026-11-18) tôi sửa nhãn cuối thành `person`, khác cả lớp gốc lẫn lớp bẫy, nên script (nhiều khả năng chỉ tính bắt được khi nhãn cuối trùng lớp gốc; tôi chưa đọc mã `review_stats.py` để xác nhận) ghi lọt và số bẫy từ 11/12 xuống 10/12. Đây là nhãn tôi tự xác nhận trên CVAT; box rất nhỏ nên đáng kiểm lại.
- Chỉ có 12 bẫy nên khoảng tin cậy rộng (55–95%); không kết luận chỉ từ con số 83.3%.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gold_manual 25 phút / 4 ảnh = 6.25 phút/ảnh (**ước tính, chưa bấm giờ**), 56 đối tượng ≈ 0.45 phút/đối tượng; batch_review 120 phút / 24 ảnh = 5 phút/ảnh (theo ghi nhận của tôi, chưa bấm giờ từng giai đoạn), 509 đối tượng ≈ 0.24 phút/đối tượng. Hai con số không so sánh trực tiếp được vì ảnh batch đông vật hơn, và cả hai đều ước tính.
- Kết luận: dùng model để tạo bản nháp cho `person`, `car`, `motorbike` rồi rà tay; **không dùng** cho `three_wheeler` nếu không sửa. Tiết kiệm thời gian có thể có, nhưng máy sót 122 vật và precision trên gold chỉ 62.1% nên không phát hành tự động.
- **Ba quy tắc giảm thiên kiến có thể kiểm tra:**
  1. Trước khi nhìn box, quét toàn ảnh và ghi số vật đếm được. Kiểm: so với số box sau duyệt, chênh lệch lớn phải giải thích được.
  2. Với mỗi box, xác nhận lớp và bốn cạnh trên ảnh gốc; lọc theo từng lớp trước khi lưu. Kiểm: tỉ lệ bắt bẫy `loose` phải đạt 3/3 (hiện 2/3 theo script).
  3. Gắn `can_review` cho ca không chắc quy tắc; QC phải gồm cả ca này và ít nhất 5 ảnh ngẫu nhiên. Kiểm: số ca `can_review` > 0 và mỗi lỗi ghi vào `qc_review.csv`.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh         | SAM (phiên bản) | Số click          | Mask gồm người lái / hàng hoá?                                              | Lỗi quan sát          | File trong `masks/`                                        |
| ----------- | --------------- | ----------------- | --------------------------------------------------------------------------- | --------------------- | ---------------------------------------------------------- |
| img_013.jpg | SAM 2.1-b       | 7 (2 dương, 5 âm) | Đã giảm dính người lái/hành khách; còn thiếu một phần bánh/khung, 2 polygon | Thiếu bánh/khung      | [tuktuk_img013_sam21b.jpg](masks/tuktuk_img013_sam21b.jpg) |
| img_004.jpg | SAM 2.1-b       | 3 (2 dương, 1 âm) | Bao phần lớn thân/mái; còn vài lỗ/biên chưa khít quanh bánh                 | Lỗ và biên quanh bánh | [tuktuk_img004_sam21b.jpg](masks/tuktuk_img004_sam21b.jpg) |
| img_017.jpg | SAM 2.1-b       | 3 (2 dương, 1 âm) | Bao phần lớn mái/thân; kính và biên cần hậu kiểm theo quy tắc mask          | Kính, biên            | [tuktuk_img017_sam21b.jpg](masks/tuktuk_img017_sam21b.jpg) |

- Đối chiếu mục 4.2 của guideline: người lái và hàng nhô ra phải tách riêng khỏi mask xe. Ở img_013, một điểm âm chưa đủ; thêm nhiều điểm âm giảm được phần dính người. Quy tắc rút ra: người ngồi sát xe cần 2 điểm âm trở lên và kiểm lại biên.
- Mask là overlay JPG, chưa nạp vào CVAT. MobileSAM chưa chạy do thiếu trọng số; Interactor SAM trong CVAT chưa thử. Tôi chưa tự kiểm lại từng ảnh mask bằng mắt trong lượt cập nhật này.

## 6. Calibration (Part 6)

- `calibration_decisions.csv` có 9 quyết định, gồm `cau_hoi_mo` (CAL-01–05, CAL-09) và `loi_may` (CAL-06–08). Các quy tắc được ghi vào `guideline_v1.1_calibrated.md`: `conf` không thay thế kiểm lớp; minibus chở khách là `bus`; xe đẩy tay không mặc nhiên là `three_wheeler`; một box chỉ bao một xe; kính trong và người là phần riêng khi tạo mask.
- **CAL-06–CAL-08 dùng ID box của snapshot nhãn cũ** (#141/#142, #190, #248). Export hiện tại đã thay đổi ở các ảnh đó nên các ID này chỉ còn là ví dụ lịch sử, chưa kiểm lại trên bản hiện tại. Bằng chứng còn đúng trên bản hiện tại: `three_wheeler` bị xoá 12 và sửa 8 trong 23 box máy (mục 2), và `provenance` cho thấy `img_018` có box đổi lớp thành `bus`.
- Phân biệt: "guideline mơ hồ" là câu hỏi xe đạp điện/xe máy điện (chưa có ví dụ để chốt); "máy sai" là tuk-tuk → car/truck và xe đẩy tay → `three_wheeler`.
- **Câu hỏi 1 (xe đạp điện / xe máy điện) vẫn mở.**

## 7. QC và quyết định (Part 6)

- Mẫu QC (chạy lại 2026-11-18 trên nhãn đã sửa): 8/24 ảnh batch, 205 đối tượng, gồm 3 ảnh ca khó (img_014, img_017, img_018; script giới hạn 3 ảnh `can_review`, img_023 không còn trong mẫu) và 5 ảnh ngẫu nhiên (img_003, img_006, img_013, img_021, img_035). Cờ `can_review` mới chỉ nằm trong `batch_reviewed.json`, chưa tick trong CVAT.
- Phiếu `qc_review.csv` do tôi rà lại bằng mắt ngày 2026-11-18: cả 8 ảnh đều `khong` (img_006 #80 và các xe khó nhận diện phía sau cố ý không khoanh; img_014 #140 đúng vùng; img_003 #17 `three_wheeler` đúng). Img_018 có 58 đối tượng nên không kiểm được từng box. Img_035 dựa trên lần rà soát 24 ảnh, chưa xem lại bằng ảnh overlay QC.
- Tỉ lệ lỗi theo phiếu: theo **đối tượng** 0/205 = 0%; theo **ảnh** 0/8 = 0%. Cận trên Wilson 95%: 32% theo ảnh (`qc_report_auto.md`), khoảng 1.8% theo đối tượng (tính tay). Mẫu 8 ảnh, trong đó 3 ảnh là ca khó chọn có chủ ý, không đại diện cho cả lô và không thể chứng minh ngưỡng 2%.
- Bẫy `loose` ở img_017 do script ghi là lọt (mục 3); đối chiếu bằng ảnh cho thấy box gốc bao hai xe và tôi đã tách đúng (Claude xem ảnh, tôi chưa tự nhìn lại).
- Script chỉ kiểm cấu trúc và số đo từ phiếu; không xác nhận được vật sót, độ ôm sát, hay việc tôi đã làm checklist bằng mắt.

**Quyết định: Hold** (2026-11-18). Lý do: (1) guideline v1.1 chưa có quy tắc chốt cho xe đạp điện/xe máy điện (mục 6; `qc_report.py --guideline-gap 1` cho HOLD); (2) mẫu QC chỉ 8 ảnh nên 0 lỗi chưa chứng minh được ngưỡng 2%; (3) bẫy 10/12 = 83.3% chỉ sát ngưỡng 80%. Tôi đã sửa 3 nhãn img_018 (#169 person; #190, #197 motorbike) cho khớp CVAT và chạy lại `validate_labels`, `review_stats`, `build_release`, `qc_sample`, `qc_report`.
Việc tiếp theo: tick các box `can_review` trong CVAT, xem lại img_035 bằng ảnh overlay QC, chốt quy tắc xe điện vào v1.1 rồi quyết định lại.

## 8. Ghi chú cá nhân

- **Công cụ AI đã dùng:** Copilot, claude.
- **Gold:** tôi gán tay 4 ảnh gold trên CVAT (56 đối tượng; so với nhãn máy chỉ 3/56 box trùng IoU > 0.99).
- **Thời gian:** gold_manual 25 phút và batch_review 120 phút đều là ước tính của tôi, chưa bấm giờ từng giai đoạn.
- **Máy / môi trường:** Windows, PyTorch CPU-only (CUDA không khả dụng), CVAT v2.74.1. Lần chạy lại `threshold_sweep.py` ban đầu lỗi NumPy 2.2.1 với matplotlib cũ; sau khi nâng cấp matplotlib thì chạy được.
- SAM 2.1-b có trọng số tại `models/sam2.1_b.pt`; MobileSAM không có trọng số cục bộ.
- **Lần sau làm khác:** bấm giờ từng giai đoạn, tick `can_review` ngay khi gặp ca khó, và chạy sweep **trước** khi merge để dùng đúng ngưỡng.
