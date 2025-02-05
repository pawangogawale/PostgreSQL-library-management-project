# PostgreSQL-library-management-project

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_project2_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.


## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure


-- 1. **Database Creation**: Created a database named `library_project2_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
-- Create table "Branch"

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
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
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);

```

-- Project task

-- 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

--Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 
--'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
```sql
Insert into books(
	isbn, book_title, category, rental_price, status, author, publisher
)
values ( '978-1-60129-456-2', 'To Kill a Mockingbird','Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')

Select * from books;
```

--Task 2: Update an Existing Member's Address
```sql
select * from members;

Update members
set member_address = '125 Main St'
where member_id = 'C101';
```
--Task 3: Delete a Record from the Issued Status Table
--Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
```sql
Select * from issued_status;

DELETE from issued_status
where issued_id = 'IS121';
```
--Task 4: Retrieve All Books Issued by a Specific Employee 
-- Objective: Select all books issued by the employee with emp_id = 'E101'.

```sql
Select issued_book_name 
from issued_status
where issued_emp_id = 'E101';
```
--Task 5: List Members Who Have Issued More Than One Book 
-- Objective: Use GROUP BY to find members who have issued more than one book.
```sql
select issued_member_id, 
count(*) from issued_status
group  by 1
having count(*)>1;
```
--3. CTAS (Create Table As Select)
--Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results 
-- each book and total book_issued_cnt**
```sql
Create table book_cnt
as
select 
	b.isbn,
	b.book_title,
	count(ist.issued_id) as no_issued
from books as b
join issued_status as ist
on b.isbn = ist.issued_book_isbn
group by 1,2;

Select * from book_cnt;

--Data Analysis & Findings
--The following SQL queries were used to address specific questions:
--Task 7. Retrieve All Books in a Specific Category:

select book_title, category from books
where category = 'Classic';
```
--Task 8: Find Total Rental Income by Category:
```sql
select 
	b.category,
	sum(b.rental_price),
	count(*)
from books as b
join issued_status as ist
on b.isbn = ist.issued_book_isbn
group by 1
```
--Task 9: List Members Who Registered in the Last 180 Days:
```sql
Select member_id,member_name, reg_date from members
where reg_date >= CURRENT_DATE - INTERVAL '180 days'
```
--Task 10 : List Employees with Their Branch Manager's Name and their branch details:
```sql
Select emp1.*,
	br.manager_id,
	emp2.emp_name as Manager
from branch as br
join employees as emp1
on br.branch_id = emp1.branch_id
join employees emp2
on br.manager_id = emp2.emp_id
```
--Task 11 : Create a Table of Books with Rental Price Above a Certain Threshold:
```sql
create table book_price_graterthan_7
as
select * from books
where rental_price > 7
```
--Task 12 : Retrieve the List of Books Not Yet Returned
```sql
select * from issued_status as i
left join return_status as r
on i.issued_id = r.issued_id
where r.issued_id is null;

```
--Advanced SQL Operations
--Task 13: Identify Members with Overdue Books
--Write a query to identify members who have overdue books (assume a 30-day return period). 
--Display the member's_id, member's name, book title, issue date, and days overdue.


-- issued status == books == members == return_status
```sql
select m.member_id,
	bk.book_title,
	ist.issued_date,
	--rt.return_date,
	current_date - ist.issued_date as overdue_days
from issued_status as ist
join books as bk
on ist.issued_book_isbn = bk.isbn
join members as m
on m.member_id = ist.issued_member_id
left join return_status rt
on ist.issued_id = rt.issued_id
where rt.return_date is null
	and
		(current_date- ist.issued_date)> 30
order by 1;
```
--Task 14: Update Book Status on Return
--Write a query to update the status of books in the books table to "Yes"  
--when they are returned (based on entries in the return_status table).
```sql

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';
-- IS104

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

UPDATE books
SET status = 'no'
WHERE isbn = '978-0-451-52994-2';

SELECT * FROM return_status
WHERE issued_id = 'IS130';

-- 
INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
VALUES
('RS125', 'IS130', CURRENT_DATE, 'Good');
SELECT * FROM return_status
WHERE issued_id = 'IS130';


-- Store Procedures
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
    
BEGIN
    -- all your logic and code
    -- inserting into returns based on users input
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES
    (p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

    SELECT 
        issued_book_isbn,
        issued_book_name
        INTO
        v_isbn,
        v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
    
END;
$$
    



-- Testing FUNCTION add_return_records

issued_id = IS135
ISBN = WHERE isbn = '978-0-307-58837-1'

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * FROM return_status
WHERE issued_id = 'IS135';

-- calling function 
CALL add_return_records('RS138', 'IS135', 'Good');



-- calling function 
CALL add_return_records('RS148', 'IS140', 'Good');
```
--Task 15: Branch Performance Report
--Create a query that generates a performance report for each branch, showing the number of books
-- issued, the number of books returned, and the total revenue generated from book rentals.

