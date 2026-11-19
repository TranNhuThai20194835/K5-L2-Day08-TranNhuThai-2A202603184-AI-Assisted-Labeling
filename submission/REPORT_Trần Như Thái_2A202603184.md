# Báo cáo Day 08 — 2A202603184 - Trần Như Thái

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao |
|---|---|---|---|
| person / motorbike / car / bus / truck | YOLO26 | mặc định 0.3 | `threshold_sweep.md`: không ngưỡng nào ≤ 0.5 đạt recall 90% trên gold (person tối đa 81%/27 vật, motorbike 69%/16, car 88%/8); bus 1 vật, truck 0 vật (< 5) nên không chọn ngưỡng riêng |
| bicycle | YOLO26 | 0.2 | recall 100%, precision 83% trên 5 vật gold |
| three_wheeler | YOLOE (prompt: tuk-tuk, cyclo, cargo tricycle) | 0.15 | recall 100%, precision 78% trên 7 vật gold |

- Lệnh `merge_prelabels.py` đã dùng: python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
python scripts/inject_traps.py --pre work/prelabel/merged_review/pre.coco.json --student "202603184"
- **Phiên bản SAM:** đã chạy cả SAM 1 và SAM 2 (xem mục 5). Lựa chọn: SAM 2.1 
- Lỗi không sửa được bằng đổi ngưỡng: máy gọi sai lớp. Đối chiếu pre-label với nhãn cuối (IoU ≥ 0.5, ~9–10 ca, khớp 9 ca trong `review_stats.md`): truck → car 4, three_wheeler → truck 2, three_wheeler → bus 2, bus → car 1, motorbike → bicycle 1. Tức xe ba bánh bị gọi là **truck / bus**, không phải car. Cách sửa: đổi lớp khi duyệt; hạ hay nâng ngưỡng không giúp vì box đúng chỗ, chỉ sai nhãn.
- **Điểm cao mà sai** (theo đối chiếu IoU của tôi, cần kiểm lại): img_024 car 0.81 và img_003 three_wheeler 0.80 bị xoá hoặc thay. **Điểm thấp mà đúng:** 19 box person có conf < 0.4 vẫn được giữ nguyên (IoU ≥ 0.9); xích lô img_035 chỉ 0.30 nhưng đúng lớp.
- Hệ quả của ngưỡng 0.15 cho three_wheeler: chọn để đạt recall 100% trên 7 vật gold, nhưng trên batch 13/26 box three_wheeler bị xoá (50%). Ngưỡng thấp tối ưu trên gold không chuyển được sang lô thật.
- Giới hạn: chọn ngưỡng và chấm trên cùng 4 ảnh gold nên số đo máy trên gold là lạc quan: ngưỡng được chọn và chấm trên cùng 64 đối tượng (precision 69.8%, KTC 95% 58–80%; recall 68.8%, KTC 95% 57–79%), không có tập kiểm riêng. Hai ngưỡng riêng (bicycle 0.2, three_wheeler 0.15) dựa trên rất ít vật (5 và 7 vật gold), nên chỉ là ước lượng thô. Bằng chứng ngưỡng không chuyển được sang lô thật: three_wheeler 0.15 đạt recall 100% trên gold nhưng ở lô batch 13/26 box lớp này bị xoá (50%). Các lớp bus (1 vật) và truck (0 vật) không đủ mẫu nên giữ ngưỡng mặc định 0.3.
## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 376 | 86.6% | 434 box máy |
| edited | 24 | 5.5% | 434 box máy |
| deleted | 32 | 7.4% | 434 box máy |
| added | 114 | 21.9% | 520 đối tượng sau duyệt |

Đã loại 22 đối tượng quanh các bẫy khỏi bảng. Ba trạng thái đầu cộng lại 432/434 (2 box còn lại có thể là `ignore_region`). Trong deleted + added có 9 ca đổi lớp, nên số vật máy sót thật ≈ 114 − 9 = 105.

| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 236 | 8 | 5 | 43 |
| bicycle | 6 | 0 | 3 | 4 |
| motorbike | 81 | 7 | 2 | 47 |
| car | 44 | 1 | 4 | 8 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 4 | 2 |
| three_wheeler | 5 | 8 | 13 | 8 |

