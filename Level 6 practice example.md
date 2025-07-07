# MySQL Database Tutorial: Algonquin College Academic Management System

## 🗂️ Key Terms Index
*Click on any linked command throughout this guide to jump to its definition*

| **Command/Concept** | **Definition** |
|---------------------|----------------|
| [CREATE DATABASE](#create-database) | Creates a new database with specified name and optional settings |
| [CREATE TABLE](#create-table) | Defines a new table structure with columns and constraints |
| [PRIMARY KEY](#primary-key) | Uniquely identifies each row in a table, cannot be NULL |
| [FOREIGN KEY](#foreign-key) | Enforces referential integrity between tables |
| [AUTO_INCREMENT](#auto-increment) | Automatically generates sequential numbers for new rows |
| [UNIQUE](#unique) | Ensures all values in a column are distinct |
| [CHECK](#check) | Validates data against specified conditions |
| [ENGINE=InnoDB](#engine-innodb) | Specifies InnoDB storage engine (supports transactions, foreign keys) |
| [CREATE USER](#create-user) | Adds a new user account to MySQL server |
| [GRANT](#grant) | Assigns privileges to users or roles |
| [CREATE ROLE](#create-role) | Defines a named collection of privileges |
| [CREATE INDEX](#create-index) | Creates an index to improve query performance |
| [CREATE PROCEDURE](#create-procedure) | Defines reusable SQL logic as a stored procedure |
| [CREATE FUNCTION](#create-function) | Defines a function that returns a value |
| [INSERT INTO](#insert-into) | Adds new rows to a table |
| [SELECT](#select) | Retrieves data from tables |
| [JOIN](#join) | Combines data from multiple tables |
| [GROUP BY](#group-by) | Groups rows sharing common values |
| [HAVING](#having) | Filters grouped results |
| [ORDER BY](#order-by) | Sorts query results |
| [DELIMITER](#delimiter) | Changes statement delimiter for procedures/functions |
| [REGEXP](#regexp) | Pattern matching using regular expressions |
| [FLUSH PRIVILEGES](#flush-privileges) | Reloads privilege tables from grant tables |
| [OPTIMIZE TABLE](#optimize-table) | Reorganizes table data for better performance |
| [BACKUP](#backup) | Creates a copy of database for recovery purposes |

---

## 🚀 Getting Started Guide

This guide is designed for beginners aiming for Bloom's Taxonomy Tier 6 (Create/Evaluate). You'll not only build a normalized database but also understand the *why* behind each step, using MySQL Workbench and MySQL Server Application.

---

## 🖥️ MySQL Server Application & Workbench Navigation

### 1. Install MySQL Server & Workbench

- **Download MySQL Server 8.0+**: [MySQL Server Download](https://dev.mysql.com/downloads/mysql/)
- **Download MySQL Workbench**: [Workbench Download](https://dev.mysql.com/downloads/workbench/)

### 2. Launch MySQL Server Application

- **Windows**: Open "MySQL Installer" or "MySQL Notifier" from Start Menu.
- **macOS/Linux**: Use terminal:  
    ```bash
    sudo systemctl start mysql
    ```

- **Verify Server is Running**:  
    In Workbench, click the connection tile (e.g., "Local instance MySQL80").  
    If you see the SQL editor, you're connected!

### 3. Navigating MySQL Workbench

- **Home Screen**: Main dashboard with connections and shortcuts
- **Create New Connection**: Click `+` next to "MySQL Connections", enter credentials, and save
- **Open SQL Editor**: Double-click your connection. Use the left sidebar for schemas, tables, and management

---

## 📊 ER Diagram: Database Design Overview

**Why ER Diagrams?**  
Entity-Relationship diagrams help visualize tables, relationships, and cardinality before coding. Analyzing these diagrams ensures your design is normalized and free from redundancy.

```mermaid
erDiagram
    Departments ||--o{ Programs : contains
    Programs ||--o{ Students : "enrolls"
    Programs ||--o{ Courses : "offers"
    Students ||--o{ Enrollments : "registers"
    Courses ||--o{ Enrollments : "has"
    Courses ||--o{ Prerequisites : "requires"
    Professors ||--o{ Course_Assignments : "teaches"
    Courses ||--o{ Course_Assignments : "assigned"
```

**Relationships:**
- Departments → Programs (1:M)
- Programs → Students (1:M)  
- Students ↔ Courses (M:N via Enrollments)
- Courses → Prerequisites (1:M)
- Professors ↔ Courses (M:N via Course_Assignments)

---

## 🔧 Step 1: Database Setup

### 1.1 Create Database

**In Workbench:**
- Click the "Create a new schema" icon (cylinder with a star).
- Name it `algonquin`, set collation to `utf8mb4_unicode_ci`, and click "Apply".

**Or run:**
```sql
CREATE DATABASE algonquin COLLATE utf8mb4_unicode_ci;
USE algonquin;
```

### 1.2 Create Admin User

**In Workbench:**
- Go to "Server" > "Users and Privileges".
- Click "Add Account", set username `ac_admin`, password, and assign privileges.

**Or run:**
```sql
CREATE USER 'ac_admin'@'localhost' IDENTIFIED BY 'secure_password_123';
GRANT ALL PRIVILEGES ON algonquin.* TO 'ac_admin'@'localhost';
FLUSH PRIVILEGES;
```

---

## 🏗️ Step 2: Create Strong Entities

**Why?**  
Strong entities are independent tables forming the backbone of your schema.

### 2.1 Departments Table

```sql
CREATE TABLE Departments (
        dept_id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(50) NOT NULL UNIQUE
) ENGINE=InnoDB;
```

### 2.2 Programs Table

```sql
CREATE TABLE Programs (
        program_id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(50) NOT NULL UNIQUE,
        min_credits INT DEFAULT 60 CHECK (min_credits >= 60),
        dept_id INT NOT NULL,
        FOREIGN KEY (dept_id) REFERENCES Departments(dept_id)
) ENGINE=InnoDB;
```

### 2.3 Students Table

```sql
CREATE TABLE Students (
        student_id BIGINT PRIMARY KEY 
                CHECK (student_id BETWEEN 100000000 AND 999999999),
        first_name VARCHAR(50) NOT NULL,
        last_name VARCHAR(50) NOT NULL,
        email VARCHAR(120) UNIQUE 
                CHECK (email REGEXP '^[a-z]+\\.[a-z]+@algonquinlive\\.com$'),
        program_id INT NOT NULL,
        birth_date DATE,
        enrollment_date DATE DEFAULT (CURRENT_DATE),
        FOREIGN KEY (program_id) REFERENCES Programs(program_id)
) ENGINE=InnoDB;
```

### 2.4 Professors Table

```sql
CREATE TABLE Professors (
        professor_id INT AUTO_INCREMENT PRIMARY KEY,
        first_name VARCHAR(50) NOT NULL,
        last_name VARCHAR(50) NOT NULL,
        email VARCHAR(120) UNIQUE 
                CHECK (email LIKE '%@algonquinlive.com'),
        department_id INT NOT NULL,
        hire_date DATE,
        FOREIGN KEY (department_id) REFERENCES Departments(dept_id)
) ENGINE=InnoDB;
```

### 2.5 Courses Table

```sql
CREATE TABLE Courses (
        course_code CHAR(7) PRIMARY KEY 
                CHECK (course_code REGEXP '^[A-Z]{3}[0-9]{4}$'),
        title VARCHAR(100) NOT NULL,
        credits INT NOT NULL CHECK (credits BETWEEN 1 AND 6),
        description TEXT
) ENGINE=InnoDB;
```

---

## 🔗 Step 3: Weak Entities & Relationships

**Why?**  
Weak entities depend on strong entities for their existence (e.g., Enrollments need Students and Courses).

### 3.1 Enrollments Table

```sql
CREATE TABLE Enrollments (
        enrollment_id INT AUTO_INCREMENT PRIMARY KEY,
        student_id BIGINT NOT NULL,
        course_code CHAR(7) NOT NULL,
        semester CHAR(5) NOT NULL CHECK (semester REGEXP '^[FWS][0-9]{4}$'),
        grade CHAR(2) CHECK (grade IN ('A+', 'A', 'B+', 'B', 'C+', 'C', 'D+', 'D', 'F', 'W')),
        FOREIGN KEY (student_id) REFERENCES Students(student_id) ON DELETE CASCADE,
        FOREIGN KEY (course_code) REFERENCES Courses(course_code)
) ENGINE=InnoDB;
```

### 3.2 Prerequisites Table

```sql
CREATE TABLE Prerequisites (
        course_code CHAR(7) NOT NULL,
        prereq_code CHAR(7) NOT NULL,
        PRIMARY KEY (course_code, prereq_code),
        FOREIGN KEY (course_code) REFERENCES Courses(course_code),
        FOREIGN KEY (prereq_code) REFERENCES Courses(course_code)
) ENGINE=InnoDB;
```

### 3.3 Course Assignments Table

```sql
CREATE TABLE Course_Assignments (
        assignment_id INT AUTO_INCREMENT PRIMARY KEY,
        course_code CHAR(7) NOT NULL,
        professor_id INT NOT NULL,
        semester CHAR(5) NOT NULL,
        FOREIGN KEY (course_code) REFERENCES Courses(course_code),
        FOREIGN KEY (professor_id) REFERENCES Professors(professor_id)
) ENGINE=InnoDB;
```

---

## ⚙️ Step 4: Optimizations & Security

### 4.1 Indexes

```sql
CREATE INDEX idx_student_name ON Students(last_name, first_name);
CREATE INDEX idx_course_title ON Courses(title);
CREATE INDEX idx_enrollment_semester ON Enrollments(semester);
CREATE INDEX idx_enrollment_grade ON Enrollments(grade);
```

### 4.2 User Roles & Permissions

```sql
CREATE ROLE 'professor';
GRANT SELECT, UPDATE(grade) ON algonquin.Enrollments TO 'professor';

CREATE ROLE 'student';
GRANT SELECT ON algonquin.Courses TO 'student';
GRANT SELECT ON algonquin.Enrollments TO 'student';

CREATE USER 'dr_smith'@'localhost' IDENTIFIED BY 'prof_password';
GRANT 'professor' TO 'dr_smith'@'localhost';
```

### 4.3 Stored Procedures

```sql
DELIMITER //

CREATE PROCEDURE RegisterStudent(
        IN student_id BIGINT,
        IN course_code CHAR(7),
        IN semester CHAR(5)
)
BEGIN
        INSERT INTO Enrollments (student_id, course_code, semester)
        VALUES (student_id, course_code, semester);
END //

CREATE FUNCTION CalculateGPA(student BIGINT) 
RETURNS DECIMAL(3,2)
DETERMINISTIC
BEGIN
        DECLARE total_points DECIMAL(10,2) DEFAULT 0;
        DECLARE total_credits INT DEFAULT 0;
        
        SELECT SUM(
                CASE grade
                        WHEN 'A+' THEN 4.5 * credits
                        WHEN 'A'  THEN 4.0 * credits
                        WHEN 'B+' THEN 3.5 * credits
                        WHEN 'B'  THEN 3.0 * credits
                        WHEN 'C+' THEN 2.5 * credits
                        WHEN 'C'  THEN 2.0 * credits
                        WHEN 'D+' THEN 1.5 * credits
                        WHEN 'D'  THEN 1.0 * credits
                        ELSE 0
                END
        ), SUM(credits)
        INTO total_points, total_credits
        FROM Enrollments e
        JOIN Courses c ON e.course_code = c.course_code
        WHERE e.student_id = student AND grade NOT IN ('W', 'F');
        
        RETURN total_points / NULLIF(total_credits, 0);
END //

DELIMITER ;
```

---

## 📊 Step 5: Insert Sample Data

### 5.1 Add Departments

```sql
INSERT INTO Departments (name) VALUES 
('School of Advanced Technology'),
('School of Business'),
('School of Health Sciences');
```

### 5.2 Add Programs

```sql
INSERT INTO Programs (name, min_credits, dept_id) VALUES
('Computer Programming', 60, 1),
('Business Administration', 60, 2),
('Nursing', 75, 3);
```

### 5.3 Add Courses

```sql
INSERT INTO Courses (course_code, title, credits) VALUES
('CST8284', 'Database Systems', 3),
('CST8285', 'Web Programming', 3),
('BUS2301', 'Business Communications', 4),
('NUR1010', 'Anatomy Fundamentals', 6);
```

### 5.4 Add Students

```sql
INSERT INTO Students (student_id, first_name, last_name, email, program_id) VALUES
(100123456, 'John', 'Doe', 'john.doe@algonquinlive.com', 1),
(100654321, 'Jane', 'Smith', 'jane.smith@algonquinlive.com', 3);
```

---

## 🔍 Step 6: Example Queries

### 6.1 Student Course Load

```sql
SELECT s.first_name, s.last_name, 
             COUNT(e.course_code) AS num_courses,
             SUM(c.credits) AS total_credits
FROM Students s
JOIN Enrollments e ON s.student_id = e.student_id
JOIN Courses c ON e.course_code = c.course_code
WHERE e.semester = 'W2024'
GROUP BY s.student_id;
```

### 6.2 Prerequisite Check

```sql
SELECT s.student_id, s.first_name, s.last_name, c.title AS course
FROM Students s
JOIN Enrollments e ON s.student_id = e.student_id
JOIN Courses c ON e.course_code = c.course_code
WHERE EXISTS (
        SELECT 1 
        FROM Prerequisites p
        WHERE p.course_code = c.course_code
            AND p.prereq_code NOT IN (
                SELECT course_code 
                FROM Enrollments 
                WHERE student_id = s.student_id AND grade NOT IN ('F', 'W')
            )
);
```

### 6.3 Professor Workload

```sql
SELECT p.first_name, p.last_name, 
             COUNT(ca.course_code) AS courses_teaching
FROM Professors p
JOIN Course_Assignments ca ON p.professor_id = ca.professor_id
WHERE ca.semester = 'W2024'
GROUP BY p.professor_id;
```

---

## 🛠️ Maintenance & Backup

### Backup Database

```bash
mysqldump -u ac_admin -p algonquin > algonquin_backup_$(date +%F).sql
```

### Optimize Tables

```sql
OPTIMIZE TABLE Students, Enrollments, Courses;
```

---

## 📝 Complete Setup Script

```sql
-- 1. DATABASE SETUP
DROP DATABASE IF EXISTS algonquin;
CREATE DATABASE algonquin COLLATE utf8mb4_unicode_ci;
USE algonquin;

-- Create admin user
DROP USER IF EXISTS 'ac_admin'@'localhost';
CREATE USER 'ac_admin'@'localhost' IDENTIFIED BY 'secure_password_123';
GRANT ALL PRIVILEGES ON algonquin.* TO 'ac_admin'@'localhost';
FLUSH PRIVILEGES;

-- 2. CREATE STRONG ENTITIES
CREATE TABLE Departments (
    dept_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
) ENGINE=InnoDB;

CREATE TABLE Programs (
    program_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    min_credits INT DEFAULT 60 CHECK (min_credits >= 60),
    dept_id INT NOT NULL,
    FOREIGN KEY (dept_id) REFERENCES Departments(dept_id)
) ENGINE=InnoDB;

CREATE TABLE Courses (
    course_code CHAR(7) PRIMARY KEY CHECK (course_code REGEXP '^[A-Z]{3}[0-9]{4}$'),
    title VARCHAR(100) NOT NULL,
    credits INT NOT NULL CHECK (credits BETWEEN 1 AND 6),
    description TEXT
) ENGINE=InnoDB;

CREATE TABLE Students (
    student_id BIGINT PRIMARY KEY CHECK (student_id BETWEEN 100000000 AND 999999999),
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(120) UNIQUE CHECK (email REGEXP '^[a-z]+\\.[a-z]+@algonquinlive\\.com$'),
    program_id INT NOT NULL,
    birth_date DATE,
    enrollment_date DATE DEFAULT (CURRENT_DATE),
    FOREIGN KEY (program_id) REFERENCES Programs(program_id)
) ENGINE=InnoDB;

CREATE TABLE Professors (
    professor_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(120) UNIQUE CHECK (email LIKE '%@algonquinlive.com'),
    department_id INT NOT NULL,
    hire_date DATE,
    FOREIGN KEY (department_id) REFERENCES Departments(dept_id)
) ENGINE=InnoDB;

-- 3. WEAK ENTITIES & RELATIONSHIPS
CREATE TABLE Enrollments (
    enrollment_id INT AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT NOT NULL,
    course_code CHAR(7) NOT NULL,
    semester CHAR(5) NOT NULL CHECK (semester REGEXP '^[FWS][0-9]{4}$'),
    grade CHAR(2) CHECK (grade IN ('A+', 'A', 'B+', 'B', 'C+', 'C', 'D+', 'D', 'F', 'W')),
    FOREIGN KEY (student_id) REFERENCES Students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (course_code) REFERENCES Courses(course_code)
) ENGINE=InnoDB;

CREATE TABLE Prerequisites (
    course_code CHAR(7) NOT NULL,
    prereq_code CHAR(7) NOT NULL,
    PRIMARY KEY (course_code, prereq_code),
    FOREIGN KEY (course_code) REFERENCES Courses(course_code),
    FOREIGN KEY (prereq_code) REFERENCES Courses(course_code)
) ENGINE=InnoDB;

CREATE TABLE Course_Assignments (
    assignment_id INT AUTO_INCREMENT PRIMARY KEY,
    course_code CHAR(7) NOT NULL,
    professor_id INT NOT NULL,
    semester CHAR(5) NOT NULL,
    FOREIGN KEY (course_code) REFERENCES Courses(course_code),
    FOREIGN KEY (professor_id) REFERENCES Professors(professor_id)
) ENGINE=InnoDB;

-- 4. OPTIMIZATIONS & SECURITY
CREATE INDEX idx_student_name ON Students(last_name, first_name);
CREATE INDEX idx_course_title ON Courses(title);
CREATE INDEX idx_enrollment_semester ON Enrollments(semester);
CREATE INDEX idx_enrollment_grade ON Enrollments(grade);

-- 5. SAMPLE DATA
INSERT INTO Departments (name) VALUES 
('School of Advanced Technology'),
('School of Business'),
('School of Health Sciences');

INSERT INTO Programs (name, min_credits, dept_id) VALUES
('Computer Programming', 60, 1),
('Business Administration', 60, 2),
('Nursing', 75, 3);

INSERT INTO Courses (course_code, title, credits) VALUES
('CST8284', 'Database Systems', 3),
('CST8285', 'Web Programming', 3),
('BUS2301', 'Business Communications', 4),
('NUR1010', 'Anatomy Fundamentals', 6);

INSERT INTO Students (student_id, first_name, last_name, email, program_id) VALUES
(100123456, 'John', 'Doe', 'john.doe@algonquinlive.com', 1),
(100654321, 'Jane', 'Smith', 'jane.smith@algonquinlive.com', 3);

-- 6. VERIFICATION
SELECT 'Database created successfully!' AS Status;
```

---

## 🌟 Why This Sequence Matters (Bloom's Tier 6)

- **Analyze**: ER diagrams and normalization prevent redundancy and ensure data integrity
- **Evaluate**: User roles and permissions enforce security best practices
- **Create**: Stored procedures and functions automate business logic
- **Apply**: Sample queries demonstrate real-world data retrieval and reporting
- **Understand**: Visual navigation in Workbench builds confidence in managing databases
- **Remember**: Regular backups and optimizations ensure data safety and performance

---

## 📚 Conclusion

You have:
- Designed a normalized, secure, and scalable database
- Used MySQL Workbench and Server Application for hands-on learning
- Visualized relationships with ER diagrams
- Practiced advanced SQL and automation

**Next Steps:**
- Experiment with more complex queries
- Try reverse engineering ER diagrams in Workbench (`Database > Reverse Engineer`)
- Build a simple frontend to interact with your database

---

## 🗂️ Command Definitions

### <a id="create-database"></a>CREATE DATABASE
Creates a new database with specified name and optional settings like character set and collation.

### <a id="create-table"></a>CREATE TABLE
Defines a new table structure with columns, data types, and constraints.

### <a id="primary-key"></a>PRIMARY KEY
Uniquely identifies each row in a table, cannot be NULL, and creates a unique index.

### <a id="foreign-key"></a>FOREIGN KEY
Enforces referential integrity by linking to a primary key in another table.

### <a id="auto-increment"></a>AUTO_INCREMENT
Automatically generates sequential numbers for new rows, typically used with primary keys.

### <a id="unique"></a>UNIQUE
Ensures all values in a column or combination of columns are distinct.

### <a id="check"></a>CHECK
Validates data against specified conditions before allowing insertion or updates.

### <a id="engine-innodb"></a>ENGINE=InnoDB
Specifies InnoDB storage engine, which supports transactions, foreign keys, and row-level locking.

### <a id="create-user"></a>CREATE USER
Adds a new user account to the MySQL server with authentication credentials.

### <a id="grant"></a>GRANT
Assigns privileges (SELECT, INSERT, UPDATE, DELETE, etc.) to users or roles.

### <a id="create-role"></a>CREATE ROLE
Defines a named collection of privileges that can be assigned to multiple users.

### <a id="create-index"></a>CREATE INDEX
Creates an index on one or more columns to improve query performance.

### <a id="create-procedure"></a>CREATE PROCEDURE
Defines reusable SQL logic that can be called with parameters.

### <a id="create-function"></a>CREATE FUNCTION
Defines a function that performs calculations and returns a value.

### <a id="insert-into"></a>INSERT INTO
Adds new rows to a table with specified values.

### <a id="select"></a>SELECT
Retrieves data from one or more tables with optional filtering and ordering.

### <a id="join"></a>JOIN
Combines data from multiple tables based on related columns.

### <a id="group-by"></a>GROUP BY
Groups rows that have the same values in specified columns for aggregate functions.

### <a id="having"></a>HAVING
Filters grouped results, similar to WHERE but for grouped data.

### <a id="order-by"></a>ORDER BY
Sorts query results by one or more columns in ascending or descending order.

### <a id="delimiter"></a>DELIMITER
Changes the statement delimiter (default semicolon) for creating procedures and functions.

### <a id="regexp"></a>REGEXP
Pattern matching using regular expressions for complex text validation.

### <a id="flush-privileges"></a>FLUSH PRIVILEGES
Reloads privilege tables, making user and privilege changes take effect immediately.

### <a id="optimize-table"></a>OPTIMIZE TABLE
Reorganizes table data and indexes for better performance and space utilization.

### <a id="backup"></a>BACKUP
Creates a copy of database structure and data for recovery purposes (using mysqldump).
