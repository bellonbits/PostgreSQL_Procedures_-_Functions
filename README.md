# 🧩 PostgreSQL Procedures & Functions

---

## 🔹 1. What Are They?

In PostgreSQL:

* A **Function** returns data (e.g., query results or a calculation).
* A **Procedure** performs an action (e.g., insert, update, or delete data).

Both are written in **PL/pgSQL (Procedural Language for SQL)**.

---

## 🔹 2. Basic Syntax

### ✅ Function Syntax

```sql
CREATE OR REPLACE FUNCTION function_name(parameter_list)
RETURNS return_type
LANGUAGE plpgsql
AS $$
BEGIN
    -- SQL statements
    RETURN some_value;
END;
$$;
```

📘 **Notes:**

* `RETURNS` defines what the function will give back.
* Use `RETURN` or `RETURN QUERY` to send results.
* Call it with:

  ```sql
  SELECT function_name(arguments);
  ```

---

### ✅ Procedure Syntax

```sql
CREATE OR REPLACE PROCEDURE procedure_name(parameter_list)
LANGUAGE plpgsql
AS $$
BEGIN
    -- SQL statements
END;
$$;
```

📘 **Notes:**

* Procedures **do not return** values.
* Call them with:

  ```sql
  CALL procedure_name(arguments);
  ```

---

## 🔹 3. Example Table: `bookshop.books`

| Column           | Type            | Description         |
| ---------------- | --------------- | ------------------- |
| `book_id`        | `serial4`       | Auto-generated ID   |
| `book_name`      | `varchar(150)`  | Title of the book   |
| `author`         | `varchar(150)`  | Author name         |
| `price`          | `numeric(10,2)` | Book price          |
| `published_date` | `date`          | Date of publication |

---

## 🔹 4. Creating a Function — Select All Books

```sql
CREATE OR REPLACE FUNCTION select_all_books()
RETURNS TABLE (
    book_id INT,
    book_name VARCHAR,
    author VARCHAR,
    price NUMERIC(10,2),
    published_date DATE
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        b.book_id, 
        b.book_name, 
        b.author, 
        b.price, 
        b.published_date
    FROM bookshop.books AS b;
END;
$$;
```

### ▶️ How to Run

```sql
SELECT * FROM select_all_books();
```

📘 **Notes:**

* `RETURN QUERY` tells PostgreSQL to output rows.
* The function behaves like a normal SQL query.

---

## 🔹 5. Creating a Procedure — Insert a Book

```sql
CREATE OR REPLACE PROCEDURE insert_book(
    p_book_name VARCHAR(150),
    p_author VARCHAR(150),
    p_price NUMERIC(10,2),
    p_published_date DATE
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO bookshop.books (book_name, author, price, published_date)
    VALUES (p_book_name, p_author, p_price, p_published_date);

    RAISE NOTICE 'Book "%", by % inserted successfully.', p_book_name, p_author;
END;
$$;
```

### ▶️ How to Run

```sql
CALL insert_book('The Alchemist', 'Paulo Coelho', 15.99, '1988-04-14');
CALL insert_book('Atomic Habits', 'James Clear', 18.50, '2018-10-16');
```

📘 **Notes:**

* `CALL` executes the procedure.
* `RAISE NOTICE` prints confirmation messages.
* `book_id` is **auto-generated** — you don't insert it manually.

---

## 🔹 6. Viewing the Inserted Data

```sql
SELECT * FROM bookshop.books;
```

---

## 🔹 7. Difference Between a Function and a Procedure

| Feature                  | Function     | Procedure                  |
| ------------------------ | ------------ | -------------------------- |
| Returns a value or table | ✅ Yes        | ❌ No                       |
| Called with              | `SELECT`     | `CALL`                     |
| Can modify data          | ⚠️ Limited   | ✅ Yes                      |
| Use case                 | Reading data | Inserting or updating data |

---

## 🔹 8. Example With Simple Error Handling

