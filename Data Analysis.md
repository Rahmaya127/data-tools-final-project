# 🍽️ Restaurant Data Analysis in R

This project demonstrates how to analyze and visualize **restaurant-related data** using **R** on **Posit Cloud (RStudio)**.
It connects to a **PostgreSQL (Supabase)** database, performs joins, summaries, and creates interactive visualizations with **ggplot2** and **plotly**.

---

## 🧩 Project Overview

The dataset models transactional relationships between **customers**, **menu items**, and **orders**.
The goal is to understand key restaurant metrics, such as popular menu items, customer ordering habits, and total sales by category.

**Tables used:**
- **customers** – details of patrons (name, email, join date)
- **menu_items** – details of available food/drink (name, category, price)
- **orders** – transactional data (customer ID, item ID, quantity, date/time)

---

## 🗂️ Database Structure (PostgreSQL)

```sql
-- Create customers table
CREATE TABLE customers (
 	id SERIAL PRIMARY KEY,
 	full_name VARCHAR(100) NOT NULL,
 	email VARCHAR(100) UNIQUE NOT NULL,
 	phone VARCHAR(20),
 	join_date DATE DEFAULT CURRENT_DATE
);

-- Create menu_items table
CREATE TABLE menu_items (
 	id SERIAL PRIMARY KEY,
 	name VARCHAR(100) NOT NULL,
 	category VARCHAR(50) NOT NULL,
 	price DECIMAL(6,2) NOT NULL CHECK (price >= 0),
 	available BOOLEAN DEFAULT TRUE
);

-- Create orders table
CREATE TABLE orders (
 	id SERIAL PRIMARY KEY,
 	customer_id INTEGER NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
 	menu_item_id INTEGER NOT NULL REFERENCES menu_items(id) ON DELETE CASCADE,
 	quantity INTEGER NOT NULL CHECK (quantity > 0),
 	order_date TIMESTAMP DEFAULT NOW()
);
```

## Setup in R (Posit Cloud)
Install and load Required Libraries
``` SQL
install.packages(c("DBI", "RPostgres", "dplyr", "ggplot2", "plotly"))
library(DBI)
library(RPostgres)
library(dplyr)
library(ggplot2)
library(plotly)
```
## Connect to the Database
```R
library(DBI)
library(RPostgres)

connect_db <- function() {
  dbConnect(
    RPostgres::Postgres(),
    host = "aws-1-eu-north-1.pooler.supabase.com",
    port = 6543,
    dbname = "postgres",
    user = "postgres.vkmjgwksyuvvizllgpdj",
    password = "Adanrahma32",
    sslmode = "require"
  )
}
```
## To list the tables
```
R source("connect_db.R")
con <- connect_db()
dbListTables(con)
```
<img width="1348" height="586" alt="image" src="https://github.com/user-attachments/assets/2fb94301-eea6-4ae5-bdaf-620f296ed8a2" />

## Load Data into R
``` R
customers <- dbReadTable(con, "customers")
menu_items  <- dbReadTable(con, "menu_items")
orders   <- dbReadTable(con, "orders")
```
<img width="1356" height="592" alt="image" src="https://github.com/user-attachments/assets/9fe166bb-651f-4e57-8369-5892f18e7bf1" />

# Visualizations
## 1. Price Distribution by Item

```R
p2 <- ggplot(menu_items, aes(x = category, y = price,
                             text = paste(name, "<br>Price:", price))) +
  geom_boxplot(outlier.shape = NA) +
  geom_jitter(width = 0.2, aes(color = category)) +
  labs(title = "Item Price Distribution by Category",
       x = "Menu Category", y = "Price ($)")
ggplotly(p2, tooltip = "text")
```

<img width="1358" height="590" alt="image" src="https://github.com/user-attachments/assets/7f09bb14-9f27-43a7-b883-d569245a357b" />

## 2. Total Revenue by Menu Category

```R
category_sales <- sales_data %>%
  group_by(category) %>%
  summarise(total_revenue = sum(sale_amount, na.rm = TRUE))

ggplot(category_sales, aes(x = reorder(category, total_revenue), y = total_revenue, fill = category)) +
  geom_col() +
  coord_flip() +
  labs(title = "Total Revenue by Menu Category",
       x = "Menu Category", y = "Total Revenue ($)", fill = "Category") +
  theme_minimal()
```
<img width="1361" height="586" alt="image" src="https://github.com/user-attachments/assets/c218e49d-de75-4309-9d96-5fe7d7d4e992" />

