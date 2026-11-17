# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 519; đối tượng sau duyệt: 519; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 512 | 512/519 = 98.7% | 519 box máy |
| edited (sửa) | 7 | 7/519 = 1.3% | 519 box máy |
| deleted (xoá) | 0 | 0/519 = 0.0% | 519 box máy |
| added (vẽ thêm) | 0 | 0/519 = 0.0% | 519 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 0.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 266 | 5 | 0 | 0 |
| bicycle | 8 | 0 | 0 | 0 |
| motorbike | 136 | 2 | 0 | 0 |
| car | 55 | 0 | 0 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 6 | 0 | 0 | 0 |
| three_wheeler | 38 | 0 | 0 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 41 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 30 | 27 | 11 | 52.6% (KTC 95%: 40–65%) | 73.2% (KTC 95%: 58–84%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - missing: 2/3
  - extra: 3/3
  - wrong_class: 3/3
  - lọt: img_035.jpg (loose), img_027.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
