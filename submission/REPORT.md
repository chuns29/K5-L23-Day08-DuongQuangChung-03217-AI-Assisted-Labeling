# Báo cáo Day 08 — 2A202603217 - Dương Quang Chung

Số đo lấy từ đầu ra script. QC và phần soạn thảo do Codex thực hiện theo yêu cầu học viên. Đây là bản đánh giá release candidate, quyết định hiện tại là Rework.

## 1. Model và recipe (Part 2–3)

| Lớp | Model | Ngưỡng dùng ở CP4 | Căn cứ |
|---|---|---|---|
| person | YOLO26m | 0.35 | Học viên chỉ định trong lệnh CP4 |
| motorbike | YOLO26m | 0.50 | Học viên chỉ định trong lệnh CP4 |
| car | YOLO26m | 0.35 | Học viên chỉ định trong lệnh CP4 |
| bicycle, bus, truck | YOLO26m | 0.30 | Ngưỡng mặc định |
| three_wheeler | YOLOE-26s; tuk-tuk, cyclo, cargo tricycle | 0.30 | Ngưỡng mặc định |

CP2 thử cấu hình CPU ở imgsz=640, gồm YOLO26n hai ngưỡng 0.25 và 0.50, YOLOE-26s và YOLO26n kết hợp SAM 2.1 tiny. Bốn dòng trong model_log.csv là r02 đến r05. CP3 chạy lại YOLO26m và YOLOE-26s ở imgsz=1024, conf=0.05 trên 36 ảnh. Các run CP3 được ghi trong work/runs.csv; hai tiến trình chạy đồng thời cùng nhận run_id r08, nên đối chiếu cả model, params và out để phân biệt.

