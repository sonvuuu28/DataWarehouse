# RETAIL SALE

## I. Retail Case Study

### 🏪 Bối cảnh doanh nghiệp

* Chuỗi **siêu thị bán lẻ thực phẩm** quy mô lớn
* ~ **100 cửa hàng** ở **5 bang**
* Mỗi cửa hàng có đầy đủ các **department**
  (grocery, frozen, dairy, meat, produce, bakery, floral, health/beauty…)
* Mỗi store có khoảng **60.000 SKU** (sản phẩm)

---

### 📊 Dữ liệu được thu ở đâu?

Có **2 điểm thu dữ liệu quan trọng** trong siêu thị:

#### 1️⃣ Front door – Quầy thu ngân (POS)

* Khi khách mua hàng
* Barcode sản phẩm được **scan**
* Ghi nhận:

  * sản phẩm nào được mua
  * số lượng
  * giá bán
  * coupon / discount
  * thời gian
  * cashier / store

👉 Đây là **nguồn dữ liệu quan trọng nhất** để đo **consumer takeaway** (bán được gì, bao nhiêu)

#### 2️⃣ Back door – Nhận hàng từ nhà cung cấp

* Khi vendor giao hàng
* Ghi nhận:

  * hàng nhập
  * số lượng
  * thời điểm
  * chi phí

👉 Phục vụ cho **logistics, tồn kho, chi phí**

---

### 🎯 Mối quan tâm của quản lý

Management tập trung vào:

* đặt hàng
* tồn kho
* bán hàng
* **tối đa hóa lợi nhuận**

Lợi nhuận đến từ:

* bán giá cao hơn
* giảm chi phí mua hàng & vận hành
* thu hút nhiều khách trong thị trường cạnh tranh

---

### 💸 Pricing & Promotion là “trận địa chính”

* Pricing và promotion là **quyết định quan trọng nhất**
* Các hình thức promotion:

  * giảm giá tạm thời
  * quảng cáo báo giấy
  * trưng bày trong store
  * coupon

👉 Giảm giá mạnh có thể:

* làm **sản lượng bán tăng vọt** (x10 lần)
* nhưng thường **bán lỗ**, không bền vững

➡️ Vì vậy:

> **Theo dõi & phân tích promotion là cực kỳ quan trọng**

---


## II. Step 1: Select the Business Process

* **Nhu cầu business**:
  Cửa hàng quan tâm đến **việc bán hàng**, đặc biệt là **tối ưu lợi nhuận** thông qua **pricing và promotion (quảng cáo, khuyến mãi)**.

* **Thực tế hệ thống vận hành**:
  Doanh nghiệp có **POS system** ghi nhận chi tiết các giao dịch mua hàng tại quầy thu ngân.

* **Suy ra business process để model**:
  👉 **Sales transactions (POS retail sales transactions)**

Process này:

* phản ánh trực tiếp hành vi mua hàng của khách
* cho phép phân tích tác động của **giá và quảng cáo** đến doanh thu & sản lượng
* có dữ liệu **sẵn có, chi tiết, đáng tin** để triển khai DW/BI

---

## III. Step 2: Declare the Grain

* Ở step 1 xác định được **nghiệp vụ mua bán** rồi

* Ta cần lưu ý: nên design **atomic grain** thay dữ liệu đã tổng hợp (Aggregated Data). Vì càng chi tiết DIM càng dễ thiết kế

* Grain: 1 dòng = 1 sản phẩm trong 1 POS transaction (Tính theo SKU)


---

## IV. Step 3: Declare the DIM

* **Ko care cột**, nghĩ OBJ trước

---


* Sau khi đã chốt grain:
1 dòng = 1 sản phẩm (SKU) trong 1 POS transaction

* Hai dimension bắt buộc và xuất hiện ngay là:
    * **Product** – sản phẩm nào
    * **Transaction** – thuộc giao dịch nào


---

* Từ primary dimension, hỏi tiếp “còn mô tả gì nữa?”

