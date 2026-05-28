# 📝 PROJECT PLANNING & DATA PREPARATION
## Project: Customer 360 Analytics Platform (RFM & Growth Rate)

---

## 🗺️ Phase 1: Data Architecture Mapping

Trước khi thực hiện các phép toán, cấu trúc dữ liệu được xác định dựa trên mối quan hệ giữa 3 bảng nguồn:

[Location] (1) -------> (N) [Customer_Register] (1) -------> (N) [Customer_Transaction]
### Chi tiết các thực thể dữ liệu (Data Dictionary):
* **`Location`**: Chứa thông tin địa lý (`LocationID`, `SubcompanyID`, `SubcompanyName`, `BranchCode`).
* **`Customer_Register`**: Quản lý vòng đời khách hàng (`ID`, `Contract`, `LocationID`, `BranchCode`, `Status`, `Created_date`, `Stop_date`).
* **`Customer_Transaction`**: Lưu vết hành vi tài chính (`TransactionID`, `CustomerID`, `Purchase_Date`, `GMV`).

---

## 🛠️ Phase 2: Execution Blueprint

### 📌 TASK 1: RFM Analysis for Customer 360 Project

#### 1.1. Trích xuất và Chuẩn hóa các chỉ số Base RFM
* **Recency (R)**: 
  * *Logic:* Tìm ngày giao dịch cuối cùng của từng khách hàng: $Max(Purchase\_Date)$.
  * *Công thức:* `DATEDIFF(day, Max(Purchase_Date), '2022-09-01')`.
* **Frequency (F)**:
  * *Logic:* Đếm số ngày có phát sinh giao dịch độc lập để tránh nhiễu (Spam nhiều đơn một ngày).
  * *Công thức gốc:* `COUNT(DISTINCT Purchase_Date)`.
  * *Chỉ số chuẩn hóa (Tránh thiên vị khách hàng lâu năm)*: Chia cho độ tuổi hợp đồng (tính bằng tháng) từ `Created_date` đến ngày chốt báo cáo `2022-09-01`.
* **Monetary (M)**:
  * *Logic:* Tính tổng giá trị giao dịch của khách hàng.
  * *Công thức:* `SUM(GMV)` (Sau đó chuẩn hóa trung bình theo số tháng hoạt động tương tự Frequency).

#### 1.2. Chấm điểm các chỉ số dựa trên IQR Method (Tứ phân vị)
* Sử dụng hàm cửa sổ `ROW_NUMBER() OVER (ORDER BY Metric)` để sắp xếp dữ liệu tăng dần cho từng chỉ số.
* Xác định các điểm mốc: **Min, Q1 (25%), Q2 (50% - Median), Q3 (75%), Max** để làm biên độ chia điểm từ 1 đến 4.
  * *Lưu ý với R:* Ngày gần nhất (DATEDIFF nhỏ) ứng với điểm R cao (4 điểm).
  * *Lưu ý với F & M:* Tần suất và số tiền càng lớn ứng với điểm F, M càng cao (4 điểm).

#### 1.3. Gộp tổ hợp & Phân hạng theo Ma trận BCG
* Ghép chuỗi các điểm số thành tổ hợp RFM (Ví dụ: `R=1, F=1, M=1` $\rightarrow$ `'111'`).
* Thực hiện thống kê tổng quan bằng cách: `COUNT(DISTINCT CustomerID)` group by `Tổ hợp RFM` và `ORDER BY total_customers DESC` để kiểm tra tổ hợp nào chiếm đa số.
* **Mapping sang nhóm khách hàng theo BCG Matrix**:
  * **Stars (VIP)**: Tổ hợp dạng `444, 443, 434...` (Chi tiêu mạnh, mua liên tục, vừa mới mua).
  * **Cash Cows (Thân thiết)**: Tổ hợp dạng `442, 441, 432...` (Mua rất đều đặn, mới mua nhưng chi tiêu mỗi đơn không quá cao).
  * **Question Marks (Tiềm năng)**: Tổ hợp dạng `414, 424...` (Mua nhiều tiền nhưng ít khi quay lại) hoặc `144, 244` (Từng là khách VIP nhưng đã lâu chưa tương tác).
  * **Dogs (Vãng lai)**: Tổ hợp dạng `111, 112, 121...` (Chi tiêu thấp, lười mua, mất tích đã lâu).

---

### 📌 TASK 2: Customer Growth Rate for BI Reporting

#### 2.1. Xử lý dữ liệu dòng thời gian (Time-series Prep)
* Trích xuất cấu trúc ngày/tháng từ 2 trường `Created_date` (Ngày bắt đầu) và `Stop_date` (Ngày hủy hợp đồng) trong bảng `Customer_Register`.

#### 2.2. Định nghĩa các chỉ số tăng trưởng (Growth Metrics)
* **New Customers (Khách hàng mới)**: Đếm số lượng `ID` có `Created_date` thuộc ngày/tháng đang xét.
* **Churned Customers (Khách hàng mất đi)**: Đếm số lượng `ID` có `Stop_date` thuộc ngày/tháng đang xét (và `Status` ghi nhận ngừng dịch vụ).
* **Retention Rate (Tỉ lệ giữ chân) & Growth Rate**: Công thức liên hệ giữa lượng đăng ký mới và lượng hủy dịch vụ theo chu kỳ để đo lường sức khỏe hệ thống.

---

## 📈 Phase 3: Dashboard & Reporting Strategy

### 3.1. Viết Report (Khai thác kết quả thực tế)
* Trả lời trực tiếp bài toán kinh doanh của Team Marketing: Từ **100k+ khách hàng ban đầu**, cấu trúc phân bổ cụ thể của từng nhóm sau khi mapping là bao nhiêu để team CSKH lên plan làm việc hiệu quả.
* Xác định rõ đâu là nhóm đang gánh doanh số chính cho doanh nghiệp và nhóm nào đang có rủi ro rời bỏ cao.

### 3.2. Ứng dụng
* Ứng dụng mô hình **BCG Matrix** để đề xuất giải pháp hành động cho từng nhóm:
  * *Nhóm Stars (VIP):* Đưa vào danh sách chăm sóc đặc biệt (VVIP), tặng quà tri ân cuối năm cá nhân hóa.
  * *Nhóm Question Marks (Tiềm năng):* Gửi các chiến dịch kích cầu, khuyến mãi trúng đích để thúc đẩy tần suất mua hàng.
  * *Nhóm Cash Cows (Thân thiết):* Duy trì combo/gói dịch vụ định kỳ để giữ dòng tiền ổn định.
  * *Nhóm Dogs (Vãng lai):* Tối ưu chi phí, không lãng phí quá nhiều ngân sách marketing vào nhóm này, chỉ dùng automation mail.
