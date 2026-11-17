# Báo cáo Day 08 — 2A202603134 - Phạm Công Bằng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.3 (mặc định) | Dẫn `r01` và `r02`: ở ngưỡng 0.25 phát hiện nhiều box thừa ở mép đường (215 box), còn ở ngưỡng 0.5 bị tụt recall các vật nhỏ ở xa (139 box). Ngưỡng 0.3 đạt cân bằng F1 tốt nhất cho tập danh mục COCO. |
| three_wheeler | YOLOE-26s, prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle' | 0.15 | Dẫn `r03`: YOLOE tìm chính xác 8 xe ba bánh ở tập pilot mà YOLO26 bỏ qua hoặc nhận nhầm. Ngưỡng 0.15 quét từ threshold sweep đạt recall 100% trên tập gold. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1** (`sam2.1_b.pt`). Lý do: SAM 2.1 cho chất lượng polygon mask vượt trội so với MobileSAM (đường biên bám rất mịn vào thân xe và người, không bị gãy góc ở nan hoa xe), thời gian xử lý khoảng 4.66s/ảnh trên CPU là chấp nhận được, và không yêu cầu đăng nhập token/quyền truy cập phức tạp như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi phân loại sai bản chất (semantic misclassification) do detector bị giới hạn trong từ vựng đóng (closed-vocabulary của COCO không có nhãn xe 3 bánh). Đổi ngưỡng chỉ làm xuất hiện hoặc biến mất box chứ không thể đổi tên nhãn. Cách sửa: Dùng mô hình open-vocabulary (YOLOE) với prompt gõ chữ riêng cho `three_wheeler`, sau đó hợp nhất (merge) với quy tắc ưu tiên box của YOLOE đè lên box `car`/`truck` của YOLO26.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Ở ảnh `img_002.jpg`, YOLO26 nhận diện chiếc xe tuk-tuk với độ tin cậy rất cao (conf 0.82) nhưng lại gán nhãn sai thành `car`.
  - Điểm thấp mà đúng: Ở ảnh `img_032.jpg`, YOLOE phát hiện chiếc xe ba gác chở đồ cồng kềnh với conf chỉ 0.18 nhưng vị trí và nhãn `three_wheeler` hoàn toàn chính xác.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Do tập gold chỉ có 4 ảnh và việc tìm ngưỡng tối ưu thực hiện in-sample (chọn và chấm trên cùng tập dữ liệu), nên các chỉ số precision/recall của máy trên gold mang tính lạc quan; khi triển khai trên dữ liệu thực tế lớn hơn sẽ có độ lệch.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 512 | 512/519 = 98.7% | 519 box máy |
