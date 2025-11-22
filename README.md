# Library-System-Management
## Project Overview

Project Title: Library Management System

Level: Intermediate

Database: library_project_2db

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

## Objectives
-Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
- CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
- TAS (Create Table As Select): Utilize CTAS to create new tables based on query results.
- Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.

## Project Structure 

## 1.Database Setup
<img width="1657" height="2116" alt="schema pgerd" src="https://github.com/user-attachments/assets/f80d6407-02af-4322-b95f-b730ed320fb4" />

* Database Creation: Created a database named library_db.
* Table Creation: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
-- Library Management System Project 2
-- Creating Branch table 


DROP TABLE IF EXISTS branch;
CREATE TABLE branch (
       branch_id VARCHAR(10) PRIMARY KEY ,
	   manager_id VARCHAR(10), 
	   branch_address VARCHAR(55),	
	   contact_no VARCHAR(10)
);

ALTER TABLE branch
ALTER COLUMN contact_no TYPE VARCHAR(100);


DROP TABLE IF EXISTS employees;
CREATE TABLE employees 
       (emp_id VARCHAR(10) PRIMARY KEY,
        emp_name	VARCHAR (25),
        position VARCHAR (10),
        salary INT,
        branch_id VARCHAR (25) --FK

  );

DROP TABLE IF EXISTS books;
CREATE TABLE books
       (isbn VARCHAR(20) PRIMARY KEY,
	   book_title VARCHAR(100),
	   category VARCHAR(20),
	   rental_price FLOAT,
	   status VARCHAR(5),
	   author VARCHAR (55),
	   publisher VARCHAR (80)
  );


DROP TABLE IF EXISTS members;
CREATE TABLE members 
       (member_id VARCHAR(20) PRIMARY KEY,
	   member_name VARCHAR(25),
	   memeber_address VARCHAR(55),
	   reg_date DATE
);

ALTER TABLE members
RENAME COLUMN memeber_address TO member_address;


DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
       (issued_id VARCHAR(10) PRIMARY KEY,
	   issued_member_id VARCHAR(10),--FK
	   issued_book_name VARCHAR(100),
	   issued_date DATE,
	   issued_book_isbn VARCHAR(50),--FK
	   issued_emp_id VARCHAR (5) --FK
);

DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status 
       (return_id VARCHAR(10) PRIMARY KEY,
	   issued_id VARCHAR(10),
	   return_book_name VARCHAR(75),
	   return_date DATE,
	   return_book_isbn VARCHAR(20)

);

--FOREIGN KEY	

ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY (issued_member_id)
REFERENCES members (member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY (issued_book_isbn)
REFERENCES books (isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees
FOREIGN KEY (issued_emp_id)
REFERENCES employees (emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id);
```
## 2.CRUD Operations

- Create: Inserted sample records into the books table.
- Read: Retrieved and displayed data from various tables.
- Update: Updated records in the employees table.
- Delete: Removed records from the members table as needed.

###  1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES 
('978-1-60119-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.'
);
SELECT * FROM books;
```

### 2.Update an Existing Member's Address

```sql
UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';
SELECT * FROM members;
```

### 3.Delete a Record from the Issued Status Table -- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
```sql
DELETE FROM issued_status
WHERE issued_id = 'IS121';
SELECT * FROM issued_status;
```

### 4 Retrieve All Books Issued by a Specific Employee -- Objective: Select all books issued by the employee with emp_id = 'E101'.

```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```

### 5.List Members Who Have Issued More Than One Book -- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT
    issued_emp_id,
    COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1
```

### 6.Create Summary Tables: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status as ist
JOIN books as b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
```

### 7. Retrieve All Books in a Specific Category:

```sql
SELECT * FROM books 
WHERE category = 'Classic';
```

### 8.Find Total Rental Income by Category:

```sql
SELECT 
    b.category,
    SUM(b.rental_price),
    COUNT(*)
FROM 
issued_status as ist
JOIN
books as b
ON b.isbn = ist.issued_book_isbn
GROUP BY 1
```

### 9.List Members Who Registered in the Last 180 Days:

```
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

### 10.List Employees with Their Branch Manager's Name and their branch details:

```sql
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.*,
    e2.emp_name as manager
FROM employees as e1
JOIN 
branch as b
ON e1.branch_id = b.branch_id    
JOIN
employees as e2
ON e2.emp_id = b.manager_id
```

### 11.Create a Table of Books with Rental Price Above a Certain Threshold:

```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

### 12.Retrieve the List of Books Not Yet Returned

```sql
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

### 13. Identify Members with Overdue Books
--Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    -- rs.return_date,
    CURRENT_DATE - ist.issued_date as over_dues_days
FROM issued_status as ist
JOIN 
members as m
    ON m.member_id = ist.issued_member_id
JOIN 
books as bk
ON bk.isbn = ist.issued_book_isbn
LEFT JOIN 
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE 
    rs.return_date IS NULL
    AND
    (CURRENT_DATE - ist.issued_date) > 30
ORDER BY 1
```


### 14.Create a Table of Active Members
--Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql
CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN (SELECT 
                        DISTINCT issued_member_id   
                    FROM issued_status
                    WHERE 
                        issued_date >= CURRENT_DATE - INTERVAL '2 month'
                    )
;

SELECT * FROM active_members;
```

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
