```sql
CREATE OR REPLACE PROCEDURE safe_insert_book(
    p_book_name VARCHAR(150),
    p_author VARCHAR(150),
    p_price NUMERIC(10,2),
    p_published_date DATE
)
LANGUAGE plpgsql
AS $$
BEGIN
    IF EXISTS (
        SELECT 1 FROM bookshop.books 
        WHERE book_name = p_book_name AND author = p_author
    ) THEN
        RAISE NOTICE 'Book "%" by % already exists. Skipping insert.', p_book_name, p_author;
    ELSE
        INSERT INTO bookshop.books (book_name, author, price, published_date)
        VALUES (p_book_name, p_author, p_price, p_published_date);
        RAISE NOTICE 'Book "%" by % inserted successfully.', p_book_name, p_author;
    END IF;
END;
$$;
```

---

## 🔹 9. Summary for Students

| Task                 | Type               | Example Command                     |
| -------------------- | ------------------ | ----------------------------------- |
| Create function      | `CREATE FUNCTION`  | —                                   |
| Create procedure     | `CREATE PROCEDURE` | —                                   |
| View all books       | Function           | `SELECT * FROM select_all_books();` |
| Insert a new book    | Procedure          | `CALL insert_book(...);`            |
| Check table contents | SQL query          | `SELECT * FROM bookshop.books;`     |

---

## 🏭 10. Real-World Industry Applications

Understanding **where** and **why** these tools are used helps students see their practical value beyond classroom exercises.

---

### 💼 **E-Commerce & Retail**

**Use Case: Order Processing**

```sql
-- Function: Calculate order total with discounts
CREATE FUNCTION calculate_order_total(order_id INT)
RETURNS NUMERIC AS $$
BEGIN
    RETURN (
        SELECT SUM(quantity * unit_price * (1 - discount_rate))
        FROM order_items
        WHERE order_id = order_id
    );
END;
$$ LANGUAGE plpgsql;

-- Procedure: Process order payment and update inventory
CREATE PROCEDURE process_order(p_order_id INT)
AS $$
BEGIN
    UPDATE orders SET status = 'paid' WHERE order_id = p_order_id;
    UPDATE inventory SET stock = stock - quantity 
    FROM order_items WHERE order_items.order_id = p_order_id;
END;
$$ LANGUAGE plpgsql;
```

**Why?** Ensures consistent business logic across applications (web, mobile, POS systems) and maintains data integrity during complex transactions.

---

### 🏦 **Banking & Finance**

**Use Case: Account Transactions**

```sql
-- Procedure: Transfer money between accounts
CREATE PROCEDURE transfer_funds(
    p_from_account INT,
    p_to_account INT,
    p_amount NUMERIC
)
AS $$
BEGIN
    -- Debit source account
    UPDATE accounts SET balance = balance - p_amount 
    WHERE account_id = p_from_account;
    
    -- Credit destination account
    UPDATE accounts SET balance = balance + p_amount 
    WHERE account_id = p_to_account;
    
    -- Log transaction
    INSERT INTO transaction_log VALUES (p_from_account, p_to_account, p_amount, NOW());
END;
$$ LANGUAGE plpgsql;
```

**Why?** Guarantees atomic operations (all steps succeed or all fail), prevents data corruption, and maintains audit trails for compliance.

---

### 🏥 **Healthcare**

**Use Case: Patient Management**

```sql
-- Function: Check bed availability by department
CREATE FUNCTION get_available_beds(p_department VARCHAR)
RETURNS INT AS $$
BEGIN
    RETURN (
        SELECT COUNT(*) FROM beds 
        WHERE department = p_department AND status = 'available'
    );
END;
$$ LANGUAGE plpgsql;

-- Procedure: Admit patient with medical history validation
CREATE PROCEDURE admit_patient(
    p_patient_id INT,
    p_bed_id INT,
    p_doctor_id INT
)
AS $$
BEGIN
    -- Check allergies and medications
    -- Assign bed
    UPDATE beds SET status = 'occupied' WHERE bed_id = p_bed_id;
    INSERT INTO admissions VALUES (p_patient_id, p_bed_id, p_doctor_id, NOW());
END;
$$ LANGUAGE plpgsql;
```

**Why?** Enforces medical protocols, prevents scheduling conflicts, and ensures regulatory compliance (HIPAA, GDPR).

---

### 📦 **Logistics & Supply Chain**

**Use Case: Inventory Management**

