# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603217 - Dương Quang Chung

Phần bổ sung cho v1.0. Các quy tắc hiện có vẫn áp dụng. Quyết định được soạn bởi Codex theo yêu cầu học viên, cần được học viên xác nhận khi nghiệm thu.

## 1. Câu hỏi mở đã chốt và phạm vi

| Câu hỏi | Quyết định vận hành | Ảnh ví dụ | case_id |
|---|---|---|---|
| Hàng nhô trên xe đạp | Giỏ và khung gắn cố định thuộc xe. Khi xét phần hàng nhô quá khoảng 20% chiều dài xe theo v1.0, dùng phần xe nhìn thấy làm mốc, không lấy chiều dài của chính kiện hàng. Nếu không xác định được giỏ cố định hay hàng rời, tick can_review và ghi ca. | img_031 | CAL-02 |

Đây là cách làm rõ mốc đo cho box detection. Chưa chốt cách mask hàng rời vì ba thử nghiệm SAM chưa có vật chở hàng phù hợp.
Câu hỏi xe đạp điện, cụm xe máy gần camera và người nhìn xuyên kính vẫn chưa có đủ ví dụ để chốt ngưỡng mới. Chưa xác định chúng là nguyên nhân chặn xử lý trong năm ảnh QC này. Khi gặp ca chưa xử lý được, đánh dấu can_review và giữ lại để làm rõ.

## 2. Ca khó và checklist bổ sung

| Ảnh | Bằng chứng | Quyết định | Quan hệ với v1.0 | case_id |
|---|---|---|---|---|
| img_003 | Box release #12 và #13 cùng bao một xe ba bánh | Trong vòng rework giữ một box toàn xe, bỏ box trùng. Rà xem các box nhỏ có phải một phần của cùng xe hay cá thể khác. | Áp dụng nguyên tắc một vật một nhãn ở mục 2, không đổi lớp. | CAL-01 |
| img_023 | Ba xe đã có ba box three_wheeler, đáp án bẫy dùng box lớn phủ nhiều xe | Giữ các cá thể tách biệt và ghi chênh lệch với box chấm. Không gộp nhiều xe để tăng điểm bẫy. | Áp dụng mục 2; đây là giới hạn của phép chấm IoU với box máy. | CAL-03 |
| img_014 | Box #134 cắt phần gương và tay lái nhìn thấy | Rà bốn cạnh với tất cả bộ phận gắn liền, gồm gương, tay lái, bánh xe; chỉnh box trong vòng rework. | Làm rõ mục 4.1.1 về phần nhìn thấy của vật. | CAL-04 |

Các quyết định CAL-01, CAL-03 và CAL-04 không đổi taxonomy hay ngưỡng đánh giá. Chúng làm rõ cách áp dụng và kiểm tra quy tắc hiện có. Các lỗi QC chưa được áp bản sửa qc_fixed trong release candidate hiện tại.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Ca liên quan |
|---|---|---|
| v1.0 | Bản giảng viên | Guideline gốc giữ nguyên |
| v1.1 | Làm rõ mốc chiều dài để xét hàng nhô; bổ sung kiểm tra box trùng, cá thể và gương xe | CAL-01 đến CAL-04 |

Ảnh ví dụ có trong data/images/img_003.jpg, img_014.jpg, img_023.jpg và img_031.jpg. Overlay có ID của các ca QC nằm trong work/qc/review/.

## 4. Ảnh ví dụ

![Ca calibration img_003](../../data/images/img_003.jpg)

![Ca calibration img_014](../../data/images/img_014.jpg)

![Ca calibration img_023](../../data/images/img_023.jpg)

![Ca calibration img_031](../../data/images/img_031.jpg)
