# Báo cáo Day 08 — 2A202603055 - Nguyễn Duy Khánh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26s (`--size s`, máy CPU) | 0.3 mặc định cho mọi lớp (threshold_sweep: không lớp nào đạt recall 90% ở ngưỡng ≤ 0.5; bus, truck < 5 vật trong gold) | r01: nhận diện đầy đủ người đi bộ, car, motorbike, truck; phân tách rõ người trên xe và xe máy. r02 (ngưỡng 0.5): box người giảm là những box người thật, nên chọn ngưỡng 0.25 / mặc định thấp. Quyết định trong model_log: dùng YOLO26 cho person / motorbike / car; không dùng cho xe ba bánh. Cỡ `s` vì máy chỉ có CPU (r01: 0.47 s/ảnh) |
| three_wheeler | YOLOE-26s, prompt: "tuk-tuk", "cyclo", "cargo tricycle" (`taxonomy.json`) | 0.15 (threshold_sweep: recall 100%, precision 78% trên 7 vật gold) | r03: tìm được xe ba bánh to ở giữa img_002 và img_032 mà r01 (YOLO26) không gán được. Quyết định trong model_log: dùng YOLOE cho three_wheeler. Đánh đổi của ngưỡng 0.15: gần như không sót xe ba bánh nhưng nhiều box thừa (batch: 23 box three_wheeler bị xoá, 12 giữ nguyên); với tôi vẽ 1 box bị sót nhanh hơn xoá box thừa, nên lần sau nên tăng ngưỡng three_wheeler, ví dụ lên 0.3. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`) vì mask khít vật, không tràn viền (r04; Part 5). SAM 2.1 chậm (3.89 s/ảnh trên CPU) và `mobile_sam` nhanh hơn, nhưng với 3 vật thì chất lượng mask quan trọng hơn tốc độ.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Ngưỡng chỉ quyết định việc giữ hay loại bỏ một hộp nhận diện (box), chứ không thể thay đổi tên lớp mà model dự đoán. YOLO26 được huấn luyện trên 80 lớp của COCO, mà không có lớp "xe ba bánh" (three_wheeler), nên dù chỉnh ngưỡng thế nào, chiếc tuk-tuk vẫn chỉ được gọi là `truck`, `car` hoặc `motorbike` (img_002: `truck`; r02: `motorbike`). Muốn nhận diện đúng loại xe đó, cách sửa là thay đổi model (dùng YOLOE cho `three_wheeler`), chứ không phải chỉnh ngưỡng (CAL-03).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: 1 box (điểm ≥ 0.7 bị xoá / đổi lớp, đã loại bẫy): img_003 `three_wheeler` điểm 0.80, box chỉ ôm phần dưới và nằm trọn trong box `three_wheeler` lớn hơn của cùng chiếc xe, đã xoá. Điểm thấp mà đúng: 28 box (điểm < 0.35, giữ nguyên IoU ≥ 0.9), ví dụ `three_wheeler` img_013 (0.18), img_022 (0.22), img_008 (0.25); `person` img_005, img_028, img_034 (0.30). Điểm số mà model đưa ra là độ tự tin của nó, chứ không phải là xác suất rằng hộp đó đúng hoàn toàn: model có thể rất tự tin với một hộp bị trùng hoặc lệch, hoặc ngược lại, có những hộp điểm thấp hơn 0.35 nhưng vẫn đúng (ví dụ xe ba bánh ở img_013 điểm chỉ 0.18 mà vẫn đúng).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: việc chọn ngưỡng rồi đánh giá lại trên cùng bộ ảnh dùng để chọn ngưỡng giống như tự đặt câu hỏi rồi tự chấm bài mình. Ngưỡng được chọn sao cho kết quả đẹp nhất trên 4 ảnh gold, nên đánh giá lại trên chính 4 ảnh đó thì kết quả chắc chắn rất tốt; khi lấy ảnh mới, kết quả thường kém hơn nhiều. Thêm nữa, chỉ có 7 vật thể xe ba bánh trong gold, nên một thay đổi nhỏ cũng làm recall lệch đến khoảng 14%. Cách chuẩn nhất là chọn ngưỡng trên tập ảnh riêng (ví dụ 8 ảnh pilot), rồi mới đánh giá trên bộ ảnh gold.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 335 | 73.8% | 454 box máy |
| edited | 60 | 13.2% | 454 box máy |
| deleted | 59 | 13.0% | 454 box máy |
| added | 123 | 23.7% | 518 đối tượng sau duyệt (≈ 114 vật máy sót thật, 9 trường hợp là đổi lớp) |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler`: số box bị xoá (23) gấp đôi số box được giữ nguyên (12). Lỗi nằm ở ngưỡng 0.15. Số liệu đối chiếu (so pre-label với nhãn cuối, IoU < 0.5 với mọi box three_wheeler cuối, đã loại bẫy): 20 box three_wheeler bị xoá, nhiều nhất ở img_003 (7 box), img_014 (3), img_009 (2), img_010 (2); 14/20 box bị xoá có điểm < 0.3, tức là sẽ bị loại nếu tăng ngưỡng lên 0.3.
- So với gold: precision / recall của **máy** 64.6% (KTC 95%: 52–75%) / 61.8% (KTC 95%: 50–72%); TP 42, FP 23, FN 26 (mẫu số: 68 đối tượng gold, 4 ảnh). Đã loại 23 đối tượng quanh bẫy khỏi bảng profile.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 9/12 = 75% (KTC 95%: 47–91%; ngưỡng đạt 80%); theo loại: loose 3/3, missing 3/3, extra 3/3, wrong_class 0/3. Loại nào lọt: cả 3 bẫy sai lớp: img_017 person thành bicycle (box rất nhỏ 23×22 px), img_031 person thành bicycle (box lớn 171×303 px), img_024 car thành truck (221×139 px); vì sao (bạn đã bỏ bước nào của checklist)? Mẫu lỗi: bắt 9/9 bẫy hình học (lỏng, sót, thừa) nhưng 0/3 bẫy sai lớp. img_017: box chỉ cao 22 px, nhãn chữ có đọc nhưng vì nhỏ và khuất nên không thấy. img_031: box person cao 303 px mang nhãn bicycle, lỗi khá lộ nhưng lọt vì bất cẩn; rút kinh nghiệm, lần sau kiểm tra kỹ hơn. img_024: phân biệt được car và truck nhưng bất cẩn; rút kinh nghiệm.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 335/454 = 73.8%, bắt bẫy 9/12 = 75% (< 80%). Đây là lần đầu tôi làm nên hơi bất cẩn, lần sau tỷ lệ sẽ cao hơn. Tôi không cảm thấy mình có thiên kiến tự động hoá: tôi không tin nhãn máy vì trông nó hợp lý, luôn làm theo checklist và tự gán nhãn, soi kỹ, luôn đối chiếu guideline và luôn tin rằng máy có sai sót. Dù tôi không cảm thấy mình tin máy, việc lọt cả 3 bẫy sai lớp cho thấy bước kiểm tra tên lớp của tôi chưa chặt; tôi sẽ cải thiện cho lần sau.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 25 phút / 4 ảnh = 6.3 phút/ảnh (gold, 77 đối tượng) vs duyệt có pre-label 147 phút / 24 ảnh = 6.1 phút/ảnh (batch, 538 đối tượng; giờ bắt đầu xác định lại sau khi làm nên 147 phút có thể thấp hơn thực tế). Thiên lệch (bạn gán gold trước nên đã quen cảnh): tôi gán gold trước nên đã quen cảnh.
- Kết luận: máy đáng dùng, sẽ giúp tăng tốc độ gán nhãn, tuy nhiên cần có người kiểm soát kỹ. Dựa trên: ảnh batch đông vật hơn (trung bình khoảng 22 vật/ảnh so với 19 ở gold) mà vẫn duyệt nhanh tương đương gán tay (6.1 so với 6.3 phút/ảnh), hơn nữa thời gian review có thể đã bị ghi thiếu; 335/454 = 73.8% box máy được giữ nguyên. Nhưng máy sót 123/518 = 23.7% vật, recall trên gold chỉ 61.8% (42/68), và lớp three_wheeler có số box bị xoá (23) gấp đôi số box được giữ (12): đây là lý do cần người kiểm soát kỹ.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Luôn bám sát guideline: mọi box `can_review` đều có một dòng trong bảng ca khó, kèm tên ảnh và quy tắc liên quan; kiểm tra bằng số ca `can_review` không ghi lý do = 0.
  2. Kiểm tra 2 vòng, vòng 1 chỉ kiểm tra tên lớp: bật hiển thị nhãn và đọc tên lớp của từng box, kể cả box nhỏ dưới 30 px; kiểm tra bằng tỉ lệ bắt bẫy sai lớp = 3/3 ở lần sau.
  3. Vòng 2 chỉ kiểm tra box: quét ảnh gốc trước khi bật nhãn máy để tìm vật sót, rồi soát box lỏng / thừa; kiểm tra bằng tỉ lệ bắt bẫy hình học giữ 9/9 và tổng bắt bẫy ≥ 80% (≥ 10/12).

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
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử (chỉ dùng `sam2_points.py` offline).

