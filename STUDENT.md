# Thông tin học viên

- Họ và tên: Dương Quang Chung
- MSHV: 2A202603217
- Lớp: L2A
- Tài khoản GitHub: chuns29
- Repo bài nộp: https://github.com/chuns29/K5-L23-Day08-DuongQuangChung-03217-AI-Assisted-Labeling
- Commit chốt bài: Chưa tạo; cập nhật khi commit và push bài hoàn chỉnh.
- Máy: Windows, Python 3.13.9, torch CPU, không có GPU được PyTorch nhận diện.
- CVAT Docker: v2.74.1

## Kết quả tóm tắt

- Máy accepted: 377/431 = 87.5%; edited: 18/431 = 4.2%; deleted: 36/431 = 8.4%, mẫu số 431 box máy sau loại vùng bẫy.
- Added: 62/457 = 13.6%, mẫu số 457 đối tượng sau duyệt trong phép so sánh đã loại vùng bẫy. Release candidate có 468 đối tượng trên 24 ảnh.
- So gold, precision máy: 45/62 = 72.6%; recall máy: 45/57 = 78.9%. Bốn ảnh gold có 61 annotation, phép đo dùng 57 đối tượng sau xử lý ignore.
- Bắt bẫy lượt duyệt đầu: 6/12 = 50.0%; sau các lượt sửa và hai chỉnh sửa của Codex: 11/12 = 91.7%. Kết quả sau không phải lượt duyệt mù.
- Thời gian gán tay: 50 phút/4 ảnh = 12.50 phút/ảnh. Duyệt lượt đầu: 76 phút/24 ảnh = 3.17 phút/ảnh, đã trừ 120 phút nghỉ. Chưa đo thời gian sửa bổ sung.
- QC do Codex kiểm 5 ảnh ngẫu nhiên, 103 đối tượng: lỗi theo ảnh 3/5 = 60.0%; theo quy ước đếm đối tượng của script 3/103 = 2.9%. Cận trên Wilson 95% theo ảnh khoảng 88%.
- Quyết định: Rework. 3/5 ảnh QC có lỗi, vượt ngưỡng 2%; cần sửa và lấy mẫu kiểm lại trước khi bàn giao.
- Nguồn: submission/qc/review_stats.json, qc_review.csv, qc_report_auto.md và submission/time_log.csv. Chi tiết các lượt sửa và cách kiểm nằm trong REPORT.md.