* Ví dụ:

    * **Date** – bán vào ngày nào
    * **Store** – bán ở cửa hàng nào
    * **Promotion** – bán dưới chương trình khuyến mãi nào
    * **Cashier** – thu ngân nào xử lý
    * **Method of payment** – hình thức thanh toán

---

* Dimension được thêm vào fact table **chỉ khi**:

  * nó có **1 giá trị duy nhất** cho mỗi combination của primary dimensions

* 🚨 Nếu thêm dimension mà:

    * làm phát sinh **thêm dòng fact**
    * hoặc buộc phải tách 1 dòng thành nhiều dòng

* 👉 Thì:

    * dimension đó **vi phạm grain**
    * **hoặc** grain đã khai báo là sai → phải quay lại Step 2

---

## V. Step 4: Declare the Fact
* Dựa vào grain build fact 

* Metric trong fact để null ok. SUM / AVG / COUNT xử lý NULL đúng. Nhưng ko được null ở FK

* Thêm cols là các **metrics cơ bản**. 
    * **VD**: `Sales Quantity`, `Regular Unit Price` (giá gốc), `Discount Unit Price` (Giá giảm), `Net Unit Price` (Giá thực trả) 

---

* Thêm cols là các **extended** 
    * **VD**: 
        * `Extended Sales Dollar Amount` = Quantity × Net Unit Price
        * `Extended Discount Dollar Amount` = Quantity × Discount Unit Price


---

* Gắn **FK** tới dim

![alt text](images/Chap3/image.png)


---

* **Derived Fact – Cost, Gross Profit**
    * `Extended Cost Dollar Amount` = `Quantity` × `Unit Cost` (Có giá nhập thì tính luôn đừng care tiền nhập, lưu bãi này kia phức tạp)
    * `Gross Profit` = `Extended Sales` − `Extended Cost`


---

* Fact **ko** nên lưu **ratio**, hãy để BI làm chuyện đó. Chỉ lưu tử mẫu
    * **VD**: `Gross Margin` = `Gross Profit` / `Revenue`
        * Revenue là doanh thu, Gross Profit là lợi nhuận, Gross Margin là tỷ lệ lợi nhuận trên doanh thu.
    * Không lưu Gross Margin, chỉ lưu tử mẫu


---

* **Ước lượng size fact**: 
    * Doanh thu: 4 tỷ đô
    * Giá trị trung bình món hàng: 2 đô
    * Sâp sỉ fact có khoảng 2 tỷ transactions


--- 
* ***Extended metric**
    *  *Aggregated được*
    *  *Mang ý nghĩa ở cả từng dòng **và** khi gộp nhiều dòng*

* **Ratio metric (vd: gross margin)**
    * *Không aggregated được*
    * *Chỉ mang ý nghĩa khi được tính lại trên **một tập dòng***
    * *Giá trị ở từng dòng **gần như vô dụng cho phân tích***


## VI. Dimension Table Details
### 1. Date Dimension
* Build trước được và 20 năm chỉ khoảng 7300 dòng.
* Build hẳn ở **dim** ko để date ở fact vì BI user **ko biết các hàm SQL.**
* **Fiscal** sinh ra để cho doanh nghiệp tự định nghĩa các mốc tgian (year, month, quarter) để tránh bị ảo tưởng bởi sự kiện truyền thống như Tết năm nay tháng 1, năm sau tháng 2. Họ tự đặt ra quý theo chiến dịch để giá hiệu quả chiến dịch trong quý đó (quý ko bắt buộc là 3 tháng)
* Dạng **text** để BI user đọc hiểu. Tránh dùng True/False, 1/0
    * Vd: Jan, Feb, Non-holiday, ...

