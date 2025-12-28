> [!NOTE]
> * Bản 21 và 22 chưa định nghĩa kiểu dữ liệu

---

## 📑 Tổng quan

* Tổng số bảng: **20 chính + 2 chưa hoàn thành(21 và 22)**
* Chia thành **5 nhóm dữ liệu**:
1. Hệ thống & Phân quyền (Core System)
2. Không gian & Con người (Space & People)
3. Dịch vụ & Tài chính (Finance)
4. Vận hành (Operations)
5. Quản lý Chi phí & Tài sản (Assets & Operational Expenses)

---

<details>
<summary>1. HỆ THỐNG & PHÂN QUYỀN (CORE SYSTEM)</summary>
  
```sql
CREATE TABLE roles (
    role_id INT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) NOT NULL, -- ADMIN, ACCOUNTANT, RESIDENT, SECURITY...
    description TEXT,
    salary DOUBLE DEFAULT 0 -- Nếu là RESIDENT thì = 0
);

CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL, -- Mật khẩu mã hóa
    full_name VARCHAR(100),
    email VARCHAR(100),
    phone VARCHAR(20),
    role_id INT,
    status VARCHAR(20) DEFAULT 'ACTIVE', -- ACTIVE, LOCKED
    FOREIGN KEY (role_id) REFERENCES roles(role_id)
);

CREATE TABLE permissions (
    perm_id INT PRIMARY KEY AUTO_INCREMENT,
    perm_code VARCHAR(50) UNIQUE NOT NULL, -- VIEW_INVOICE, EDIT_USER...
    description TEXT
);

CREATE TABLE role_permissions (
    role_id INT,
    perm_id INT,
    PRIMARY KEY (role_id, perm_id),
    FOREIGN KEY (role_id) REFERENCES roles(role_id),
    FOREIGN KEY (perm_id) REFERENCES permissions(perm_id)
);

```

</details>

---

<details>
<summary>2. KHÔNG GIAN & CON NGƯỜI (SPACE & PEOPLE)</summary>

```sql
CREATE TABLE blocks (
    block_id INT PRIMARY KEY AUTO_INCREMENT,
    block_name VARCHAR(50) NOT NULL, -- Tòa A, Tòa B
    total_floors INT,
    number_apartments INT
);

CREATE TABLE apartment_types (
    type_id INT PRIMARY KEY AUTO_INCREMENT,
    type_name VARCHAR(50), -- NORMAL, SPECIAL, LUXURY
    price_per_m2 DECIMAL(15, 2),
    management_fee_per_m2 DECIMAL(15, 2)
);

CREATE TABLE apartments (
    apartment_id VARCHAR(20) PRIMARY KEY, -- P101, P102
    block_id INT,
    area FLOAT, -- Diện tích m2
    type_id INT,
    floor_number INT,
    status VARCHAR(20) DEFAULT 'VACANT', -- VACANT, OCCUPIED
    FOREIGN KEY (block_id) REFERENCES blocks(block_id),
    FOREIGN KEY (type_id) REFERENCES apartment_types(type_id)
);

CREATE TABLE residents (
    resident_id INT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100) NOT NULL,
    user_id INT,
    phone VARCHAR(20),
    identity_card VARCHAR(20),
    hometown VARCHAR(200),
    dob DATE,
    gender VARCHAR(10),
    apartment_id VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (apartment_id) REFERENCES apartments(apartment_id)
);

CREATE TABLE living_history (
    history_id INT PRIMARY KEY AUTO_INCREMENT,
    resident_id INT,
    apartment_id VARCHAR(20),
    start_date DATE,
    end_date DATE,
    type VARCHAR(20), -- OWNER, TENANT
    rent_price DECIMAL(15, 2) DEFAULT 0,
    FOREIGN KEY (resident_id) REFERENCES residents(resident_id),
    FOREIGN KEY (apartment_id) REFERENCES apartments(apartment_id)
);

CREATE TABLE vehicles (
    vehicle_id INT PRIMARY KEY AUTO_INCREMENT,
    resident_id INT,
    plate_number VARCHAR(20) UNIQUE,
    vehicle_type VARCHAR(20), -- MOTORBIKE, CAR, BICYCLE
    status VARCHAR(20), -- IN_PARKING, OUTSIDE, LOCKED
    FOREIGN KEY (resident_id) REFERENCES residents(resident_id)
);

```

