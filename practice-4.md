# Звіт з практичної роботи. Перетворення ER-моделі на реляційну схему

## 1. Таблиця відповідності: «сутність → таблиця, зв'язок → правило перетворення»

| Елемент концептуальної моделі | Елемент реляційної моделі (Таблиця) | Правило перетворення / Примітка |
| :--- | :--- | :--- |
| Сутність `АКАДЕМ_ГРУПА` | Таблиця `academic_groups` | Створення базової таблиці. |
| Сутність `СТУДЕНТ` | Таблиця `students` | Створення базової таблиці. |
| Сутність `ВИКЛАДАЧ` | Таблиця `teachers` | Створення базової таблиці. |
| Сутність `ДИСЦИПЛІНА` | Таблиця `subjects` | Створення базової таблиці. |
| Зв'язок 1:N (Група -> Студент) | Зовнішний ключ `group_id` у `students` | **Зв'язок 1:N:** Зовнішній ключ (FK) додається у таблицю на стороні «багато» (до студентів). |
| Зв'язок M:N (Викладач <-> Дисципліна) | Проміжна таблиця `teacher_subjects` | **Зв'язок M:N:** Створюється нова таблиця розв'язки. Вона містить зовнішні ключі обох батьківських таблиць. |
| Зв'язок 1:N (Навантаження -> Заняття) | Зовнішний ключ `teacher_subject_id` у `lessons`| Зовнішній ключ додається до таблиці `lessons` (Заняття). |
| Зв'язок 1:N (Заняття -> Оцінка) | Зовнішний ключ `lesson_id` у `grades` | Зовнішній ключ додається до таблиці `grades` (Оцінка). |
| Зв'язок 1:N (Студент -> Оцінка) | Зовнішний ключ `student_id` у `grades` | Зовнішній ключ додається до таблиці `grades` (Оцінка). |

---

## 2. Перелік обмежень цілісності для кожної таблиці

| Таблиця | Атрибут | Тип даних | Обмеження цілісності (Constraints) |
| :--- | :--- | :--- | :--- |
| **academic_groups** | `group_id` | SERIAL (INT) | **PRIMARY KEY** |
| | `name` | VARCHAR(50) | **NOT NULL**, **UNIQUE** (наприклад, "ІПЗ-31" не може повторюватися) |
| | `entry_year` | INT | **NOT NULL**, **CHECK** (entry_year >= 2000) |
| **students** | `student_id` | SERIAL (INT) | **PRIMARY KEY** |
| | `group_id` | INT | **FOREIGN KEY**, **NOT NULL**, **ON DELETE RESTRICT** (не можна видалити групу, якщо там є студенти) |
| | `full_name` | VARCHAR(100)| **NOT NULL** |
| | `email` | VARCHAR(100)| **UNIQUE** |
| **teachers** | `teacher_id` | SERIAL (INT) | **PRIMARY KEY** |
| | `full_name` | VARCHAR(100)| **NOT NULL** |
| | `position` | VARCHAR(50) | **DEFAULT** 'Викладач' |
| **subjects** | `subject_id` | SERIAL (INT) | **PRIMARY KEY** |
| | `name` | VARCHAR(100)| **NOT NULL**, **UNIQUE** |
| | `credits` | INT | **NOT NULL**, **CHECK** (credits > 0) |
| **teacher_subjects**| `id` | SERIAL (INT) | **PRIMARY KEY** |
| | `teacher_id` | INT | **FOREIGN KEY**, **ON DELETE CASCADE** (якщо видаляють викладача, видаляється і його навантаження) |
| | `subject_id` | INT | **FOREIGN KEY**, **ON DELETE CASCADE** |
| **lessons** | `lesson_id` | SERIAL (INT) | **PRIMARY KEY** |
| | `teacher_subject_id`| INT | **FOREIGN KEY**, **ON DELETE CASCADE** |
| | `group_id` | INT | **FOREIGN KEY**, **ON DELETE CASCADE** |
| | `date_time` | TIMESTAMP | **NOT NULL** |
| **grades** | `grade_id` | SERIAL (INT) | **PRIMARY KEY** |
| | `student_id` | INT | **FOREIGN KEY**, **ON DELETE CASCADE** |
| | `lesson_id` | INT | **FOREIGN KEY**, **ON DELETE CASCADE** |
| | `value` | INT | **NOT NULL**, **CHECK** (value >= 0 AND value <= 100) |

---

## 3. Пояснення розв'язання зв'язку M:N
У концептуальній моделі між `Викладачем` та `Дисципліною` існував зв'язок багато-до-багатьох (один викладач веде кілька предметів, один предмет ведуть кілька викладачів). 

Згідно з Першою нормальною формою (1NF) реляційних баз даних, кожна клітинка таблиці повинна містити лише одне атомарне значення. Ми не можемо записати масив ID дисциплін у рядок викладача. Тому зв'язок M:N завжди перетворюється на **окрему таблицю зв'язку** (у нашому випадку `teacher_subjects`). Ця таблиця містить свій сурогатний первинний ключ, а також зовнішні ключі, що посилаються на таблиці викладачів та дисциплін, розбиваючи один зв'язок M:N на два зв'язки 1:N.

---

## 4. Реляційна схема (Фізична модель даних)

*Схема згенерована автоматично. Вона відображає фізичні назви таблиць, типи даних та зовнішні ключі.*

```mermaid
erDiagram
    academic_groups ||--|{ students : "group_id"
    academic_groups ||--o{ lessons : "group_id"
    
    teachers ||--o{ teacher_subjects : "teacher_id"
    subjects ||--o{ teacher_subjects : "subject_id"
    
    teacher_subjects ||--o{ lessons : "teacher_subject_id"
    lessons ||--o{ grades : "lesson_id"
    students ||--o{ grades : "student_id"

    academic_groups {
        INT group_id PK
        VARCHAR name UK
        INT entry_year
    }
    students {
        INT student_id PK
        INT group_id FK
        VARCHAR full_name
        VARCHAR email UK
    }
    teachers {
        INT teacher_id PK
        VARCHAR full_name
        VARCHAR position
    }
    subjects {
        INT subject_id PK
        VARCHAR name UK
        INT credits
    }
    teacher_subjects {
        INT id PK
        INT teacher_id FK
        INT subject_id FK
    }
    lessons {
        INT lesson_id PK
        INT teacher_subject_id FK
        INT group_id FK
        TIMESTAMP date_time
    }
    grades {
        INT grade_id PK
        INT student_id FK
        INT lesson_id FK
        INT value
    }
