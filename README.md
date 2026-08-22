# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library\_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

!\[Library\_project](https://github.com/najirh/Library-System-Management---P2/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1\. Database Setup

!\[ERD](https://github.com/najirh/Library-System-Management---P2/blob/main/library\_erd.png)

* **Database Creation**: Created a database named `library\_db`.
* **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library\_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch\_id VARCHAR(10) PRIMARY KEY,
            manager\_id VARCHAR(10),
            branch\_address VARCHAR(30),
            contact\_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp\_id VARCHAR(10) PRIMARY KEY,
            emp\_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch\_id VARCHAR(10),
            FOREIGN KEY (branch\_id) REFERENCES  branch(branch\_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member\_id VARCHAR(10) PRIMARY KEY,
            member\_name VARCHAR(30),
            member\_address VARCHAR(30),
            reg\_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book\_title VARCHAR(80),
            category VARCHAR(30),
            rental\_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);



-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued\_status;
CREATE TABLE issued\_status
(
            issued\_id VARCHAR(10) PRIMARY KEY,
            issued\_member\_id VARCHAR(30),
            issued\_book\_name VARCHAR(80),
            issued\_date DATE,
            issued\_book\_isbn VARCHAR(50),
            issued\_emp\_id VARCHAR(10),
            FOREIGN KEY (issued\_member\_id) REFERENCES members(member\_id),
            FOREIGN KEY (issued\_emp\_id) REFERENCES employees(emp\_id),
            FOREIGN KEY (issued\_book\_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return\_status;
CREATE TABLE return\_status
(
            return\_id VARCHAR(10) PRIMARY KEY,
            issued\_id VARCHAR(30),
            return\_book\_name VARCHAR(80),
            return\_date DATE,
            return\_book\_isbn VARCHAR(50),
            FOREIGN KEY (return\_book\_isbn) REFERENCES books(isbn)
);

```

### 2\. CRUD Operations

* **Create**: Inserted sample records into the `books` table.
* **Read**: Retrieved and displayed data from various tables.
* **Update**: Updated records in the `employees` table.
* **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott \& Co.')"

```sql
INSERT INTO books(isbn, book\_title, category, rental\_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott \& Co.');
SELECT \* FROM books;
```

**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member\_address = '125 Oak St'
WHERE member\_id = 'C103';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued\_id = 'IS121' from the issued\_status table.

```sql
DELETE FROM issued\_status
WHERE   issued\_id =   'IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp\_id = 'E101'.

```sql
SELECT \* FROM issued\_status
WHERE issued\_emp\_id = 'E101'
```



**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT
    issued\_emp\_id,
    COUNT(\*)
FROM issued\_status
GROUP BY 1
HAVING COUNT(\*) > 1
```

### 3\. CTAS (Create Table As Select)

* **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book\_issued\_cnt\*\*

```sql
CREATE TABLE book\_issued\_cnt AS
SELECT b.isbn, b.book\_title, COUNT(ist.issued\_id) AS issue\_count
FROM issued\_status as ist
JOIN books as b
ON ist.issued\_book\_isbn = b.isbn
GROUP BY b.isbn, b.book\_title;
```



### 4\. Data Analysis \& Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT \* FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT 
    b.category,
    SUM(b.rental\_price),
    COUNT(\*)
FROM 
issued\_status as ist
JOIN
books as b
ON b.isbn = ist.issued\_book\_isbn
GROUP BY 1
```

9. **List Members Who Registered in the Last 180 Days**:

```sql
SELECT \* FROM members
WHERE reg\_date >= CURRENT\_DATE - INTERVAL '180 days';
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT 
    e1.emp\_id,
    e1.emp\_name,
    e1.position,
    e1.salary,
    b.\*,
    e2.emp\_name as manager
FROM employees as e1
JOIN 
branch as b
ON e1.branch\_id = b.branch\_id    
JOIN
employees as e2
ON e2.emp\_id = b.manager\_id
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:

```sql
CREATE TABLE expensive\_books AS
SELECT \* FROM books
WHERE rental\_price > 7.00;
```

Task 12: **Retrieve the List of Books Not Yet Returned**

```sql
SELECT \* FROM issued\_status as ist
LEFT JOIN
return\_status as rs
ON rs.issued\_id = ist.issued\_id
WHERE rs.return\_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's\_id, member's name, book title, issue date, and days overdue.

```sql
SELECT 
    ist.issued\_member\_id,
    m.member\_name,
    bk.book\_title,
    ist.issued\_date,
    -- rs.return\_date,
    CURRENT\_DATE - ist.issued\_date as over\_dues\_days
FROM issued\_status as ist
JOIN 
members as m
    ON m.member\_id = ist.issued\_member\_id
JOIN 
books as bk
ON bk.isbn = ist.issued\_book\_isbn
LEFT JOIN 
return\_status as rs
ON rs.issued\_id = ist.issued\_id
WHERE 
    rs.return\_date IS NULL
    AND
    (CURRENT\_DATE - ist.issued\_date) > 30
ORDER BY 1
```



**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return\_status table).



```sql

CREATE OR REPLACE PROCEDURE add\_return\_records(p\_return\_id VARCHAR(10), p\_issued\_id VARCHAR(10), p\_book\_quality VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v\_isbn VARCHAR(50);
    v\_book\_name VARCHAR(80);
    
BEGIN
    -- all your logic and code
    -- inserting into returns based on users input
    INSERT INTO return\_status(return\_id, issued\_id, return\_date, book\_quality)
    VALUES
    (p\_return\_id, p\_issued\_id, CURRENT\_DATE, p\_book\_quality);

    SELECT 
        issued\_book\_isbn,
        issued\_book\_name
        INTO
        v\_isbn,
        v\_book\_name
    FROM issued\_status
    WHERE issued\_id = p\_issued\_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v\_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v\_book\_name;
    
END;
$$


-- Testing FUNCTION add\_return\_records

issued\_id = IS135
ISBN = WHERE isbn = '978-0-307-58837-1'

SELECT \* FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT \* FROM issued\_status
WHERE issued\_book\_isbn = '978-0-307-58837-1';

SELECT \* FROM return\_status
WHERE issued\_id = 'IS135';

-- calling function 
CALL add\_return\_records('RS138', 'IS135', 'Good');

-- calling function 
CALL add\_return\_records('RS148', 'IS140', 'Good');

```





**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch\_reports
AS
SELECT 
    b.branch\_id,
    b.manager\_id,
    COUNT(ist.issued\_id) as number\_book\_issued,
    COUNT(rs.return\_id) as number\_of\_book\_return,
    SUM(bk.rental\_price) as total\_revenue
FROM issued\_status as ist
JOIN 
employees as e
ON e.emp\_id = ist.issued\_emp\_id
JOIN
branch as b
ON e.branch\_id = b.branch\_id
LEFT JOIN
return\_status as rs
ON rs.issued\_id = ist.issued\_id
JOIN 
books as bk
ON ist.issued\_book\_isbn = bk.isbn
GROUP BY 1, 2;

SELECT \* FROM branch\_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active\_members containing members who have issued at least one book in the last 2 months.

```sql

CREATE TABLE active\_members
AS
SELECT \* FROM members
WHERE member\_id IN (SELECT 
                        DISTINCT issued\_member\_id   
                    FROM issued\_status
                    WHERE 
                        issued\_date >= CURRENT\_DATE - INTERVAL '2 month'
                    )
;

SELECT \* FROM active\_members;

```



**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT 
    e.emp\_name,
    b.\*,
    COUNT(ist.issued\_id) as no\_book\_issued
FROM issued\_status as ist
JOIN
employees as e
ON e.emp\_id = ist.issued\_emp\_id
JOIN
branch as b
ON e.branch\_id = b.branch\_id
GROUP BY 1, 2
```

**Task 18: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.



**Task 19: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book\_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

CREATE OR REPLACE PROCEDURE issue\_book(p\_issued\_id VARCHAR(10), p\_issued\_member\_id VARCHAR(30), p\_issued\_book\_isbn VARCHAR(30), p\_issued\_emp\_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
-- all the variabable
    v\_status VARCHAR(10);

BEGIN
-- all the code
    -- checking if book is available 'yes'
    SELECT 
        status 
        INTO
        v\_status
    FROM books
    WHERE isbn = p\_issued\_book\_isbn;

    IF v\_status = 'yes' THEN

        INSERT INTO issued\_status(issued\_id, issued\_member\_id, issued\_date, issued\_book\_isbn, issued\_emp\_id)
        VALUES
        (p\_issued\_id, p\_issued\_member\_id, CURRENT\_DATE, p\_issued\_book\_isbn, p\_issued\_emp\_id);

        UPDATE books
            SET status = 'no'
        WHERE isbn = p\_issued\_book\_isbn;

        RAISE NOTICE 'Book records added successfully for book isbn : %', p\_issued\_book\_isbn;


    ELSE
        RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book\_isbn: %', p\_issued\_book\_isbn;
    END IF;
END;
$$

-- Testing The function
SELECT \* FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT \* FROM issued\_status;

CALL issue\_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue\_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT \* FROM books
WHERE isbn = '978-0-375-41398-8'

```



**Task 20: Create Table As Select (CTAS)**
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
The number of overdue books.
The total fines, with each day's fine calculated at $0.50.
The number of books issued by each member.
The resulting table should show:
Member ID
Number of overdue books
Total fines



## Reports

* **Database Schema**: Detailed table structures and relationships.
* **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
* **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.

```sh
   git clone https://github.com/najirh/Library-System-Management---P2.git
   ```

2. **Set Up the Database**: Execute the SQL scripts in the `database\_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis\_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## Author - PRAMOD KUMAR UPADHYAY

This project showcases SQL skills essential for database management and analysis. For more content on SQL and data analysis, connect with me through the following channels:

* **YouTube**: [Subscribe to my channel for tutorials and insights](https://www.youtube.com/@zero_analyst)
* **Instagram**: [Follow me for daily tips and updates](https://www.instagram.com/zero_analyst/)
* **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/najirr)
* **Discord**: [Join our community for learning and collaboration](https://discord.gg/36h5f2Z5PK)

Thank you for your interest in this project!