</details>

---

<details>
<summary>3. DỊCH VỤ & TÀI CHÍNH (FINANCE)</summary>

```sql
CREATE TABLE services (
    service_id INT PRIMARY KEY AUTO_INCREMENT,
    service_name VARCHAR(100), -- Điện, Nước, Phí QL, Gửi xe
    unit_price DECIMAL(15, 2),
    unit_type VARCHAR(20) -- kWh, m3, tháng, chiếc
);

CREATE TABLE service_usage (
    usage_id INT PRIMARY KEY AUTO_INCREMENT,
    apartment_id VARCHAR(20),
    service_id INT,
    month INT,
    year INT,
    old_index INT,
    new_index INT,
    total_consumed INT, -- new_index - old_index
    FOREIGN KEY (apartment_id) REFERENCES apartments(apartment_id),
    FOREIGN KEY (service_id) REFERENCES services(service_id)
);

CREATE TABLE invoices (
    invoice_id INT PRIMARY KEY AUTO_INCREMENT,
    apartment_id VARCHAR(20),
    month INT,
    year INT,
    total_amount DECIMAL(15, 2),
    status VARCHAR(20) DEFAULT 'UNPAID', -- UNPAID, PAID, OVERDUE
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (apartment_id) REFERENCES apartments(apartment_id)
);

CREATE TABLE invoice_details (
    detail_id INT PRIMARY KEY AUTO_INCREMENT,
    invoice_id INT,
    service_id INT,
    amount DECIMAL(15, 2),
    FOREIGN KEY (invoice_id) REFERENCES invoices(invoice_id),
    FOREIGN KEY (service_id) REFERENCES services(service_id)
);

CREATE TABLE payments (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    invoice_id INT,
    amount_paid DECIMAL(15, 2),
    payment_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    method VARCHAR(50), -- CASH, TRANSFER
    collector_id INT,
    FOREIGN KEY (invoice_id) REFERENCES invoices(invoice_id),
    FOREIGN KEY (collector_id) REFERENCES users(user_id)
);

```


</details>

---

<details>
<summary>4. VẬN HÀNH (OPERATIONS)</summary>

```sql
CREATE TABLE complaints (
    complaint_id INT PRIMARY KEY AUTO_INCREMENT,
    resident_id INT,
    title VARCHAR(200),
    content TEXT,
    status VARCHAR(20) DEFAULT 'PENDING', -- PENDING, PROCESSING, DONE
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    resolved_at DATETIME,
    FOREIGN KEY (resident_id) REFERENCES residents(resident_id)
);

CREATE TABLE notifications (
    notify_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200),
    message TEXT,
    target_role VARCHAR(50), -- ALL, RESIDENT, BLOCK_A
    created_by INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES users(user_id)
);

```

</details>

---

<details>
<summary>5. QUẢN LÝ CHI PHÍ & TÀI SẢN (ASSETS & OPERATIONAL EXPENSES)</summary>

```sql
CREATE TABLE expense_categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100),
    description TEXT
);

CREATE TABLE operational_expenses (
    expense_id INT PRIMARY KEY AUTO_INCREMENT,
    category_id INT,
    amount DECIMAL(15, 2),
    expense_date DATETIME,
    payee VARCHAR(100),
    approved_by INT,
    FOREIGN KEY (category_id) REFERENCES expense_categories(category_id),
    FOREIGN KEY (approved_by) REFERENCES users(user_id)
);

CREATE TABLE facility_assets (
    asset_id INT PRIMARY KEY AUTO_INCREMENT,
    asset_name VARCHAR(100),
    location VARCHAR(100),
    status VARCHAR(50), -- GOOD, BROKEN, UNDER_MAINTENANCE
    purchase_date DATE,
    value DECIMAL(15, 2)
);

CREATE TABLE maintenance_logs (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    asset_id INT,
    maintainer_name VARCHAR(100),
    description TEXT,
    cost DECIMAL(15, 2), -- Cộng vào operational_expenses thông qua logic nghiệp vụ
    maintenance_date DATE,
    next_maintenance_date DATE,
    FOREIGN KEY (asset_id) REFERENCES facility_assets(asset_id)
);

```

</details>
