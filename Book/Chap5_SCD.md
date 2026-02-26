# SCD
SCD là thay đổi dữ liệu các entity (hay các attribute của DIM). Nhưng nên bàn trước các bên business entity nào, thuộc tính nào cần được track lịch sử, cái nào ko cần.
- Ko có lịch sử ko đc business chấp nhận.
- Thừa lịch sử nặng query.

## I. 4 Basic SCD
### 0. Type 0 - Retain Origin
Giữ nguyên trạng thái ban đầu:
- **Attribute**: logic ko đổi
- **VD**: dim_date, durable_key,...

---

### 1. Type 1 - Overwrite
**Ghi đè** giá trị, chỉ **quan tâm hiện tại** dim, **không giữ lịch sử**.

| Product Key | SKU      | Product     | Department |
| ----------- | -------- | ----------- | ---------- |
| 12345       | ABC922-Z | IntelliKidz | Education  |
| 12345       | ABC922-Z | IntelliKidz | Strategy   |


Nên dùng type 1 khi muốn **sửa lỗi chính tả** hoặc **đổi chuẩn hóa** 
- "Educaiton" → "Education"
- "HCM City" → "Ho Chi Minh City"
👉 Lịch sử không có giá trị phân tích


`❗Cảnh báo`:
- Report sẽ thay đổi. Các phép tính tổng hợp toang.
- VD:
    - Ngày 31/1, Product IntelliKidz còn ở Education
    - Ngày 1/2, Bạn dùng Type 1 → ghi đè sang Strategy
    - Các bảo doanh thu từng department sẽ khác với trước ngày 31/1

---

### 2. Type 2 - Add New Row
Thêm 1 dòng mới vào dim, lưu lịch sử, các cột có giá trị mới:
- `Surrogate Key`
- `Effective Date`
- `Expire Date`
- `Current`

**Không** dính lỗi của **Type 1** nữa

`❗Cảnh báo`:
- Count theo **SK** sẽ **sai**. Count theo Natural Key **DISTINCT**

---

**Trước khi đổi**

    | Product Key | SKU      | Product     | Dept      | Eff Date   | Exp Date   | Current |
    | ----------- | -------- | ----------- | --------- | ---------- | ---------- | ------- |
    | 12345       | ABC922-Z | IntelliKidz | Education | 2012-01-01 | 9999-12-31 | Y       |


**Sau khi đổi**

    | Product Key | SKU      | Product     | Dept      | Eff Date   | Exp Date   | Current |
    | ----------- | -------- | ----------- | --------- | ---------- | ---------- | ------- |
    | 12345       | ABC922-Z | IntelliKidz | Education | 2012-01-01 | 2013-01-31 | N       |
    | 25984       | ABC922-Z | IntelliKidz | Strategy  | 2013-02-01 | 9999-12-31 | Y       |


---

### 3. Type 3 - Add New Attribute
- Xem **1** phiên lịch sử trước thôi

    | Product Key | Current Dept | Prior Dept |
    | ----------- | ------------ | ---------- |
    | 12345       | Strategy     | Education  |

- **Ý nghĩa**:
    - Người dùng có 2 góc nhìn, Dùng fact join nếu theo prior thì sao, theo current thì sao.

- **Type 3 dùng khi** 
    - Department trước tái cấu trúc
    - District trước sáp nhập
    - Territory 2022

- Type 3 mở rộng
    - Nhiều cột:
        - Current Department
        - 2023 Department
        - 2022 Department
        - 2021 Department

----

### 4. Type 4 - Add Mini-Dimension
- **`Vấn đề`**:
    - **DIM gốc to** quá  
    - **Attribute thường xuyên đổi** (score, thu nhập tuổi,.. ) 
- -> ko thể chơi **type 2** vì đổi add thêm dòng phình dim. 

- **`Giải pháp`**:
    - Mở thêm 1 dim khác
    - Lôi thuộc thay đổi, độ nhiễu cao (cái nào ok thì lôi ko ép).
    - Chia band/bucket

![alt text](images/Chap5/image-5.png)

- **`Trade-off`**:
    - Trả lời câu hỏi theo nhóm ko theo chi tiết unique nên cân nhắc thuộc tính lôi ra.

- **`Lưu ý`**:
    - Có thể cắm FK demographic_dim cho customer_dim. (Type 5)
    - Khác với 2 và 3 dim nắm giữ lịch sử hoàn toàn. Ở 4 đã có sự giúp sức của fact để lưu lịch sử

----

