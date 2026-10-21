# Nova University – Student Management Module

A console application, written in Python, for maintaining basic student and faculty information and running simple academic checks: attendance eligibility, total and average marks, pass/fail status and letter grade.

No database, API, GUI or third-party package is used. Python 3.8 or later is enough.

## Business problem and purpose

Nova University needs a small, maintainable tool to record students and faculty and to apply its academic rules consistently. The application:

- stores each student's ID, contact details, department, attendance and marks in Python, Excel and SQL;
- stores faculty contact details and department;
- decides attendance eligibility for the examination (75% or more);
- calculates total marks, average marks, result status (pass at an average of 50 or above) and grade;
- rejects invalid input instead of crashing.

## Design approach

The requirements describe people, students, and rules that apply to students. The design follows them directly:

| Module | Responsibility |
|---|---|
| `validators.py` | Checks and cleans raw input (names, emails, phone numbers, marks, attendance). Raises `ValidationError` with a user-readable message. |
| `academic_rules.py` | Pure functions for totals, averages, grades, pass rule and attendance eligibility. Holds the business rules in one place. |
| `models.py` | Domain classes: the abstract `Person`, plus `Student` and `Faculty`. |
| `records.py` | `UniversityRecords`, an in-memory store with duplicate checks and ID lookups. |
| `main.py` | Console menu and input handling. Contains no business logic. |
| `test_student_management.py` | Unit tests for boundaries, rules, validation, records and role-based display. |

Two choices are worth noting:

- **Rules are separate from data.** `Student` asks `academic_rules` for its grade and result instead of computing them itself. A change to a grade band or the pass mark is made in one place, and the rules can be reused by any other record type, such as a report.
- **The UI is separate from the domain.** The menu code only collects input and prints output. The classes know nothing about `input()` or `print()`, so they can be tested without simulating a keyboard.

## OOP concepts used

- **Abstraction:** `Person` is an `ABC` with an abstract `role` property. A `Person` cannot be created directly.
- **Inheritance:** `Student` and `Faculty` inherit name, email and phone handling from `Person`, so that logic is not repeated.
- **Polymorphism:** `display_profile()` is defined once in `Person`. Each subclass supplies its own `role` and detail lines, so a list of mixed people is shown correctly without type checks. The student profile shows attendance; the faculty profile does not.
- **Encapsulation:** Attributes are private (`_name`, `_marks`, and so on) and exposed through read-only properties. Changes go through methods such as `update_mark()` and `update_attendance()`, which validate before saving. `marks` returns a copy, so outside code cannot bypass validation.
- **Reusable logic:** Academic rules are plain functions in `academic_rules.py`. Each rule is written once and used by `Student`.

## Business rules and assumptions

The brief leaves some details open. These are the decisions taken:

| Topic | Rule implemented |
|---|---|
| Attendance | Number from 0 to 100 (decimals allowed). 75 or more is eligible for the examination. |
| Marks | Each of Python, SQL and Excel is a number from 0 to 100. All three are required. |
| Total | Sum of the three marks. |
| Average | Total divided by three. |
| Pass | Average of 50 or more is "Pass"; otherwise "Fail". Attendance does not affect the pass result, which keeps the eligibility rule and the pass rule separate, as the brief states them. |
| Grade | A: 85 and above; B: 70 to 84.99; C: 60 to 69.99; D: 50 to 59.99; F: below 50. The brief gives no grade bands, so these are an assumption and can be changed in `academic_rules.py`. |
| Email | Must look like `name@example.com`. |
| Phone | 7 to 15 digits, optionally starting with `+`. Spaces, hyphens and brackets are removed. |
| IDs | Student and faculty IDs must be unique within their own group. |

## How the application works

When started, the application shows a menu:

1. **Add Student:** collects the ID, name, email, phone, department, attendance and the three marks. Each field is asked for again until it is valid.
2. **View Students:** shows the profile of every student.
3. **Check Attendance Eligibility:** asks for a student ID and reports whether attendance is 75% or more.
4. **View Academic Result:** shows attendance, eligibility, each mark, total, average, result and grade.
5. **Add Faculty:** collects the faculty ID, name, email, phone and department.
6. **View Faculty:** shows the profile of every faculty member.
7. **Exit**

Records are held in memory, so they are lost when the program exits.

## How to run

From the project folder:

```bash
python main.py
```

To run the tests:

```bash
python -m unittest -v
```

## Sample session

Input is shown after each prompt. The example includes an invalid attendance value (105), which is rejected and asked for again.

```
Choose an option (1-7): 1
-- Add Student --
Student ID: S001
Name: Asha Kumar
Email: asha@nova.edu
Phone: 98765-43210
Department: Computer Science
Attendance (%): 105
  Invalid input: Attendance must be between 0 and 100. Please try again.
Attendance (%): 80
Python marks: 70
SQL marks: 60
Excel marks: 80
  Student added successfully.

Choose an option (1-7): 4
Student ID (blank to cancel): S001
Academic result for Asha Kumar (S001)
Attendance: 80.0% - eligible for examination
Python    : 70.00
SQL       : 60.00
Excel     : 80.00
Total     : 210.00
Average   : 70.00
Result    : Pass
Grade     : B

Choose an option (1-7): 3
Student ID (blank to cancel): S999
  No student found with ID S999. Try again or leave blank to cancel.
Student ID (blank to cancel):
```

## Test coverage

The tests in `test_student_management.py` (28 in total) cover:

- **Attendance boundaries:** 75 is eligible, 74.9 is not; 0 and 100 are accepted; -5, 105 and non-numeric values are rejected.
- **Marks:** 0 and 100 are accepted; -1, 101 and non-numeric values are rejected; all three subjects are required.
- **Academic rules:** totals and averages, each grade band boundary (85, 70, 60, 50 and just below each), the pass boundary at 50, and a failing student.
- **Validation:** invalid email, invalid phone, phone normalisation, empty IDs.
- **Records:** several students stored together, duplicate IDs rejected, unknown IDs raise an error, faculty stored separately.
- **Role-based display:** student and faculty profiles show the correct fields.

Result of the last run: `Ran 28 tests ... OK`.

## Project structure

```
nova-university-oop/
├── README.md
├── main.py
├── models.py
├── records.py
├── academic_rules.py
├── validators.py
└── test_student_management.py
```
