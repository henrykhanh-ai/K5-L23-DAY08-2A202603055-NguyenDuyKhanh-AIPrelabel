# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | <<YOLO26m>> | <<theo lớp, từ threshold_sweep>> | <<>> |
| three_wheeler | <<YOLOE-26s, prompt: ...>> | <<>> | <<>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk) | SAM 2.1 (`sam2.1_b.pt`) | 4 dương + 0 âm | có: mask gồm cả người lái và người phía ngoài bị lem vào | mask phủ khoảng 90% xe + người; bị lem ra ngoài, dính thêm phần ghế, bàn và người phía ngoài; theo đề xuất v1.1, người lái bị tô chung với xe là lỗi | `tuktuk_img013.jpg` |
| img_013 (tuk-tuk, chạy lại thêm click âm) | SAM 2.1 (`sam2.1_b.pt`) | 4 dương + 4 âm | có: người lái và hàng trong xe vẫn bị tô cùng một polygon với xe | 4 click âm (2 người, 1 bàn, 1 ghế bên ngoài) làm mask không còn dính các vật bên ngoài; người + hàng trong xe vẫn nằm trong mask; theo đề xuất v1.1, người lái bị tô chung với xe là lỗi | `tuktuk_img013_neg.jpg` |
| img_017 (tuk-tuk) | SAM 2.1 (`sam2.1_b.pt`) | 1 dương + 0 âm | không áp dụng: xe không có người lái và không có hàng; không lem người bên ngoài | mask phủ toàn xe tuk-tuk gồm cả mui; không bị lem | `xe3banh_img017.jpg` |
| img_010 (xích lô) | SAM 2.1 (`sam2.1_b.pt`) | 3 dương + 0 âm | không tô người lái | mask phủ toàn xe; không tô người lái; không tô biển số xe | `xichlo_img010.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu?
  - **Quy tắc giúp được:** (4) hai vật chạm nhau: ở img_013, sau khi thêm 4 click âm thì 2 người, bàn và ghế bên ngoài không còn dính mask; với người và hàng trong xe, click thuộc xe nên xe được coi là vật phía trước và người lái được tô trong vùng chạm, cùng một polygon. (2) lỗ thủng: ở img_010, khung xích lô có nhiều khoảng hở (thanh sắt, nan bánh xe), mask nên tô kín và SAM đã tô kín. (3) bóng đổ / kính xe: mask không ăn vào bóng đổ dưới gầm xe; kính xe thuộc vật nên có mask.
  - **Quy tắc thiếu / mơ hồ:** mục 4.2 không nói người lái ngồi trên xe có thuộc mask xe ba bánh không (quan sát: ở img_013 SAM tô người lái chung mask với xe, không thành polygon riêng; ở img_010 SAM không tô người lái); cũng không nói rõ người ngồi *bên trong* xe thì vật nào là "vật phía trước" ở vùng chạm. Biển số thuộc xe nhưng ở img_010 không được mask, chỉ bị lem; guideline chỉ nhắc kính xe, không nhắc biển số / phụ kiện gắn xe. Ngưỡng "lệch biên ≤ ~3 px" chỉ kiểm tra được bằng mắt trên ảnh overlay khoảng 50%.
  - **Đề xuất cho v1.1:** người lái là vật riêng (polygon `person` riêng); phần người lái bị xe che thì thuộc xe; tách biệt rõ ràng để không nhầm lẫn với mask của các vật khác như người đi bộ.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>

### Danh sách ca `can_review` và `ignore_region` (trích từ nhãn đã export)

| File | Ảnh | Box | Lớp / loại | Vì sao không chắc quy tắc (bạn điền) |
|---|---|---|---|---|
| gold_manual.json | img_020 | #43 | person, `can_review` | <<>> |
| gold_manual.json | img_036 | #55 | person, `can_review` | <<>> |
| gold_manual.json | img_036 | #60 | person, `can_review` | <<>> |
| gold_manual.json | img_036 | #68 | person, `can_review` | <<>> |
| batch_reviewed.json | img_024 | CVAT #4298 | car, `can_review` | <<>> |
| gold_manual.json | img_001 | 3 vùng | `ignore_region` | <<>> |
| gold_manual.json | img_036 | 6 vùng | `ignore_region` | <<>> |
| batch_reviewed.json | img_006, img_018, img_024, img_026 | mỗi ảnh 2 vùng | `ignore_region` | <<>> |
