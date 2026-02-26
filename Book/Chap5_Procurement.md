# Procurement

## Tổng quan
Chương V nói đến cách xử lý khi dữ liệu trong dimension thay đổi theo thời gian. và xây dựng các mô hình dữ liệu cho mua hàng


## Procurement In Value Chain
### Vì sao procurement quan trọng?
Với nhiều công ty, procurement ảnh hưởng **trực tiếp đến lợi nhuận**:
- Retailer / Distributor: mua đúng hàng, đúng giá để bán
    - Ko phải món hàng lúc nào cx bán 1 giá (Tùy dịp)
- Manufacturing: mua nguyên vật liệu với chi phí tối ưu

💰 **Tiết kiệm chi phí lớn** có thể đạt được nhờ:

* Giảm số lượng supplier
* Đàm phán với supplier ưu tiên
* Hợp đồng dài hạn / mua số lượng lớn (Gom đơn chi nhánh mua số lượng lớn cho rẻ)

---

### Procurement gắn với Demand Planning

Quy trình logic:

1️⃣ Dự báo nhu cầu (Demand Forecast)

2️⃣ Procurement đảm bảo:

* Đúng loại hàng
* Đúng số lượng
* Giá rẻ nhất có thể

Procurement bao gồm nhiều hoạt động:

* Đàm phán hợp đồng
* Tạo requisition
* Tạo Purchase Order (PO)
* Theo dõi nhận hàng
* Duyệt thanh toán

---

### Các câu hỏi phân tích phổ biến trong Procurement

Hệ thống BI cần trả lời được những câu hỏi như:

**📦 Mua gì – từ ai – giá bao nhiêu?**

* Sản phẩm/nguyên vật liệu nào được mua nhiều nhất?
* Có bao nhiêu vendor cung cấp cùng một sản phẩm?
* Giá mua giữa các vendor khác nhau thế nào?
* Nếu gom nhu cầu toàn công ty:

  * Có thể giảm số vendor không?
  * Có thể đàm phán giá tốt hơn không?

**🚨 Maverick Spending**

* Nhân viên có mua từ **vendor không được ưu tiên** không?
* Có vi phạm hợp đồng đã đàm phán không?

**💸 Giá mua có đúng hợp đồng không?**
* Có chênh lệch giữa:
  * Giá hợp đồng
  * Giá thực tế mua (purchase price variance)?

**🚚 Hiệu suất của Vendor**
* Tỷ lệ giao đủ hàng (fill rate)
* Giao đúng hạn hay trễ?
* Đơn hàng bị backorder bao nhiêu?
* Tỷ lệ hàng bị reject khi kiểm tra chất lượng

---

## I. 4 Steps
![alt text](images/Chap5/image.png)

### 1. Chọn Business Process
là `Procurement`

### 2. Grain 
- 1 dòng = 1 giao dịch mua bán

### 3. DIM liên quan
- `Transaction Date` – ngày phát sinh giao dịch (Conformed Dim)
- `Product` – hàng hóa / nguyên vật liệu (Conformed Dim)
- `Vendor` – nhà cung cấp
- `Contract Terms` – điều khoản hợp đồng
- `Procurement Transaction Type` – loại giao dịch (PO, receipt, payment…)

*Nếu là doanh nghiệp manufacturing có thêm*
- `Raw Materials Dimension`: Nguyên liệu

### 4. Các measure (facts)
Fact table lưu các số liệu như:
- Số lượng mua
- Giá trị tiền (dollar amount)
Trong fact Procument để mã hợp đồng.

### 5. Back to Dim
`Vendor Dim`:
- Mỗi vendor 1 dòng
- Có các thuộc tính mô tả để phân tích hiệu suất vendor

`dim_contract_terms`
- Mỗi dòng đại diện cho một bộ điều khoản đã đàm phán

`Procurement Transaction Type Dimension`
- Dùng để:
    - Group
    - Filter

Ví dụ: chỉ xem PO, chỉ xem receipt…

- Ko nên tạo dim hợp đồng nên tạo dim điều khoản thôi.

---

`Procurement Transaction Fact`
| Date Key | Product Key | Vendor Key | Contract Terms Key | Txn Type Key | Contract Number | Qty | Dollar Amount |
| -------: | ----------: | ---------: | -----------------: | -----------: | --------------- | --: | ------------- |
| 20240105 |         201 |        101 |                301 |          401 | CT-2023-001     | 100 | 50,000,000    |
| 20240110 |         201 |        101 |                301 |          402 | CT-2023-001     | 100 | 50,000,000    |
| 20240110 |         201 |        101 |                301 |          403 | CT-2023-001     | 100 | 50,000,000    |
| 20240105 |         202 |        102 |                302 |          401 | CT-2023-014     | 200 | 40,000,000    |


`Procurement Transaction Type Dimension`
| Transaction Type Key | Description    | Category  |
| -------------------: | -------------- | --------- |
|                  401 | Purchase Order | Ordering  |
|                  402 | Goods Receipt  | Receiving |
|                  403 | Vendor Payment | Payment   |


`dim_contract_terms`
| Contract Terms Key | Description             | Terms Type |
| -----------------: | ----------------------- | ---------- |
|                301 | Fixed price, Net 30     | Standard   |
|                302 | Volume discount, Net 45 | Volume     |

- Dòng 1: giá cố định, sau 30 ngày thanh toán. Điều khoản bthg
- Dòng 2: Mua càng nhiều thì càng rẻ, Được 45 ngày mới phải thanh toán. Đây là điều khoản ưu đãi do mua số lượng lớn

## II. Bus Matrix & single or multi fact
Tùy vào quy mô, định nghĩa business có thể tách 1 fact procurement thành nhiều fact.

Việc tách nhiều fact quản lý nhiều hơn nhưng ETL đơn giản hơn

Procurement không phải 1 hành động mua, mà là cả một chuỗi “xin – đặt – giao – nhận – đòi – trả (Cân nhắc có nên tách ko).

![alt text](images/Chap5/image-1.png)

![alt text](images/Chap5/image-2.png)

![alt text](images/Chap5/image-3.png)


## III. Complementary Procurement Snapshot
Transaction Procument chưa trả lời được câu hỏi 
- Bước nào trong procurement bị kẹt
- Mất bao lâu từ đặt hàng đến trả tiền

Do nằm rải rác nhiều fact

![alt text](images/Chap5/image-4.png)


