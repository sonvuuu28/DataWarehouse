# 4 STEPS 

## I. Quy trình 4 bước thiết kế DIM
### 1. Chọn quy trình (Business Process)
Quy trình là 1 hoạt động của doanh nghiệp, đặc điểm:
- `Business Process` thường là động từ vì đại diện cho các hoạt động. DIM mô tả bối cảnh liên quan event
- `Business Process` được hỗ trợ bởi operational process
- `Business Process` sinh ra các metrics
- `Business Process` có input và output

--- 

**Nhiệm vụ DE:**
- Lắng nghe câu hỏi của business để suy ra business process
- Các metrics đến từ process
- Process xảy ra ở đâu, khi nào

> VD: Tôi muốn xem doanh thu ???

--- 

**Lưu ý:**
- Ko nhầm lẫn quy trình là phòng ban

--- 

### 2. Xác định Grain
"Một row trong fact table mô tả sự kiện gì?"

Ví dụ grain đúng cách (nói bằng ngôn ngữ business):
- 1 dòng = 1 lần quét 1 sản phẩm trong 1 giao dịch bán hàng
- 1 dòng = 1 dòng chi tiết trong hóa đơn bác sĩ
- 1 dòng = 1 boarding pass được quét ở cổng sân bay
- 1 dòng = snapshot tồn kho mỗi ngày cho mỗi mặt hàng
- 1 dòng = 1 tài khoản ngân hàng mỗi tháng


--- 

### 3. Xác định Dimension
"Business mô tả sự kiện này như thế nào?"

Sau khi đã chốt grain (1 dòng fact là 1 sự kiện gì), thì bước này gần như… tự rơi ra.



---

#### 1️⃣ Dimension là “cách business kể câu chuyện”

Fact cho bạn **con số**
Dimension cho bạn **ngữ cảnh**

Dimension trả lời mấy câu quen thuộc:

* **Ai** làm?
* **Cái gì**?
* **Ở đâu**?
* **Khi nào**?
* **Vì sao**?
* **Bằng cách nào**?

👉 Đây chính là:

> *who / what / where / when / why / how*

---

#### 2️⃣ Nếu grain rõ → dimension rất dễ chọn

Ví dụ grain:

> “1 dòng = 1 sản phẩm được bán trong 1 giao dịch”

Thì dimension gần như hiện ra ngay:

* **Date** – bán lúc nào
* **Product** – bán sản phẩm gì
* **Customer** – ai mua
* **Store / Facility** – mua ở đâu
* **Employee** – ai bán
* **Channel** – online hay offline
* **Promotion** – bán vì chương trình gì

👉 Dimension **chỉ có 1 giá trị cho mỗi row fact**
(1 dòng bán hàng thì chỉ có 1 khách, 1 sản phẩm, 1 thời điểm…)

---

#### 3️⃣ Dimension = bảng chữ (text), không phải số

Sau khi chọn dimension, việc tiếp theo là:
👉 **liệt kê các thuộc tính mô tả (attributes)**

Ví dụ:

**Product dimension**

* product_name
* brand
* category
* subcategory
* size
* color

**Customer dimension**

* customer_name
* gender
* age_group
* city
* loyalty_level

👉 Dimension là nơi business **filter, group, slice, dice** dữ liệu.

---

### 4. Xác định Fact
> **Business process này đang ĐO cái gì?**

---

#### 1️⃣ Fact = thứ process đo được

Fact là:

* **chỉ số hiệu năng (performance metrics)**
* thứ mà business **muốn phân tích, so sánh, theo dõi**

Ví dụ:

* số lượng bán
* doanh thu
* chi phí
* thời gian xử lý

👉 Thường là **số** và **cộng được**.

---

#### 2️⃣ Fact PHẢI đúng grain (luật thép)

Kimball rất nghiêm chỗ này:

> **Mọi fact phải “true to the grain”**

Nghĩa là:

* Nếu grain là

  > “1 dòng = 1 sản phẩm được bán”
* thì fact chỉ được là:

  * quantity
  * sales_amount
  * discount_amount

❌ Không được:

* tổng doanh thu ngày
* tồn kho cuối tháng
* số dư tài khoản

👉 Fact nào thuộc **grain khác** → **fact table khác**

---

#### 3️⃣ Grain là hàng rào chống “fact lạc loài”

Grain đóng vai trò như **bảo vệ cửa** 🚪

* Fact đúng → vào
* Fact sai grain → mời đi chỗ khác

Nếu không có grain:

* Fact nào nhìn “có vẻ hợp” cũng bị nhét vào
* Fact table biến thành **nồi lẩu số liệu** 🍲

---

#### 4️⃣ Fact thường là số cộng được (additive)

Phần lớn fact:

* cộng theo time
* cộng theo product
* cộng theo customer

Ví dụ tốt:

* quantity
* revenue
* cost

Một số fact **không cộng hoàn toàn** (semi / non-additive) thì:

* càng cần grain rõ để không dùng sai

---

#### 5️⃣ Đừng thiết kế chỉ bằng cách nhìn source data

Kimball cảnh báo rất nặng đoạn này ⚠️

Sai lầm phổ biến:

* mở DB source
* thấy cột nào → cho vào fact
* thấy bảng nào → cho vào dimension

👉 Đây là **data-driven**, không phải **business-driven**.

Hệ quả:

* design nhìn “đủ cột”
* nhưng business không dùng được
* hoặc số liệu không trả lời đúng câu hỏi họ cần

---

#### 6️⃣ Phải kết hợp: Business + Source data

Thiết kế đúng là:

* nghe business muốn đo gì
* hiểu process ngoài đời
* rồi **so với**:

  * source có ghi nhận không
  * ghi nhận ở mức nào
  * có đáng tin không

👉 Không có business input → data **vô nghĩa**
👉 Không hiểu source → design **viển vông**

---