- **Lớp tệ nhất: three_wheeler.** Chỉ 5/26 box giữ nguyên (19.2%), 13 bị xoá, 8 sửa, 8 vẽ thêm. Ngược lại person tốt nhất: 236/249 box giữ nguyên (94.8%). **motorbike** là lớp bị sót nhiều nhất: 47 vật phải vẽ thêm, phần lớn ở ảnh dày xe máy (005, 018, 021, 024, 028, 029). Ảnh dẫn: img_003 (three_wheeler 0.80 bị xoá), img_014 và img_023 (three_wheeler 0.63 bị xoá).
- So với gold: precision **69.8%** (KTC 95%: 58–80%), recall **68.8%** (KTC 95%: 57–79%); TP 44, FP 19, FN 20; mẫu số recall: 64 đối tượng gold trong 4 ảnh (img_001, 012, 020, 036). Con số lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **10/12 = 83.3%** (KTC 95%: 55–95%; ngưỡng đạt ≥ 80%). Theo loại: missing 3/3, wrong_class 3/3, extra 3/3, **loose 1/3**.
- Lọt: **img_003 (loose, person #19)** và **img_006 (loose, person #75)**. Tôi đối chiếu `batch_reviewed.json` với `traps_key.json`: cả hai box vẫn trùng khớp 100% (IoU 1.00) với box lỏng đã cài, và chỉ khớp 0.52 và 0.67 với box gốc. Bẫy loose bắt được duy nhất là img_013 (IoU với box gốc 0.96). Cả hai ảnh lọt đều dày (20 và 24 đối tượng).
- Bước checklist bỏ: bước cuối "lỏng" (so cạnh box với vật). 
- Accepted 86.6% và bắt bẫy 83.3% khá nhất quán, không có dấu hiệu duyệt cho qua hàng loạt. Điểm yếu là **lỗi hình học**: đã bắt hết lỗi thiếu, thừa, sai lớp nhưng lọt 2/3 box lỏng. Chỉ 12 bẫy nên khoảng tin cậy rộng.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay 42 phút / 4 ảnh = **10.5 phút/ảnh** (≈ 36 giây/đối tượng, 70 nhãn gồm 6 `ignore_region`); duyệt có pre-label 300 phút / 24 ảnh = **12.5 phút/ảnh** (≈ 32.5 giây/đối tượng, 553 nhãn). Thiên lệch: bạn gán gold trước nên đã quen cảnh; ảnh batch dày hơn (23 so với 17.5 đối tượng/ảnh).
- Kết luận: **chỉ đáng dùng cho một số lớp, và chưa tiết kiệm thời gian tổng thể.** Theo từng đối tượng, thời gian xấp xỉ gán tay (36 so với 32.5 giây). Máy tốt ở person (94.8% giữ nguyên) và car (44/49 = 89.8%), kém ở three_wheeler (19.2%) và bỏ sót nhiều motorbike (47 vật). Với three_wheeler nên gán tay hoặc nâng ngưỡng hơn 0.15.
- **Ba quy tắc giảm thiên kiến của tôi:**
  1. Quét ảnh và đếm số vật tự thấy **trước** khi bật nhãn máy. Kiểm tra: số `added` theo lớp ở lô sau (mục tiêu giảm motorbike 47 vật bị sót).
  2. Với mỗi ảnh, kiểm cạnh box của các vật lớn nhất so với biên vật (sai số ≤ ~2 px theo v1.0 mục 4.1.1) trước khi sang ảnh tiếp. Kiểm tra: bắt bẫy loose 3/3 ở lô sau.
  3. Mọi box `three_wheeler` và mọi box `conf` < 0.5 được duyệt riêng một lượt, không dùng `conf` để bỏ qua việc nhìn ảnh (CAL-03). Kiểm tra: không còn box three_wheeler sai lớp (máy gọi truck / bus) trong mẫu QC.

## 5. Mask SAM cho 3 vật (Part 5)

Số click đếm từ chấm trên ảnh (xanh = dương, đỏ = âm): mỗi ảnh 2 dương + 1 âm.

| Ảnh | SAM | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File |
|---|---|---|---|---|---|
| img_013 | SAM 1 | 3 | Gần như không gồm người lái và khách | Loang sang mặt, tay người lái; vài mảng thủng ở khung xe | `tuktuk_img013_sam1.jpg` |
| img_013 | SAM 2 | 3 | **Gồm cả người lái và khách ngồi**, dù đã click âm lên người lái | Mask phủ qua khe khung xe; vài lỗ nhỏ rải rác trên áo người lái | `tuktuk_img013_sam2.jpg` |
| img_004 | SAM 2 | 3 | Không gồm người đứng bên cạnh | Mép dưới sát bóng đổ bị răng cưa; mảng thủng ở kính xe | `tuktuk_img004_sam2.jpg` |
| img_017 | SAM 2 | 3 | Không có người; gồm cả khoang ghế | Click âm (chấm đỏ) nằm trên mui xe, trong vùng mask, nên không có tác dụng loại; <<kiểm lại ý định>> | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline 4.2: quy tắc "người là vật riêng" (mục 5 và 4.1.4) giúp xác định mask xe không được gồm người. Quy tắc 4.2.2 (lỗ thủng) giúp xử lý khe khung xe. Thiếu: chưa chốt kính chắn gió (mục 6 câu 5): mask ở img_004 và img_017 đều gồm kính. 
- So sánh SAM 1 và SAM 2 trên img_013: SAM 2 gọn ở biên xe nhưng nuốt cả người ngồi trong xe, vi phạm quy tắc người là vật riêng; SAM 1 tách người tốt hơn nhưng loang sang mặt và tay. Cả hai cần thêm click âm trên người.
- Interactor SAM trong CVAT: không.

## 6. Calibration (Part 6)

- Số quyết định: **5** (`calibration_decisions.csv`: 3 `cau_hoi_mo`, 2 `loi_may`). Quan trọng nhất: **CAL-01** (hàng nhô, ngưỡng ~20% và cách đo chiều dài xe, vì máy bỏ khay hàng ở img_031) và **CAL-03** (không đặt ngưỡng `conf` để bỏ qua việc nhìn ảnh: xích lô 0.30 và xe đạp 0.45 đúng lớp, còn 2/12 bẫy lọt).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu 2, 3, 7 của mục 6 v1.0; không phủ `ignore_region` lên vật đã gán (CAL-04); xoá box máy trong cụm nhỏ trước khi vẽ `ignore_region` (CAL-05). Câu 1, 4, 5, 6 chưa chốt vì 5 ảnh mẫu không có ca tương ứng.
- Nguyên nhân gốc: CAL-01, CAL-02, CAL-03 do guideline v1.0 chưa trả lời (mục 6); CAL-04, CAL-05 do máy (chồng box và `ignore_region`, cụm nhỏ ở xa). Ngoài ra không có box nào được tick `can_review` (0 trên 623 nhãn gold + batch), dù v1.0 yêu cầu tick khi chưa chắc quy tắc: nên chưa có ca `can_review` nào làm nguyên liệu calibration.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42): img_003, 006, 014, 031, 035; 98 đối tượng. Người kiểm: tôi (**hạn chế**: tôi vừa là người duyệt, nên không độc lập).
- Lỗi theo **đối tượng**: **2/98 = 2.0%** · theo **ảnh**: **2/5 = 40%** (KTC 95% Wilson: 12–77%). Ngưỡng Release ≤ 2% theo ảnh: **vượt**.
- Top lỗi: sai vị trí/kích thước 2/2. Ví dụ: img_003 person #19 (box lệch ~30 px trái và lên, IoU với box gốc 0.52); img_006 person #75 (IoU 0.67). Hai lỗi này chính là hai bẫy loose lọt (mục 3), nên là lỗi cùng loại lặp lại.
- Giới hạn mẫu nhỏ: 5 ảnh, 2 lỗi. Cận trên 95% của tỉ lệ ảnh lỗi thật là 77%. Mẫu quá nhỏ để ước lượng chính xác tỉ lệ lỗi cả lô, và chỉ cho biết có lỗi hệ thống ở ảnh dày; 3 ảnh còn lại được đọc nhanh nên "khong lỗi" là yếu.
- Script kiểm được / không kiểm được: `qc_report.py` chỉ đếm theo phiếu do người điền, nên không tự phát hiện box lỏng; hai lỗi trên chỉ biết nhờ `traps_key.json`. Nó cũng không kiểm được vật cả tôi và máy cùng bỏ sót, hay việc tôi có làm theo checklist không. Các box lỏng không phải bẫy có thể còn nằm trong 19 ảnh chưa vào mẫu.
- Lưu ý dữ liệu: `qc_report_auto.md` trong zip được sinh trước khi điền `qc_review.csv` (còn cảnh báo "5 ảnh CHƯA kiểm" và gợi ý RELEASE 0%). Cần chạy lại `qc_report.py` sau khi chốt phiếu; số ở trên là số tôi tính thủ công từ phiếu đã sửa.