| edited | 7 | 7/519 = 1.3% | 519 box máy |
| deleted | 0 | 0/519 = 0.0% | 519 box máy |
| added | 0 | 0/519 = 0.0% | 519 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` và `person` ở xa là kém nhất. Ở `img_002.jpg` và `img_013.jpg`, máy nhầm lẫn giữa tuk-tuk và ô tô con. Ở `img_018.jpg`, người đi bộ ở xa bị che khuất một phần nên máy bỏ sót bounding box.
- So với gold: precision / recall của **máy** Precision 30/57 = 52.6% (KTC 95%: 40–65%) / Recall 30/41 = 73.2% (KTC 95%: 58–84%) (mẫu số: 41 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3% (KTC 95%: 55–95%); loại nào lọt: lọt 1 bẫy `loose` ở ảnh `img_035.jpg` (box xe máy bị rộng một khoảng phía sau) và 1 bẫy `missing` ở ảnh `img_027.jpg` (mất 1 xe máy nhỏ 15x24px ở hậu cảnh xa); vì sao: Do người duyệt bị mỏi mắt và lướt qua nhanh ở các ảnh có quá nhiều chi tiết, bỏ qua bước 0 của checklist (không tắt nhãn máy để soi ảnh thô trước).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ Accepted cao (98.7%) đi kèm với việc vẫn để lọt 2 bẫy phản ánh rõ rệt hiện tượng thiên kiến tự động hóa (automation bias): người duyệt có xu hướng tin tưởng vào các đề xuất trực quan của máy, dễ dãi bấm Accept mà không kiểm tra kỹ các góc khuất hoặc viền mép đối tượng.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 25 phút / 4 ảnh = 6.25 phút/ảnh vs duyệt 68 phút / 24 ảnh = 2.83 phút/ảnh. Tiết kiệm hơn 54% thời gian mỗi ảnh. Thiên lệch: Giai đoạn gán gold làm trước nên tốn thời gian đọc hiểu quy tắc guideline; khi chuyển sang duyệt 24 ảnh thì thao tác đã thuần thục hơn.
- Kết luận: RẤT ĐÁNG DÙNG. Tiết kiệm được hơn một nửa thời gian làm việc, đặc biệt hiệu quả với các đối tượng phổ biến (`person`, `motorbike`, `car`). Tuy nhiên bắt buộc phải có quy trình kiểm soát bẫy để chống thiên kiến.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quy tắc 15 giây soi ảnh thô (Checklist Step 0)*: Trước khi bật lớp nhãn máy trên CVAT, luôn tắt nhãn và dành 15 giây quét bằng mắt để tìm các vật thể nhỏ/bị che khuất ở hậu cảnh.
  2. *Quy tắc Chèn bẫy mù kiểm định*: Luôn trộn tối thiểu 5–10% ảnh bẫy vào quy trình làm việc; nếu kết quả bắt bẫy dưới 80% thì yêu cầu duyệt lại toàn bộ lô ảnh.
  3. *Quy tắc Giới hạn phiên duyệt (Pomodoro 25 phút)*: Cứ sau 8–10 ảnh (khoảng 25 phút) phải nghỉ giải lao 3–5 phút để mắt phục hồi độ nhạy, tránh tình trạng bấm Accept hàng loạt.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 2 click (1 pos, 1 neg) | Tách riêng được thân xe tuk-tuk, không ăn sang người lái | Cần đặt thêm click âm ở phần kính chắn gió | tuktuk_img013.jpg |
| img_003.jpg | SAM 2.1 (`sam2.1_b.pt`) | 2 click (1 pos, 1 neg) | Bao phủ trọn vẹn thân và bánh xe ba gác | Mask lan nhẹ xuống vệt bóng đen dưới mặt đường | vehicle_img003.jpg |
| img_004.jpg | SAM 2.1 (`sam2.1_b.pt`) | 2 click (1 pos, 1 neg) | Phân tách tốt người và xe đạp | Phần nan hoa bánh xe còn một vài khoảng hở nhỏ | person_bike_img004.jpg |

- Đối chiếu guideline mục 4.2: Quy tắc 4.2 yêu cầu phân tách độc lập giữa người và phương tiện. Khi dùng SAM với prompt điểm, cần tối thiểu 1 điểm âm (negative point) đặt vào người lái để mask không tự động gộp cả hai thành một khối.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Chạy offline bằng script `sam2_points.py` giúp chủ động chọn tọa độ click chính xác trên pixel ảnh gốc và xuất mask ổn định, không bị phụ thuộc vào kết nối mạng hay dịch vụ serverless Nuclio của CVAT.

## 6. Calibration (Part 6)

- Số quyết định: 3 quyết định (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): Quyết định `CAL-01` (chốt mọi phương tiện 3 bánh có mui che hoặc thùng hàng đều là `three_wheeler`).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung chi tiết nhận diện `three_wheeler`, hướng dẫn tách riêng `person` và `motorbike`, và quy định dùng nhãn `ignore_region` cho các cụm phương tiện nhỏ hơn 20px ở xa.
- Nguyên nhân gốc: Kết hợp giữa giới hạn từ vựng đóng của mô hình AI (lỗi máy) và các trường hợp biên chưa được làm rõ trong bản guideline v1.0.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh ca khó `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: Phạm Công Bằng (hạn chế: vừa là người duyệt vừa là người tự kiểm định chất lượng nội bộ).
- Lỗi theo **đối tượng**: 0/191 = 0.0% · theo **ảnh**: 0/8 = 0.0% (ngưỡng Release ≤ 2%).
- Top lỗi: Không ghi nhận lỗi sai sót nào trong mẫu kiểm định 8 ảnh ngẫu nhiên.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 ảnh lỗi trong 8 ảnh mẫu, cận trên khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi thật lên tới 32%. Ngưỡng 2% không thể được chứng minh tuyệt đối bằng mẫu nhỏ như vậy; đây là hạn chế thống kê cần lưu ý.
- Script kiểm được / không kiểm được: Script kiểm tra được tính toàn vẹn cú pháp COCO, nhãn hợp lệ trong taxonomy, độ phủ của box; không kiểm tra được độ bám viền pixel thực tế và các đối tượng mà cả người lẫn máy cùng bỏ sót.

**Quyết định: RELEASE**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release):
1. Không có ca vướng mắc guideline chưa chốt và không có ảnh lỗi dữ liệu -> Không HOLD.
2. Tỉ lệ ảnh lỗi trong mẫu là 0.0% <= 2%, không có lỗi hệ thống, tỉ lệ bắt bẫy đạt 83.3% >= 80% -> Không REWORK.
3. Thỏa mãn đầy đủ các tiêu chuẩn kiểm định nghiệm thu dữ liệu -> **RELEASE**.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Antigravity làm trợ lý chạy tự động các câu lệnh script, kiểm thử validation và tính toán các chỉ số thống kê. Mọi nhãn gán và quyết định đều được kiểm tra kỹ lưỡng.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không bị nhiễm. 4 ảnh gold được tự gán tay hoàn toàn trên CVAT trước khi chạy bất kỳ mô hình AI nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce RTX 3050 Laptop GPU (4GB VRAM), CVAT Docker v2.74.1. Cài đặt diễn ra thuận lợi, không gặp trục trặc lớn.
- Dùng cache `data/cache/` thay vì tự chạy? Có dùng file kết quả chạy sẵn trong `data/cache/` của 36 ảnh ở ngưỡng 0.05 để tiết kiệm thời gian suy luận toàn bộ dataset.
- Lỗi / bug script đã gặp (nếu có): Không gặp lỗi trong mã nguồn đề bài.
- Một điều bạn làm khác lần sau: Sẽ chú ý phóng to (zoom in) kỹ hơn ở các khu vực rìa hậu cảnh của các bức ảnh góc rộng để không bị lọt các bẫy chi tiết nhỏ như ở `img_027`.
