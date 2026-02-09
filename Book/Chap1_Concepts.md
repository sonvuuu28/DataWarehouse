# DW/BI/Dimensional Model
1. DW/BI xuất phát từ business question ko phải vì tool, tech, data.
2. Simplicity is a key. 
- User dễ hiểu
- Query Optimization
- System sẽ sống lâu

## Mục lục
[I. Different Worlds of Data Capture and Data Analysis](#i-different-worlds-of-data-capture-and-data-analysis)

[II. Goals of Data Warehousing and Business Intelligence](#ii-goals-of-data-warehousing-and-business-intelligence)

## I. Different Worlds of Data Capture and Data Analysis
| Tiêu chí                   | Operational Systems (OLTP)           | DW / BI Systems                |
| -------------------------- | ------------------------------------ | ------------------------------ |
| **Mục đích**               | Ghi nhận & vận hành nghiệp vụ        | Phân tích & ra quyết định      |
| **Vai trò**                | *Put data in*                        | *Get data out*                 |
| **Kiểu workload**          | Transaction ngắn, lặp lại            | Query dài, ad-hoc              |
| **Đơn vị xử lý**           | 1 transaction / 1 record             | Hàng trăm → hàng triệu records |
| **Tối ưu cho**             | Write nhanh, concurrency cao         | Read & aggregation nhanh       |
| **Lịch sử dữ liệu**        | Ít hoặc không lưu                    | Bắt buộc lưu lịch sử           |
| **Tính ổn định query**     | Query cố định                        | Query thay đổi liên tục        |

`Lưu ý:`
- Copy nguyên OLTP sang hệ khác rồi gọi đó là Data Warehouse -> SAI
- Chấp nhận: Denormalize, Dimensional Model, Aggregation, History.

## II. Goals of Data Warehousing and Business Intelligence
#### `Vấn đề`
- "We collect tons of data, but we can’t access it."
- "We need to slice and dice the data every which way." (Thích tư duy cắt lát 1 câu dài giữ keyword)
- "Business people need to get at the data easily."
- "Just show me what is important."
- "We spend entire meetings arguing about who has the right numbers rather than making decisions."
- "We want people to use information to support more fact-based decision making."

#### `7 mục tiêu`
1. Dễ truy cập – dễ hiểu – nhanh
2. Nhất quán & đáng tin
3. Thích nghi với thay đổi
4. Kịp thời (TIMELY)
5. Bảo mật dữ liệu
6. Nền tảng ra quyết định
7. Business phải chấp nhận & sử dụng

#### `Metaphor`
Kimball ví von người DW/BI designers là những người làm báo. Publisher:
- Biết độc giả là ai
- Họ muốn đọc gì
- Ai là nhóm độc giả "giá trị cao"
- Mở rộng tập độc giả
- Chọn bài hay
- Layout dễ đọc
- Văn phong nhất quán
- Kiểm chứng độ chính xác
- Cập nhật theo thay đổi của độc giả

**Đừng** đánh đổi trải nghiệm đọc vì tối ưu vận hành, tool xây dựng, schema rối rắm làm người đọc chẳng hiểu gì.

## III. Dimensional Modeling Introduction
Vẫn mục tiêu giữ chân người đọc, ko phức tạp hóa layout.

| Tiêu chí | 3NF / Normalized | Dimensional |
|--------|------------------|-------------|
| Mục tiêu | Giao dịch, cập nhật | Phân tích, ra quyết định |
| Số bảng | Nhiều | Ít |
| Join | Phức tạp | Đơn giản |
| Business hiểu | ❌ | ✅ |
| BI query | Chậm | Nhanh |

### `Star Schemas VS OLAP Cubes`
- Star schema: Dimensional model triển khai trên RDBMS
- OLAP cube: Dimensional model triển khai trên multi-dimensional

**Star Schema** chủ yếu quản lý cấu trúc dữ liệu và business semantics cốt lõi, chỉ làm giàu ở mức độ có kiểm soát.
Việc cố gắng nhồi quá nhiều logic phân tích vào star schema sẽ khiến mô hình phức tạp, khó bảo trì và dễ gãy khi business thay đổi.

**OLAP Cube** được xây dựng trên star schema để triển khai thêm các góc nhìn kinh doanh, thường dưới dạng flattened / pre-aggregated views, giúp người dùng business drill-down / roll-up trực quan, nhanh và dễ hiểu hơn.


### `Fact Table`
- 1 row là 1 event busisness
- Nên là số 
- Nhiều FK
- Ít cột

### `Dimension Table`
- Dimension = **ngữ cảnh mô tả** cho fact  
  → trả lời: **who / what / where / when / how / why**


## IV. Kimball Architecture
### `Định nghĩa:`
Kimball ko thích chuẩn hóa 3NF trong quá trình ETL mặc dù dữ liệu đầu vào có thể đến từ các hệ thống 3NF hoặc các tập tin dữ liệu rời rạc. 

Kimball thích gì business hơn cho kết quả ngay, lấy luôn cột dữ liệu cần thiết làm star schema luôn.

### `Steps:`
1. Thống nhất các định nghĩa và business rules dùng chung (ví dụ: Customer, Product, Date, Revenue…) để tránh tình trạng mỗi phòng ban hiểu cùng một thực thể theo cách khác nhau.

2. Xây dựng các data mart dưới dạng dimensional models và sử dụng conformed dimensions.

3. **Tập hợp các data mart tạo thành Enterprise Data Warehouse**
   Một hoặc nhiều data mart, miễn là được thiết kế đúng chuẩn và tích hợp thông qua conformed dimensions, **đều được xem là Data Warehouse theo quan điểm Kimball**.

### `Chú ý`
Kimball ko cực đoan, gây cho các phòng ban khó chịu khi thống nhất rule. Vì nghiệp vụ mỗi phòng sẽ mỗi khác. 

Kimball cho thêm các cột boolean, extend table, snowflake nhẹ.

Tuân theo quy tắc 80-20:
- 80 dùng core dim. 
- 20 cho nghiệp vụ đặc thù

Nên tập trung vào dim trước. (Ngồi bàn trước dim là gì với business sau đó mới build fact)

Quan tâm đến nghiệp vụ kinh doanh thay vì report.

```
Khi business nói:

    “Tôi cần báo cáo doanh thu theo khu vực”

Kimball dịch ngược thành:

    Business process nào? → Sales

    Measurement event? → Order / Invoice

    Grain? → 1 dòng hàng / 1 đơn

    Dimension? → Customer, Product, Date, Region
```
---

## V. Bus Matrix
Bản đồ cho biết business process nào dùng chung dimension nào

``Team IT + Business team``

| Business Process \ Dimension | Date | Customer | Product | Store | Channel |
| ---------------------------- | ---- | -------- | ------- | ----- | ------- |
| Sales                        | ✅    | ✅        | ✅       | ✅     | ✅       |
| Inventory                    | ✅    | ❌        | ✅       | ✅     | ❌       |
| Shipment                     | ✅    | ✅        | ❌       | ✅     | ❌       |
| Returns                      | ✅    | ✅        | ✅       | ✅     | ❌       |





