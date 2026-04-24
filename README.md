# library-management-system-sql

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.


## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.

## Project Structure

### 1. Database Setup
![ERD](https://github.com/najirh/Library-System-Management---P2/blob/main/library_erd.png)

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_db;

-- Library Sysytem Management SQL PROJECT

-- CREATE DATABASE Library 


--Create table "Branch"
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);

-- Create table "Employee"

CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10) -- FK
           );

-- Create table "Members"

CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);

-- Create table "Books"

CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);

-- Create table "IssueStatus"

CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),--FK
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50), -- FK
            issued_emp_id VARCHAR(10) -- FK
            );

			
-- Create table "ReturnStatus"

CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50)
         
);

--Define relation between tables by adding foreign key

--FOREIGN KEY

ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY(issued_member_id)
REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY(issued_book_isbn)
REFERENCES books(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees
FOREIGN KEY(issued_emp_id)
REFERENCES employees(emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branch
FOREIGN KEY(branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY(issued_id)
REFERENCES issued_status(issued_id);

-- use commands - getting error while uploading files in issued_table

SELECT * FROM issued_status;

INSERT INTO issued_status(issued_id)
VALUES ('IS101');

SELECT * FROM issued_status WHERE issued_id = 'IS105';

INSERT INTO issued_status(issued_id)
VALUES ('IS105');

SELECT * FROM issued_status WHERE issued_id = 'IS105';

SELECT * FROM issued_status;

SELECT issued_id FROM issued_status;

INSERT INTO issued_status(issued_id)
VALUES ('IS103');

----Project Task

2-- CRUD Operations
-- Create: Inserted sample records into the books table.
-- Read: Retrieved and displayed data from various tables.
-- Update: Updated records in the employees table.
-- Delete: Removed records from the members table as needed.

--Task 1. Create a New Book Record--
-- '978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

SELECT * FROM books;

--Task 2: Update an Existing Member's Address 

UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';
SELECT * FROM members;

-- Task 3: Delete a Record from the Issued Status Table 
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

SELECT * FROM issued_status;

DELETE FROM issued_status
WHERE issued_id = 'IS121';

-- Task 4: Retrieve All Books Issued by a Specific Employee -- Objective: Select all books issued by the employee with emp_id = 'E101'

SELECT * FROM issued_status
WHERE issued_emp_id ='E101';

-- 5: List Members Who Have Issued More Than One Book -- Objective: Use GROUP BY to find members who have issued more than one book.
SELECT 
      issued_emp_id,
     COUNT(issued_id) as total_book_issued
FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(issued_id) > 1;

--3--CTAS (Create Table As Select)
--Task 6: Create Summary Tables: 
--Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

CREATE TABLE book_issued_cnt 
AS
SELECT
     b.isbn,
	 b.book_title,
	 COUNT(ist.issued_id) as no_issued
FROM books as b
JOIN
issued_status as ist
ON ist.issued_book_isbn =b.isbn
GROUP BY 1, 2;

SELECT * FROM  book_issued_cnt;

--4. Data Analysis & Findings
--The following SQL queries were used to address specific questions:

--Task 7. Retrieve All Books in a Specific Category:

SELECT * FROM  books
WHERE category = 'Classic'

--Task 8: Find Total Rental Income by Category:
SELECT 
     b.category,
	 SUM(b.rental_price),
	 COUNT(*)
	 FROM books as b
JOIN
issued_status as ist
ON ist.issued_book_isbn =b.isbn
GROUP BY 1

--Task 9: List Members Who Registered in the Last 180 Days:

SELECT *
FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';

--FIRST INSERT DATA FOR 180 DAYS INTERVAL from current days

INSERT INTO members(member_id, member_name, member_address, reg_date)
VALUES
('C120', 'ram', '111 Main St', '2026-04-24');

-- Task 10: List Employees with Their Branch Manager's Name and their branch details:

SELECT 
e1.*,
b.manager_id,
e2.emp_name as manager
FROM employees as e1
JOIN
branch as b
ON b.branch_id = e1.branch_id
JOIN
employees as e2
ON b.manager_id = e2.emp_id

-- Task 11. Create a Table of Books with Rental Price Above a Certain Threshold 7 USD:

CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;

SELECT * FROM expensive_books;

-- Task 12: Retrieve the List of Books Not Yet Returned

SELECT * 
FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;


## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.
   ```sh


2. **Set Up the Database**: Execute the SQL scripts in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## 👩‍💻 Author
Varsha Palwalia
QA Engineer | SQL | Testing Enthusiast





