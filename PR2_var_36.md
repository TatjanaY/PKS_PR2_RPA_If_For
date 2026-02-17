# Вариант 36

**Тематика:** Анализ выполнения личных целей и прогнозирование риска срыва в мобильном приложении для учёта целей

---

## Контекст задачи

Мобильное приложение для учёта личных целей позволяет пользователям фиксировать задачи, устанавливать сроки, отслеживать прогресс и получать напоминания. Платформа анализирует данные о выполнении целей, чтобы:

* выявлять цели с низкой активностью и риском невыполнения;
* прогнозировать вероятность срыва целей;
* автоматически классифицировать цели и пользователей по уровню риска;
* предоставлять агрегированную метрику риска (`total_forecast_risk`).

Необходимо реализовать RPA-процесс, который:

* собирает данные по пользователям и их целям;
* рассчитывает показатели прогресса и вовлечённости;
* прогнозирует риск срыва цели;
* формирует Excel-отчёт;
* выводит итоговое уведомление.

---

# Задание 1. Анализ выполнения целей за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `Goals_Week1.xlsx`, лист `UserGoals`.

### Структура таблицы:

| Поле            | Тип     | Смысл                                                 |
| --------------- | ------- | ----------------------------------------------------- |
| UserID          | string  | Идентификатор пользователя                            |
| GoalID          | string  | Уникальный идентификатор цели                         |
| GoalCategory    | string  | Категория цели (Фитнес, Карьера, Образование, Личное) |
| StartDate       | date    | Дата начала цели                                      |
| EndDate         | date    | Дата окончания цели                                   |
| ProgressPercent | number  | Текущий процент выполнения цели                       |
| TasksCompleted  | integer | Количество выполненных задач                          |
| TasksTotal      | integer | Общее количество задач                                |
| LastUpdateDate  | date    | Дата последнего обновления статуса цели               |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик               | Начальное значение | Смысл                                  |
| --------------------- | ------------------ | -------------------------------------- |
| total_goals           | 0                  | Всего целей                            |
| low_progress_goals    | 0                  | Цели с низким прогрессом (<50%)        |
| overdue_goals         | 0                  | Просроченные цели                      |
| inactive_users        | 0                  | Пользователи без обновлений            |
| total_tasks_completed | 0                  | Суммарное количество выполненных задач |
| total_tasks           | 0                  | Суммарное количество задач             |
| total_forecast_risk   | 0                  | Суммарный прогнозный риск срыва целей  |

---

## 3. Преобразование данных

* ProgressPercent → number
* TasksCompleted, TasksTotal → integer
* StartDate, EndDate, LastUpdateDate → date
* UserID, GoalID → удалить пробелы, верхний регистр

---

## 4. Расчёты (по каждой строке)

* `progress_ratio = TasksCompleted / max(TasksTotal,1) * 100`
* `days_remaining = EndDate - CurrentDate`
* если `ProgressPercent < 50` → `low_progress_goals += 1`
* если `CurrentDate > EndDate AND ProgressPercent < 100` → `overdue_goals += 1`
* если `LastUpdateDate < CurrentDate - 7` → `inactive_users += 1`
* `total_tasks_completed += TasksCompleted`
* `total_tasks += TasksTotal`
* `forecast_risk = max(0, 100 - ProgressPercent + (50 - progress_ratio)/2)`
* `total_forecast_risk += forecast_risk`

---

## 5. Классификация

* если `forecast_risk > 70` → статус `"Critical Risk"`
* если `forecast_risk > 40` → статус `"High Risk"`
* иначе → `"On Track"`

---

## 6. Итоговый Excel

Файл `Goals_Result.xlsx`, лист `Week1_Analysis`:

| UserID | GoalID | GoalCategory | ProgressPercent | TasksCompleted | TasksTotal | ForecastRisk | Status |

---

## 7. Итоговое уведомление

```
f"""Анализ целей за неделю завершён.

Всего целей: {total_goals}
Целей с низким прогрессом: {low_progress_goals}
Просроченные цели: {overdue_goals}
Неактивные пользователи: {inactive_users}
Суммарное количество выполненных задач: {total_tasks_completed} из {total_tasks}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сохранён в Goals_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска срыва

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `Goals_Month.xlsx`, лист `UserGoals_Month`.

---

## 2. Инициализация переменных

| Переменная            | Начальное значение | Смысл                                  |
| --------------------- | ------------------ | -------------------------------------- |
| total_goals           | 0                  | Всего целей                            |
| low_progress_goals    | 0                  | Цели с низким прогрессом               |
| overdue_goals         | 0                  | Просроченные цели                      |
| inactive_users        | 0                  | Пользователи без обновлений            |
| total_tasks_completed | 0                  | Суммарное количество выполненных задач |
| total_tasks           | 0                  | Суммарное количество задач             |
| high_risk_goals       | 0                  | Цели с высоким прогнозным риском       |
| critical_risk_goals   | 0                  | Цели с критическим прогнозным риском   |
| total_forecast_risk   | 0                  | Суммарный прогнозный риск срыва целей  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
progress_ratio = TasksCompleted / max(TasksTotal,1) * 100
days_remaining = EndDate - CurrentDate
progress_deficit = max(0, 100 - ProgressPercent)
task_deficit = max(0, TasksTotal - TasksCompleted)
forecast_risk = progress_deficit + task_deficit + (7 - days_remaining)
total_forecast_risk += forecast_risk
```

---

## 4. Логика классификации риска

* если `forecast_risk > 70` → `risk_category = "Critical Risk"`, `critical_risk_goals += 1`
* если `forecast_risk > 40` → `risk_category = "High Risk"`, `high_risk_goals += 1`
* иначе → `risk_category = "On Track"`

---

## 5. Итоговый Excel

Файл `Goals_Result.xlsx`, лист `Month_Analysis`:

| UserID | GoalID | GoalCategory | ProgressPercent | TasksCompleted | TasksTotal | ForecastRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ целей за месяц завершён.

Всего целей: {total_goals}
Целей с высоким риском: {high_risk_goals}
Целей с критическим риском: {critical_risk_goals}
Суммарный прогнозный риск срыва: {total_forecast_risk}

Отчёт сформирован в Goals_Result.xlsx (лист Month_Analysis).
"""
```
