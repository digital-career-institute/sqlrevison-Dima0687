# DDL Commands:
# Create a Database:
Task: Create a new database named "SchoolDB" with an appropriate character set.
# Create Tables:

Task: Design and create a table named "Students" with the following columns:
# Classrooms:

classroom_id (Primary Key)
classroom_name
teacher_id

#  Students:
student_id (Primary Key)
student_name
classroom_id (Foreign Key referencing classroom_id in the "Classrooms" table)
grade

# Modify Table Structure:

Task: Add a new column named "Gender" (VARCHAR, Maximum 10 characters) to the "Students" table.

# DML Commands:
Insert Data:

Task: Insert at least 5 records into the "Students" table.
# Update Records:

Task: Update the "Age" of students who are in the "Class" 10 to 16.
# Delete Records:

Task: Delete all records of students whose "Age" is less than 10.
# DQL Command:
# Select Data:

Task: Write a SELECT query to retrieve the "FirstName," "LastName," and "Age" of all students in the "SchoolDB."
# Aggregate Functions:

Task: Use aggregate functions to find the average age of all students.

# Primary Key and Foreign Key:
# Define Primary Key:

Task: Explain the concept of a primary key and its significance. Also, ensure that the "StudentID" column in the "Students" table is a primary key.
# Create Foreign Key:

Task: Create a new table named "Courses" with columns: "CourseID" (Integer, Primary Key) and "CourseName" (VARCHAR, Maximum 50 characters). Then, add a foreign key constraint in the "Students" table referencing the "CourseID" in the "Courses" table.


Write SQL queries to perform the following tasks:

# List all students with their respective classrooms and teacher names.

# Show the average grade for each classroom. Display the classroom name and the average grade.

# Find the names of students who belong to a classroom taught by a specific teacher (you can choose a teacher_id).

# List all classrooms along with the number of students in each classroom.

# Show the details of classrooms that have an average grade above a certain value (you can choose a specific grade).