Lệnh CP4 thực tế:
```powershell
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

threshold_sweep.md gợi ý motorbike=0.10 với recall 12/13 = 92.3%, three_wheeler=0.15 với recall 6/6 = 100%. CP4 dùng chuỗi ngưỡng riêng do học viên gửi, nên không được trình bày như đã dùng chuỗi tối ưu CP3. Trong sweep, person tối đa 21/24 = 87.5%, car tối đa 7/9 = 77.8%. Bicycle có 4 mẫu, bus 1, truck 0 nên dùng mặc định 0.3. Three_wheeler thực tế có 6 mẫu và đủ điều kiện chọn ngưỡng.

Chọn ngưỡng và đo trên cùng 4 ảnh gold là in-sample, có thể cho kết quả lạc quan. Mỗi lớp có ít đối tượng nên ngưỡng chỉ là ước lượng thô.
YOLO26n gọi xe ba bánh bên trái img_011 là truck ở conf=0.34; đổi ngưỡng không đổi được lớp đầu ra đó. YOLOE bổ sung lớp three_wheeler nhưng vẫn cần người phân biệt cá thể.
Ở img_002, YOLOE nhận tuk-tuk xanh với conf=0.97. Ở img_011, người áo hồng có conf=0.28 vẫn hiện hữu nhưng mất khi tăng conf lên 0.50.
SAM dùng sam2.1_t.pt trên CPU vì đây là trọng số đã có; mục 5 ghi kết quả thực tế và các giới hạn.

## 2. AI error profile

Nguồn qc/review_stats.json. Script loại các đối tượng nằm quanh bẫy khỏi bảng chất lượng máy.

| Trạng thái | Số đo | Mẫu số |
|---|---|---|
| accepted | 377/431 = 87.5% | 431 box máy |
| edited | 18/431 = 4.2% | 431 box máy |
| deleted | 36/431 = 8.4% | 431 box máy |
| added | 62/457 = 13.6% | 457 đối tượng sau duyệt trong phép so sánh đã loại vùng bẫy |

Có 6 trường hợp đổi lớp trong deleted và added, nên added trừ đổi lớp là khoảng 56 vật máy sót theo phép ghép của script. Không diễn giải đây là toàn bộ vật thật bị bỏ sót.
Three_wheeler có 10 accepted, 2 edited, 10 deleted và 9 added. Tỉ lệ deleted 10/22 = 45.5% box máy của lớp này cao nhất trong các lớp được thống kê; img_023 cho thấy ranh giới cá thể là một nguyên nhân phải rà lại.
So gold, máy có TP=45, FP=17, FN=12, precision 45/62 = 72.6%, recall 45/57 = 78.9%. Bốn ảnh gold có 61 annotation tổng cộng, phép đo detection dùng 57 đối tượng sau xử lý ignore của script. Release candidate chứa 468 đối tượng trên 24 ảnh batch; mẫu số 457 trong error profile khác tổng này vì loại vùng bẫy.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Lượt duyệt đầu bắt 6/12 = 50.0%, trong đó missing 0/3, wrong_class 2/3, loose 2/3 và extra 2/3. Học viên sửa theo phản hồi và tăng lên 9/12 = 75.0%. Codex sau đó chỉnh box xe máy ở img_035 và box người ở img_029 theo ảnh và đáp án đã được mở. Kết quả hiện tại là 11/12 = 91.7%, gồm missing 3/3, wrong_class 2/3, loose 3/3 và extra 3/3.
Ảnh img_023 vẫn bị tính lọt vì phép chấm so với box máy lớn phủ nhiều tuk-tuk, trong khi nhãn hiện tại tách ba xe thành ba box. Đã giữ các cá thể riêng, chưa sửa đáp án hay script để tăng điểm.

Accepted cao ở lượt đầu, 382/431 = 88.6%, đi cùng bắt bẫy 6/12 = 50.0%. Việc cả 3/3 bẫy missing bị lọt cho thấy lượt tìm vật chưa có box cần được củng cố. Chưa có log thao tác đủ để kết luận học viên đã bỏ bước nào.
Điểm sau khi xem phản hồi và có Codex sửa không còn đo riêng khả năng duyệt mù của học viên.

## 4. Máy có đáng dùng không?

Gold làm từ 17:40 đến 18:30, 50 phút cho 4 ảnh, trung bình 12.50 phút/ảnh. Lượt duyệt batch từ 18:58 đến 22:14, nghỉ 20:00–22:00, nên thời gian làm là 76 phút cho 24 ảnh, trung bình 3.17 phút/ảnh. Nguồn time_log.csv. Thời gian các lượt sửa bổ sung chưa được bấm giờ và chưa cộng vào log.

Theo hai lượt đầu, giảm thời gian mỗi ảnh là (12.50 - 3.17)/12.50 = 74.7%. Gold và batch khác độ khó, cỡ mẫu khác nhau, học viên đã quen cảnh sau khi gán gold và chưa tính rework, nên đây chỉ là so sánh mô tả.
Pre-label có ích như bản đề xuất cho người rà ảnh. Kết quả QC ở mục 7 yêu cầu tiếp tục sửa trước khi dùng release để huấn luyện hoặc bàn giao.

Ba quy tắc cho lượt duyệt tiếp:
1. Với đủ 24/24 ảnh, ẩn toàn bộ box và quét bốn góc trước khi hiện nhãn, ghi đã làm trong nhật ký từng ảnh.
2. Duyệt theo thứ tự thiếu, thừa, sai lớp, sai biên trên 24/24 ảnh, kiểm cả box điểm cao và mỗi cá thể có một box.
3. Sau mỗi tối đa 10 ảnh, nghỉ 3–5 phút và ghi thời gian; mỗi ca chưa rõ quy tắc có can_review và một dòng mô tả để calibration.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 tiny, sam2.1_t.pt, CPU | 5 (3 dương, 2 âm) | Còn lấn sang người lái và người phía sau khung xe; không thử vật chở hàng | Phủ một phần người dù có điểm âm; các khoảng trống giữa thanh khung chưa tách sạch. Diện tích 291086 px (41,7% ảnh), 1 polygon | tuktuk_img013.jpg |
| img_032.jpg | SAM 2.1 tiny, sam2.1_t.pt, CPU | 5 (3 dương, 2 âm) | Không thấy phủ người đứng cạnh; ghế và khung xe được giữ | Một số khe trống giữa khung dưới ghế còn bị tô; bánh và ghế bám tương đối sát. Diện tích 49196 px (8,3% ảnh), 1 polygon | cyclo_img032.jpg |
| img_020.jpg | SAM 2.1 tiny, sam2.1_t.pt, CPU | 7 (4 dương, 3 âm) | Còn phủ một phần người ngồi và vùng bên trong xe; không thử vật chở hàng | Mui có vài vùng bị bỏ sót; khoảng nhìn xuyên khung chưa tách sạch. Diện tích 84502 px (12,1% ảnh), 1 polygon | tuktuk_img020.jpg |

Codex chọn tọa độ prompt, chạy SAM và ghi nhận xét từ ảnh overlay. Học viên đã xem lại ba ảnh kết quả. Các phép đo diện tích lấy từ đầu ra sam2_points.py. Lệnh tái lập nằm trong work/cp5_commands.ps1 (thư mục làm việc cục bộ).

- Đối chiếu guideline mục 4.2.1, mask cần bám đường biên phần nhìn thấy. Ảnh 032 giữ được phần ghế và bánh xe tương đối sát, còn ảnh 020 có các vùng mui bị bỏ sót. Chưa đo sai lệch biên theo pixel nên chưa xác nhận đạt dung sai khoảng 3 px.
- Theo mục 4.2.2, lỗ nhỏ hơn khoảng 5% diện tích vật có thể được giữ trong mask, còn lỗ lớn cần loại ra. Các khe khung xe trong ba ảnh cần xét theo quy tắc này. Việc mask chỉ có một polygon chưa chứng minh các lỗ đã được xử lý đúng.
- Mục 4.2.4 yêu cầu tách các vật chạm nhau và gán vùng tiếp xúc cho vật phía trước. Kết hợp với quy tắc người và xe là các vật riêng ở mục 4.1.4, phần người bị tô trong mask xe ở ảnh 013 và 020 vẫn là lỗi cần chỉnh. Điểm âm đã dùng chưa loại sạch những vùng đó.
- Mục 4.2.3 xác định kính thuộc xe và bóng đổ nằm ngoài vật. Guideline vẫn để mở cách xử lý người nhìn xuyên kính ở mục 6.5, cũng như hàng nhô ra trong mask ở mục 6.3. Ba lượt thử này chưa đủ để chốt quy tắc cho hàng hoá.
- Khi xem kết quả SAM, cần kiểm tra riêng mui, bánh, thanh khung, vùng người và khoảng trống. Chất lượng ở ảnh 020 với 7 click vẫn còn lỗi, vì vậy số click trong ba ca này chưa cho thấy mask đạt yêu cầu chỉ nhờ tăng prompt.
- Ba lượt thử dùng sam2_points.py offline với điểm dương và điểm âm, sau đó xuất ảnh overlay. Chưa thử Interactor SAM trực tiếp trong CVAT nên chưa có kết quả so sánh hai cách.

## 6. Calibration

Đã ghi 4 quyết định trong qc/calibration_decisions.csv, gồm 2 ca loi_may, 1 cau_hoi_mo và 1 bay_lot. Bản bổ sung guideline v1.1 đồng bộ CAL-01 đến CAL-04, có ảnh ví dụ.
CAL-02 làm rõ mốc chiều dài xe khi xét hàng nhô, với img_031. CAL-01, CAL-03 và CAL-04 chủ yếu là áp dụng quy tắc đã có về cá thể và biên vật, bổ sung checklist để giảm lỗi lặp.
Các ca này chưa được áp qc_fixed vào nhãn trong vòng QC hiện tại. Bản release ở đây là candidate còn lỗi.
Các câu hỏi xe điện, ngưỡng cụm xe gần camera và mask hàng rời chưa có đủ ví dụ để xác lập quy tắc mới. Khi gặp ca liên quan chưa giải được phải giữ lại để làm rõ.

## 7. QC và quyết định

Chạy build_release.py, sau đó qc_sample.py --n 5 --seed 42. Mẫu gồm 5 ảnh ngẫu nhiên và 0 ca can_review, tổng 103 đối tượng, từ 24 ảnh batch. Năm ảnh là img_003, img_006, img_014, img_031 và img_035.
Người thực hiện QC lần này là Codex qua ảnh gốc, overlay có ID và tọa độ COCO. Học viên chưa kiểm độc lập phiếu QC này. Codex đã sửa hai box ở vòng trước nên đánh giá có nguy cơ phụ thuộc người sửa.

| Lỗi ghi nhận | Ảnh | Bằng chứng |
|---|---|---|
| Thừa | img_003 | #13 là phần thùng và bánh của cùng xe đã có #12 |
| Bỏ sót | img_006 | Xe máy đỗ phía sau xe bạc, đuôi và biển số gần (490,460), chưa có box riêng |
| Sai vị trí/kích thước | img_014 | #134 bắt đầu y=372, cắt gương và tay lái nhìn thấy phía trên |
| Chưa tính lỗi xác nhận | img_031 | Biên dưới #386 bị giỏ và bánh xe che; chưa đủ chắc chỉ từ overlay để kết luận lỗi, cần học viên đối chiếu thêm |
| Chưa thấy lỗi rõ trong lượt này | img_035 | Đã kiểm 37 đối tượng của ảnh; kết quả này không chứng minh ảnh hoàn toàn sạch |

Phiếu QC ghi 3 mục lỗi xác nhận, tương ứng 2 đối tượng đã có ID và một vật bị bỏ sót chưa có ID. Script tính 3/103 = 2.9% theo đối tượng và 3/5 = 60.0% theo ảnh. Mẫu số 103 là số nhãn có sẵn trong mẫu, chưa cộng vật bị bỏ sót, nên đây là quy ước đếm của bài.
Ba loại lỗi hình học, thừa và bỏ sót mỗi loại chiếm 1/3 = 33.3% mục lỗi xác nhận. Ca nghi ngờ ở img_031 không được tính là lỗi đã xác nhận. Không kết luận các loại sai lớp, thuộc tính hoặc mask bằng 0 trên toàn bộ batch chỉ từ mẫu detection này.
Wilson 95% cho 3/5 ảnh lỗi có cận trên khoảng 88%. Với 5 ảnh, không thể chứng minh tỉ lệ lỗi thực tế dưới 2%.

Theo cây quyết định, chưa thấy dữ liệu hỏng hoặc một ca chưa rõ quy tắc ngăn xử lý ba lỗi QC đã ghi. Tỉ lệ ảnh lỗi 3/5 = 60.0% vượt ngưỡng 2%, nên chọn Rework. Điểm bẫy hiện tại 11/12 = 91.7% sau sửa không bù được các lỗi nhãn còn tồn tại.
Script kiểm được cấu trúc COCO, đủ ảnh, lớp hợp lệ, kích thước box và tính số đo từ phiếu. Độ ôm sát vật, cá thể bị cả máy lẫn người bỏ sót và việc làm đúng checklist cần kiểm trực quan.

**Quyết định: Rework**, ngày 2026-11-17. Bản release chưa sẵn sàng bàn giao.
Lộ trình sửa là rà ba ảnh có lỗi và các ảnh tương tự, xác nhận lỗi với học viên, sửa trong CVAT, xuất bản mới, build lại release và lấy mẫu QC mới có lưu lịch sử. Giữ số đo vòng này làm bằng chứng trước sửa; chưa tự nhận vòng sau đã đạt.

## 8. Ghi chú cá nhân

- Công cụ Codex cài thư viện, tạo Project và Task CVAT Docker, chạy model và script, export COCO qua API cục bộ, chọn prompt SAM, soạn nhận xét và kiểm QC. Học viên tự gán gold và duyệt batch lần đầu, sau đó xem ảnh SAM. Sau phản hồi bẫy, Codex sửa hai box trên batch. Quyết định và nhận xét trong bản này được Codex soạn theo yêu cầu, học viên cần đọc và xác nhận trước khi nộp.
- Gold do học viên gán tay. Trước khi export gold, Codex từng mở bốn ảnh gốc theo yêu cầu gán hộ, nhưng không thêm hoặc sửa annotation gold. Các model all chỉ chạy sau khi học viên báo đã gán xong và CVAT đã có nhãn. Hội thoại có giải thích quy tắc chung về che khuất và mép ảnh. Chưa có bằng chứng đã cho học viên xem pre-label gold trước khi hoàn tất.
- Máy Windows, Python 3.13.9, torch CPU; CVAT Docker v2.74.1. Sửa thiếu PyYAML, transformers và huggingface_hub. YOLOE lần đầu cài CLIP và tải mobileclip2_b.ts.
- Đã chạy model tại máy; không dùng data/cache thay cho hai lượt all. CP2 dùng YOLO26n và SAM tiny; CP3 dùng YOLO26m theo yêu cầu.
- Không đổi rules trong taxonomy, không sửa đầu ra số đo bằng tay. API download export dùng X-Accel-Redirect nên phải đọc ZIP tương ứng trong container. Việc chạy đồng thời hai model CP3 tạo trùng run_id r08 trong work/runs.csv; các tham số, model và thư mục out vẫn phân biệt được từng lượt.
- Các sửa sau lượt duyệt đầu chưa có thời gian đo riêng. Commit chứa bản bài nộp được ghi trong STUDENT.md. Repo đích do học viên chỉ định là chuns29/K5-L23-Day08-DuongQuangChung-03217-AI-Assisted-Labeling.
- Lần sau cần hoàn thành một lượt tìm vật sót khi đang ẩn nhãn, lưu checkpoint trước khi xem đáp án và ghi thời gian từng vòng rework để so sánh chi phí đầy đủ.
