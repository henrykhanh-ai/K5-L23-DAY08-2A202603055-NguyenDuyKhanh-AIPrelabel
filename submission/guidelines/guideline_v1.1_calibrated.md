# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603055 - Nguyễn Duy Khánh

> Chỉ ghi phần thay đổi so với v1.0. Mỗi ca khó có ảnh ví dụ và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | `motorbike` | Không có ảnh phù hợp trong bộ dữ liệu; mô tả: xe máy điện / xe đạp điện dáng scooter (có yên, sàn để chân, bánh nhỏ, không đạp) gán `motorbike` (không đưa ảnh ngoài bộ dữ liệu vào repo, theo RULES.md) | CAL-04 |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | `ignore_region` | img_018, img_024 | CAL-05 |
| 3 | Hàng nhô ra: box / mask xe có gồm hàng không? | Hàng kèm trong xe thì box / mask chung với xe; không tính hàng nếu nhô ra quá 20% chiều dài xe (giữ mục 4.1 quy tắc 5 của v1.0, áp dụng cả cho mask) | img_013 | CAL-06 |
| 4 | Người ngồi trong xe ba bánh có mui che | Person tách riêng thành 1 lớp (box / mask riêng); ngưỡng "còn thấy ≥ 20%" giữ như v1.0 | img_013 | CAL-07 |
| 5 | Kính chắn gió / cửa sổ xe: thuộc mask xe hay không? | Kính xe thuộc vật nên có mask (giữ mục 4.2 quy tắc 3) | img_013, img_017 | CAL-02 |
| 6 | Box máy chồng nhau quá nhiều trong ảnh đêm: gộp hay giữ? | Gán lại từng box | Không có ảnh phù hợp trong bộ dữ liệu; mô tả: đám đông ban đêm dưới đèn đường, người đứng sát nhau nên box máy chồng lên nhau, mỗi người phải gán lại một box riêng (không đưa ảnh ngoài bộ dữ liệu vào repo, theo RULES.md) | CAL-08 |
| 7 | Mức tin cậy tối thiểu để chấp nhận box máy không cần sửa? | 80% (`conf` ≥ 0.80); chỉ là ngưỡng tin cậy, box vẫn phải duyệt theo checklist | Không có ảnh phù hợp trong bộ dữ liệu; mô tả: đường phố lúc chạng vạng, máy vẽ nhiều box `car`: các box điểm > 0.80 (0.81, 0.82, 0.86, 0.91, 0.92) ôm đúng xe nên được chấp nhận; các box điểm < 0.80 (0.78, 0.30) ở cụm xe xa, chồng lên nhau phải duyệt lại từng box (không đưa ảnh ngoài bộ dữ liệu vào repo, theo RULES.md) | CAL-09 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_010 | Người lái ngồi trên xe ba bánh, chạm / bị xe che một phần | SAM 2.1 (`sam2.1_b`): img_013 tô người lái chung mask với xe; img_010 không tô người lái | Người lái là vật riêng; phần bị xe che thuộc xe | **Mục 4.2, quy tắc 5 (mới):** người lái xe ba bánh là vật riêng (polygon `person` riêng), không tô chung vào mask `three_wheeler`; phần người lái bị xe che thì thuộc xe; tách biệt rõ ràng để không nhầm với mask người đi bộ (CAL-01) |
| img_002, img_032 | Tuk-tuk / xe ba bánh to ở giữa ảnh | YOLO26 không nhận ra tuk-tuk; img_002 gọi một xe ba bánh là `truck`; ngưỡng 0.5 gán nhầm thành `motorbike` | Dùng YOLO26 cho person / motorbike / car; không dùng cho xe ba bánh; dùng YOLOE cho `three_wheeler` | Không đổi định nghĩa lớp (mục 2 v1.0 đã đủ): box máy gọi `truck` / `motorbike` cho tuk-tuk, xích lô, ba gác phải đổi thành `three_wheeler` (CAL-03) |
| img_006, img_018, img_024, img_026 | Xe nhỏ hoặc người nhỏ chồng lên nhau, mờ nhòe | — | Đặt `ignore_region` | **Mục 4 (bổ sung):** vật nhỏ chồng lên nhau, mờ nhòe dùng `ignore_region` (CAL-10) |
| img_024 | Ô tô di chuyển nhanh, không thấy rõ (box tick `can_review`) | — | Gán `ignore_region` | Cùng quy tắc CAL-10 (CAL-11) |
| img_020 (gold) | Box person #43 tick `can_review`: không chắc có nên gán, ngưỡng nhìn thấy bao nhiêu pixel | — | Gán `person` vì box cao 83 px, vượt xa 20 px, cả chiều cao và diện tích nhìn thấy | Không đổi guideline: mục 4.1 quy tắc 2 và 6 đã đủ (CAL-13) |
| img_036 (gold) | 3 box person #55, #60, #68 ở xa, mờ (cao 22–34 px), tick `can_review` | — | Đổi sang `ignore_region` theo CAL-10; gold đã nộp giữ nguyên theo v1.0 | Cùng quy tắc CAL-10 (CAL-14) |
| img_017, img_031, img_024 | 3 bẫy sai lớp lọt (person thành bicycle; car thành truck) | (bẫy cài vào pre-label) | Lọt do kiểm tra không kỹ; đã sửa lại | Không đổi guideline: lỗi người duyệt, không phải guideline mơ hồ (CAL-12) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt cả 7 câu hỏi mở của mục 6; thêm mục 4.2 quy tắc 5 (người lái là vật riêng); bổ sung mục 4 (vật nhỏ chồng nhau, mờ nhòe dùng `ignore_region`); ghi ca máy gọi xe ba bánh là truck / motorbike | CAL-01 → CAL-14 |