```SQL
-- Task: Create a new database named "SchoolDB" with an appropriate character set.
-- makes not really sense as a command in pgAdmin
CREATE DATABASE schooldb;
-- should be used in mysql, but in postgresql we need \c or \connect, both aren't working within query in pgAdmin
-- USE schooldb;

CREATE TABLE teachers (
	teacher_id SERIAL PRIMARY KEY,
	teacher_name VARCHAR(50)
);

INSERT INTO teachers (teacher_name)
VALUES
  ('Mr. Johnson'),
  ('Ms. Smith'),
  ('Mrs. Williams'),
  ('Dr. Brown'),
  ('Prof. Miller');

-- classroom_id (Primary Key) classroom_name teacher_id
CREATE TABLE classrooms (
	classroom_id SERIAL PRIMARY KEY,
	classroom_name VARCHAR(50),
	teacher_id INT,
	FOREIGN KEY ( teacher_id ) REFERENCES teachers( teacher_id )
);

INSERT INTO classrooms (classroom_name, teacher_id)
VALUES
  ('Math Class', 1),
  ('English Class', 2),
  ('Science Class', 3),
  ('History Class', 4),
  ('Art Class', 5);

-- student_id (Primary Key) student_name classroom_id (Foreign Key referencing classroom_id in the "Classrooms" table) grade 
-- age is here missing so i inserted it by myself
CREATE TABLE students (
	student_id SERIAL PRIMARY KEY,
	student_firstname VARCHAR(50),
	student_lastname VARCHAR(50),
	classroom_id INT,
	FOREIGN KEY (classroom_id) REFERENCES classrooms(classroom_id),
	grade DECIMAL(5,2),
	age INT
);

-- Task: Add a new column named "Gender" (VARCHAR, Maximum 10 characters) to the "Students" table.
ALTER TABLE students
ADD COLUMN GENDER VARCHAR(10);

-- Task: Insert at least 5 records into the "Students" table.
INSERT INTO students (student_firstname, student_lastname, classroom_id, grade, age, gender)
VALUES
  ('John', 'Doe', 1, 95.75, 10, 'Male'),
  ('Jane', 'Smith', 2, 88.50, 11, 'Female'),
  ('Michael', 'Johnson', 1, 75.25, 9, 'Male'),
  ('Emily', 'Williams', 3, 92.00, 12, 'Female'),
  ('Christopher', 'Jones', 2, 85.00, 10, 'Male'),
  ('Sophia', 'Brown', 1, 92.50, 12, 'Female'),
  ('Liam', 'Miller', 2, 88.75, 11, 'Male'),
  ('Olivia', 'Davis', 3, 95.00, 10, 'Female'),
  ('Noah', 'Wilson', 1, 90.25, 12, 'Male'),
  ('Emma', 'Moore', 2, 85.50, 11, 'Female');
  
-- Task: Update the "Age" of students who are in the "Class" 10 to 16.
UPDATE students
SET age = 16
WHERE classroom_id IN ( SELECT 
				classroom_id
			FROM classrooms
			WHERE classroom_name = 'Math Class'
		  	)
AND age = 10;

-- Task: Delete all records of students whose "Age" is less than 10.
DELETE FROM students
WHERE age < 10;

-- Task: Write a SELECT query to retrieve the "FirstName," "LastName," and "Age" of all students in the "SchoolDB."
SELECT
	student_firstname AS firstname,
	student_lastname AS lastname,
	age
FROM students;

-- Task: Explain the concept of a primary key and its significance. Also, ensure that the "StudentID" column in the "Students" table is a primary key.
-- primary key is a unique identifier for a record in a database table
-- uniqueness, non-null, indexed for enhancing speed of data retrieval, relationships

-- Isn't it the same as classrooms? We have e.g. 'Math Class', 'Art Class', etc. ?
-- Task: Create a new table named "Courses" with columns: "CourseID" (Integer, Primary Key) and "CourseName" (VARCHAR, Maximum 50 characters). 
-- Then, add a foreign key constraint in the "Students" table referencing the "CourseID" in the "Courses" table.

CREATE TABLE courses (
	courseid INT PRIMARY KEY,
	coursename VARCHAR(50)
);

ALTER TABLE students
ADD COLUMN courseid INT,
ADD CONSTRAINT fk_students_course
FOREIGN KEY ( courseid ) REFERENCES courses( courseid );

-- List all students with their respective classrooms and teacher names.
SELECT 
	CONCAT(s.student_firstname, ' ', s.student_lastname ) AS student_fullname,
	s.age,
	cl.classroom_name,
	t.teacher_name
FROM students s
JOIN classrooms cl ON s.classroom_id = cl.classroom_id
JOIN teachers t ON cl.teacher_id = t.teacher_id;

-- Show the average grade for each classroom. Display the classroom name and the average grade.

SELECT
	cl.classroom_name,
	TRUNC( AVG( s.grade ) ) AS average_grade
FROM students s
JOIN classrooms cl ON s.classroom_id = cl.classroom_id
GROUP BY
	cl.classroom_name;
	
-- Find the names of students who belong to a classroom taught by a specific teacher (you can choose a teacher_id).

SELECT
	CONCAT(s.student_firstname, ' ', s.student_lastname) AS student_fullname,
	cl.classroom_name
FROM students s
JOIN classrooms cl ON s.classroom_id = cl.classroom_id
JOIN teachers t ON t.teacher_id = cl.teacher_id
WHERE ( t.teacher_id = 1 ) AND ( cl.classroom_id = s.classroom_id )
GROUP BY cl.classroom_name, student_fullname;


-- List all classrooms along with the number of students in each classroom.

SELECT
    cl.classroom_name,
    COUNT(s.student_id) AS number_of_students
FROM
    classrooms cl
LEFT JOIN students s ON cl.classroom_id = s.classroom_id
GROUP BY
    cl.classroom_name;

-- Show the details of classrooms that have an average grade above a certain value (you can choose a specific grade).

SELECT
    cl.classroom_name,
    TRUNC( AVG(s.grade), 2 ) AS average_grade
FROM
    classrooms cl
JOIN students s ON cl.classroom_id = s.classroom_id
GROUP BY
    cl.classroom_name
HAVING
    AVG(s.grade) > 90;
```
