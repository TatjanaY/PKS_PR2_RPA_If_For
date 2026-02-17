# Вариант 1

**Тематика:** Анализ прогресса и активности студентов на веб-платформе онлайн-курсов

---

## Контекст задачи

Веб-платформа онлайн-курсов обслуживает сотни студентов и десятки курсов одновременно. Каждый студент может проходить несколько курсов параллельно, сдавать домашние задания и тесты, а преподаватели отслеживают успеваемость и вовлечённость.

Руководство сталкивается с типичными проблемами:

* часть студентов не приступает к занятиям вовремя;
* некоторые курсы показывают высокий процент невыполненных заданий;
* нет единого анализа вовлечённости и успеваемости по курсам;
* требуется автоматизированный контроль и классификация студентов по уровню прогресса.

**Цель кейса:** реализовать RPA-процесс, который собирает данные о студентах и их прогрессе, рассчитывает показатели вовлечённости, классифицирует студентов по уровню риска отставания, формирует итоговый отчёт в Excel и выводит уведомление в модальном окне.

---

# Задание 1. Анализ активности студентов Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале необходимо оценить активность студентов на платформе, выявить студентов с низкой вовлечённостью и подготовить сводный отчёт по прогрессу и вовлечённости.

---

## 1. Подготовка данных (Excel)

Создать файл `OnlineCourses_Q1.xlsx` с листом `StudentProgress_Q1`.

**Структура таблицы:**

| Поле                 | Тип     | Смысл                                           |
| -------------------- | ------- | ----------------------------------------------- |
| StudentID            | string  | Уникальный идентификатор студента               |
| CourseID             | string  | Уникальный идентификатор курса                  |
| EnrollmentStatus     | string  | Статус регистрации: Active, Inactive, Completed |
| TotalModules         | integer | Общее количество модулей курса                  |
| CompletedModules     | integer | Количество пройденных модулей                   |
| AssignmentsSubmitted | integer | Количество сданных заданий                      |
| StartDate            | date    | Дата начала курса                               |
| EndDate              | date    | Дата завершения курса                           |
| CourseName           | string  | Название курса                                  |

**Методический комментарий:** таблица отражает исходные данные для анализа активности студентов. Данные генерируются через DeepSeek для разнообразия курсов и дат.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик                | Начальное значение | Смысл                                     |
| ---------------------- | ------------------ | ----------------------------------------- |
| total_students         | 0                  | Общее количество студентов                |
| inactive_students      | 0                  | Студенты со статусом Inactive             |
| active_students        | 0                  | Студенты со статусом Active               |
| completed_students     | 0                  | Студенты, завершившие курс                |
| low_progress_students  | 0                  | Студенты с низким прогрессом              |
| overdue_courses        | 0                  | Курсы, просроченные по срокам             |
| risk_students          | 0                  | Студенты с высоким риском отставания      |
| critical_students      | 0                  | Студенты с критическим уровнем отставания |
| totalIncompleteModules | 0                  | Суммарное количество непройденных модулей |

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `TotalModules, CompletedModules, AssignmentsSubmitted → integer`
* `StartDate, EndDate → date`
* `StudentID, CourseID, CourseName → удалить пробелы, верхний регистр`

**Методический комментарий:** корректные типы данных обеспечивают точность вычислений процента прохождения курса и сравнения дат.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

* `incomplete_modules = TotalModules - CompletedModules`
  *количество непройденных модулей*

* `progress_percent = CompletedModules / TotalModules * 100`
  *процент прохождения курса*

* `remaining_days = EndDate - CurrentDate`
  *количество дней до завершения курса; отрицательное значение → просрочка*

**Методический комментарий:** эти расчёты позволяют определить уровень прогресса студента и срок выполнения курса.

---

## 5. Логика классификации студентов и курсов (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

* если `EndDate < CurrentDate AND EnrollmentStatus != "Completed"` → `overdue_courses += 1`

**Составные условия:**

* если `progress_percent < 50` → `low_progress_students += 1`, `totalIncompleteModules += incomplete_modules`
* если `progress_percent < 30 OR EndDate < CurrentDate` → `risk_students += 1`

**Вложенные условия:**

* если `EnrollmentStatus = "Inactive"` → `inactive_students += 1`
* если `EnrollmentStatus = "Active"` → `active_students += 1`
* если `EnrollmentStatus = "Completed"` → `completed_students += 1`
* если `progress_percent < 20` → `critical_students += 1`