```sql
-- Function: Predict reorder date based on consumption rate
CREATE FUNCTION calculate_reorder_date(p_product_id INT)
RETURNS DATE AS $$
DECLARE
    current_stock INT;
    daily_usage NUMERIC;
BEGIN
    SELECT stock INTO current_stock FROM inventory WHERE product_id = p_product_id;
    SELECT AVG(daily_sales) INTO daily_usage FROM sales_history WHERE product_id = p_product_id;
    
    RETURN CURRENT_DATE + (current_stock / daily_usage)::INT;
END;
$$ LANGUAGE plpgsql;
```

**Why?** Automates supply chain decisions, reduces manual calculations, and optimizes warehouse operations.

---

### 🎮 **Gaming & Entertainment**

**Use Case: Leaderboards and Achievements**

```sql
-- Function: Get player rank
CREATE FUNCTION get_player_rank(p_player_id INT)
RETURNS INT AS $$
BEGIN
    RETURN (
        SELECT COUNT(*) + 1 FROM players 
        WHERE score > (SELECT score FROM players WHERE id = p_player_id)
    );
END;
$$ LANGUAGE plpgsql;

-- Procedure: Award achievement and update stats
CREATE PROCEDURE award_achievement(p_player_id INT, p_achievement_id INT)
AS $$
BEGIN
    INSERT INTO player_achievements VALUES (p_player_id, p_achievement_id, NOW());
    UPDATE players SET total_achievements = total_achievements + 1;
END;
$$ LANGUAGE plpgsql;
```

**Why?** Handles high-traffic operations efficiently and maintains consistent game logic across servers.

---

### 📊 **Analytics & Reporting**

**Use Case: Business Intelligence**

```sql
-- Function: Calculate monthly revenue by category
CREATE FUNCTION monthly_revenue_report(p_year INT, p_month INT)
RETURNS TABLE (category VARCHAR, revenue NUMERIC) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        c.category_name,
        SUM(o.total_amount)
    FROM orders o
    JOIN products p ON o.product_id = p.id
    JOIN categories c ON p.category_id = c.id
    WHERE EXTRACT(YEAR FROM o.order_date) = p_year
      AND EXTRACT(MONTH FROM o.order_date) = p_month
    GROUP BY c.category_name;
END;
$$ LANGUAGE plpgsql;
```

**Why?** Pre-aggregates complex reports, improves dashboard performance, and standardizes KPI calculations.

---

### 🔐 **Security & Compliance**

**Use Case: Data Privacy (GDPR/CCPA)**

```sql
-- Procedure: Anonymize user data upon deletion request
CREATE PROCEDURE anonymize_user_data(p_user_id INT)
AS $$
BEGIN
    UPDATE users SET 
        email = 'deleted_' || p_user_id || '@example.com',
        phone = NULL,
        address = NULL,
        deleted_at = NOW()
    WHERE id = p_user_id;
    
    INSERT INTO audit_log VALUES ('user_anonymized', p_user_id, NOW());
END;
$$ LANGUAGE plpgsql;
```

**Why?** Ensures legal compliance, maintains data lineage for audits, and automates privacy workflows.

---

## 🎯 Key Industry Benefits

| Benefit                  | Description                                                      |
| ------------------------ | ---------------------------------------------------------------- |
| **Performance**          | Reduces network overhead by processing data on the database side |
| **Security**             | Centralizes business logic and controls data access              |
| **Consistency**          | Ensures same rules apply across all applications                 |
| **Maintainability**      | Update logic once instead of in multiple codebases               |
| **Transaction Safety**   | Guarantees atomic operations for critical business processes     |
| **Code Reusability**     | Shared logic across web, mobile, and internal tools              |
| **Regulatory Compliance** | Enforces data handling rules and audit trails                    |

---

## 💡 Career Relevance

**Job Roles That Use This:**
- Database Administrators (DBAs)
- Backend Developers
- Data Engineers
- DevOps Engineers
- Business Intelligence Analysts
- Financial Systems Developers

**Interview Topics:**
- "Explain the difference between functions and stored procedures"
- "How would you handle a money transfer between accounts?"
- "Design a procedure to maintain inventory consistency"
- "What are the performance benefits of database functions?"

---

Students who master functions and procedures gain skills directly applicable to real-world systems, making them valuable in any data-driven industry.
