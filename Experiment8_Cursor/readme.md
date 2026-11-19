# Experiment 8: PL/SQL Cursor Programs

## AIM
To write and execute PL/SQL programs using cursors and exception handling to manage runtime errors effectively and display appropriate messages.

## THEORY

In PL/SQL, cursors are used to handle query result sets row-by-row. 

There are two types of cursors:

- Implicit Cursors: Automatically created by PL/SQL for single-row queries.
- Explicit Cursors: Declared and controlled by the programmer for multi-row queries.

Types of Explicit Cursors:

1. Simple Cursor: Basic cursor to iterate over multiple rows.

2. Parameterized Cursor: Accepts parameters to filter the result dynamically.

3. Cursor FOR Loop: Simplifies cursor operations (open, fetch, close).

4. %ROWTYPE Cursor: Fetches entire row into a record using %ROWTYPE.

5. Cursor with FOR UPDATE: Used for row-level locking and updating the rows while looping.

**Syntax:**
```sql
DECLARE 
   <declarations section> 
BEGIN 
   <executable command(s)>
EXCEPTION 
   <exception handling> 
END;
```

### Basic Components of PL/SQL Block:

- DECLARE: Section to declare variables and constants.
- BEGIN: The execution section that contains PL/SQL statements.
- EXCEPTION: Handles errors or exceptions that occur in the program.
- END: Marks the end of the PL/SQL block.

**Exception Handling**

PL/SQL provides a robust mechanism to handle runtime errors using exception handling blocks. When an error occurs during execution, control is passed to the EXCEPTION section, where specific or general errors can be handled gracefully.

### Components of Exception Handling:
- Predefined Exceptions: Automatically raised by PL/SQL for common errors (e.g., NO_DATA_FOUND, TOO_MANY_ROWS, ZERO_DIVIDE).
- User-defined Exceptions: Declared explicitly in the declaration section using the EXCEPTION keyword.
- WHEN OTHERS: A generic handler for all exceptions not handled explicitly.

```sql
BEGIN
   -- Statements
EXCEPTION
   WHEN exception_name THEN
      -- Handling code
   WHEN OTHERS THEN
      -- Handling for unknown errors
END;
```

### **Question 1: Simple Cursor with Exception Handling**

**Write a PL/SQL program using a simple cursor to fetch employee names and designations from the `employees` table. Implement exception handling for the following cases:**

1. **NO_DATA_FOUND**: When no rows are fetched.
2. **OTHERS**: Any other unexpected errors during execution.

**Steps:**

- Create an `employees` table with fields `emp_id`, `emp_name`, and `designation`.
- Insert some sample data into the table.
- Use a simple cursor to fetch and display employee names and designations.
- Implement exception handling to catch the relevant exceptions and display appropriate messages.

**Output:**  
The program should display the employee details or an error message.
## Query ##
```
SET SERVEROUTPUT ON;

CREATE TABLE employees (
    emp_id NUMBER PRIMARY KEY,
    emp_name VARCHAR2(50),
    designation VARCHAR2(50)
);

INSERT INTO employees VALUES (1, 'Ravi', 'Manager');
INSERT INTO employees VALUES (2, 'Priya', 'Developer');
INSERT INTO employees VALUES (3, 'Arun', 'Tester');

COMMIT;

DECLARE
    CURSOR emp_cursor IS
        SELECT emp_name, designation
        FROM employees;

    v_emp_name employees.emp_name%TYPE;
    v_designation employees.designation%TYPE;
BEGIN
    OPEN emp_cursor;

    LOOP
        FETCH emp_cursor INTO v_emp_name, v_designation;

        EXIT WHEN emp_cursor%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(
            'Employee Name: ' || v_emp_name ||
            ', Designation: ' || v_designation
        );
    END LOOP;

    CLOSE emp_cursor;

    IF emp_cursor%ROWCOUNT = 0 THEN
        RAISE NO_DATA_FOUND;
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: No employee records found.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```

<img width="940" height="418" alt="image" src="https://github.com/user-attachments/assets/9b5123dc-bb5d-4ac6-a250-75bd24dbc0ca" />

---

### **Question 2: Parameterized Cursor with Exception Handling**

**Write a PL/SQL program using a parameterized cursor to retrieve and display employees with a salary in a given range. Implement exception handling for the following errors:**

1. **NO_DATA_FOUND**: When no employees meet the salary criteria.
2. **OTHERS**: For any unexpected errors during the execution.

**Steps:**

- Modify the `employees` table by adding a `salary` column.
- Insert sample salary values for the employees.
- Use a parameterized cursor to accept a salary range as input and fetch employees within that range.
- Implement exception handling to catch and display relevant error messages.

