# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 434; đối tượng sau duyệt: 520; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 376 | 376/434 = 86.6% | 434 box máy |
| edited (sửa) | 24 | 24/434 = 5.5% | 434 box máy |
| deleted (xoá) | 32 | 32/434 = 7.4% | 434 box máy |
| added (vẽ thêm) | 114 | 114/520 = 21.9% | 520 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 105.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 236 | 8 | 5 | 43 |
| bicycle | 6 | 0 | 3 | 4 |
| motorbike | 81 | 7 | 2 | 47 |
| car | 44 | 1 | 4 | 8 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 4 | 2 |
| three_wheeler | 5 | 8 | 13 | 8 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 64 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 44 | 19 | 20 | 69.8% (KTC 95%: 58–80%) | 68.8% (KTC 95%: 57–79%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - loose: 1/3
  - wrong_class: 3/3
  - extra: 3/3
  - lọt: img_003.jpg (loose), img_006.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
