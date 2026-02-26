# BUS

## I. Enterprise Data Warehouse Bus Architecture
* EDW lớn phức tạp cần có sơ đồ kiến trúc tổng quát thể hiện được các quy trình và bối cảnh

![alt text](images/Chap4/image.png)

### 1. Bus Matrix
* Bus Matrix giúp xác định các dim liên quan business process.

* Bus Matrix là bản đồ tổng thể, trực quan, dễ dùng → lập kế hoạch, quản lý dự án, giao tiếp nội bộ và với lãnh đạo, quản trị dữ liệu.

![alt text](images/Chap4/image-1.png)

---


### 2. Stakeholder Matrix
* Stakeholder Matrix cho biết phòng ban nào liên quan đến quy trình nào.

* Stakeholder Matrix = tập trung vào người và nhóm → ai cần tham gia, ai quan tâm → quản lý dự án hiệu quả.

![alt text](images/Chap4/image-2.png)

## II. Mistake khi làm Bus
### 1. Hàng
- `Quá rộng`: Làm theo phòng ban mà ko làm theo business process
- `Quá hẹp`: Làm theo báo cáo mà ko làm theo business process 

---

### 2. Cột
- Lỗi chọn thực thể tổng quát. Chọn person thay vì customer, employee, vendor,..
- Dính lỗi Hierarcy, ko granularity.

## III. Conformed DIM
- Dim được **dùng chung** giữa các fact -> `Fact x Dim`, `Dim x Fact` => **Fact** gặp **Fact** qua **Conformed DIM**

- Trả lời câu hỏi
    - *Sản phẩm nào đang tồn kho cao, đơn mở nhiều, nhưng bán chậm?*

- Lợi ích:
    - Drill Across: Cho các metrics của các fact về cùng 1 report
    - Kết nối `fact_sales` (Sales Qty), `fact_inventory` (Inventory Qty), `fact_orders` (Open Orders Qty) 👉 Tất cả hiển thị theo cùng Product (dim_product).
    
    ![alt text](images/Chap4/image-3.png)

## IV. Shrunken DIM (Bẻ Hieracy)
### 1. Subset theo cột
![alt text](images/Chap4/image-4.png)

- Là **dim con** Chứa subset cột của dim cha

- Lí do:
    - Vì **fact không có key** của **dim cha**

- Ví dụ:
    - `Retail Sales fact` → grain = product (SKU) (trong product có brand)
    - `Forecast fact` → grain = brand


- Cách thức:
    - `fact_sales` X `dim_product` (Group by brand chẳng hạn)
    - `fact_forecast` X `dim_brand` (Group by brand chẳng hạn)
    - Outer join 2 bảng -> Thấy được sự chênh giữa forecast và thực bán nhờ drill across.

```text
fact_sales
Brand A | 100
Brand B | 200

fact_forecast
Brand B | 180
Brand C | 300

Final
Brand A | 100 | NULL
Brand B | 200 | 180
Brand C | NULL | 300
```

---

### 2. Subset theo dòng
- Lấy 1 tệp data theo scope

- Tách dim ra thành dim con

- Tách fact ra thành fact con

- Ví dụ: Tập đoàn lớn làm mảng sản phẩm riêng -> Tránh DA bị ngợp hoặc đơn giản ko cho phép họ xem.

---

### 3. Daily -> Month
Là case đặc biệt vừa row vừa col

1️⃣ Subset ROW

* Lấy **chỉ các ngày cuối tháng** từ `dim_date`
* 1 row = 1 tháng

---

2️⃣ Subset COLUMN

* ❌ Bỏ các cột daily:

  * weekday
  * holiday
  * day_of_month
* ✅ Giữ cột month-level:

  * month
  * year
  * yyyy-mm

---

### 4. EDW ngoài lề 
EDW Thất bại Khi:
- Doanh nghiệp có chứa cty con
- Không thống nhất được định nghĩa

-> Ưu tiên doanh nghiệp hơn là mô hình

Conformed dimension là bài toán tổ chức & chính trị, không phải bài toán kỹ thuật. IT ko có quyền.
Không có data governance → đừng mơ enterprise analytics.

- Hãy cố tìm điểm chung (attribute) giữa các stakeholder để drill across.

### 5. Conformed Fact
Là fact dùng chung chỉ chiếm **5%** effort.

Chỉ gọi là conformed fact khi metric có **cùng ngữ nghĩa 100%**. **Khác** -> bỏ conformed tạo nhiều fact.

**Không cố ép** conform nếu ko đc.

Case khác đơn vị đo:
- Ở warehouse dùng **thùng**
- Ở Store dùng **chai**
    - Ko tạo 2 fact. 
    - Không để BI user tính toán nhức đầu
    -  **Tạo 2 cột measurements là được**

```text
fact_inventory_flow
- quantity_cases
- quantity_bottles
```