# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Mai Hoàng Anh |
| MSSV | 23001824 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/23001824-HoangAnh/K4-Track2-Day21-2A202601118-NguyenMaiHoangAnh |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 2 | 100 | 0.10 | 3 | 0.7109 | 0.878 |
| 3 | 200 | 0.10 | 5 | 0.7149 | 0.874 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.10`, `max_depth=5`.

**Lý do:** Lần chạy 3 có F1 cao nhất (0.7149), phù hợp quality gate dù accuracy thấp hơn lần 2 là 0.004. Điều này cho thấy accuracy và chất lượng lớp dương không hoàn toàn đồng biến trên dữ liệu lệch lớp. Tăng từ 50 lên 100 cây cùng learning rate giúp F1 cải thiện mạnh; 200 cây chỉ tăng nhẹ, thể hiện lợi ích giảm dần.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Lớp thu nhập trên 50K chiếm 24.8% tập huấn luyện, nên mô hình luôn dự đoán “thu nhập thấp” vẫn đạt 75.2% accuracy nhưng bỏ sót toàn bộ lớp cần phát hiện và có F1 lớp dương bằng 0. F1 kết hợp precision và recall, buộc mô hình vừa hạn chế dương tính sai vừa tìm được đủ trường hợp thu nhập cao. Quality gate dùng F1 lớp dương (`pos_label=1`), không dùng `weighted` vì lớp đa số có thể che lấp kết quả kém, và không dùng `macro` vì mục tiêu là kiểm soát trực tiếp lớp thu nhập cao.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| DVC S3 xung đột phiên bản | Boto3 cũ không tương thích dependency hiện tại | Ghim phiên bản tương thích và chạy lại test |
| Push đầu không kích hoạt Actions | Actions của repo fork chưa được bật | Bật Actions và xác nhận push mới chạy đủ bốn job |
| EC2 cần hạn chế bề mặt tấn công | SSH và cổng dịch vụ rộng là không cần thiết | Dùng SSM, đóng cổng 22 và giới hạn cổng 8080 theo IP |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.874 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.882 |

**Nhận xét:** Thêm 22.361 mẫu làm F1 tăng 0.0205 và accuracy tăng 0.008 trên holdout cố định. Mức tăng vừa phải cho thấy thêm dữ liệu cùng phân phối không bảo đảm một bước nhảy lớn.
