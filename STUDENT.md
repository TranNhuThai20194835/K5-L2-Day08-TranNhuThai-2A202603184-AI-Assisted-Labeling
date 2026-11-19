# Thông tin học viên

- Họ và tên: Trần Như Thái  
- MSHV: 2A202603184 
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: TranNhuThai20194835 
- Repo bài nộp (URL): https://github.com/TranNhuThai20194835/K5-L2-Day08-TranNhuThai-2A202603184-AI-Assisted-Labeling.git
- Commit chốt bài (hash): 'Hoan thanh bai lab Day08'
- Máy: Window, GPU
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 376 / 24 / 32 (mẫu số: 434 box máy; tức 86.6% / 5.5% / 7.4%); added: 114 (mẫu số: 520 đối tượng sau duyệt; tức 21.9%)
- So với gold: precision / recall của **máy** 69.8% / 68.8% (KTC 95%: 58–80% / 57–79%; mẫu số recall: 64 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 10.5 phút/ảnh (42 phút / 4 ảnh); duyệt có pre-label 12.5 phút/ảnh (300 phút / 24 ảnh)
- Lỗi QC: 40% theo ảnh (2/5), 2.0% theo đối tượng (2/98)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 2/5 = 40% vượt ngưỡng 2% và lỗi box lỏng lặp ở cả hai ảnh (img_003 #19, img_006 #75), dù bắt bẫy 10/12 = 83.3% đạt ngưỡng.