**Output:**  
The program should display the employee details within the specified salary range or an error message if no data is found.
## Query ##
```
SET SERVEROUTPUT ON;

-- Step 1: Add salary column
ALTER TABLE employees ADD salary NUMBER;

-- Step 2: Insert salary values
UPDATE employees SET salary = 50000 WHERE emp_id = 1;
UPDATE employees SET salary = 35000 WHERE emp_id = 2;
UPDATE employees SET salary = 25000 WHERE emp_id = 3;

COMMIT;

-- Step 3: Parameterized Cursor Program
DECLARE
    CURSOR emp_cursor (min_sal NUMBER, max_sal NUMBER) IS
        SELECT emp_id, emp_name, designation, salary
        FROM employees
        WHERE salary BETWEEN min_sal AND max_sal;

    v_id employees.emp_id%TYPE;
    v_name employees.emp_name%TYPE;
    v_designation employees.designation%TYPE;
    v_salary employees.salary%TYPE;

    v_count NUMBER := 0;

BEGIN
    OPEN emp_cursor(30000, 60000);

    LOOP
        FETCH emp_cursor INTO v_id, v_name, v_designation, v_salary;

        EXIT WHEN emp_cursor%NOTFOUND;

        v_count := v_count + 1;

        DBMS_OUTPUT.PUT_LINE('Employee ID: ' || v_id);
        DBMS_OUTPUT.PUT_LINE('Employee Name: ' || v_name);
        DBMS_OUTPUT.PUT_LINE('Designation: ' || v_designation);
        DBMS_OUTPUT.PUT_LINE('Salary: ' || v_salary);
        DBMS_OUTPUT.PUT_LINE('----------------------');
    END LOOP;

    CLOSE emp_cursor;

    IF v_count = 0 THEN
        RAISE NO_DATA_FOUND;
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No employees found in the given salary range.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Unexpected Error: ' || SQLERRM);

END;
/
```
<img width="942" height="412" alt="image" src="https://github.com/user-attachments/assets/24869820-5ad5-4011-996c-adc2981fb159" />

---

### **Question 3: Cursor FOR Loop with Exception Handling**

**Write a PL/SQL program using a cursor FOR loop to retrieve and display all employee names and their department numbers from the `employees` table. Implement exception handling for the following cases:**

1. **NO_DATA_FOUND**: If no employees are found in the database.
2. **OTHERS**: For any other unexpected errors.

**Steps:**

- Modify the `employees` table by adding a `dept_no` column.
- Insert sample department numbers for employees.
- Use a cursor FOR loop to fetch and display employee names along with their department numbers.
- Implement exception handling to catch the relevant exceptions.

**Output:**  
The program should display employee names with their department numbers or the appropriate error message if no data is found.
## Query ##
```
SET SERVEROUTPUT ON;

-- Step 1: Add department number column
ALTER TABLE employees ADD dept_no NUMBER;

-- Step 2: Insert department numbers
UPDATE employees SET dept_no = 10 WHERE emp_id = 1;
UPDATE employees SET dept_no = 20 WHERE emp_id = 2;
UPDATE employees SET dept_no = 30 WHERE emp_id = 3;

COMMIT;

-- Step 3: Cursor FOR Loop Program
DECLARE
    v_count NUMBER := 0;

    CURSOR emp_cursor IS
        SELECT emp_name, dept_no
        FROM employees;

BEGIN
    FOR emp_rec IN emp_cursor
    LOOP
        v_count := v_count + 1;

        DBMS_OUTPUT.PUT_LINE('Employee Name: ' || emp_rec.emp_name);
        DBMS_OUTPUT.PUT_LINE('Department Number: ' || emp_rec.dept_no);
        DBMS_OUTPUT.PUT_LINE('----------------------');
    END LOOP;

    IF v_count = 0 THEN
        RAISE NO_DATA_FOUND;
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No employee records found.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Unexpected Error: ' || SQLERRM);

END;
/
```
<img width="932" height="421" alt="image" src="https://github.com/user-attachments/assets/67a2481d-1c88-4c71-8cc2-17c2046a1708" />

---

### **Question 4: Cursor with `%ROWTYPE` and Exception Handling**

**Write a PL/SQL program that uses a cursor with `%ROWTYPE` to fetch and display complete employee records (emp_id, emp_name, designation, salary). Implement exception handling for the following errors:**

1. **NO_DATA_FOUND**: When no employees are found in the database.
2. **OTHERS**: For any other errors that occur.

**Steps:**

