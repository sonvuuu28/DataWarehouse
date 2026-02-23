# TECHNIQUE

## I. Fundamental Concepts
### 1. Thu thập yêu cầu nghiệp vụ & thực tế dữ liệu
1. Business Process: làm việc với business team xem họ cần gì.
2. Data Realities: Họp với data source expert xem dữ liệu có gì.
3. Cân bằng nhu cầu với những gì ta có.


### 2. Thiết kế cùng với team business
* Data modeler là người làm model
* Business Stakeholder là người xác nhận đúng/sai
* Data governance đảm bảo ngữ nghĩa thống nhất
=> tránh đúng kỹ thuật sai nghiệp vụ


### 3. Quy trình thiết kế Dimensional 4 bước
1. Chọn business process
2. Khai báo grain 
3. Xác định dim
4. Xác định fact

### 4. Business Process
Là 1 hoạt động vận hành, sinh ra facts (Tương ứng 1 dòng trong Bus Matrix)

**VD**: Nhận đơn, xử lý khiếu nại bảo hành, đăng ký học,..

### 5. Grain
Là 1 dòng của bảng fact. Nó đại diện event của hoạt động vận hành đó.

Grain phải được xác định trước khi chọn dimension và fact.

### 6. Dim
Bảng thực thể liên quan đến 1 event giúp hiểu bối cảnh.

Trả lời Ai? Cái gì? Ở đâu? Khi nào? Vì sao? Như thế nào?

Tốn công nhất nhưng quyết định trải nghiệm BI.

### 7. Fact 
Là số **kết quả** đo lường của 1 sự kiện nghiệp vụ

Thường là numeric

### 8. Star Schema & OLAP Cube
Star Schema lưu trong RDBMS gồm bảng fact và dim

Olap Cube giống cái report, phục ad-hoc.

## II. Fact Table Technique
### 1. Ba loại numberic
1. Additive
- Là cột cộng được theo mọi chiều
- VD: Doanh thu cộng bao nhiêu row đều được(Theo ngày, 10 ngày, 1 tháng,...).

2. Semi-additive
- Là số đo trạng thái tại 1 thời điểm
- VD: Số dư tài khoản, Tồn kho (Ko cộng được với ngày khác chỉ lấy lastest record)

3. Non-additive
- Là tỷ lệ, trung bình, phần trăm


### 2. Null
Fact table cho phép có null. Nhưng `FK` ko được là null. Trong dim nên có 1 dòng đặc biệt:
- Unknow
- Not Applicable

### 3. Conformed Facts
2 fact có cột cùng tên <=> cột đó cùng ý nghĩa nghiệp vụ.

VD: Fact_sale, fact_billing
có chu g cột sale_amount <=> tiền thu được chưa tính thuế.

### 4. Ba loại fact
1. `Transaction`

    Mỗi dòng là 1 sự kiện => data sinh ra khi có sự kiện xảy ra.
    
    **VD**: 1 hóa đơn, 1 cú click,..

2. `Periodic Snapshot`

    Mỗi dòng là 1 snapshot theo thời gian => data sinh ra theo chu kỳ.

    **VD**: Tồn kho cuối ngày, Số dư tài khoản cuối tháng, ...


3. `Accumulating Snapshot`

    Mỗi dòng là 1 event của 1 quy trình  => data sinh ra nhưng được update nhiều lần.

    Fact table duy nhất cho phép update

    **VD**: đơn hàng, hồ sơ, ..

### 5. Factless Fact Table
Factless fact table = bảng đại diện cho MỘT LOẠI EVENT
dùng để xác nhận sự tồn tại của event đó,

KHÔNG chứa số liệu tính toán


VD: Fact_Attendance
| date_key | student_key | class_key | teacher_key |
| -------- | ----------- | --------- | ----------- |
| 20240201 | 101         | 501       | 12          |



### 6. Aggregate Fact Table
- Fact đã được tổng hợp sẵn
- Mục tiêu duy nhất: chạy nhanh

### 7. Consolidated Fact Table
Gộp nhiều nghiệp vụ cùng vào 1 fact. Miễn  cùng grain.

VD: Doanh thu thật, doanh thu dự đoán

## III. DIM Table Technique
### 1. Surrogate Key
### 2. Không để null
### 3. Calendar Date Dimension
- Bảng dim này ko cần Surrogate Key
- BẮT BUỘC có 1 dòng đặc biệt: Unknown cho data thiếu ngày.

```
date_key (YYYYMMDD) -- Ngoại lệ duy nhất được dùng key có ý nghĩa
full_date
day_of_week
week_number
month
quarter
year
fiscal_period
is_holiday
holiday_name
```

### 4. Junk Dimension
Dim chứa thuộc tính lặt vặt làm gọn fact


## IV. Integration via Conformed Dimensions
### 1. Conformed Dimensions
Hai bảng fact dùng chung bảng dim => dim là conformed dim

=> Có thể làm report của 2 fact theo thuộc tính dim

