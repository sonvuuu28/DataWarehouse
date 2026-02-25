# INVENTORY

## I. Value Chain 
* Value Chain là chuỗi các business process

* Mỗi business process sinh ra 1 hoặc vài fact.

* Ko ép làm đủ business process

![alt text](images/image-8.png)

## II. Mô hình Inventory
* Có 3 mô hình

### 1. Periodic Snapshot
* Ảnh chụp tồn kho **phổ biến nhất**

* Trả lời câu hỏi:
    * *Hôm nay/ Tuần này tồn bao nhiêu?*
    * *Xu hướng biến đổi theo thời gian?*

* Vấn đề của Periodic Snapshot là dữ liệu sinh ra đều đặn `cấp số cộng`. Lí do mỗi ngày đều cập nhật số lượng từng sản phẩm.
    * Tại sao hàng ko lấy ra vẫn phải cập nhật -> Vì đơn giản cho User BI, ko muốn họ lag/window func/join từa lưa ra.

* Giải pháp khi quá nặng
    * Giữ 60 ngày gần nhất
    * periodic lớn hơn -> tính theo tuần hẵn cập nhật

![alt text](images/image-9.png)


* **Nâng cấp fact** bằng thêm metrics mang tính business hơn. Dashboard chỉ group by date là ra insight
    * Nếu chỉ có số lượng chỉ trả lời được hàng tồn là bao nhiêu
    * Không trả lời được:
        * *Hàng chạy nhanh hay chậm?*
        * *Tồn này là tốt hay kẹt?*
        * *Bao lâu thì hết hàng?*

![alt text](images/image-10.png)


* `Quantity sold`: 
    * “Trong period này, hàng quay được mấy vòng?”
        * Hôm qua bán 20 hộp, 120 tồn / 20 bán mỗi ngày = 6 ngày nữa hết, Day of supply là 6, Nếu days of supply lớn nghĩa tồn kho kẹt
        * inventory_turns (daily) = quantity_sold / quantity_on_hand
        * inventory_turns (no-daily) = total_quantity_sold / average_daily_quantity_on_hand
        * tính được số vòng (turn) sẽ cân nhắc được có nên bán sản phẩm này nx ko đó.

* `Inventory Value at Cost`
    * tồn kho bị “đóng tiền” bao nhiêu
    * Bao nhiêu tiền đang nằm chết trong kho?

* `Inventory Value at Price`
    * = 120 hộp × 30k = 3.6 triệu
    * Họ so được: price value – cost value = margin tiềm năng

![alt text](images/image-11.png)


### 2. Transaction Inventory
* **Ghi mọi biến động (+ / –)**
    * Nhập kho
    * Xuất kho
    * Điều chỉnh
    * Move location

* Trả lời câu hỏi:
    * *“Hàng đã di chuyển như thế nào?”*
    * *Một ngày kho xử lý bao nhiêu giao dịch?*
    * *Vendor nào bị return vì lỗi nhiều nhất?*

* Transaction fact rất chi tiết, nhưng **KHÔNG** nên dùng nó một mình để phân tích tồn kho tổng thể → **phải có snapshot fact đi kèm**

* Tác giả đang cảnh báo `thiết kế sai` 👇
    * Ví dụ:
        * Receive: có vendor
        * Ship: có customer
        * Pick: không có vendor, không có customer

    * Nếu bạn cố nhét tất cả vào 1 transaction fact duy nhất
    * 👉 thì:
        * nhiều cột FK sẽ NULL loạn xạ
        * grain không còn sạch
        * BI user rất khó hiểu

    * 📌 Vì vậy:
        * Nếu các event có grain / dimension khác nhau → hãy **tách thành nhiều fact table Inventory**

``` text
fact_inventory_receive
- date_key
- product_key
- warehouse_key
- vendor_key
- quantity_received

fact_inventory_ship
- date_key
- product_key
- warehouse_key
- customer_key
- quantity_shipped


fact_inventory_pick
- date_key
- product_key
- warehouse_key
- quantity_picked
```


![alt text](images/image-12.png)

---

**`🎬 Bối cảnh thực tế`**

* Ngày: **2026-01-05**
* Kho: **WH01**
* Sản phẩm: **SKU_A**
* Vendor giao **100 units**
* Sau đó:

  * putaway vào kệ
  * pick 40 units
  * ship ra store

---

**`1️⃣ Inventory Transaction Type Dimension`**

| transaction_type_key | description         | group    |
| -------------------- | ------------------- | -------- |
| 1                    | Receive from Vendor | Inbound  |
| 2                    | Putaway to Bin      | Internal |
| 3                    | Pick for Shipment   | Internal |
| 4                    | Ship to Store       | Outbound |

---

**`2️⃣ Warehouse Inventory **Transaction Fact`****

👉 **1 dòng = 1 hành động**

| date_key | warehouse_key | product_key | transaction_type_key | transaction_number | quantity | dollar_amount |
| -------- | ------------- | ----------- | -------------------- | ------------------ | -------- | ------------- |
| 20260105 | WH01          | SKU_A       | 1                    | RCV1001            | +100     | +10,000       |
| 20260105 | WH01          | SKU_A       | 2                    | PUT1001            | 0        | 0             |
| 20260106 | WH01          | SKU_A       | 3                    | PICK2001           | -40      | -4,000        |
| 20260106 | WH01          | SKU_A       | 4                    | SHP3001            | -40      | -4,000        |

📌 Lưu ý:

* **Putaway** không đổi số lượng → quantity = 0
* Pick & Ship đều là movement → số âm
* Fact **không biết tồn kho hiện tại**

---

**`3️⃣ Inventory Snapshot Fact (để so sánh)`**

👉 **1 dòng = trạng thái cuối ngày**

| date_key | warehouse_key | product_key | quantity_on_hand |
| -------- | ------------- | ----------- | ---------------- |
| 20260105 | WH01          | SKU_A       | 100              |
| 20260106 | WH01          | SKU_A       | 60               |






### 3. Accumulating Snapshot
* Theo dõi một đối tượng từ lúc bắt đầu → qua các mốc → đến khi kết thúc, tất cả nằm trên 1 dòng fact

* Trả lời câu hỏi:
    * *Một lô hàng mất bao lâu từ nhận → xuất?*
    * *Kẹt ở bước nào lâu nhất? (inspection, bin, shipping)*
    * *% lô hàng chưa xuất sau 7 ngày*
    * *Warehouse nào xử lý chậm?*

* Thực hiện phép UPDATE khác với 2 cái kia only insert

![alt text](images/image-13.png)

![alt text](images/image-14.png)

---

Tóm lại, Transaction Fact xem được chi tiết từng hành động còn accumulate biết các mốc tgian 
(ở chỗ transaction có thế làm được nhưng viết SQL cả ngày)

### 4. Complementary Fact Table Types
* **Periodic snapshot** & **Accumulating**
    * Dùng bảng accumulating vận hành, viết job ETL cứ mỗi tuần/tháng tổng hợp 1 bảng periodic.
    * Khi hết tháng, bảng tích lũy trở thành bảng tháng trong snapshot, và bạn bắt đầu bảng tích lũy mới cho tháng tiếp theo.

* **Transaction** & **Snapshot**
    * Hai loại này không thể dễ dàng gộp vào cùng một bảng vì bản chất dữ liệu khác nhau.

* Đừng sợ dữ liệu dư thừa
    * Mỗi bảng cung cấp một góc nhìn khác nhau về cùng một câu chuyện kinh doanh.