- Modify the `employees` table by adding `emp_id`, `emp_name`, `designation`, and `salary` fields.
- Insert sample data into the `employees` table.
- Declare a cursor using `%ROWTYPE` to fetch complete rows from the `employees` table.
- Implement exception handling to catch the relevant exceptions and display appropriate messages.

**Output:**  
The program should display employee records or the appropriate error message if no data is found.
## Query ##
```
SET SERVEROUTPUT ON;

-- Step 1: Insert sample employee data
INSERT INTO employees (emp_id, emp_name, designation, salary)
VALUES (1, 'Ravi', 'Manager', 50000);

INSERT INTO employees (emp_id, emp_name, designation, salary)
VALUES (2, 'Priya', 'Developer', 35000);

INSERT INTO employees (emp_id, emp_name, designation, salary)
VALUES (3, 'Arun', 'Tester', 25000);

COMMIT;

-- Step 2: Cursor using %ROWTYPE
DECLARE
    CURSOR emp_cursor IS
        SELECT emp_id, emp_name, designation, salary
        FROM employees;

    emp_rec employees%ROWTYPE;
    v_count NUMBER := 0;

BEGIN
    OPEN emp_cursor;

    LOOP
        FETCH emp_cursor INTO emp_rec.emp_id,
                             emp_rec.emp_name,
                             emp_rec.designation,
                             emp_rec.salary;

        EXIT WHEN emp_cursor%NOTFOUND;

        v_count := v_count + 1;

        DBMS_OUTPUT.PUT_LINE('Employee ID: ' || emp_rec.emp_id);
        DBMS_OUTPUT.PUT_LINE('Employee Name: ' || emp_rec.emp_name);
        DBMS_OUTPUT.PUT_LINE('Designation: ' || emp_rec.designation);
        DBMS_OUTPUT.PUT_LINE('Salary: ' || emp_rec.salary);
        DBMS_OUTPUT.PUT_LINE('----------------------');
    END LOOP;

    CLOSE emp_cursor;

    IF v_count = 0 THEN
        RAISE NO_DATA_FOUND;
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No employee records found.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Unexpected Error: ' || SQLERRM);

END;
/
```
<img width="942" height="401" alt="image" src="https://github.com/user-attachments/assets/e6ce68a1-e88b-4a61-98f1-d290b04eff89" />

---

### **Question 5: Cursor with FOR UPDATE Clause and Exception Handling**

**Write a PL/SQL program using a cursor with the `FOR UPDATE` clause to update the salary of employees in a specific department. Implement exception handling for the following cases:**

1. **NO_DATA_FOUND**: If no rows are affected by the update.
2. **OTHERS**: For any unexpected errors during execution.

**Steps:**

- Modify the `employees` table to include a `dept_no` and `salary` field.
- Insert sample data into the `employees` table with different department numbers.
- Use a cursor with the `FOR UPDATE` clause to lock the rows of employees in a specific department and update their salary.
- Implement exception handling to handle `NO_DATA_FOUND` or other errors that may occur.
- 

**Output:**  
The program should update employee salaries and display a message, or it should display an error message if no data is found.
## Query ##
```
SET SERVEROUTPUT ON;

-- Step 1: Update sample department and salary data
UPDATE employees SET dept_no = 10, salary = 50000 WHERE emp_id = 1;
UPDATE employees SET dept_no = 20, salary = 35000 WHERE emp_id = 2;
UPDATE employees SET dept_no = 10, salary = 25000 WHERE emp_id = 3;

COMMIT;

-- Step 2: Cursor with FOR UPDATE Clause
DECLARE
    CURSOR emp_cursor IS
        SELECT emp_id, emp_name, salary
        FROM employees
        WHERE dept_no = 10
        FOR UPDATE OF salary;

    v_count NUMBER := 0;

BEGIN
    FOR emp_rec IN emp_cursor
    LOOP
        UPDATE employees
        SET salary = salary + 5000
        WHERE CURRENT OF emp_cursor;

        v_count := v_count + 1;

        DBMS_OUTPUT.PUT_LINE('Employee Name: ' || emp_rec.emp_name);
        DBMS_OUTPUT.PUT_LINE('Updated Salary: ' || (emp_rec.salary + 5000));
        DBMS_OUTPUT.PUT_LINE('----------------------');
    END LOOP;

    IF v_count = 0 THEN
        RAISE NO_DATA_FOUND;
    END IF;

    COMMIT;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('No employees found in the given department.');

    WHEN OTHERS THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Unexpected Error: ' || SQLERRM);

END;
/
```
<img width="947" height="427" alt="image" src="https://github.com/user-attachments/assets/d953a48c-03ae-4d8f-b511-239b7cafdada" />

---

## RESULT
Thus, the program successfully executed and displayed employee details using a cursor. 

