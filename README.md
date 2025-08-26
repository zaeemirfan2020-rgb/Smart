#Smart Course Recommendation & Enrollment System (SCRES)
-- SQL-Only Project
-- ===============================

-- Step 1: Create Database
CREATE DATABASE scres_db;
USE scres_db;

-- Step 2: Create Tables

CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    major VARCHAR(100),
    gpa DECIMAL(3,2),
    enrollment_year INT
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    description TEXT,
    credit_hours INT,
    tags TEXT
);

CREATE TABLE prerequisites (
    id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT,
    required_course_id INT,
    FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE CASCADE,
    FOREIGN KEY (required_course_id) REFERENCES courses(course_id) ON DELETE CASCADE
);

CREATE TABLE instructors (
    instructor_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100),
    department VARCHAR(100)
);

CREATE TABLE course_instructors (
    id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT,
    instructor_id INT,
    FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE CASCADE,
    FOREIGN KEY (instructor_id) REFERENCES instructors(instructor_id) ON DELETE CASCADE
);

CREATE TABLE course_schedule (
    schedule_id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT,
    day_of_week ENUM('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'),
    time_slot VARCHAR(20),
    room_number VARCHAR(20),
    FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE CASCADE
);

CREATE TABLE student_interests (
    id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT,
    interest_tag VARCHAR(50),
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE
);

CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT,
    course_id INT,
    semester VARCHAR(20),
    status ENUM('enrolled', 'waitlisted', 'dropped') DEFAULT 'enrolled',
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE CASCADE
);

CREATE TABLE grades (
    grade_id INT PRIMARY KEY AUTO_INCREMENT,
    enrollment_id INT,
    score DECIMAL(5,2),
    grade CHAR(2),
    FOREIGN KEY (enrollment_id) REFERENCES enrollments(enrollment_id) ON DELETE CASCADE
);

-- Step 3: Insert Sample Data

INSERT INTO students (full_name, email, major, gpa, enrollment_year) VALUES
('Ali Raza', 'ali@example.com', 'Computer Science', 3.50, 2022),
('Sara Khan', 'sara@example.com', 'AI', 3.90, 2021),
('Hamza Shah', 'hamza@example.com', 'Data Science', 2.80, 2023);

INSERT INTO instructors (name, email, department) VALUES
('Dr. Ahmed', 'ahmed@uni.edu', 'CS'),
('Ms. Fatima', 'fatima@uni.edu', 'Math'),
('Mr. Bilal', 'bilal@uni.edu', 'AI');

INSERT INTO courses (name, description, credit_hours, tags) VALUES
('Introduction to AI', 'Basics of Artificial Intelligence', 3, 'AI,Logic,Machine Learning'),
('Data Structures', 'Stacks, Queues, Trees', 4, 'Algorithms,Programming'),
('Calculus I', 'Fundamentals of Calculus', 3, 'Math'),
('Machine Learning', 'Supervised and Unsupervised learning', 3, 'AI,ML,Data'),
('Database Systems', 'Relational DB, SQL, Transactions', 3, 'SQL,Database');

INSERT INTO course_instructors (course_id, instructor_id) VALUES
(1, 3), (2, 1), (3, 2), (4, 3), (5, 1);

INSERT INTO prerequisites (course_id, required_course_id) VALUES
(4, 1),
(4, 2);

INSERT INTO course_schedule (course_id, day_of_week, time_slot, room_number) VALUES
(1, 'Monday', '09:00-10:30', 'CS101'),
(2, 'Tuesday', '10:00-11:30', 'CS102'),
(3, 'Wednesday', '08:00-09:30', 'MATH201'),
(4, 'Thursday', '12:00-13:30', 'AI303'),
(5, 'Friday', '11:00-12:30', 'DB202');

INSERT INTO student_interests (student_id, interest_tag) VALUES
(1, 'AI'), (1, 'Machine Learning'), (2, 'Data'), (2, 'AI'), (3, 'Math');

INSERT INTO enrollments (student_id, course_id, semester, status) VALUES
(1, 1, 'Fall 2024', 'enrolled'),
(1, 2, 'Fall 2024', 'enrolled'),
(1, 4, 'Spring 2025', 'waitlisted'),
(2, 1, 'Fall 2024', 'enrolled'),
(2, 4, 'Spring 2025', 'enrolled'),
(3, 3, 'Fall 2024', 'enrolled');

INSERT INTO grades (enrollment_id, score, grade) VALUES
(1, 88.5, 'A'),
(2, 77.0, 'B+'),
(4, 92.0, 'A+'),
(6, 65.5, 'C');

-- Step 4: Demonstration Queries

-- 1. All students and their enrolled courses
SELECT s.full_name, c.name AS course, e.semester, e.status
FROM enrollments e
JOIN students s ON e.student_id = s.student_id
JOIN courses c ON e.course_id = c.course_id;

-- 2. All prerequisites
SELECT c1.name AS course, c2.name AS prerequisite
FROM prerequisites p
JOIN courses c1 ON p.course_id = c1.course_id
JOIN courses c2 ON p.required_course_id = c2.course_id;

-- 3. Recommended courses for a student based on interests
SELECT DISTINCT c.name
FROM courses c
JOIN student_interests si ON c.tags LIKE CONCAT('%', si.interest_tag, '%')
WHERE si.student_id = 1 AND c.course_id NOT IN (
    SELECT course_id FROM enrollments WHERE student_id = 1
);

-- 4. Student transcript
SELECT s.full_name, c.name AS course, g.score, g.grade
FROM grades g
JOIN enrollments e ON g.enrollment_id = e.enrollment_id
JOIN students s ON e.student_id = s.student_id
JOIN courses c ON e.course_id = c.course_id
WHERE s.student_id = 1;

-- 5. Check for schedule conflicts for a student
SELECT cs1.course_id AS course1, cs2.course_id AS course2, cs1.day_of_week, cs1.time_slot
FROM course_schedule cs1
JOIN course_schedule cs2 
    ON cs1.day_of_week = cs2.day_of_week
    AND cs1.time_slot = cs2.time_slot
    AND cs1.course_id < cs2.course_id
WHERE cs1.course_id IN (SELECT course_id FROM enrollments WHERE student_id = 1)
AND cs2.course_id IN (SELECT course_id FROM enrollments WHERE student_id = 1);
