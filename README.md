
# Phân tích cổ phiếu ngân hàng Việt Nam (BID, ACB, MBB)

## Mô tả dự án

Project này là một **notebook phân tích dữ liệu chứng khoán Việt Nam (nhóm ngân hàng)**, tập trung vào các mã **BID, ACB, MBB** trong giai đoạn **2024-01-01 → 2024-11-30**. Mục tiêu là **lấy dữ liệu giá**, **trực quan hóa**, **đo lường lợi suất/rủi ro**, và **thử pipeline dự đoán giá** ở mức cơ bản.

## Nội dung chính

- **Thu thập dữ liệu**  
  Tải lịch sử giá/khối lượng các mã BID, ACB, MBB từ `vnquant` (nguồn CafeF) trong khoảng 2024-01-01 → 2024-11-30, chuẩn hóa về DataFrame để xử lý.
- **Trực quan hóa**  
  Vẽ đường giá (open/high/low/close/adjust) kèm khối lượng/giá trị giao dịch theo thời gian để quan sát xu hướng và thanh khoản.
- **Return & Risk**  
  Tính `daily_return` cho từng mã, so sánh lợi suất trung bình; pivot để tính tương quan (correlation) giữa các mã; tính các đường trung bình động (SMA/EMA 30/60/200) để đọc xu hướng; chuẩn bị ước lượng rủi ro đuôi (VaR).
- **ML cơ bản**  
  Xây pipeline nhỏ: chia train/test theo thứ tự thời gian, thử mô hình dự đoán giá đóng cửa (ví dụ linear regression/random forest), trực quan hóa giá thật vs giá dự đoán.

## Cách chạy

- **Mở notebook**: `Analysis.ipynb`
- **Chạy tất cả cell** theo thứ tự trong notebook (Run All / Run Cells).

## Yêu cầu (gợi ý)

- Python 3.x
- Thư viện: `vnquant`, `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`
