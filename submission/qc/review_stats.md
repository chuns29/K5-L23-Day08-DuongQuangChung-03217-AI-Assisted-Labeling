# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 431; đối tượng sau duyệt: 457; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 377 | 377/431 = 87.5% | 431 box máy |
| edited (sửa) | 18 | 18/431 = 4.2% | 431 box máy |
| deleted (xoá) | 36 | 36/431 = 8.4% | 431 box máy |
| added (vẽ thêm) | 62 | 62/457 = 13.6% | 457 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 56.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 243 | 2 | 7 | 18 |
| bicycle | 5 | 0 | 4 | 2 |
| motorbike | 74 | 13 | 2 | 24 |
| car | 40 | 1 | 9 | 4 |
| bus | 2 | 0 | 1 | 2 |
| truck | 3 | 0 | 3 | 3 |
| three_wheeler | 10 | 2 | 10 | 9 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 57 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 45 | 17 | 12 | 72.6% (KTC 95%: 60–82%) | 78.9% (KTC 95%: 67–88%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - wrong_class: 2/3
  - loose: 3/3
  - extra: 3/3
  - lọt: img_023.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
