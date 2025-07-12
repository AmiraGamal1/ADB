# Lab 4: Database Security Measures

## Objective

To teach students how to implement basic database security mechanisms including:

- Authentication
- Role-based access control (RBAC)
- Data encryption
- SQL injection prevention in a DBMS environment

## Description

In this lab, students will:

- Understand how authentication and authorization work in a DBMS
- Learn to create user roles and assign privileges to tables and views
- Explore encryption for sensitive data
- Simulate SQL injection and implement preventive measures

---

## Tasks & Examples

## 1. Create Database and Tables

- Create a DB (`university_db`)
- Create Tables: `students`, `courses`
- Insert data

![Alt text](relative/path/to/image.png)
![Alt text](relative/path/to/image.png)

---

## 2. Create User Roles and Assign Permissions

In this step, we create specific MySQL users and assign them limited access to certain database objects (like tables).  
This helps control what actions each user can perform, such as allowing only `SELECT` access without `INSERT` or `UPDATE`.

We’ll create a read-only user named `data_viewer` and grant access to the tables `students` and `courses`.


```sql
-- Create a new user
CREATE USER 'data_viewer'@'localhost' IDENTIFIED BY 'viewerpass123';

-- Grant SELECT permission directly on specific tables
GRANT SELECT ON university_db.students TO 'analyst'@'localhost';
GRANT SELECT ON university_db.courses TO 'analyst'@'localhost';
```
---
## 3. Create Roles and Assign to User

Roles group privileges into reusable sets.  
Instead of assigning permissions to each user individually, we create a role (e.g., `readonly`) with specific privileges and assign it to multiple users.  
This is easier to manage, especially for large teams.

We’ll create a role named `readonly` and assign it to the `data_viewer` user we created earlier.

```sql
-- Create a read-only role
CREATE ROLE 'readonly';

-- Grant SELECT permissions to the role
GRANT SELECT ON university_db.* TO 'readonly';

-- Assign role to user
GRANT 'readonly' TO 'data_viewer'@'localhost';
```

**Expected Output:**

User can only view data; cannot insert, update, or delete.

---
### Test Instructions

1. Log in as `data_viewer`.
   ![Alt text](relative/path/to/image.png)

   
2. Click the **( + )** icon on the welcome screen to **Add a New Connection**.
   ![Alt text](relative/path/to/image.png)
   
   
3. Fill out the connection details and click **Test Connection**.
   ![Alt text](relative/path/to/image.png)

   
4. When you receive a successful connection pop-up message, press **OK**.
   ![Alt text](relative/path/to/image.png)




### Run the Following Code

```sql
-- Activate the readonly role
SET ROLE 'readonly';

-- Select the database
USE university_db;

-- ✅ This should work (SELECT permission granted)
SELECT * FROM students;

-- ❌ This should fail (no INSERT permission for students table)
INSERT INTO students (name, email) VALUES ('Test User', 'test@example.com');

-- ❌ This should fail (no UPDATE permission for teacher table)
UPDATE teachers SET salary = 90000 WHERE id = 1;
```
---
### Direct Grant vs Role-Based Access Control (RBAC)

Granting privileges directly to users (e.g.,  
`GRANT SELECT ON students TO 'data_viewer'`) works for individual access,  
but using roles (e.g.,  
`GRANT SELECT ON students TO 'readonly'; GRANT 'readonly' TO 'data_viewer';`)  
is more scalable and easier to manage across multiple users.

---

## 4. Column-Level Encryption (Simulated)

**What is it?**

Column-level encryption allows you to encrypt sensitive data (like Social Security Numbers, passwords, or medical records) at the column level within a table.  
This ensures that even if someone gains access to the database, the data remains unreadable without the decryption key.

MySQL provides built-in functions:

- `AES_ENCRYPT(data, key)` → encrypts data  
- `AES_DECRYPT(data, key)` → decrypts encrypted data  

💡 Encrypted data is stored as binary (e.g., `VARBINARY`), and both encryption & decryption must use the same key.

---

### Example: Encrypting Student SSNs in the previously created `students` table

**1. Insert encrypted data into the students table**
```sql
INSERT INTO students (name, email, ssn)
VALUES (
    'Mazen',
    'mazen@student.edu',
    AES_ENCRYPT('123-45-6789', 'secret_key')
);
```

