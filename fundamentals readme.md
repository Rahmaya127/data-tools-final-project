# 🧱 Data Fundamentals Project: Admin Roles & Security in Supabase

## 📖 Overview
This project extends the **Data Tools Final Project** by adding **role-based access control**, **Row Level Security (RLS)**, and **Supabase Auth integration**.  
You’ll manage Admin and User privileges, secure your tables, and automate user creation when new Auth accounts are registered.

---

## 📚 Table of Contents
1. [Overview](#-overview)  
2. [Database Setup](#-database-setup)  
3. [Roles & Policies](#-roles--policies)  
4. [Admin Function](#-admin-function)  
5. [Automatic User Sync](#-automatic-user-sync)  
6. [Assigning Roles](#-assigning-roles)  
7. [Testing & Verification](#-testing--verification)  
8. [Deliverables](#-deliverables)  
9. [Sample Queries](#-sample-queries)

---

## 🏗️ Database Setup
We build upon the previous project’s database (`customers`, `menu_items`, and `orders` tables) and add authentication and role-based security.

```sql
-- 1️⃣ Add Auth & Role columns (safe if not exist)
ALTER TABLE customers 
ADD COLUMN IF NOT EXISTS auth_id UUID;

ALTER TABLE customers 
ADD COLUMN IF NOT EXISTS role TEXT DEFAULT 'user';
```

Next, enable **Row Level Security** on all relevant tables:
```sql
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE menu_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
```

---

## 🔒 Roles & Policies
### 🧍 User Role
Users can:
- View **only their own** orders  
- Insert **only their own** orders  

```sql
-- USERS: Can view only their own orders
CREATE POLICY "Users view own orders"
ON orders
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM customers
    WHERE customers.id = orders.customer_id
      AND customers.auth_id = auth.uid()
  )
);

-- USERS: Can insert only their own orders
CREATE POLICY "Users insert own orders"
ON orders
FOR INSERT
WITH CHECK (
  EXISTS (
    SELECT 1 FROM customers
    WHERE customers.id = orders.customer_id
      AND customers.auth_id = auth.uid()
  )
);
```

### 👑 Admin Role
Admins can **read, insert, update, and delete** all data.

```sql
-- ADMINS: Full access
CREATE POLICY "Admins full access to orders"
ON orders
FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM customers
    WHERE customers.auth_id = auth.uid()
      AND role = 'admin'
  )
);
```

---

## ⚙️ Admin Function
Admins have a special function to delete any order:

```sql
CREATE OR REPLACE FUNCTION delete_order(order_id INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
BEGIN
  IF EXISTS (
    SELECT 1 
    FROM customers 
    WHERE auth_id = auth.uid() 
      AND role = 'admin'
  ) THEN
    DELETE FROM orders WHERE id = order_id;
  ELSE
    RAISE EXCEPTION 'Permission denied: only admins can delete orders';
  END IF;
END;
$$;
```

This ensures only **admins** can perform deletions.

---

## 🔁 Automatic User Sync
When a new Supabase Auth user is created, a **trigger** automatically inserts them into the `customers` table with the default `user` role.

```sql
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.customers (full_name, email, auth_id, role)
  VALUES (
    COALESCE(NEW.raw_user_meta_data->>'full_name', 'New User'),
    NEW.email,
    NEW.id,
    'user'
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

DROP TRIGGER IF EXISTS on_auth_user_created ON auth.users;

CREATE TRIGGER on_auth_user_created
AFTER INSERT ON auth.users
FOR EACH ROW EXECUTE FUNCTION handle_new_user();
```

---

## 🧑‍💼 Assigning Roles
You can manually promote yourself or others to **admin** using:
```sql
UPDATE customers
SET role = 'admin'
WHERE email = 'rahma@gmail.com';
```

---

## 🔄 UUID Linking for Old Customers
For older records (created before Auth integration), generate unique UUIDs:
```sql
UPDATE customers
SET auth_id = gen_random_uuid()
WHERE auth_id IS NULL;
```

---
## Admin can add new users using magic email links in supabase
<img width="1855" height="363" alt="image" src="https://github.com/user-attachments/assets/c8dea1ee-5a93-4eaa-ad48-bd4a9c8fefa5" />

## 🧪 Testing & Verification
To verify setup:

### 1️⃣ Check all customers
```sql
SELECT * FROM customers;
```
**Users based on roles**
<img width="1820" height="477" alt="image" src="https://github.com/user-attachments/assets/62b6fe06-76f6-47de-b8a2-751849d0c212" />

### 2️⃣ Check all menu items
```sql
SELECT * FROM menu_items;
```
**Output for all menu items**
<img width="1875" height="469" alt="image" src="https://github.com/user-attachments/assets/e56ddb51-2728-434c-abe0-8dc4234464c1" />


### 3️⃣ Insert an order (as logged-in user)
```sql
INSERT INTO orders (customer_id, menu_item_id, quantity)
VALUES (1, 3, 2);
```

### 4️⃣ View only your own orders
```sql
SELECT * FROM orders;
```

### 5️⃣ Admin-only delete function
```sql
SELECT delete_order(2);
```

### 6️⃣ Test RLS visibility (should be restricted per role)
```sql

-- As admin, should return all orders
SELECT * FROM orders;
```
**All orders output**
<img width="1854" height="504" alt="image" src="https://github.com/user-attachments/assets/4537a451-f26b-48a9-871a-c60ecf527fcb" />

---

## 📦 Deliverables
- Supabase database with **RLS** enabled  
- **Admin/User** roles and security policies  
- **Admin-only function** (`delete_order`)  
- **Automatic user creation trigger**  
- **Comprehensive documentation** (this README)  

---

✅ **End of Project: Admin Roles & Security in Supabase**
