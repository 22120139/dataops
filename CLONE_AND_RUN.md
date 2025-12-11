# Hướng dẫn Clone và Chạy Project

## Yêu cầu ban đầu
- **Docker Desktop** (đã cài & chạy)
- **Git** (để clone)
- **Terminal/Bash** (Windows: PowerShell hoặc WSL)

## Các bước chạy

### Bước 1: Clone repository
```bash
git clone https://github.com/22120139/dataops.git
cd dataops
```

### Bước 2: Build Docker images
```bash
docker-compose build --no-cache
```
⏱️ **Mất ~5-10 phút** (phụ thuộc vào tốc độ internet & máy)

### Bước 3: Start services
```bash
docker-compose up -d
```

### Bước 4: Initialize Database (SQL Server)
Chờ **1-2 phút** để SQL Server khởi động xong, rồi chạy:

```bash
docker-compose exec sqlserver /tmp/restore_db.sh
```

**Output mong đợi:** `Restore is complete on database 'AdventureWorks2014'`

### Bước 5: Configure SQL Server User
Tạo login và user cho DBT/Airflow:

```bash
# Xem các login hiện tại
docker-compose exec sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "YourStrong@Passw0rd" -Q "SELECT name FROM sys.sql_logins WHERE type = 'S';"

# Tạo login & user mới
docker-compose exec sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "YourStrong@Passw0rd" -Q "CREATE LOGIN imrandbtnew WITH PASSWORD = 'Imran@12345'; USE AdventureWorks2014; CREATE USER imrandbtnew FOR LOGIN imrandbtnew; ALTER ROLE db_owner ADD MEMBER imrandbtnew;"
```

### Bước 6: Test DBT Connection
```bash
docker-compose exec dbt dbt debug
```

**Output mong đợi:** 
```
Connection test: [OK connection ok]
All checks passed!
```

### Bước 7: Verify All Services Running
```bash
docker-compose ps
```

**Mong đợi:** 6 containers running (sqlserver, postgres, dbt, airflow-webserver, airflow-scheduler)

---

## 🎯 Xác nhận Setup thành công

### ✅ 1. Airflow UI
- **URL:** http://localhost:8080
- **Username:** admin
- **Password:** admin

### ✅ 2. DBT Models
```bash
docker-compose exec dbt dbt run
```
**Output mong đợi:** All models run successfully ✓

### ✅ 3. SQL Server Data
Dùng **Azure Data Studio** hoặc **SQL Server Management Studio:**
- **Server:** localhost,1433
- **Database:** AdventureWorks2014
- **User:** imrandbtnew (hoặc sa)
- **Password:** Imran@12345 (hoặc YourStrong@Passw0rd)

Query để kiểm tra:
```sql
SELECT COUNT(*) FROM dbo.brnz_customers;
SELECT COUNT(*) FROM dbo.slvr_customers;
SELECT COUNT(*) FROM dbo.gld_customer_metrics;
```

---

## ⚠️ Troubleshooting

### ❌ Lỗi: "service 'X' is not running"
```bash
# Kiểm tra logs
docker-compose logs [service-name]

# Restart service
docker-compose restart [service-name]
```

### ❌ Lỗi: "Login timeout expired" (DBT debug fail)
- **Nguyên nhân:** SQL Server chưa khởi động xong hoặc user chưa được tạo
- **Cách sửa:**
  ```bash
  # Chờ SQL Server khởi động xong (~30 giây)
  docker-compose logs sqlserver | tail -20
  
  # Rồi tạo user lại (Bước 5)
  ```

### ❌ Lỗi: "no space left on device" (Docker build fail)
```bash
# Dọn Docker cache
docker system prune -a --volumes

# Rebuild
docker-compose build --no-cache
```

### ❌ Lỗi: Port already in use (8080, 1433)
Sửa `docker-compose.yml`:
```yaml
services:
  sqlserver:
    ports:
      - "1434:1433"  # Thay từ 1433 sang 1434
  
  airflow-webserver:
    ports:
      - "8081:8080"  # Thay từ 8080 sang 8081
```

### ❌ Lỗi: Permission denied on logs
```bash
# Reset logs directory
rm -rf airflow/logs
mkdir -p airflow/logs
chmod 777 airflow/logs

# Restart
docker-compose down
docker-compose up -d
```

---

## 📋 Architecture

```
Clone từ GitHub
      ↓
docker-compose build
      ↓
docker-compose up -d
      ↓
[SQL Server] ← Restore AdventureWorks2014.bak
      ↓
[DBT] ← Test connection & Run models
      ↓
[Airflow] ← Schedule DBT transformations
      ↓
[Postgres] ← Store Airflow metadata
      ↓
✅ Ready to use!
```

---

## 📚 Các file quan trọng
- **docker-compose.yml** - Service configuration
- **dbt/profiles.yml** - DBT connection settings (dùng `sqlserver` hostname)
- **dbt/dbt_project.yml** - DBT project config
- **airflow/dags/dbt_dag.py** - Airflow DAG definition
- **NEW_COMPUTER_SETUP.md** - Chi tiết hơn nếu có vấn đề

---

## ⏱️ Thời gian setup
- **Build images:** 5-10 phút
- **Start services:** 2-3 phút
- **Database restore:** 1-2 phút
- **DBT test:** < 1 phút
- **Total:** ~10-15 phút

---

## 🚀 Bước tiếp theo (sau khi setup)
1. Xem DAG trên Airflow UI (http://localhost:8080)
2. Trigger DBT DAG manually để test
3. Kiểm tra dữ liệu trên SQL Server
4. Modify dbt/models theo business logic
5. Push changes lên GitHub

---

**Có vấn đề?** Xem chi tiết ở `TROUBLESHOOTING.md` hoặc `NEW_COMPUTER_SETUP.md`
