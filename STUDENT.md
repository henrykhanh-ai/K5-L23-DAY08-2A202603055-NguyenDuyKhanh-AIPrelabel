# Thông tin học viên

- Họ và tên: Nguyễn Duy Khánh
- MSHV: 2A202603055
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: henrykhanh-ai
- Repo bài nộp (URL): https://github.com/henrykhanh-ai/K5-L23-DAY08-2A202603055-NguyenDuyKhanh-AIPrelabel
- Commit chốt bài (hash): 5771860
- Máy: Windows (x64), không có GPU (PyTorch CPU 2.15.0)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 335/454 = 73.8% / 60/454 = 13.2% / 59/454 = 13.0% (mẫu số: 454 box máy); added: 123/518 = 23.7% (mẫu số: 518 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 64.6% / 61.8% (mẫu số: 68 đối tượng gold, 4 ảnh)
- Bắt bẫy: 9/12 = 75.0%
- Thời gian: gán tay 6.3 phút/ảnh (25 phút / 4 ảnh); duyệt có pre-label 6.1 phút/ảnh (147 phút / 24 ảnh, có thể ghi thiếu)
- Lỗi QC: 16.7% theo ảnh (1/6), 0.6% theo đối tượng (1/154)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 1/6 = 16.7% > 2% và bắt bẫy 9/12 = 75% < 80%.
