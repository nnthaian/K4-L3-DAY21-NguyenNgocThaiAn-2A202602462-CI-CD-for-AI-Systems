# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Ngọc Thái An |
| MSSV | 2A202602462 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/nnthaian/K4-L3-DAY21-NguyenNgocThaiAn-2A202602462-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 2 | 50 | 0.5 | 2 | 0.7048 | 0.876 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.874 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.1`, `max_depth=3`.

**Lý do:** Cấu hình 3 có F1 cao nhất nhưng chỉ hơn cấu hình 1 khoảng 0.004, trong khi dùng gấp đôi số cây và độ sâu lớn hơn. Tôi chọn cấu hình 1 để cân bằng chất lượng và độ phức tạp; F1 vẫn vượt ngưỡng 0.65. Lần có accuracy cao nhất không trùng với lần có F1 cao nhất, cho thấy accuracy chưa phản ánh tốt lớp thu nhập cao. Learning rate lớn với ít cây hơn cũng cho F1 thấp hơn.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Dữ liệu Adult mất cân bằng: lớp thu nhập trên 50K chỉ chiếm khoảng 25%. Mô hình luôn đoán “thu nhập thấp” vẫn đạt gần 75% accuracy nhưng không phát hiện được lớp dương. F1 kết hợp precision và recall nên chỉ cao khi mô hình vừa hạn chế dự đoán dương sai, vừa tìm được nhiều trường hợp thu nhập cao. Do quality gate cần ngăn mô hình bỏ sót lớp thiểu số, F1 phù hợp hơn accuracy. Tôi dùng `f1_score(y_true, y_pred)` cho lớp dương mặc định; `average="weighted"` bị lớp đa số kéo lên, còn `average="macro"` không tập trung riêng vào lớp thu nhập cao.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| DVC không truy cập S3 | Sai bucket và thiếu quyền AWS | Sửa remote, IAM/bucket policy và GitHub Secrets |
| Release không load model | scikit-learn giữa runner và EC2 khác nhau | Pin `scikit-learn==1.7.2` cho train và serve |
| API không truy cập từ máy cá nhân | Security Group chưa mở TCP 8080 | Mở cổng 8080 giới hạn theo IP cá nhân |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (chỉ `train_batch1`) | 0.7109 | 0.878 |
| Bước 3 (thêm `train_batch2`) | 0.7014 | 0.874 |

**Nhận xét:** Sau khi thêm dữ liệu, F1 giảm 0.0095 và accuracy giảm 0.004 nhưng vẫn qua quality gate. Hai batch cùng nguồn, có phân phối tương tự nên dữ liệu mới không nhất thiết thêm tín hiệu. Giá trị chính của Bước 3 là chứng minh quy trình DVC, huấn luyện, kiểm tra và triển khai lại chạy tự động.