## 6. Calibration (Part 6)

- Số quyết định: 14 (`calibration_decisions.csv`: 8 `cau_hoi_mo` CAL-01, 02, 04–09; 1 `loi_may` CAL-03; 4 `can_review` CAL-10, 11, 13, 14; 1 `bay_lot` CAL-12); quan trọng nhất (dẫn `CAL-xx`): CAL-01 (người lái xe ba bánh là vật riêng; SAM tô người lái chung mask với xe ở img_013).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt đủ 7 câu hỏi mở của mục 6 v1.0 (CAL-02, 04–09); mục 4.2 quy tắc 5 mới: người lái xe ba bánh là vật riêng (CAL-01); mục 4 bổ sung: vật nhỏ chồng nhau, mờ nhòe dùng `ignore_region` (CAL-10).
- Nguyên nhân gốc: chủ yếu đến từ máy vẫn còn sai sót và người duyệt chưa có kinh nghiệm, cần thêm thời gian để chắc chắn cải thiện.

## 7. QC và quyết định (Part 6)

- Mẫu: 6 ảnh, 154 đối tượng (1 ảnh `can_review`: img_024 + 5 ngẫu nhiên: img_003, img_006, img_013, img_014, img_033; seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 2/154 = 1.3% · theo **ảnh**: 2/6 = 33.3% (ngưỡng Release ≤ 2%). Ghi chú: lúc QC bằng mắt tôi ghi img_024 là không lỗi; khi đối chiếu đáp án bẫy mới thấy box #304 car bị gán truck vẫn còn trong mẫu, nên đã sửa phiếu QC cho trung thực.
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: `bo_sot` 1 lỗi (img_013: bỏ sót xe bên cạnh, phía trước id 127); `sai_lop` 1 lỗi (img_024: box #304 car bị gán truck).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 2 ảnh lỗi trong 6 ảnh mẫu, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 70%; ngưỡng 2% không thể được chứng minh bằng mẫu 6 ảnh.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script kiểm được tên file, lớp có trong taxonomy, box nằm trong ảnh, đủ ảnh, các số đo (accepted / edited / deleted / added, bắt bẫy, tỉ lệ lỗi QC). Script không kiểm được box có ôm sát vật không, lớp có đúng thực tế không, vật mà cả người duyệt lẫn máy cùng bỏ sót (ví dụ xe bị sót ở img_013, chỉ phát hiện khi QC bằng mắt), và việc người duyệt có theo checklist thật hay không.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không Hold vì 7 câu hỏi mở đã được chốt ở v1.1; Rework vì ảnh có lỗi 2/6 = 33.3% > 2% và bắt bẫy 9/12 = 75% < 80%; dữ liệu release cần chính xác tuyệt đối theo metrics.
Nếu Rework / Hold, việc cần làm tiếp: sửa xe bị sót ở img_013; sửa 3 bẫy sai lớp (img_017, img_031, img_024) và car #4298 ở img_024 rồi xuất `qc_fixed.json`; duyệt lại cả lô; chạy lại `build_release.py` và `qc_sample.py --force`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude (claude.ai, kết nối máy tính qua Claude desktop app). Dùng Claude để tóm tắt nội dung, đưa ra trình tự công việc cần làm, hỗ trợ viết report và hỗ trợ phân tích lỗi để khắc phục cho lần sau. Cụ thể: Claude cài đặt / tạo Project và Task trên CVAT, nạp ảnh và pre-label, export nhãn; chạy các script đo và kiểm tra (không mở `traps_key.json` trước khi export `batch_reviewed.json`); trích số liệu từ file đầu ra; giải thích khái niệm; đặt câu hỏi và sắp xếp câu trả lời của tôi vào các file nộp (mục 1 phần lỗi không sửa được bằng ngưỡng, điểm cao mà sai / điểm thấp mà đúng, và giới hạn: tôi viết lại dựa trên giải thích khái niệm của Claude). Tôi tự làm: gán tay gold, review 24 ảnh batch, xem ảnh và nhận xét, đánh giá mask, mọi quyết định calibration, kiểm QC, quyết định Rework. Kiểm tra: tự mở ảnh kết quả đối chiếu với nhận xét, chạy `validate_labels` / `check_submission`, đọc lại từng câu trong REPORT.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không: gán gold 18:05–18:30 ngày 17/11, trước khi chạy model lần đầu ở Part 2; hoàn toàn không nhìn pre-label và không hỏi AI về 4 ảnh gold trước khi gán xong.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows x64, không GPU (PyTorch 2.15 CPU), Python 3.12; CVAT v2.74.1 (Docker). Vướng mắc: Docker Desktop chưa chạy lúc đầu; thiếu `transformers` (đã cài); không có `yolo26m.pt` nên dùng `--size s`; PowerShell mới mở mất biến `$Py`; file CSV bị khoá khi đang mở trong Excel.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy cả 6 lần (r01–r06).
- Lỗi / bug script đã gặp (nếu có): không gặp bug script; không sửa script nào.
- Một điều bạn làm khác lần sau: tập trung cải thiện để không bỏ sót box và kiểm tra kỹ tên lớp, đặc biệt với vật thể lộ rõ.

### Danh sách ca `can_review` và `ignore_region` (trích từ nhãn đã export)

| File | Ảnh | Box | Lớp / loại | Vì sao không chắc quy tắc / quyết định |
|---|---|---|---|---|
| gold_manual.json | img_020 | #43 | person, `can_review` | Không chắc có nên gán person và ngưỡng nhìn thấy bao nhiêu pixel; chốt gán person vì box cao 83 px (CAL-13) |
| gold_manual.json | img_036 | #55 | person, `can_review` | Ở xa, mờ, không thấy rõ; theo v1.1 là `ignore_region`, gold giữ nguyên (CAL-14) |
| gold_manual.json | img_036 | #60 | person, `can_review` | Như #55 (CAL-14) |
| gold_manual.json | img_036 | #68 | person, `can_review` | Như #55 (CAL-14) |
| batch_reviewed.json | img_024 | CVAT #4298 | car, `can_review` | Xe di chuyển nhanh, không thấy rõ vật thể; chốt `ignore_region` (CAL-11) |
| gold_manual.json | img_001 | 3 vùng | `ignore_region` | Vật quá nhỏ (cao < 20 px) theo cảnh báo của `validate_labels` (mục 4.1 quy tắc 6) |
| gold_manual.json | img_036 | 6 vùng | `ignore_region` | Vật quá nhỏ (cao < 20 px) theo cảnh báo của `validate_labels` (mục 4.1 quy tắc 6) |
| batch_reviewed.json | img_006, img_018, img_024, img_026 | mỗi ảnh 2 vùng | `ignore_region` | Xe / người nhỏ chồng nhau, mờ nhòe; khối xe máy dày (CAL-05, CAL-10) |