* Nếu **user BI quá yếu** có thể thêm các thuộc tính để họ đọc hơn là suy luận.

    | Attribute            | Thực chất giúp user                  |
    | -------------------- | ------------------------------------ |
    | IsCurrentMonth       | “Cho tôi số tháng hiện tại”          |
    | IsCurrentFiscalMonth | “Tháng tài chính hiện tại là gì?”    |
    | IsFiscalMonthEnd     | “Đây có phải cuối kỳ chốt sổ không?” |
    | IsPrior60Days        | “60 ngày gần nhất”                   |
    | LagDay               | “Hôm nay / hôm qua / hôm kia”        |


![alt text](images/Chap3/image-1.png)

![alt text](images/Chap3/image-2.png)

---

### 2. Time of Day
Theo giờ, phút luôn thì có **2 cách**

1. Tạo hẳng **dim time** trong đó row có thể chia theo **bucket** hoặc theo từng phút 1 có 1440 rows tương đương 1440 phút/ngày
2. Để trong **fact** nếu ko filter hay group mà chỉ tính `gap`, `duration`.


### 3. Product Dimension
* `Product` sẽ lquan đến `Brand`, `Category`, `Department`. Ko cần để **FK** trỏ đến các table khác -> **Flatten** luôn ở Product (Đừng sợ nặng, sợ rắc rối hơn).

* Ngoài hierarchy attribute sẽ có các filter, drill down attribute. **VD**: Diet Type, Fat Content, ..

* Nếu **Natural Key** có ý nghĩa hãy thêm các cột code để giải mã cho BI user hiểu luôn.
    * VD: sku_code = "BR12-ICE-500-XL"
    ```text
    product_dim
    - product_key
    - sku_code (NK)
    - brand
    - category
    - size
    - package_type
    ```

---

* **Numberic Value** nằm ở **Fact** hay ở **Dim**. Theo Kimball:
    * Fact nếu giá trị số đó để tính toán
    * Dim nếu giá trị để drill down, Filter
    * Nằm ở cả 2 đều ok
    * VD: `Unit price`, cá nhân mình nghĩ sẽ để ở cả 2


![alt text](images/Chap3/image-3.png)

### 4. Store Dimension
* `Store` ko chỉ là **địa điểm** mà là **thực thể kinh doanh** 
* Store sẽ liên quan đến Store → City-State → County → State → Zip (City-State vì có city trùng tên ở bang khác nhau)

![alt text](images/Chap3/image-4.png)

* Khi một DIM có chứa các ngày mà BI user muốn filter và drill down theo calendar / fiscal, thì nên dùng `role-playing Date Dimension` (thường qua VIEW) để làm cho dimension “giàu ngữ nghĩa” (semantic-rich) và dễ dùng cho BI.

    ```sql
    CREATE OR REPLACE TEMP VIEW first_open_date AS
    SELECT
        date_key              AS first_open_date_key,
        full_date             AS first_open_date,
        fiscal_year           AS first_open_fiscal_year,
        fiscal_quarter        AS first_open_fiscal_quarter,
        fiscal_month          AS first_open_fiscal_month,
        calendar_year         AS first_open_calendar_year,
        weekday_indicator     AS first_open_weekday_indicator,
        holiday_indicator     AS first_open_holiday_indicator
    FROM date_dim;
    ```

---

### 5. Promotion Dimension
* Một dòng sale = bán trong điều kiện khuyến mãi nào?

* Vấn đề cần lưu ý:
    * Có bán được nhiều hàng hơn so với bìnhh thường ko.
    * Có bị `ăn trước-hụt sau` ko. (Chỉ tăng trong lúc promo, sau promo hụt người mua luôn)
    * Cannibalization – Có ăn thịt sản phẩm khác không? (SP A tăng, SP B giảm 👉 Tổng category có thể không tăng.)
    * Có lời ko?

* Tạo 1 record placeholder để trỏ tránh null ko thể join hoặc làm User BI khó hiểu.

* Không để null các metrics. Hãy dùng giá trị "Unknown"

![alt text](images/Chap3/image-5.png)


---

### 6. Dimension khác
* **1** row fact trỏ đến **1** row dim -> Perfect