**Quyết định: Rework**, ngày 2026-11-19. Lý do (theo Hold → Rework → Release): không Hold vì guideline đã rõ hơn nhờ v1.1 và dữ liệu không hỏng; **Rework** vì ảnh lỗi 2/5 = 40% vượt ngưỡng 2% và lỗi box lỏng lặp ở cả hai ảnh; bắt bẫy 10/12 = 83.3% đạt ngưỡng nhưng loose chỉ bắt 1/3.
Việc cần làm tiếp: (1) sửa 2 box lỏng (img_003 #19, img_006 #75) và rà lỗi ôm sát toàn lô, ưu tiên ảnh dày ≥ 20 vật; (2) kiểm lại 26 box three_wheeler (13 bị xoá) theo CAL-03; (3) chạy lại `build_release.py`, rồi `qc_sample.py` với n lớn hơn có ảnh `can_review`, và `qc_report.py`; (4) áp v1.1 lên 6 ảnh dày xe máy (005, 018, 021, 024, 028, 029).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Claude, dùng để soạn nháp `calibration_decisions.csv`, `guideline_v1.1_calibrated.md`, `qc_review.csv` 
- Gold có bị nhiễm không? không 
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windoư/GPU/CVAT 2.81.1
- Dùng cache `data/cache/` thay vì tự chạy? không
- Lỗi / bug đã gặp: file nộp ban đầu có vài lỗi định dạng (tiêu đề `calibration_decisions.csv` bị mất chữ `c`, `time_log.csv` có dấu phẩy chưa bọc nháy, `qc_review.csv` dùng `no` thay `khong`); đã sửa.
- Một điều tôi làm khác lần sau: không
