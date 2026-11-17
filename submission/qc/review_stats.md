# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 454; đối tượng sau duyệt: 518; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 335 | 335/454 = 73.8% | 454 box máy |
| edited (sửa) | 60 | 60/454 = 13.2% | 454 box máy |
| deleted (xoá) | 59 | 59/454 = 13.0% | 454 box máy |
| added (vẽ thêm) | 123 | 123/518 = 23.7% | 518 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 114.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 193 | 20 | 12 | 54 |
| bicycle | 3 | 2 | 1 | 6 |
| motorbike | 81 | 27 | 17 | 35 |
| car | 41 | 3 | 4 | 19 |
| bus | 3 | 0 | 0 | 1 |
| truck | 2 | 0 | 2 | 2 |
| three_wheeler | 12 | 8 | 23 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 68 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 42 | 23 | 26 | 64.6% (KTC 95%: 52–75%) | 61.8% (KTC 95%: 50–72%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - missing: 3/3
  - extra: 3/3
  - wrong_class: 0/3
  - lọt: img_017.jpg (wrong_class), img_031.jpg (wrong_class), img_024.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
