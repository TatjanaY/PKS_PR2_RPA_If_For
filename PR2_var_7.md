# Вариант 7

**Тематика:** Анализ результатов и прогресса пользователей на платформе онлайн-тестирования

---

## Контекст задачи

Платформа онлайн-тестирования используется для подготовки студентов и корпоративного обучения. Каждый день создаются новые тесты, а пользователи проходят их с разной скоростью и разным уровнем успеха.

Руководство сталкивается с проблемами:

* часть тестов содержит большое количество не пройденных заданий;
* некоторые пользователи не успевают пройти тесты в срок;
* нет сводной классификации пользователей по уровню успеваемости и риску невыполнения тестов;
* требуется анализ прогресса и предупреждение о возможных проблемах с успеваемостью.

**Цель кейса:** реализовать RPA-процесс, который анализирует результаты пользователей, рассчитывает показатели успеваемости, классифицирует пользователей по уровню риска, формирует итоговый отчёт в Excel и выводит уведомление в модальном окне.

---

# Задание 1. Анализ результатов тестирования Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале на платформе было проведено несколько сотен тестов. Необходимо выявить пользователей с низкой успеваемостью, просроченные тесты и подготовить сводный отчёт по прогрессу.

---

## 1. Подготовка данных (Excel)

Создать файл `OnlineTests_Q1.xlsx` с листом `UserResults_Q1`.

**Структура таблицы:**

| Поле           | Тип     | Смысл                                           |
| -------------- | ------- | ----------------------------------------------- |
| UserID         | string  | Уникальный идентификатор пользователя           |
| TestID         | string  | Уникальный идентификатор теста                  |
| TestStatus     | string  | Статус теста: NotStarted, InProgress, Completed |
| TotalQuestions | integer | Общее количество вопросов в тесте               |
| CorrectAnswers | integer | Количество правильных ответов                   |
| StartDate      | date    | Дата начала теста                               |
| EndDate        | date    | Плановая дата окончания теста                   |
| Course         | string  | Название курса                                  |

**Методический комментарий:** таблица отражает исходные данные для анализа прогресса пользователей. Данные генерируются через DeepSeek для разнообразия курсов и дат тестирования.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик               | Начальное значение | Смысл                                                   |
| --------------------- | ------------------ | ------------------------------------------------------- |
| total_tests           | 0                  | Общее количество тестов                                 |
| notstarted_tests      | 0                  | Количество тестов со статусом NotStarted                |
| inprogress_tests      | 0                  | Количество тестов со статусом InProgress                |
| completed_tests       | 0                  | Количество завершённых тестов                           |
| low_performance_users | 0                  | Пользователи с низкой успеваемостью                     |
| overdue_tests         | 0                  | Тесты, не завершённые в срок                            |
| risk_users            | 0                  | Пользователи с высоким риском невыполнения тестов       |
| critical_users        | 0                  | Пользователи с критическим уровнем невыполненных тестов |
| totalIncorrectAnswers | 0                  | Суммарное количество неправильных ответов               |

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `TotalQuestions, CorrectAnswers → integer`
* `StartDate, EndDate → date`
* `UserID, TestID, Course → удалить пробелы, верхний регистр`

**Методический комментарий:** корректные типы данных обеспечивают точность вычислений процента правильных ответов и сравнение дат.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

* `incorrect_answers = TotalQuestions - CorrectAnswers`
  *количество неправильных ответов*

* `performance_percent = CorrectAnswers / TotalQuestions * 100`
  *процент правильных ответов*

* `remaining_days = EndDate - CurrentDate`
  *количество дней до окончания теста; отрицательное значение → просрочка*

**Методический комментарий:** эти расчёты позволяют определить уровень успеваемости пользователя и срок выполнения теста.

---

## 5. Логика классификации пользователей и тестов (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

* если `EndDate < CurrentDate AND TestStatus != "Completed"` → `overdue_tests += 1`

**Составные условия:**

* если `performance_percent < 50` → `low_performance_users += 1`, `totalIncorrectAnswers += incorrect_answers`
* если `performance_percent < 30 OR EndDate < CurrentDate` → `risk_users += 1`

**Вложенные условия:**

* если `TestStatus = "NotStarted"` → `notstarted_tests += 1`
* если `TestStatus = "InProgress"` → `inprogress_tests += 1`
* если `TestStatus = "Completed"` → `completed_tests += 1`
* если `performance_percent < 20` → `critical_users += 1`

