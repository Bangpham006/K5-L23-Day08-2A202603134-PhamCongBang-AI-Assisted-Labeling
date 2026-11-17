# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603134 - Phạm Công Bằng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Người lái và người ngồi sau xe máy/xe đạp có vẽ gộp với xe không? | Tách riêng: người ngồi trên xe gán nhãn `person` riêng (tính từ đỉnh đầu/mũ bảo hiểm đến chân), xe gán `motorbike` hoặc `bicycle` riêng. Nếu người bị che khuất > 85% chỉ thấy chỏm đầu thì không vẽ `person`. | img_004.jpg, img_018.jpg | CAL-02 |
| 2 | Xe đạp điện, xe máy điện xếp vào đâu? | Nếu có bàn đạp cơ và dáng thanh mảnh xếp vào `bicycle`; nếu dáng xe tay ga không bàn đạp xếp vào `motorbike`. | img_011.jpg | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002.jpg, img_013.jpg | Xe tuk-tuk, xe ba gác có mui che hoặc thùng chở hàng phía sau | YOLO26m nhận nhầm thành `car` hoặc `truck` | Luôn gán `three_wheeler` cho tất cả phương tiện 3 bánh chở người hoặc chở hàng | Mục 2: Bổ sung định nghĩa `three_wheeler` bao gồm tuk-tuk có cabin kín và xe ba gác chở vật liệu |
| img_018.jpg, img_021.jpg | Bãi đỗ xe hoặc cụm xe máy ở hậu cảnh xa kích thước < 20px | Máy vẽ sót hoặc tạo box vụn không chính xác | Không vẽ riêng lẻ từng xe nhỏ li ti; vẽ 1 box lớn bao quanh cụm và gán `ignore_region` | Mục 4: Quy tắc gán nhãn `ignore_region` cho các cụm phương tiện dày đặc có kích thước < 20px |
| img_035.jpg | Xe máy bị che khuất một nửa bởi biển hiệu và xe khác | Máy tạo box lỏng bao cả khoảng trống nền | Siết sát mép nhìn thấy thực tế của xe máy | Mục 3: Bounding box phải ôm sát các phần nhìn thấy của xe, không bao khoảng trống xung quanh |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên ban đầu | Bản cơ sở chuẩn |
| v1.1 | Chốt quy tắc tách người/xe máy; phân loại xe 3 bánh có mui; bổ sung hướng dẫn dùng `ignore_region` cho cụm xe nhỏ < 20px ở hậu cảnh | Khắc phục lỗi nhầm lẫn của model AI và làm rõ các ca phân vân (CAL-01, CAL-02, CAL-03) |
