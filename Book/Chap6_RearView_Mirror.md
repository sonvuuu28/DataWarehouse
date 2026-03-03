Đừng chỉ chăm làm model phản ánh quá khứ. Hãy làm model phản ứng động cơ.

```text
Fact_Order
- order_date
- product
- customer
- quantity
- revenue
```


```text
Fact_Sales_Call
- call_date
- sales
- customer
- duration

Fact_Quotation
- quote_date
- sales
- customer
- quote_amount

Fact_Lead
- lead_date
- sales
- source
```

Giờ BI làm được:

- Thấy sales activity giảm ➜ cảnh báo sớm

- Quote nhiều nhưng order ít ➜ vấn đề pricing

- Lead (Người có khả năng mua hàng) nhiều nhưng không convert ➜ vấn đề chất lượng lead