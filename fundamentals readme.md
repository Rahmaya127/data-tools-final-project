# Supabase Admin Roles & Security Project

<a name="readme-top"></a>

<!-- TABLE OF CONTENTS -->

# 📗 Table of Contents

- [📖 About the Project](#about-project)
  - [🛠 Built With](#built-with)
    - [Tech Stack](#tech-stack)
    - [Key Features](#key-features)
  - [💻 Getting Started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Setup](#setup)
    - [Database Schema](#database-schema)
  - [📸 Screenshots](#screenshots)
  - [👥 Authors](#authors)
  - [🔭 Future Features](#future-features)
  - [🤝 Contributing](#contributing)

---

# 📖 Supabase Admin Roles & Security Project <a name="about-project"></a>

**Supabase Admin Roles & Security Project** builds upon the **Restaurant Ordering SQL Project** by introducing **Row Level Security (RLS)**, **role-based access control**, and **custom SQL functions**.  
It’s designed as part of the **Data Fundamentals Unit (Data Tools)** — a **4-week project** that teaches secure database administration in Supabase (Postgres).

---

## 🛠 Built With <a name="built-with"></a>

### Tech Stack <a name="tech-stack"></a>

- SQL (PostgreSQL)
- Supabase (Cloud-hosted Postgres)
- Supabase Auth (Email/Magic Link)

---

### Key Features <a name="key-features"></a>

- ✅ **Row Level Security (RLS)** enabled on all tables  
- 🧑‍💼 **Admin and User roles** implemented  
- 🔒 **Custom SQL policies** for secure CRUD access  
- 🧩 **Admin-only SQL function** for order deletion  
- 🗂️ **Documentation** via `README.md` and `security_notes.md`  

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 💻 Getting Started <a name="getting-started"></a>

Follow these steps to set up and secure your Supabase database.

### Prerequisites <a name="prerequisites"></a>

You’ll need:  
- A [Supabase Account](https://supabase.com/)  
- Basic SQL knowledge  
- Your **Restaurant Ordering SQL Project** schema

---

### Setup <a name="setup"></a>

1. Go to your Supabase project.  
2. Reuse the `customers`, `menu_items`, and `orders` tables from your previous project.  
3. Enable **Row Level Security (RLS)** for each table:

```sql
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE menu_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
```

---

### Database Schema <a name="database-schema"></a>

**Tables:**  
- `customers`: Stores user information and roles  
- `menu_items`: Lists available restaurant items  
- `orders`: Links customers to their menu selections  

**Add a Role Column:**

```sql
ALTER TABLE customers ADD COLUMN role TEXT DEFAULT 'user';
UPDATE customers SET role = 'admin' WHERE email = 'otieno@gmail.com';
```

**Admin and User Policies:**

```sql
--Enable Row Level Security (RLS) for each table:
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE menu_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Add a column to link customers to Supabase Auth users
-- (This stores the user's UUID from Supabase Auth)
ALTER TABLE customers ADD COLUMN auth_id UUID DEFAULT NULL;

ALTER TABLE customers ADD COLUMN role TEXT DEFAULT 'user';
UPDATE customers SET role = 'admin' WHERE email = 'otieno@gmail.com';



-- Users can only see/insert their own orders
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

--Admins have full access to all orders
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

**Admin-only Function:**

```sql
CREATE OR REPLACE FUNCTION delete_order(order_id INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
BEGIN
  -- Only allow admins to delete orders
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

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 📸 Screenshots <a name="screenshots"></a>

**Customers Table**
<img width="1893" height="476" alt="customers table" src="https://github.com/user-attachments/assets/placeholder-customers" />

**Menu Items Table**
<img width="1881" height="445" alt="menu items table" src="https://github.com/user-attachments/assets/placeholder-menu" />

**Orders Table**
<img width="1902" height="517" alt="orders table" src="https://github.com/user-attachments/assets/placeholder-orders" />

**RLS and Policies in Supabase**
<img width="1064" height="577" alt="rls policies" src="https://github.com/user-attachments/assets/placeholder-rls" />

**ERD Diagram**
<img width="1064" height="577" alt="erd diagram" src="https://github.com/user-attachments/assets/placeholder-erd" />

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 👥 Authors <a name="authors"></a>

👤 **Your Name Here**  
- GitHub: [@yourgithub](https://github.com/yourgithub)  
- Twitter: [@yourtwitter](https://twitter.com/yourtwitter)  
- LinkedIn: [@yourlinkedin](https://linkedin.com/in/yourlinkedin)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 🔭 Future Features <a name="future-features"></a>

- [ ] Add **Payment Transactions Table**  
- [ ] Integrate **Inventory Management**  
- [ ] Connect **R or Python** for security auditing and visualization  

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## 🤝 Contributing <a name="contributing"></a>

Contributions, issues, and feature requests are welcome!  
Feel free to check the [issues page](../../issues/).

<p align="right">(<a href="#readme-top">back to top</a>)</p>
