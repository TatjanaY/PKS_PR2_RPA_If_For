# Вариант 16

**Тематика:** Онлайн-обучение программированию — анализ учебной активности и прогноз академического риска

---

## Контекст задачи

Платформа онлайн-обучения программированию объединяет несколько образовательных треков (Frontend, Backend, Data Engineering). Студенты проходят модули, выполняют практические задания и сдают промежуточные тесты.

Академический отдел фиксирует ряд проблем:

* часть студентов замедляет темп прохождения модулей;
* растёт количество несданных практических заданий;
* к концу модуля увеличивается доля низких оценок;
* отсутствует единый механизм прогнозирования академического риска по группе.

Необходимо реализовать автоматизированный RPA-процесс, который анализирует текущую учебную активность, рассчитывает показатели прогресса, классифицирует студентов по уровню риска академического отставания и формирует итоговый отчёт в Excel с уведомлением в модальном окне.

---

# Задание 1. Анализ учебной активности за модуль M1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** завершён первый учебный модуль. Требуется оценить текущий прогресс студентов, выявить отстающих и сформировать сводную статистику для академического совета.

---

## 1. Подготовка данных (Excel)

Создать файл `Programming_Module_M1.xlsx`, лист `Module_M1`.

**Структура таблицы:**

| Поле             | Тип     | Смысл                                          |
| ---------------- | ------- | ---------------------------------------------- |
| StudentID        | string  | Уникальный идентификатор студента              |
| Track            | string  | Образовательный трек (Frontend, Backend, Data) |
| TotalLessons     | integer | Общее количество занятий в модуле              |
| CompletedLessons | integer | Количество завершённых занятий                 |
| TasksTotal       | integer | Общее количество практических заданий          |
| TasksSubmitted   | integer | Количество сданных заданий                     |
| AverageScore     | decimal | Средний балл за задания (0–100)                |
| ModuleStart      | date    | Дата начала модуля                             |
| ModuleEnd        | date    | Дата завершения модуля                         |
| EnrollmentStatus | string  | Статус (Active, Inactive, Completed)           |

Минимум 20 записей.

**Методический комментарий:** таблица отражает академическую активность и дисциплину студентов в рамках конкретного модуля.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик                 | Начальное значение | Смысл                                   |
| ----------------------- | ------------------ | --------------------------------------- |
| total_students          | 0                  | Общее количество обработанных студентов |
| active_students         | 0                  | Студенты со статусом Active             |
| inactive_students       | 0                  | Студенты со статусом Inactive           |
| completed_students      | 0                  | Студенты, завершившие модуль            |
| low_progress_students   | 0                  | Студенты с прогрессом менее 50%         |
| critical_students       | 0                  | Студенты с прогрессом менее 25%         |
| overdue_students        | 0                  | Студенты с просроченным модулем         |
| total_unsubmitted_tasks | 0                  | Суммарное количество несданных заданий  |
| total_forecast_risk     | 0                  | Суммарный прогнозный академический риск |

**Методический комментарий:**
`total_forecast_risk` суммирует баллы риска:

* Critical → +3
* High Risk → +2
* Moderate → +1
* On Track → 0

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `TotalLessons, CompletedLessons, TasksTotal, TasksSubmitted → integer`
* `AverageScore → decimal`
* `ModuleStart, ModuleEnd → date`
* `StudentID, Track, EnrollmentStatus → удалить пробелы, верхний регистр`

**Методический комментарий:** корректное приведение типов необходимо для точных расчётов процентов выполнения и анализа сроков.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

* `lesson_progress = CompletedLessons / TotalLessons * 100`
  — процент завершённых занятий

* `task_completion_rate = TasksSubmitted / TasksTotal * 100`
  — процент сданных заданий

* `unsubmitted_tasks = TasksTotal - TasksSubmitted`
  — количество несданных заданий

* `remaining_days = ModuleEnd - CurrentDate`
  — количество дней до завершения модуля

* `performance_index = (lesson_progress * 0.4) + (task_completion_rate * 0.3) + (AverageScore * 0.3)`
  — интегральный показатель академической эффективности

**Методический комментарий:** совокупность этих показателей позволяет комплексно оценить прогресс и академическую дисциплину.

---

## 5. Логика классификации студентов (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

* если `ModuleEnd < CurrentDate AND EnrollmentStatus != "Completed"` → `overdue_students += 1`

**Составные условия:**

* если `lesson_progress < 50` → `low_progress_students += 1`
* если `lesson_progress < 25` → `critical_students += 1`
* если `task_completion_rate < 60` → `total_unsubmitted_tasks += unsubmitted_tasks`

