# Звіт з практичної роботи №7. Визначення схем у SQL/DDL
**Виконав:** студент групи ІПЗ-41 Камінський Денис

---

## 1. SQL-скрипт відтворення схеми (`schema.sql`)

Скрипт написаний для СКБД **PostgreSQL**. Він створює схему `college` (якщо її ще немає) та послідовно створює батьківські (`departments`, `students`) і дочірні (`courses`, `enrollments`) таблиці з усіма необхідними обмеженнями цілісності.

```sql
-- 1. Підготовка схеми
CREATE SCHEMA IF NOT EXISTS college;

-- Очищення таблиць у зворотному порядку залежностей (для повторного запуску з нуля)
DROP TABLE IF EXISTS college.enrollments;
DROP TABLE IF EXISTS college.courses;
DROP TABLE IF EXISTS college.students;
DROP TABLE IF EXISTS college.departments;

-- 2. Таблиця підрозділів / кафедр (Батьківська)
CREATE TABLE college.departments (
    department_id SMALLINT GENERATED ALWAYS AS IDENTITY,
    code VARCHAR(10) NOT NULL,
    name VARCHAR(150) NOT NULL,
    CONSTRAINT pk_departments PRIMARY KEY (department_id),
    CONSTRAINT uq_departments_code UNIQUE (code)
);

-- 3. Таблиця студентів (Батьківська, необхідна для зв'язку в enrollments)
CREATE TABLE college.students (
    student_id INTEGER GENERATED ALWAYS AS IDENTITY,
    full_name VARCHAR(100) NOT NULL,
    group_code VARCHAR(15) NOT NULL,
    CONSTRAINT pk_students PRIMARY KEY (student_id)
);

-- 4. Таблиця курсів (Дочірня до departments)
CREATE TABLE college.courses (
    course_id INTEGER GENERATED ALWAYS AS IDENTITY,
    department_id SMALLINT NOT NULL,
    title VARCHAR(120) NOT NULL,
    credits SMALLINT NOT NULL,
    tuition_fee NUMERIC(10, 2) NOT NULL DEFAULT 0.00,
    CONSTRAINT pk_courses PRIMARY KEY (course_id),
    CONSTRAINT fk_courses_department FOREIGN KEY (department_id) 
        REFERENCES college.departments (department_id) 
        ON DELETE RESTRICT,
    CONSTRAINT chk_courses_credits CHECK (credits BETWEEN 1 AND 30),
    CONSTRAINT chk_courses_fee CHECK (tuition_fee >= 0)
);

-- 5. Таблиця зв'язку (Запис студентів на курси — M:N)
CREATE TABLE college.enrollments (
    student_id INTEGER NOT NULL,
    course_id INTEGER NOT NULL,
    enrolled_at DATE NOT NULL DEFAULT CURRENT_DATE,
    grade SMALLINT,
    CONSTRAINT pk_enrollments PRIMARY KEY (student_id, course_id),
    CONSTRAINT fk_enrollments_student FOREIGN KEY (student_id) 
        REFERENCES college.students (student_id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_enrollments_course FOREIGN KEY (course_id) 
        REFERENCES college.courses (course_id) 
        ON DELETE RESTRICT,
    CONSTRAINT chk_enrollments_grade CHECK (grade BETWEEN 0 AND 100)
);
