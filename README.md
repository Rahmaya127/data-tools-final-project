
<div align="center">
  <!-- You are encouraged to replace this logo with your own! Otherwise you can also remove it. -->
  <img width="314" height="285" alt="image" src="https://github.com/user-attachments/assets/20661293-a214-4004-9042-657102fb0710" />
  <br/>

  <h3><b>Rahma Adan Readme</b></h3>

</div>

# Restaurant Ordering SQL Project

<a name="readme-top"></a>

<!-- TABLE OF CONTENTS -->

# 📗 Table of Contents

- [Restaurant Ordering SQL Project](#about-project)
- [📗 Table of Contents](#-table-of-contents)
- [📖 Restaurant Ordering SQL Project](#about-project)
  - [🛠 Built With ](#-built-with-)
    - [Tech Stack ](#tech-stack-)
    - [Key Features ](#key-features-)
  - [💻 Getting Started ](#-getting-started-)
    - [Prerequisites](#prerequisites)
    - [Setup](#setup)
    - [Usage](#usage)
  - [👥 Authors ](#-authors-)
  - [🔭 Future Features ](#-future-features-)
  - [🤝 Contributing ](#-contributing-)

<!-- PROJECT DESCRIPTION -->

# 📖 Restaurant Ordering SQL Project <a name="about-project"></a>

**Restaurant Ordering SQL Project** is a relational database that manages a restaurant’s core operations — customer records, menu items, and orders. It uses **PostgreSQL** to store and query structured data efficiently.

This project simulates a basic **Restaurant Management System**, where each customer can place orders from a menu, and the system keeps track of order details including prices and dates.

## 🛠 Built With <a name="built-with"></a>

### Tech Stack <a name="tech-stack"></a>

- **SQL**
- **PostgreSQL**
- **Supabase (optional)**

### Key Features <a name="key-features"></a>

[✔️] **Customer Table** — Stores customer information including names, emails, and join dates.  
[✔️] **Menu Items Table** — Contains menu categories, prices, and availability.  
[✔️] **Orders Table** — Links customers to menu items and records quantities and order timestamps.  
[✔️] **Relational Integrity** — Uses foreign keys and constraints to maintain clean data relationships.  
[✔️] **Test Queries** — Includes sample SQL queries to validate the setup.  

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## 💻 Getting Started <a name="getting-started"></a>

To rebuild or test this database locally or in Supabase, follow the steps below.

### Prerequisites

You need:

- [A Supabase account](https://supabase.com/) or local PostgreSQL setup  
- Basic understanding of [SQL](https://www.w3schools.com/sql/)  
- A SQL client (e.g., pgAdmin, DBeaver, or Supabase SQL Editor)

### Setup

You can copy the schema and data into your SQL editor or clone this repo to your local directory:

```sh
  git clone https://github.com/yourusername/restaurant-sql-db
  cd restaurant-sql-db
```

### DB Schema

The database consists of **3 main tables** with **5 records each**.

#### SQL Schema:

```sql
-- Drop old tables if they exist
DROP TABLE IF EXISTS orders CASCADE;
DROP TABLE IF EXISTS customers CASCADE;
DROP TABLE IF EXISTS menu_items CASCADE;

-- Create customers table
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),
    join_date DATE DEFAULT CURRENT_DATE
);

-- Insert sample customers
INSERT INTO customers (full_name, email, phone, join_date)
VALUES
('Ken Otieno', 'otieno@gmail.com', '555-1234', '2025-10-01'),
('Jane Wanjiku', 'jane@gmail.com', '555-5678', '2025-10-03'),
('Rahma Adan', 'rahma@gmail.com', '555-8765', '2025-10-10'),
('Aisha Ali', 'aisha@gmail.com', '555-3456', '2025-10-12'),
('Maria Salome', 'maria@gmail.com', '555-4321', '2025-10-15');

-- Create menu_items table
CREATE TABLE menu_items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    price DECIMAL(6,2) NOT NULL CHECK (price >= 0),
    available BOOLEAN DEFAULT TRUE
);

-- Insert sample menu items
INSERT INTO menu_items (name, category, price, available)
VALUES
('Cheeseburger', 'Main', 8.99, TRUE),
('Caesar Salad', 'Starter', 5.49, TRUE),
('Chocolate Cake', 'Dessert', 4.99, TRUE),
('Latte', 'Drink', 3.50, TRUE),
('Grilled Salmon', 'Main', 12.99, FALSE);

-- Create orders table
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
    menu_item_id INTEGER NOT NULL REFERENCES menu_items(id) ON DELETE CASCADE,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    order_date TIMESTAMP DEFAULT NOW()
);

-- Insert sample orders
INSERT INTO orders (customer_id, menu_item_id, quantity, order_date)
VALUES
(1, 1, 2, '2025-10-20 12:30:00'),
(2, 3, 1, '2025-10-21 13:15:00'),
(3, 4, 2, '2025-10-22 09:45:00'),
(4, 1, 1, '2025-10-22 18:00:00'),
(5, 2, 3, '2025-10-23 11:20:00');
```

### Test Queries

```sql
-- Show all customers
SELECT * FROM customers;
```
-- Show all menu items
```sql
SELECT * FROM menu_items;
```
-- Show order details with join
```sql
SELECT 
    o.id AS order_id,
    c.full_name AS customer,
    m.name AS item,
    o.quantity,
    (m.price * o.quantity) AS total_price,
    o.order_date
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN menu_items m ON o.menu_item_id = m.id
ORDER BY o.order_date;
```

### Screenshots from my test queries in Supabase

**Customers Table**

<img width="1906" height="807" alt="image" src="https://github.com/user-attachments/assets/96706ee7-595c-4de0-b7c1-6389259f863e" />


**Menu Items Table**

<img width="1904" height="703" alt="image" src="https://github.com/user-attachments/assets/84e9334a-0592-4495-b59b-7f51e48b31a7" />


**Orders Table**

<img width="1893" height="691" alt="image" src="https://github.com/user-attachments/assets/25dac8e1-6ff1-4b13-9a6d-21939cff708a" />


**ERD (Entity Relationship Diagram)**

<img width="1604" height="900" alt="image" src="https://github.com/user-attachments/assets/6b21b4c1-aa7c-4909-8a3e-6e120d2cfc49" />


## 👥 Authors <a name="authors"></a>

👤 **Rahma Adan**

- GitHub: [@yourgithub](https://github.com/Rahmaya127)   
- LinkedIn: [@rahma Adan](https://github.com/Rahmaya127)  

## 🔭 Future Features <a name="future-features"></a>

- [ ] Add a **payments** table for order transactions  
- [ ] Include **staff** and **inventory** tables  
- [ ] Integrate **visual dashboards** for order analysis via R or Python  

## 🤝 Contributing <a name="contributing"></a>

Contributions, issues, and feature requests are welcome!  

Feel free to check the **[issues page](../../issues/)** if you’d like to collaborate.

<p align="right">(<a href="#readme-top">back to top</a>)</p>