**Методический комментарий:** простые условия фиксируют факты просрочки, составные — выявляют студентов с низким прогрессом, вложенные — итоговую категорию студентов.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `OnlineCourses_Result.xlsx`, лист `Analysis_Q1` с полями:

| StudentID | CourseID | ProgressPercent | RemainingDays | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ активности студентов Q1 завершён.

Всего студентов: {total_students}
Активных: {active_students}
Неактивных: {inactive_students}
Завершили курс: {completed_students}

Студенты с низким прогрессом: {low_progress_students}
Критические: {critical_students}
Просроченные курсы: {overdue_courses}
Рисковые студенты: {risk_students}
Суммарные непройденные модули: {totalIncompleteModules}

Отчёт сохранён в OnlineCourses_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ активности Q2 с прогнозной оценкой риска

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале количество студентов и курсов увеличилось. Необходимо прогнозировать риск отставания по каждому студенту, используя цикл по строкам таблицы и прогнозные расчёты.

---

## 1. Подготовка данных (Excel)

Файл `OnlineCourses_Q2.xlsx`, лист `StudentProgress_Q2` с той же структурой, что и Q1. Данные генерируются через DeepSeek.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик                | Начальное значение | Смысл                                           |
| ---------------------- | ------------------ | ----------------------------------------------- |
| total_students         | 0                  | Общее количество студентов                      |
| inactive_students      | 0                  | Студенты со статусом Inactive                   |
| active_students        | 0                  | Студенты со статусом Active                     |
| completed_students     | 0                  | Студенты, завершившие курс                      |
| low_progress_students  | 0                  | Студенты с низким прогрессом                    |
| overdue_courses        | 0                  | Просроченные курсы                              |
| risk_students          | 0                  | Студенты с высоким прогнозным риском            |
| critical_students      | 0                  | Студенты с критическим прогнозным риском        |
| totalIncompleteModules | 0                  | Суммарное количество непройденных модулей       |
| totalForecastRisk      | 0                  | Суммарный прогнозный риск по портфелю студентов |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы и формат данных:

* `TotalModules, CompletedModules, AssignmentsSubmitted → integer`
* `StartDate, EndDate → date`
* `StudentID, CourseID, CourseName → удалить пробелы, верхний регистр`

2. Вычислить:

* `incomplete_modules = TotalModules - CompletedModules`
* `progress_percent = CompletedModules / TotalModules * 100`
* `remaining_days = EndDate - CurrentDate`
* `total_duration_days = EndDate - StartDate`
* `forecast_progress_drop = (50 - progress_percent) * (remaining_days / total_duration_days)`

**Методический комментарий:** прогнозное снижение прогресса отражает потенциальный риск невыполнения курса или отставания студента.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

* если `EnrollmentStatus = "Inactive"`:

  * если `remaining_days < 0` → `risk_category = "Critical"`, `critical_students += 1`
  * иначе → `risk_category = "At Risk"`, `risk_students += 1`
* если `EnrollmentStatus = "Active"`:

  * если `forecast_progress_drop > 30` → `risk_category = "High Risk"`, `critical_students += 1`
  * если `forecast_progress_drop > 15` → `risk_category = "Moderate Risk"`, `risk_students += 1`
  * иначе → `risk_category = "On Track"`
* если `EnrollmentStatus = "Completed"` → `risk_category = "Completed"`

**Методический комментарий:** выполняется для каждой строки, чтобы классифицировать студентов по прогнозному риску отставания.

---

## 5. Формирование результирующего Excel (Excel)

Файл `OnlineCourses_Result.xlsx`, лист `Analysis_Q2` с полями:

| StudentID | CourseID | ProgressPercent | RemainingDays | ForecastProgressDrop | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Расширенный анализ активности студентов Q2 завершён.

Всего студентов: {total_students}
Активных: {active_students}
Неактивных: {inactive_students}
Завершили курс: {completed_students}

Критические студенты: {critical_students}
Рисковые студенты: {risk_students}
Просроченные курсы: {overdue_courses}
Суммарные непройденные модули: {totalIncompleteModules}
Суммарный прогнозный риск портфеля: {totalForecastRisk}%

Отчёт сформирован в OnlineCourses_Result.xlsx (лист Analysis_Q2).
"""
```

---
