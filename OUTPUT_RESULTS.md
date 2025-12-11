# Kết quả Output của DBT ETL Pipeline

## 📊 Tổng quan

Pipeline này biến dữ liệu thô (AdventureWorks2014) thành các bảng phân tích có giá trị cao, theo mô hình **Bronze → Silver → Gold**.

---

## 🏗️ Kiến trúc Layers

### **BRONZE Layer** (Raw Data)
**Mục đích:** Copy dữ liệu gốc từ source tables mà không thay đổi  
**Bảng:**
- `brnz_customers` - Dữ liệu khách hàng gốc
- `brnz_products` - Dữ liệu sản phẩm gốc
- `brnz_sales_orders` - Dữ liệu đơn hàng gốc

---

### **SILVER Layer** (Cleaned & Standardized)
**Mục đích:** Clean, transform, standardize dữ liệu  
**Bảng:**

#### 1. **slvr_customers**
```
INPUT: brnz_customers
TRANSFORMATIONS:
  - Đổi tên cột sang snake_case
  - Join với Person table để lấy FirstName, LastName
  - Handle NULL: coalesce → 'Unknown'
  - Tạo full_name = FirstName + ' ' + LastName

OUTPUT COLUMNS:
  ✓ customer_id
  ✓ first_name (cleaned)
  ✓ last_name (cleaned)
  ✓ full_name (concatenated)
  ✓ email_promotion
  ✓ store_id
  ✓ territory_id
  ✓ last_modified_date
```

#### 2. **slvr_products**
- Standardize product data từ bronze
- Handle missing descriptions & colors
- Calculate cost vs. list price

#### 3. **slvr_sales_orders**
- Enrich sales data với customer & product info
- Add discount flags
- Standardize order channels

---

### **GOLD Layer** (Business Metrics)
**Mục đích:** Aggregated metrics cho Business Analytics  
**Bảng:**

#### 1. **gld_customer_metrics** 📈
```
Mục đích: Phân tích khách hàng chi tiết

INPUT: slvr_customers + slvr_sales_orders

KỈ NƯỚC:
┌─────────────────────────────────┐
│ customer_id, full_name          │ ← Thông tin khách hàng
├─────────────────────────────────┤
│ total_orders                    │ ← Số đơn hàng
│ total_revenue                   │ ← Tổng doanh thu
│ avg_order_value                 │ ← Giá trị đơn hàng trung bình
│ total_items_purchased           │ ← Tổng sản phẩm mua
│ first_order_date, last_order_date │ ← Khoảng thời gian mua hàng
│ orders_with_discount            │ ← Số đơn có discount
└─────────────────────────────────┘

VÍ DỤ OUTPUT:
| customer_id | full_name      | total_orders | total_revenue | avg_order_value |
|-------------|----------------|--------------|---------------|-----------------|
| 1001        | John Doe       | 5            | $2,450.00     | $490.00         |
| 1002        | Jane Smith     | 12           | $8,900.00     | $741.67         |

USAGE:
- Segmentation: VIP customers, repeat buyers
- CLV (Customer Lifetime Value) analysis
- Churn prediction
```

#### 2. **gld_sales_summary** 📊
```
Mục đích: Daily/aggregated sales overview

INPUT: slvr_sales_orders

METRICS:
┌──────────────────────────────────┐
│ order_date                       │ ← Ngày bán
├──────────────────────────────────┤
│ total_orders                     │ ← Số lượng đơn
│ unique_customers                 │ ← Số khách hàng khác nhau
│ total_items_sold                 │ ← Tổng items
│ total_revenue                    │ ← Tổng doanh thu
│ avg_order_line_value             │ ← Giá trị order line trung bình
│ online_orders / offline_orders   │ ← Channel analysis
│ discounted_revenue               │ ← Doanh thu từ discount
└──────────────────────────────────┘

VÍ DỤ OUTPUT:
| order_date | total_orders | unique_customers | total_revenue | online_orders |
|------------|--------------|------------------|---------------|---------------|
| 2024-01-01 | 45           | 38               | $12,500.00    | 28            |
| 2024-01-02 | 52           | 41               | $14,200.00    | 35            |

USAGE:
- Sales dashboard (Tableau, Power BI)
- Trend analysis
- Channel performance (Online vs Offline)
- Discount impact analysis
```