**VD**: Sales vs Inventory theo Brand(Dim product)

### 2. Bus Architecture
Kiến trúc xây dựng DWH theo từng nghiệp vụ dùng chung dim.


### 3. Bus Matrix
| Business Process ↓ / Dimension → | Date | Product | Customer | Store |
| -------------------------------- | ---- | ------- | -------- | ----- |
| Sales                            | ✅    | ✅       | ✅        | ✅     |
| Inventory                        | ✅    | ✅       | ❌        | ✅     |
| Procurement                      | ✅    | ✅       | ❌        | ❌     |


Dùng để:
- Dim nào cần conformed
- Build nào trước
- Nói chuyện với business


### 4. Detailed Bus Matrix
Bus matrix nhưng chi tiết hơn:
- 1 process → nhiều fact
- ghi rõ grain
- ghi rõ measure


### 5. Stakeholder Matrix
Bảng để xác định nghiệp vụ gắn với phòng ban 

| Process   | Sales | Finance | Marketing |
| --------- | ----- | ------- | --------- |
| Sales     | ✅     | ✅       | ✅         |
| Inventory | ❌     | ✅       | ❌         |


Dùng để:
- Mời ai họp
- Ai là stackholder


## V. Slowly Changing Dimension
| Type | Lịch sử      | Tăng row | Cơ Chế             |
| ---- | ------------ | -------- | -------------------- |
| 0    | ❌            | ❌        | Không cho update chỉ insert 1 lần |
| 1    | ❌            | ❌        | Đè lên giá trị cũ |
| 2    | ✅            | ✅        | Thêm dòng mới, cần cột `effective_date`, `expiration_date`, `current_flag` |
| 3    | ⚠️ hạn chế   | ❌        | 1 current và 1 quá khứ, `current`, `previous` |
| 4    | ✅            | ✅      | Tách nhóm thuộc tính hay thay đổi ra bảng riêng, Fact chứa FK của cả main Dim và mini Dim |
| 5    | ✅ + hiện tại | ✅      | Tách nhóm thuộc tính hay thay đổi ra bảng riêng, Fact chứa FK của main Dim, main Dim chứa mini Dim (SnowFlake)  |
| 6    | ✅ + hiện tại | ✅        | Mỗi record đều lưu type ban đầu và type hiện tại, giúp xem hiện tại |
| 7    | ✅ + hiện tại | ✅        | Dim chứa SK và DK, SK xem quá khứ cụ thể, DK coi full         |


## VI. Advanced Fact Table Techniques
1. SK có hoặc ko đều được
2. Tránh Centipede Fact Table (dim junk)
3. Data số có thể thuộc fact hoặc dim (tính toán, conversion ở case 8 -> fact, Filter -> Dim)
4. Tính lag (Ở accumulating snapshot tránh để user tính tay -> hãy dùng lag)
5. Làm chi tiết ko làm header (Thay vì Orders, Order_Items -> làm 1 th)
6. Fact table lỗ lãi -> hãy để mọi thứ ổn định và được chấp nhận bởi doanh nghiệp mới làm
7. Đa tiền tệ (Nên có 1 cột tiền gốc và tiền quy đổi, có thêm dim_currency để biết cái nào là tiền gốc)
8. Đa hệ đo. Mỗi phòng ban muốn xem 1 đơn vị đo khác nhau. Ví dụ: Theo chai, Theo thùng, Theo Công. 
    - Chọn 1 hệ đo chuẩn. VD: theo chai
    - Cài các số chia. VD: Bao chai 1 thùng (24). Đây là số chia

9. ko nên tính chỉ số YTD ở DW. Hãy để cube/BI tính. vì nhu cầu sẽ thay đổi nhiều.

10. Ko join 2 fact table vì fact value là line ko phải header. Join gây sai dữ liệu. Nên aggregated trc khi join -> gọi là drill across.
11. Timespan ở Fact (SCD type 2) chỉ nên dùng khi fact cập nhật hằng ngày là vô nghĩa.

| date  | product | warehouse | on_hand_qty |
| ----- | ------- | --------- | ----------- |
| 01-01 | X       | A         | 100         |
| 01-02 | X       | A         | 100         |
| 01-03 | X       | A         | 100         |
| 01-04 | X       | A         | 100         |

=> áp dụng SCD type 2 khi nào có data thì add rồi thêm cột expiration_date, effective_date.

## VII. Advanced Dim Techniques
1. `Dim Join Dim gặp SCD type 2 => Broke.`

    VD: Customer_Dim chứa FK của Geography_Dim. Khi Geo_Dim có SCD type 2, thay đổi vùng mới thì để record => customer thêm record => phình to.

    => Cách giải quyết đưa FK cho Fact giữ. tránh dim join dim

2. `Bridge table`

    Giải quyết vấn many-to-many



    ```text
    fact_visit
    └── diagnosis_group_key
            ↓
    bridge_diagnosis
            ↓
    diagnosis_dim
    ```

3. `Bridge Table có SCD type 2 để track lịch sử`