**Вложенные условия:**

* если `EnrollmentStatus = "Inactive"` → `inactive_students += 1`
* если `EnrollmentStatus = "Active"` → `active_students += 1`
* если `EnrollmentStatus = "Completed"` → `completed_students += 1`

**Формирование категории риска:**

* если `performance_index < 50` → `risk_category = "Critical"`, `total_forecast_risk += 3`
* если `performance_index >= 50 AND performance_index < 65` → `risk_category = "High Risk"`, `total_forecast_risk += 2`
* если `performance_index >= 65 AND performance_index < 80` → `risk_category = "Moderate"`, `total_forecast_risk += 1`
* иначе → `risk_category = "On Track"`

**Методический комментарий:** простые условия фиксируют просрочку, составные — низкий прогресс, вложенные — статус студента, итоговая часть — прогнозный риск.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `Programming_Module_Result.xlsx`, лист `Analysis_M1`.

| StudentID | Track | LessonProgress | TaskCompletionRate | PerformanceIndex | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ модуля M1 завершён.

Всего студентов: {total_students}
Активных: {active_students}
Неактивных: {inactive_students}
Завершили модуль: {completed_students}

Студенты с низким прогрессом: {low_progress_students}
Критические: {critical_students}
Просроченные: {overdue_students}
Несданные задания (суммарно): {total_unsubmitted_tasks}
Суммарный прогнозный академический риск: {total_forecast_risk}

Отчёт сохранён в Programming_Module_Result.xlsx (лист Analysis_M1).
"""
```

---

# Задание 2. Прогноз академического риска для модуля M2 (расширенный уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** во втором модуле увеличилась сложность материала. Необходимо спрогнозировать возможное снижение прогресса и заранее определить студентов с повышенным риском академического отставания.

---

## 1. Подготовка данных (Excel)

Файл `Programming_Module_M2.xlsx`, лист `Module_M2` с той же структурой, дополнительно:

| Поле                     | Тип     | Смысл                       |
| ------------------------ | ------- | --------------------------- |
| ForecastLessonsCompleted | integer | Прогноз завершённых занятий |
| ForecastScore            | decimal | Прогноз среднего балла      |

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

Используются те же счётчики, включая `total_forecast_risk`.

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы данных аналогично заданию 1.

2. Вычислить:

* `forecast_lesson_progress = ForecastLessonsCompleted / TotalLessons * 100`
* `forecast_performance_index = (forecast_lesson_progress * 0.4) + (task_completion_rate * 0.3) + (ForecastScore * 0.3)`
* `performance_delta = forecast_performance_index - performance_index`

**Методический комментарий:** показатель `performance_delta` отражает прогноз изменения академической эффективности.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

* если `EnrollmentStatus = "Inactive"`:

  * если `remaining_days < 0` → `risk_category = "Critical"`, `critical_students += 1`, `total_forecast_risk += 3`
  * иначе → `risk_category = "High Risk"`, `total_forecast_risk += 2`

* если `EnrollmentStatus = "Active"`:

  * если `forecast_performance_index < 50 OR performance_delta < -15`
    → `risk_category = "Critical"`, `critical_students += 1`, `total_forecast_risk += 3`

  * если `forecast_performance_index < 65`
    → `risk_category = "High Risk"`, `total_forecast_risk += 2`

  * если `forecast_performance_index < 80`
    → `risk_category = "Moderate"`, `total_forecast_risk += 1`

  * иначе → `risk_category = "On Track"`

* если `EnrollmentStatus = "Completed"` → `risk_category = "Completed"`

**Методический комментарий:** выполняется для каждой строки; простые условия фиксируют статус, составные — снижение показателей, вложенные — итоговую прогнозную категорию и накопление значения `total_forecast_risk`.

---

## 5. Формирование результирующего Excel (Excel)

Файл `Programming_Module_Result.xlsx`, лист `Analysis_M2`.

| StudentID | ForecastLessonProgress | ForecastPerformanceIndex | PerformanceDelta | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Прогноз академического риска для модуля M2 выполнен.

Всего студентов: {total_students}
Критические студенты: {critical_students}
Суммарный прогнозный риск портфеля: {total_forecast_risk}

Если значение превышает 300 — высокий риск по группе.
Если 150–300 — средний риск.
Если ниже 150 — низкий риск.

Отчёт сформирован в Programming_Module_Result.xlsx (лист Analysis_M2).
"""
```
