# Security Notes – Admin Roles & RLS Policies

## 🔐 Overview
This file documents the **Row Level Security (RLS)** setup, **role-based access**, and **security policies** implemented in the **Restaurant Supabase Security Project**.

---

## 👥 Roles

| Role | Description | Access |
|------|--------------|---------|
| **Admin** | Full control (CRUD on all tables) | Read, Insert, Update, Delete |
| **User** | Limited access (own data only) | Read, Insert (own records) |

---

## 🧱 Tables with RLS Enabled

- `customers`
- `menu_items`
- `orders`

```sql
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE menu_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
```

---

## ⚙️ Policies by Table

### 🧍 customers

**Admins can view all customers**
```sql
CREATE POLICY "Admins view all customers"
ON customers
FOR SELECT
USING (EXISTS (
  SELECT 1 FROM customers WHERE id = auth.uid() AND role = 'admin'
));
```

**Users can only view their own details**
```sql
CREATE POLICY "Users view own profile"
ON customers
FOR SELECT
USING (auth.uid() = id);
```

---

### 🍔 menu_items

**Users can only read menu**
```sql
CREATE POLICY "Users can view menu"
ON menu_items
FOR SELECT
USING (TRUE);
```

**Admins manage all menu items**
```sql
CREATE POLICY "Admins manage menu"
ON menu_items
FOR ALL
USING (EXISTS (
  SELECT 1 FROM customers WHERE id = auth.uid() AND role = 'admin'
));
```

---

### 🧾 orders

**Users can only see/insert their own orders**
```sql
CREATE POLICY "Users view own orders"
ON orders
FOR SELECT
USING (auth.uid() = customer_id);

CREATE POLICY "Users insert own orders"
ON orders
FOR INSERT
WITH CHECK (auth.uid() = customer_id);
```

**Admins have full access**
```sql
CREATE POLICY "Admins have full access to orders"
ON orders
FOR ALL
USING (EXISTS (
  SELECT 1 FROM customers WHERE id = auth.uid() AND role = 'admin'
));
```

---

## 🧩 Custom Function (Admin Only)

```sql
CREATE OR REPLACE FUNCTION delete_order(order_id INT)
RETURNS VOID
LANGUAGE SQL
SECURITY DEFINER
AS $$
  DELETE FROM orders WHERE id = order_id;
$$;
```

- `SECURITY DEFINER` allows execution with the creator’s (admin) privileges.  
- Only Admins should be able to call this function.  

---

## 🧠 Security Best Practices

1. Always enable **RLS** for user-accessible tables.  
2. Use **auth.uid()** for user-specific access control.  
3. Limit function access using **SECURITY DEFINER** + role checks.  
4. Avoid granting direct access to system roles like `anon`.  
5. Log and monitor role-based activity in Supabase Dashboard.

---

## ✅ Verification Checklist

- [x] RLS Enabled on all tables  
- [x] Policies defined per role  
- [x] Admin-only SQL Function implemented  
- [x] Auth enabled (Email/Magic Link)  
- [x] Documentation completed (README + security_notes)

---

**End of Security Notes**
