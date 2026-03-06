# SubSystems & Techniques
## I. Requirement
Gồm 10 yêu cầu:

### 1. Business Needs
Business cần gì?
- KPI nào?
- Đo cái gì?
- Khi KPI tụt → họ drill-down vào đâu?

Suy ra:
- Report nào là core
- Report nào là nice-to-have
- Cái nào cần drill (theo ngày / sản phẩm / region)

### 2. Compliance – Có bị audit, soi, kiện không?

Câu hỏi ETL cần trả lời:

* Số này **từ đâu ra?**
* Có bị sửa không?
* Có chứng minh được không?

📌 Dẫn tới:

* audit dimension
* lineage
* raw data archive
* checksum / reconciliation

👉 Nếu công ty dính:

* tài chính
* thuế
* ngân hàng
* telecom

→ **ETL phải “giải trình được”**

---

### 3. Data Quality – Dữ liệu bẩn đến mức nào?
* field nào **biết chắc là bẩn**
* field nào **phải monitor liên tục**
* fix ở source hay fix ở ETL?

---

### 4. Security – Ai được xem cái gì?
* dữ liệu nhạy cảm nào?
* ai được xem?
* backup có mã hóa không?
* archive có an toàn không?

---

### 5. Data Integration – Có cần nối dữ liệu giữa hệ thống không?

Ví dụ:

* Customer ở CRM ≠ Customer ở ERP
* Product code mỗi hệ khác

📌 ETL phải:

* conform dimension
* thống nhất KPI

👉 Dùng **bus matrix** để quyết định:

* process nào bắt buộc join với process nào
* cái nào làm trước, cái nào để sau

---

### 6. Data Latency – Dữ liệu trễ bao lâu thì chịu được?

* daily
* intra-day
* near real-time
* real-time

> Batch → streaming **KHÔNG phải nâng cấp nhẹ**, mà là **đổi kiến trúc**

👉 Với bạn:

* daily report → ETL batch là OK
* đừng nhận bừa “realtime” nếu không cần

---

### 7. Archiving & Lineage – Có giữ data cũ không?

Kimball nói rất thẳng:

> **Lưu data rẻ hơn build lại ETL**

📌 Khuyến nghị:

* stage sau mỗi bước lớn
* archive raw + cleaned + conformed
* có metadata: data này từ đâu, xử lý thế nào

👉 Lý do:

* source có thể xoá
* logic ETL có thể thay đổi
* audit quay lại hỏi

---

### 8. BI Delivery Interfaces – Đưa data cho BI thế nào?

Quan điểm Kimball (rất gắt):

> ❌ Đưa normalized schema cho BI là vô trách nhiệm

ETL + modeling phải:

* đưa **dimensional model**
* query nhanh
* dễ dùng
* BI tool chạy mượt

📌 Phải chốt:

* expose bảng nào
* cube nào
* index, aggregate nào

👉 ETL **phục vụ BI**, không phải ngược lại

---

### 9. Available Skills – Team bạn làm nổi không?
* biết SQL tới đâu
* ETL tool gì
* có ai maintain không

---

### 10. Legacy Licenses – Bị ép dùng tool cũ không?


## II. Subsystem (Tóm tắt)
Gồm **34 subsystem** chia làm **4 nhóm lớn**:

### 1. Extract
- full extract
- incremental
- CDC

---

### 2. Clean & Conform (5 subsystems)
* Chuẩn hóa code
* Fix data bẩn
* Mapping master data
* Validate rule
* **Theo dõi lỗi chất lượng dữ liệu** (audit, reject, out-of-bound)

---

### 3. Load vào dimensional model (13 subsystems)
* Slowly Changing Dimension (Type 1/2/3)
* Fact load
* Accumulating snapshot update
* Surrogate key management
* Late arriving data

---

### 4. Vận hành ETL (13 subsystems)
* Retry
* Restart
* Logging
* Alert
* SLA
* Job dependency
* Metadata
* Versioning

---

## II. Subsystem chi tiết
### 1. Data Profiling
Tìm hiểu, đánh giá dữ liệu source
- **Bước 1**: Xem xét data source nào dùng được -> không ổn loại bỏ
- **Bước 2**: Hiểu data/table chi tiết. 
    - Vì có những data tốt cho production nhưng ko tốt cho analytics.
    - Field nào thiếu ổn định/thiếu dữ liệu, ..

> Đây sẽ là tiền đề cho phần data processing.

### 2. CDC
Ba cách đó là:

Có kiểu snapshot-based nữa.

---

**1. Log-based CDC (Debezium)**

Ý chính:

* Database nào cũng có transaction log
* Log ghi lại tất cả thao tác insert / update / delete
* CDC application đọc log để biết dữ liệu nào thay đổi

Vì sao phổ biến?

* Không cần query table
* Ít ảnh hưởng hiệu năng hệ thống nguồn

Nhược điểm:

* Mỗi DB có format log khác nhau
* Làm CDC cho nhiều DB khác nhau thì phức tạp

---

**2. Timestamp-based CDC** 

Ý chính:

* Table có cột thời gian (`updated_at`, `last_modified`)
* CDC tool query những record:

  * được tạo
  * hoặc sửa
  * sau một mốc thời gian

Ưu điểm:

* Dễ làm
* Dễ hiểu

Nhược điểm lớn:

* Query nhiều → tăng tải DB
* ❌ Không bắt được DELETE (vì record bị xóa thì timestamp cũng mất)

---

**3. Trigger-based CDC**

Ý chính:

* Tạo trigger trong DB
* Khi insert / update / delete xảy ra:

  * trigger chạy
  * ghi dữ liệu thay đổi vào change table / shadow table

Ưu điểm:

* Dễ triển khai
* Bắt được cả delete

Nhược điểm:

* Mỗi transaction đều kích hoạt trigger
* → tốn tài nguyên DB
* Có thể ảnh hưởng OLTP