```sql
Drop table if exists Branch_Report;

create table Branch_Report 
AS
select 
	e.branch_id,
	ist.issued_emp_id,
	sum(b.rental_price) rental_income,
	count(ist.issued_id) total_issued, 
	count(rt.return_id) total_return
from issued_status as ist
left join return_status as rt
on ist.issued_id = rt.issued_id
join employees as e
on e.emp_id = ist.issued_emp_id
join books as b
on ist.issued_book_isbn = b.isbn
group by e.branch_id,ist.issued_emp_id
order by 1;

Select Branch_id, 
	sum(Rental_income) Rental_income,
	sum(total_issued) total_issued,
	sum(total_return) total_return
from Branch_Report 
group by 1;
```
--Task 16: CTAS: Create a Table of Active Members
--Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing 
--members who have issued at least one book in the last 2 months.
```sql
Create table Active_members
as
Select member_id from members
where member_id in (
select distinct(issued_member_id) from issued_status
where 
	issued_date >= current_date - INTERVAL'2 months'
);

Select * from active_members;
```
--Task 17: Find Employees with the Most Book Issues Processed
--Write a query to find the top 3 employees who have processed the most book issues. 
--Display the employee name, number of books processed, and their branch.
```sql

Drop table if exists top3_employees

create table top3_employees
as
(Select issued_emp_id as emp_id, count(issued_id) as no_book_issued from issued_status
group by 1
order by 2 desc
limit 3);

Select emp.emp_name, t3emp.no_book_issued, emp.branch_id from top3_employees as t3emp
join 
	employees as emp
on t3emp.emp_id = emp.emp_id;
```
--Task 18: Identify Members Issuing High-Risk Books
--Write a query to identify members who have issued books more than twice with the status 
--"damaged" in the books table. Display the member name, book title, and the number of times 
--they've issued damaged books.
```sql
drop table if exists damage_books
create table damage_books
as
(
Select ist.issued_member_id,ist.issued_book_isbn,m.member_name, rt.book_quality
from return_status as rt
join issued_status as ist
on ist.issued_id = rt.issued_id
join members as m
on m.member_id = ist.issued_member_id
where rt.book_quality = 'Damaged'
);


select d.member_name,b.book_title, count(d.book_quality) from damage_books as d
join books as b
on d.issued_book_isbn = b.isbn
group by 1,2
```
--Task 19: Stored Procedure 
--Objective: Create a stored procedure to manage the status of books in a library system. 
--Description: Write a stored procedure that updates the status of a book in the library based 
--on its issuance. The procedure should function as follows: The stored procedure should take 
--the book_id as an input parameter. The procedure should first check if the book is 
--available (status = 'yes'). If the book is available, it should be issued, and the status in 
--the books table should be updated to 'no'. If the book is not available (status = 'no'), the 
--procedure should return an error message indicating that the book is currently not available.

```sql
Create or replace procedure book_issuance
	(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10))
language plpgsql
as $$
Declare
	v_status varchar(10);

begin
	Select status into v_status from books
	where isbn = p_issued_book_isbn;
		if v_status = 'yes' then

		Insert into issued_status(issued_id, issued_member_id,issued_date, issued_book_isbn, issued_emp_id)
		values (p_issued_id, p_issued_member_id,current_date, p_issued_book_isbn, p_issued_emp_id);

		update books
		set status = 'no'
		where isbn = p_issued_book_isbn;

		RAISE NOTICE 'Book records added successfully for book isbn : %', p_issued_book_isbn;

		else
		  RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;

		end if;
		
end;
$$

--Testing The function
SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;

CALL book_issuance('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL book_issuance('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'
```

--Task 20: Create Table As Select (CTAS) 
--Objective: Create a CTAS (Create Table As Select) query to identify 
--overdue books and calculate fines.
--Description: Write a CTAS query to create a new table that lists each member and the books 
--they have issued but not returned within 30 days. The table should include: The number of 
--overdue books. The total fines, with each day's fine calculated at $0.50. The number of 
--books issued by each member. The resulting table should show: Member ID Number of overdue 
--books Total fines

```sql
drop table if exists overdues;
Create table overdues
as
select ist.issued_member_id, ((rt.return_date - ist.issued_date) - 30) no_overdue_days from issued_status as ist
join return_status as rt
on ist.issued_id = rt.issued_id;

select issued_member_id,
	count(no_overdue_days) overdues, 
	sum(no_overdue_days * 0.50) total_fine
from overdues
group by 1;
```