4. Label để DIM ko fact

5. Khi mà phân nhóm cluster nên tạo 1 bảng riêng chứa PK, 1 bảng định nghĩa loại cluster để nghiên cứu nhóm đó. Dễ giao, hợp để mining data

    VD: Khách hàng sẽ rời bỏ doanh nghiệp trong quý 1. Tạo bảng customer_churn_Q1 chứa khóa durable của các customer. Việc này giúp ta theo dõi lịch sử dễ dàng hơn, người làm BI dễ tương tác hơn.

```text
study_group
-----------
study_group_code      -- CHURN_Q1_2025
study_group_name      -- Customers likely to churn in Q1
as_of_date
description
status


study_group_member
------------------
study_group_code
customer_durable_key

```

6. BI User thích dimension chứa luôn các metric phânb band (chi tiêu trên 100M, tổng chi tiêu HIGH)


7. Thay vì quy ước trước trong document ta để quy ước vào 1 table phục vụ định nghĩa phân band

8. Nếu ứng dụng đa múi giờ làm dual FK

9. Step dim để biết user xài tới đâu thì stop

```
dim_step
--------
step_key
step_number        -- bước thứ mấy
steps_remaining    -- còn mấy bước nữa

```

còn 1 kiểu nữa là 1 dòng fact chứa đủ quy trình, mỗi cột gắn datatime vô luôn

|            | Accumulating Snapshot | Step Dimension       |
| ---------- | --------------------- | -------------------- |
| Grain      | 1 process             | 1 step               |
| Số dòng    | Ít                    | Nhiều                |
| Update     | UPDATE nhiều lần      | INSERT-only          |
| Phù hợp    | Batch ETL             | Realtime / streaming |


10. Hot swappable dim trong finance


11. No generic table

1 bảng có type để định user là employee/customer => ko được 


12. Audit_dim cho biết record từ đợt ETL, lúc nào, end khi nào,...

13. Ghi fact trước mặc cho dim đã tồn tại hay chưa. Điều chỉnh sau khi mọi thứ hoàn thành


---
---


# Special Purpose Schemas

Các **design pattern** dưới đây được dùng cho **những use case đặc biệt**.

---

## 1. Supertype & Subtype Schemas

### (For Heterogeneous Products)

* **Bối cảnh**

  * Financial services và nhiều ngành khác thường có **rất nhiều loại sản phẩm khác nhau**.
  * Ví dụ: ngân hàng bán lẻ có:

    * checking accounts
    * mortgages
    * business loans
  * Tất cả đều là **account**, nhưng:

    * facts khác nhau
    * attributes khác nhau

* **Vấn đề**

  * Cố xây **1 fact table duy nhất**:

    * chứa union của *tất cả facts*
    * join với dim chứa *tất cả attributes*
  * → **Sẽ thất bại**

    * vì có **hàng trăm facts & attributes không tương thích**

* **Giải pháp**

  * Xây:

    1. **Supertype fact table**

       * Chỉ chứa **phần giao nhau (intersection)** của facts giữa các account types
       * Đi kèm **supertype dimension table** chứa các attributes chung
    2. **Subtype fact tables**

       * Mỗi subtype có:

         * fact table riêng
         * dimension tables riêng

* **Tên gọi khác**

  * Supertype / Subtype fact tables
  * = **Core fact tables / Custom fact tables**

* **Tham chiếu**

  * Chapter 10 – Financial Services (p.293)
  * Chapter 14 – Healthcare (p.347)
  * Chapter 16 – Insurance (p.384)

---

## 2. Real-Time Fact Tables

* **Đặc điểm**

  * Cần được **update thường xuyên hơn** so với batch nightly truyền thống

* **Kỹ thuật triển khai**

  * Phụ thuộc vào:

    * DBMS
    * hoặc OLAP cube
  * Ví dụ:

    * **Hot partition**

      * Một partition của fact table được:

        * pin trong physical memory
        * **không build index**
        * **không build aggregation**
    * **Deferred updating**

      * Cho phép query hiện tại chạy xong
      * Sau đó mới thực hiện update

* **Tham chiếu**

  * Chapter 8 – Customer Relationship Management (p.260)
  * Chapter 20 – ETL System Process and Tasks (p.520)

---

## 3. Error Event Schemas

* **Mục tiêu**

  * Quản lý **data quality** trong data warehouse

* **Cách làm**

  * Dùng hệ thống:

    * data quality screens
    * data quality filters
  * Kiểm tra dữ liệu khi:

    * data chảy từ source systems
    * sang BI platform

* **Khi phát hiện lỗi**

  * Mỗi lỗi được ghi nhận thành **một event**
  * Event này được lưu trong:

    * **một dimensional schema đặc biệt**
    * **chỉ dùng trong ETL back room**

* **Cấu trúc schema**

  1. **Error Event Fact Table**

     * Grain: **1 error event**
  2. **Error Event Detail Fact Table**

     * Grain:

       * từng column
       * của từng table
       * tham gia vào error event

---
