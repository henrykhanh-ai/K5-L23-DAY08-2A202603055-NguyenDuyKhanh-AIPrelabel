# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 154 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 2/154 = 1.3% | 154 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/6 = 33.3% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 335/454 = 73.8% / 60/454 = 13.2% / 59/454 = 13.0% | 454 box máy |
| Máy: added | 123/518 = 23.7% | 518 đối tượng sau duyệt |
| Bắt lỗi bẫy | 9/12 = 75.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 1 | img_013.jpg |
| sai_lop | 1 | img_024.jpg |

## Giới hạn thống kê
- 2 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **70%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 33.3% > 2%
- bắt bẫy 75% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