**Методический комментарий:** простые условия фиксируют факты просрочки, составные — выявляют пользователей с низкой успеваемостью, вложенные — итоговую категорию пользователей.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `OnlineTests_Result.xlsx`, лист `Analysis_Q1` с полями:

| UserID | TestID | PerformancePercent | RemainingDays | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ тестирования Q1 завершён.

Всего тестов: {total_tests}
Не начато: {notstarted_tests}
В процессе: {inprogress_tests}
Завершено: {completed_tests}

Пользователи с низкой успеваемостью: {low_performance_users}
Критические: {critical_users}
Просроченные тесты: {overdue_tests}
Рисковые: {risk_users}
Суммарные неправильные ответы: {totalIncorrectAnswers}

Отчёт сохранён в OnlineTests_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ тестирования Q2 с прогнозной оценкой риска

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале число пользователей и тестов увеличилось. Необходимо прогнозировать риск невыполнения тестов и низкой успеваемости, используя цикл по строкам таблицы и прогнозные расчёты.

---

## 1. Подготовка данных (Excel)

Файл `OnlineTests_Q2.xlsx`, лист `UserResults_Q2` с той же структурой, что и Q1. Данные генерируются через DeepSeek.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик               | Начальное значение | Смысл                                               |
| --------------------- | ------------------ | --------------------------------------------------- |
| total_tests           | 0                  | Общее количество тестов                             |
| notstarted_tests      | 0                  | Тесты со статусом NotStarted                        |
| inprogress_tests      | 0                  | Тесты со статусом InProgress                        |
| completed_tests       | 0                  | Тесты со статусом Completed                         |
| low_performance_users | 0                  | Пользователи с низкой успеваемостью                 |
| overdue_tests         | 0                  | Тесты с просроченной датой                          |
| risk_users            | 0                  | Пользователи с высоким прогнозным риском            |
| critical_users        | 0                  | Пользователи с критическим прогнозным риском        |
| totalIncorrectAnswers | 0                  | Суммарное количество неправильных ответов           |
| totalForecastRisk     | 0                  | Суммарный прогнозный риск по портфелю пользователей |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы и формат данных:

* `TotalQuestions, CorrectAnswers → integer`
* `StartDate, EndDate → date`
* `UserID, TestID, Course → удалить пробелы, верхний регистр`

2. Вычислить:

* `incorrect_answers = TotalQuestions - CorrectAnswers`
* `performance_percent = CorrectAnswers / TotalQuestions * 100`
* `remaining_days = EndDate - CurrentDate`
* `total_duration_days = EndDate - StartDate`
* `forecast_performance_drop = (50 - performance_percent) * (remaining_days / total_duration_days)`

**Методический комментарий:** прогнозное снижение успеваемости отражает потенциальный риск невыполнения теста или снижения результата.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

* если `TestStatus = "NotStarted"`:

  * если `remaining_days < 0` → `risk_category = "Critical"`, `critical_users += 1`
  * иначе → `risk_category = "At Risk"`, `risk_users += 1`
* если `TestStatus = "InProgress"`:

  * если `forecast_performance_drop > 30` → `risk_category = "High Risk"`, `critical_users += 1`
  * если `forecast_performance_drop > 15` → `risk_category = "Moderate Risk"`, `risk_users += 1`
  * иначе → `risk_category = "On Track"`
* если `TestStatus = "Completed"` → `risk_category = "Completed"`

**Методический комментарий:** выполняется для каждой строки, чтобы классифицировать пользователей по финансовым и временным рискам.

---

## 5. Формирование результирующего Excel (Excel)

Файл `OnlineTests_Result.xlsx`, лист `Analysis_Q2` с полями:

| UserID | TestID | PerformancePercent | RemainingDays | ForecastPerformanceDrop | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Расширенный анализ тестирования Q2 завершён.

Всего тестов: {total_tests}
Не начато: {notstarted_tests}
В процессе: {inprogress_tests}
Завершено: {completed_tests}

Критических пользователей: {critical_users}
Рисковых пользователей: {risk_users}
Просроченные тесты: {overdue_tests}
Суммарные неправильные ответы: {totalIncorrectAnswers}
Суммарный прогнозный риск портфеля: {totalForecastRisk}%

Отчёт сформирован в OnlineTests_Result.xlsx (лист Analysis_Q2).
"""
```

---