# Data Dictionary

## 📚 Data Dictionary: Restaurant Operations Database

This dictionary describes the tables and columns used in the PostgreSQL database for the restaurant data analysis project.

---

### 1. `customers` Table: Patron Information

| Column Name | Data Type (PostgreSQL) | Description | Constraints / Notes |
| :--- | :--- | :--- | :--- |
| **id** | `SERIAL` | **Primary Key** for the table. Unique customer identifier. | `PRIMARY KEY`, `NOT NULL` |
| **full_name** | `VARCHAR(100)` | The customer's full name. | `NOT NULL` |
| **email** | `VARCHAR(100)` | The customer's email address. | `UNIQUE`, `NOT NULL` |
| **phone** | `VARCHAR(20)` | The customer's contact phone number. | Optional |
| **join_date** | `DATE` | The date the customer was added to the system. | Default is `CURRENT_DATE` |

---

### 2. `menu_items` Table: Menu Details

| Column Name | Data Type (PostgreSQL) | Description | Constraints / Notes |
| :--- | :--- | :--- | :--- |
| **id** | `SERIAL` | **Primary Key** for the table. Unique menu item identifier. | `PRIMARY KEY`, `NOT NULL` |
| **name** | `VARCHAR(100)` | The name of the menu item (e.g., 'Cheeseburger'). | `NOT NULL` |
| **category** | `VARCHAR(50)` | The category of the item (e.g., 'Main', 'Drink', 'Dessert'). | `NOT NULL` |
| **price** | `DECIMAL(6,2)` | The selling price of the menu item. | `NOT NULL`, Must be $ >= 0$ |
| **available** | `BOOLEAN` | Indicates if the item is currently available for order. | Default is `TRUE` |

---

### 3. `orders` Table: Transactional Data

| Column Name | Data Type (PostgreSQL) | Description | Constraints / Notes |
| :--- | :--- | :--- | :--- |
| **id** | `SERIAL` | **Primary Key** for the table. Unique order identifier. | `PRIMARY KEY`, `NOT NULL` |
| **customer_id** | `INTEGER` | **Foreign Key** referencing the `customers.id`. | `NOT NULL`, `REFERENCES customers(id)` |
| **menu_item_id** | `INTEGER` | **Foreign Key** referencing the `menu_items.id`. | `NOT NULL`, `REFERENCES menu_items(id)` |
| **quantity** | `INTEGER` | The number of units of the item ordered in this transaction. | `NOT NULL`, Must be $ > 0$ |
| **order_date** | `TIMESTAMP` | The exact date and time the order was placed. | Default is `NOW()` |

## Authors

**Project Owner / Maintainer**  
- Adan Rahma Jilo
- GitHub: github.com/Rahmaya
- Contact: adanrahma32@gmail.com

---

## Future Features

- Expand schema: shipments, buyers, inventory, and transactions.  
- Add time-series pricing and seasonal trend analysis.  
- Small Shiny dashboard combining interactive maps, price trends, and producer filters.  
- Automate daily price ingestion from CSV uploads or API.

---

## Contributing

Contributions are welcome. Please:
1. Fork the repo.  
2. Create a feature branch (`git checkout -b feature/your-feature`).  
3. Commit your changes and open a Pull Request with a clear description.

---

## Support

If you find this project useful, give it a ⭐️ on GitHub and share feedback via Issues.

---

## Acknowledgements

- [Supabase](https://supabase.com) — PostgreSQL hosting  
- [Posit](https://posit.co) — RStudio / Posit Cloud environment for analysis

---

## FAQ

**1. How do I run the schema?**  
Open Supabase SQL Editor and paste the `Schema SQL` section; run it to create tables and seed data.

**2. I get `Tenant or user not found` when connecting — why?**  
That means your DB credentials (host, user, password) are incorrect or don’t match the Supabase project. Double-check Project → Settings → Database → Connection info and use the database **user/password** (not API keys) in `connect_db()`.

**3. Can I add more sample data?**  
Yes — append `INSERT` statements or upload CSVs into Supabase table editor.

**4. Can I use this with another DB (MySQL)?**  
This schema and the provided R code are written for PostgreSQL (Supabase). Converting to MySQL will require datatype and connection adjustments.

---

## License

This project is released under the **MIT License**. See `LICENSE` for details.