#### 3. **gld_product_performance** 🎯
```
Mục đích: Phân tích hiệu suất sản phẩm

INPUT: slvr_products + slvr_sales_orders

METRICS:
┌────────────────────────────────────┐
│ product_id, product_name          │ ← Product info
│ subcategory_name, color           │ ← Category
│ list_price, standard_cost          │ ← Pricing
├────────────────────────────────────┤
│ total_orders                       │ ← Số lần bán
│ total_quantity_sold                │ ← Tổng units sold
│ total_revenue                      │ ← Doanh thu sản phẩm
│ avg_selling_price                  │ ← Giá bán trung bình
│ total_profit                       │ ← Lợi nhuận thực tế
│ profit_margin_pct                  │ ← % lợi nhuận
└────────────────────────────────────┘

VÍ DỤ OUTPUT:
| product_name    | total_revenue | total_profit | profit_margin_pct |
|-----------------|---------------|--------------|-------------------|
| Mountain Bike   | $50,000       | $20,000      | 40%               |
| Road Bike       | $35,000       | $12,250      | 35%               |

USAGE:
- Product portfolio analysis
- Pricing strategy
- Inventory optimization
- Sales target by product
```

---

## 📈 Data Flow

```
AdventureWorks2014 (Raw Source)
        ↓
    [BRONZE LAYER]
    - brnz_customers ✓
    - brnz_products ✓
    - brnz_sales_orders ✓
        ↓
    [SILVER LAYER]
    - slvr_customers (join + clean + standardize) ✓
    - slvr_products (clean + standardize) ✓
    - slvr_sales_orders (enrich + standardize) ✓
        ↓
    [GOLD LAYER]
    - gld_customer_metrics (customer analysis) ✓
    - gld_sales_summary (daily KPIs) ✓
    - gld_product_performance (product analysis) ✓
        ↓
    [BI / ANALYTICS]
    - Dashboards
    - Reports
    - Predictions
```

---

## 🎯 Ứng dụng thực tế

### **1. Marketing Team**
```
Dùng: gld_customer_metrics
- Tìm high-value customers → target campaigns
- Segment khách hàng theo revenue
- Identify churn risk customers
```

### **2. Sales Team**
```
Dùng: gld_sales_summary + gld_customer_metrics
- Daily/weekly sales performance
- Sales target tracking
- Channel performance (Online vs Offline)
```

### **3. Product Management**
```
Dùng: gld_product_performance
- Which products are profitable?
- Pricing recommendations
- Product mix optimization
```

### **4. Finance**
```
Dùng: gld_product_performance
- Revenue forecasting
- Margin analysis
- Cost optimization
```

---

## 📊 Số liệu từ AdventureWorks2014

Khi chạy `dbt run`, bạn sẽ tạo ra:

| Layer | Table | Row Count | Purpose |
|-------|-------|-----------|---------|
| Bronze | brnz_customers | ~19K | Raw customer data |
| Bronze | brnz_products | ~500 | Raw product data |
| Bronze | brnz_sales_orders | ~121K | Raw sales data |
| **Silver** | **slvr_customers** | **~19K** | Cleaned customer master |
| **Silver** | **slvr_products** | **~500** | Cleaned product master |
| **Silver** | **slvr_sales_orders** | **~121K** | Enriched sales transactions |
| **Gold** | **gld_customer_metrics** | **~19K** | Customer KPIs |
| **Gold** | **gld_sales_summary** | **~365** | Daily sales aggregation |
| **Gold** | **gld_product_performance** | **~500** | Product-level metrics |

---

## 🚀 Cách dùng kết quả

### **1. Query Gold tables trực tiếp:**
```sql
-- Top 10 customers by revenue
SELECT TOP 10 full_name, total_revenue, total_orders
FROM dbo.gld_customer_metrics
ORDER BY total_revenue DESC;

-- Today's sales performance
SELECT order_date, total_orders, total_revenue, online_orders
FROM dbo.gld_sales_summary
WHERE order_date = CAST(GETDATE() AS DATE);

-- Most profitable products
SELECT TOP 10 product_name, total_profit, profit_margin_pct
FROM dbo.gld_product_performance
ORDER BY profit_margin_pct DESC;
```

### **2. Connect BI tools (Tableau, Power BI, Looker):**
- Kết nối SQL Server
- Dùng Gold tables làm data source
- Tạo interactive dashboards

### **3. Export để tổng hợp báo cáo:**
```bash
# Export tới Excel/CSV
docker-compose exec sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost \
  -U imrandbtnew \
  -P Imran@12345 \
  -d AdventureWorks2014 \
  -Q "SELECT * FROM gld_customer_metrics" \
  -o report.csv
```

---

## ✅ Kết luận

**Kết quả chung của pipeline:**
✓ **3 tables Bronze** (raw copy)  
✓ **3 tables Silver** (cleaned + standardized)  
✓ **3 tables Gold** (business metrics)  
✓ **121K+ transactions** được transform & aggregate  
✓ **Ready for BI & Analytics**  
✓ **Scheduled chạy tự động qua Airflow**

Pipeline này cung cấp dữ liệu sạch, có cấu trúc, và aggregated metrics để hỗ trợ decision-making ở tất cả các bộ phận (Marketing, Sales, Product, Finance).