The `secret_key` is the encryption key used for encrypting sensitive data. You must use the **same key** to decrypt the data later.

**2. Select and Decrypt Data**

The following SQL query selects the `name` and `email` columns and decrypts the `ssn` (social security number) column using the AES encryption key `secret_key`:

```sql
SELECT name, email, AES_DECRYPT(ssn, 'secret_key') AS decrypted_ssn
FROM students;
```
**Note:** If you use the wrong key, `decrypted_ssn` will return `NULL`.


---

## 4. Simulate SQL Injection & Prevention

SQL injection is a common attack where an attacker inserts malicious SQL code into input fields to manipulate queries. This section demonstrates an unsafe query and how to prevent it using prepared statements.

### Example (Using the admin “Root” editor)

### Simulate SQL Injection (Unsafe Query)

This demonstrates how malicious input like `'' OR '1'='1'` can trick a query into returning all rows, even if the condition should match only one row.

**1. Simulate unsafe user input**

```sql
SET @user_input = "' OR '1'='1";
```
**2. Dynamically build an unsafe SQL query**
```sql
SET @query = CONCAT("SELECT * FROM students WHERE name = '", @user_input, "'");
```
**3. Prepare and execute the query**
```sql
PREPARE stmt FROM @query;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```
**Effect:** Returns all users due to the injected
---
### Prevent SQL Injection- Safe Approach (Prepared Statements)

Prepared statements separate SQL code from user input. This ensures that input values are treated as data, not executable SQL, thus protecting the query from injection attacks.

**Step 1:** Prepare a safe query with a placeholder (Tells MySQL to prepare an SQL query in advance with a placeholder `?` for user input. Why it’s safe: The query structure is fixed and cannot be changed by user input.)
```sql
PREPARE stmt FROM 'SELECT * FROM students WHERE name = ?';
```
**Step 2:** Define the user input (Stores user input into a variable. Why it’s safe: The value 'Sara' is treated as data, not part of the query itself.)
```sql
SET @name_input = "Sara";
```

**Step 3:** Execute the statement safely using bound input ((What it does: Runs the query and plugs in the value stored in @name_input. Why it’s safe: MySQL automatically escapes and quotes the input, blocking any attempt to inject SQL.)
```sql
EXECUTE stmt USING @name_input;
DEALLOCATE PREPARE stmt;
```

**Expected Result:**  Returns only user ‘Sara’ without risk of injection.

--- 
**Try changing the input to** something malicious like `' OR '1'='1`, still MySQL will treats it as a `string`, not as part of the `SQL logic`.

PREPARE stmt FROM 'SELECT * FROM students WHERE name = ?';
SET @name_input = "' OR '1'='1"; 
EXECUTE stmt USING @name_input;
EXECUTE stmt USING @name_input;

**Expected Result:** No row will return.
--- 

# Assignment: Database Security 🔧

## Tasks to Complete

### 1. Role-Based and Column-Level Access Control

- **Create and test users:**
  - One user with **read-only access** to the entire database.
  - One user with **read and insert** permissions, but **no update or delete**.
  - One user with access to **only specific columns** (e.g., can view names but **not emails or IDs**).

- **Screenshot Requirements:**
  - User creation commands and privilege assignment.
  - Evidence of successful and failed attempts to access or modify tables or columns.

---

### 2. Column-Level Encryption

- Encrypt a sensitive column (e.g., national ID).

- Show how:
  - Data looks when encrypted.
  - Data can be safely decrypted.

- **Screenshot Requirements:**
  - Encrypted data insertion.
  - Decryption query and its output.
  - View comparing encrypted binary data vs readable decrypted results.

---

### 3. SQL Injection: Unsafe vs Safe

- **Report:**
  - Brief explanation (1-2 pages) of what SQL injection is and why preventing it is important.
  - Common prevention methods, e.g.:
    - Prepared statements
    - Input sanitization
    - Least Privilege Principle

---

## What to Submit

- A **PDF document** including:
  - Screenshots for each task.
  - 1-2 sentence caption under each screenshot explaining what it shows.
  - A 1-2 page report on SQL injection (as described above).




