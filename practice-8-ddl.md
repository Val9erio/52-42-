# Звіт з практичної роботи №8. Створення структури бази та її об'єктів (SQL DDL)
**Виконав:** студент групи ІПЗ-41 Камінський Денис

---

## 1. Повний DDL-скрипт схеми (`schema.sql`)

Цей скрипт повністю відтворює реляційну схему бази даних коледжу з нуля. У ньому враховано всі таблиці (`departments`, `students`, `teachers`, `subjects`, `teacher_subjects`, `lessons`, `grades`), типи даних, первинні та зовнішні ключі, а також правила каскадного видалення і обмеження цілісності (`CHECK`, `UNIQUE`, `NOT NULL`).

```sql
-- Створення схеми
CREATE SCHEMA IF NOT EXISTS college;

-- Очищення існуючих об'єктів у зворотному порядку залежностей
DROP TABLE IF EXISTS college.grades CASCADE;
DROP TABLE IF EXISTS college.lessons CASCADE;
DROP TABLE IF EXISTS college.teacher_subjects CASCADE;
DROP TABLE IF EXISTS college.subjects CASCADE;
DROP TABLE IF EXISTS college.teachers CASCADE;
DROP TABLE IF EXISTS college.students CASCADE;
DROP TABLE IF EXISTS college.academic_groups CASCADE;
DROP TABLE IF EXISTS college.departments CASCADE;

-- 1. Таблиця підрозділів / кафедр
CREATE TABLE college.departments (
    department_id SMALLINT GENERATED ALWAYS AS IDENTITY,
    code VARCHAR(10) NOT NULL,
    name VARCHAR(150) NOT NULL,
    CONSTRAINT pk_departments PRIMARY KEY (department_id),
    CONSTRAINT uq_departments_code UNIQUE (code)
);

-- 2. Таблиця академічних груп
CREATE TABLE college.academic_groups (
    group_id INTEGER GENERATED ALWAYS AS IDENTITY,
    name VARCHAR(50) NOT NULL,
    entry_year INT NOT NULL,
    CONSTRAINT pk_academic_groups PRIMARY KEY (group_id),
    CONSTRAINT uq_academic_groups_name UNIQUE (name),
    CONSTRAINT chk_entry_year CHECK (entry_year >= 2000)
);

-- 3. Таблиця студентів
CREATE TABLE college.students (
    student_id INTEGER GENERATED ALWAYS AS IDENTITY,
    group_id INT NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    CONSTRAINT pk_students PRIMARY KEY (student_id),
    CONSTRAINT uq_students_email UNIQUE (email),
    CONSTRAINT fk_students_group FOREIGN KEY (group_id) 
        REFERENCES college.academic_groups (group_id) 
        ON DELETE RESTRICT
);

-- 4. Таблиця викладачів
CREATE TABLE college.teachers (
    teacher_id INTEGER GENERATED ALWAYS AS IDENTITY,
    full_name VARCHAR(100) NOT NULL,
    position VARCHAR(50) NOT NULL DEFAULT 'Викладач',
    CONSTRAINT pk_teachers PRIMARY KEY (teacher_id)
);

-- 5. Таблиця навчальних дисциплін
CREATE TABLE college.subjects (
    subject_id INTEGER GENERATED ALWAYS AS IDENTITY,
    department_id SMALLINT NOT NULL,
    name VARCHAR(100) NOT NULL,
    credits INT NOT NULL,
    CONSTRAINT pk_subjects PRIMARY KEY (subject_id),
    CONSTRAINT uq_subjects_name UNIQUE (name),
    CONSTRAINT fk_subjects_department FOREIGN KEY (department_id) 
        REFERENCES college.departments (department_id) 
        ON DELETE RESTRICT,
    CONSTRAINT chk_credits CHECK (credits > 0)
);

-- 6. Проміжна таблиця зв'язку M:N (Навантаження: викладач — дисципліна)
CREATE TABLE college.teacher_subjects (
    id INTEGER GENERATED ALWAYS AS IDENTITY,
    teacher_id INT NOT NULL,
    subject_id INT NOT NULL,
    CONSTRAINT pk_teacher_subjects PRIMARY KEY (id),
    CONSTRAINT uq_teacher_subject UNIQUE (teacher_id, subject_id),
    CONSTRAINT fk_ts_teacher FOREIGN KEY (teacher_id) 
        REFERENCES college.teachers (teacher_id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_ts_subject FOREIGN KEY (subject_id) 
        REFERENCES college.subjects (subject_id) 
        ON DELETE CASCADE
);

-- 7. Таблиця занять
CREATE TABLE college.lessons (
    lesson_id INTEGER GENERATED ALWAYS AS IDENTITY,
    teacher_subject_id INT NOT NULL,
    group_id INT NOT NULL,
    date_time TIMESTAMP NOT NULL,
    CONSTRAINT pk_lessons PRIMARY KEY (lesson_id),
    CONSTRAINT fk_lessons_ts FOREIGN KEY (teacher_subject_id) 
        REFERENCES college.teacher_subjects (id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_lessons_group FOREIGN KEY (group_id) 
        REFERENCES college.academic_groups (group_id) 
        ON DELETE CASCADE
);

-- 8. Таблиця оцінок
CREATE TABLE college.grades (
    grade_id INTEGER GENERATED ALWAYS AS IDENTITY,
    student_id INT NOT NULL,
    lesson_id INT NOT NULL,
    value INT NOT NULL,
    CONSTRAINT pk_grades PRIMARY KEY (grade_id),
    CONSTRAINT fk_grades_student FOREIGN KEY (student_id) 
        REFERENCES college.students (student_id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_grades_lesson FOREIGN KEY (lesson_id) 
        REFERENCES college.lessons (lesson_id) 
        ON DELETE CASCADE,
    CONSTRAINT chk_grade_value CHECK (value >= 0 AND value <= 100)
);
