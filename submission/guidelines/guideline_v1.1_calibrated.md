# Label contract — Phố châu Á — BỔ SUNG v1.1 — <<2A202603184-Trần Như Thái>>

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? Ngưỡng? | Không đổi ngưỡng của 4.1.2 và 4.1.6. Xe cao ≥ 20 px và thấy ≥ 20% diện tích gán box riêng, kể cả sát nhau và ở gần camera. Xe thấy < 20% không gán. `ignore_region` chỉ cho cụm ≥ 5 vật cao < 20 px. | img_006 | CAL-02 |
| 3 | Hàng nhô ra: box xe và mask có gồm hàng không? | Giữ ngưỡng ~20% của 4.1.5. Chiều dài xe đo từ mép ngoài bánh này đến mép ngoài bánh kia. Hàng nhô theo hướng đó ≤ ~20% thì box ôm cả hàng, > ~20% thì box dừng ở xe. Mask dùng cùng ngưỡng; hàng gắn cố định (giỏ, khay buộc vào giá) thuộc mask xe. | img_031 (khay quả hồng nhô trái ~17%, ước từ ảnh; đo lại trong CVAT) | CAL-01 |
| 7 | Có đặt mức `conf` tối thiểu để chấp nhận box máy không cần sửa? | Không đặt. Mọi box máy đều phải đối chiếu với ảnh. `conf` chỉ xếp thứ tự duyệt: box `conf` < 0.5 và mọi box `three_wheeler` duyệt trước. Lý do: xích lô 0.30 (img_035) và xe đạp 0.45 (img_031) đúng lớp dù điểm thấp; 2/12 bẫy vẫn lọt. | img_035, img_031 | CAL-03 |

**Chưa chốt** (5 ảnh mẫu QC không có ca tương ứng; xử lý tạm: gán lớp bạn tin nhất và tick `can_review`):

| # | Câu hỏi | Vì sao chưa chốt |
|---|---|---|
| 1 | Xe đạp điện / xe máy điện: `bicycle` hay `motorbike`? | Không có xe điện trong 5 ảnh mẫu |
| 4 | Ngưỡng "còn thấy ≥ 20%" cho người trong xe ba bánh có mui | Cần đo độ lệch giữa hai lần đánh giá trên cùng ảnh; chưa làm |
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe không? | Không có ô tô trong 5 ảnh mẫu |
| 6 | Box máy chồng nhau quá nhiều ở ảnh đêm | Không có ảnh đêm trong 5 ảnh mẫu |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_006 | Hẻm đỗ xe máy dày | Tạo `ignore_region` #82 phủ lên vùng đã có box `motorbike` (#78...) | Giữ box xe đủ điều kiện; `ignore_region` chỉ bao phần không gán được, thu nhỏ hoặc xoá nếu phủ lên box | Mục 2, lớp `ignore_region`: thêm "không phủ lên vật đã gán" (CAL-04) |
| img_014 | Cụm xe và người nhỏ ở xa | Sinh nhiều box chồng nhãn (#133, #135, #137, #138) (YOLO26) | Cụm ≥ 5 vật cao < 20 px: xoá box máy trong cụm, vẽ một `ignore_region`; vật cao ≥ 20 px và tách rõ giữ box | Mục 4.1.6: thêm bước xoá box máy trước khi vẽ `ignore_region` (CAL-05) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2, 3, 7 của mục 6; làm rõ cách đo chiều dài xe (4.1.5); thêm "không phủ `ignore_region` lên vật đã gán" (mục 2); thêm bước xoá box máy trong cụm nhỏ (4.1.6); `conf` chỉ để xếp thứ tự duyệt | CAL-01 đến CAL-05 |