## II. 3 Hybrid SCD
### 5. Type 5 = 4 + 1
- Type 4 tách attribute thay đổi nhiều ra dim, chia band
- Type 1 Overwrite, no care lịch sử
---
Đinh nghĩa lại:
- Áp dụng Type 4 tách attribute thay đổi nhiều ra dim, chia band
- Ở bảng dim gốc, customer_dim gắn thêm FK
- Khác lạ maybe User BI ko thích

**`Trả lời câu hỏi`**:
- Doanh thu 2022 của những customer hiện đang thu nhập High
    - Fact 2022
    - Join Customer → Current Demographics Key
    - Xem thử hiện tại thu nhập họ sao vì FK trỏ tới dim_demographic hiện tại.
- Hiện tại có bao nhiêu customer:
    - Age band 26–30
    - Income medium

![alt text](images/Chap5/image-6.png)

----

### 6. Type 6 = 2 + 3
- Type 2: có đầy đủ lịch sử, thêm row mới và có các cột thay đổi giá trị: SK, Effective_date, Expire_Date, is_current
- Type 3: Thêm cột previous nữa

---
- **`Lưu ý`**
    - Update current_attribute của tất cả row khi có row mới được add.

![alt text](images/Chap5/image-7.png)

- Có thể thấy mỗi khi có row mới được add và value hiện tại bị đổi. Tất cả cột current đều đổi theo value hiện tại

----

### 7. Type 7 
- Sinh ra do type 6 ko thể đáp ứng nếu BI user xem prev, current của 100 thuộc tính.
- **`Cách làm`**
    - Tạo 1 view chỉ chứa current, user làm việc 90% ở đây
    - Khi cần lịch sử họ sẽ joint fact, dim chính sau
----
- Fact table sẽ giữ 2 khóa cho cùng 1 dimension:
    - Product Key (SK)
        - → trỏ vào dimension Type 2
        - → dùng khi muốn xem giá trị lịch sử đúng tại thời điểm phát sinh fact

    - Durable Product Key (DK / NK bền vững)
        - → trỏ vào dimension Type 1 (current)
        - → dùng khi muốn xem giá trị hiện tại của product, bất kể fact xảy ra lúc nào

![alt text](images/Chap5/image-8.png)

## III. Report Type 7 đặc biệt
- 1 câu hỏi cực dị: Muốn xem dữ liệu tại thời điểm bất kì nhưng theo kiến trúc hiện tại
    - VD: 2022 team A thuộc tổ chức Sales VN, nhưng sau đó team A thuộc tổ chức Sales SEA
    - NHƯNG, muốn xem giả sử team A vẫn thuộc tổ chức cũ thì bây giờ doanh thu team cũ sẽ ra sao


**`dim_org`**
| org_sk | org_dk | org_name | parent_org | effective_date | expiration_date |
| ------ | ------ | -------- | ---------- | -------------- | --------------- |
| 101    | A      | Team A   | Sales VN   | 2022-01-01     | 2023-06-30      |
| 102    | A      | Team A   | Sales SEA  | 2023-07-01     | 9999-12-31      |
| 201    | B      | Team B   | Sales VN   | 2022-01-01     | 9999-12-31      |


**`fact_sales`**
| sale_date  | amount | org_sk_at_event | org_dk |
| ---------- | ------ | --------------- | ------ |
| 2022-03-01 | 100    | 101             | A      |
| 2022-11-01 | 150    | 101             | A      |
| 2024-01-10 | 200    | 102             | A      |
| 2022-05-01 | 120    | 201             | B      |
| 2024-02-01 | 180    | 201             | B      |


**`Fact sau khi join`**
| sale_date  | amount | org | parent_org (as-of 01-12-2022) |
| ---------- | ------ | --- | ----------------------------- |
| 2022-03-01 | 100    | A   | Sales VN                      |
| 2022-11-01 | 150    | A   | Sales VN                      |
| 2024-01-10 | 200    | A   | Sales VN ❗                    |
| 2022-05-01 | 120    | B   | Sales VN                      |
| 2024-02-01 | 180    | B   | Sales VN                      |


---

Bước 1: Filter dimension tại ngày đó

Bước 2: Join fact bằng durable key


---

## IV. Tóm lại
| Câu hỏi                    | SCD phù hợp    |
| -------------------------- | -------------- |
| Lúc đó là sao?             | Type 2 / 4     |
| Hiện tại là sao?           | Type 1         |
| So sánh lúc đó vs hiện tại | Type 5 / 6 / 7 |
| As-of bất kỳ trong quá khứ | **Type 7**     |