* Nhưng khi gặp 1 row fact trỏ đến nhiều dim **NHƯNG** grain đã tốt thì đừng back lại step 2. Thay vào đó **tạo fact khác** mặc dù chung business process
    * **VD**: 1 transaction có nhiều phương thức thanh toán
        * Ngoài fact_sales tạo thêm fact_payment:
        * `Type 1`: 
            | transaction_key | payment_method_key | payment_amount |
            | --------------- | ------------------ | -------------- |
            | T001            | Cash               | 30             |
            | T001            | Credit Card        | 50             |

        * `Type 2`: 
            | transaction_key | cash_amt | card_amt | voucher_amt |
            | --------------- | -------- | -------- | ----------- |


* **`Degenerate Dimension`**: Object nào chỉ còn mỗi key, không có descriptive attribute dùng để filter / group / label → thì KHÔNG cần (và KHÔNG nên) tạo dimension table 

---

### 7. Factless Fact Tables
* Factless fact table là để track những sự kiện **ko xảy ra như fact bình thường**

* Muốn theo dõi sản phẩm ko bán được. Ko như SQL expert có thể **giao hợp** tìm ra kết quả. Ở đây tạo fact mới luôn cho User BI dễ thao tác.

* Cũng có FK attribute như fact bình thường. Nhưng lại ko có metrics đo lường hay extended metrics. Đặc biệt có thêm dòng **Promotion Count** để đếm.

![alt text](images/Chap3/image-6.png)

---

### 8. Xử lý Key
* **Surrogate Key**: là key của DW cho DIM và fact join với nhau.
* **Natural Key**: là key của các source khác. Là **1 attribute** của của dim. Cùng 1 thực thể nhưng có nhiều nguồn nên **giữ đủ source**
    * `Cách 1`: Gắn mã nguồn vào key
        * VD: SAP|43251, CRM|6539152

    * `Cách 2`: Giữ nhiều natural key. Dim có thể nhiều cột natural key
    
* **Durable Key**: là key nhận diện 1 entity **MẶC CHO** entity xuất từ nguồn nào, từ phiên bảng nào (SCD), natural key ra sao.
    * VD: **cùng 1 customer** nhưng có sự khác nhau sau -> cần durable key.
        | Nguồn | Natural key |
        | ----- | ----------- |
        | CRM   | C123        |
        | ERP   | 99881       |
        | POS   | KH-77       |


---

### 9. Degenerate Dim
* Trong thực tế có trường hợp đa chi nhánh, mỗi chi nhánh dùng 1 máy POS. Khả năng các mấy POS sinh ra transaction_id trùng.

* Bảng `fact_sales` lúc này record bị trùng nhau. -> ban đầu `transaction_dim` được xem là **DD** -> lúc này cần tạo `transaction_dim` để chống trùng bằng cách gắn SK.

```text
Transaction ID
      |
      |-- unique, stable, small
      |      → Degenerate Dimension (cột trong fact)
      |
      |-- NOT unique / reused / bulky / BI cần
             → Tạo Dimension
             → Gán Surrogate Key
             → KHÔNG còn degenerate
```


---

### 10. Date Dimension Smart Keys
* Date Dim ko cần dùng SK vô nghĩa hãy dùng luôn yyyymmdd -> tốt cho partition, delete này kia.

* Nhưng **Smart Keys** ko phải là thứ dành User BI bỏ qua date dim

---

### 11. Fact Table Surrogate Key
Chỉ ứng dụng cho ETL ko phải user BI

* Giúp nhận diện 1 row ngay lập tức khi ETL
* Trong trường hợp load Fail track được dòng nào để tiếp tục

---

### 12. Centipede Fact Tables with Too Many Dimensions
* Tránh thiết kế bảng như con rết -> tốn disk + chậm query . Cán dẹp các bảng cùng nhánh (Hierarcy). 

* Phần lớn business process: 👉 < 20 dimensions



![alt text](images/Chap3/image-7.png)