# Вариант 19

**Тематика:** Анализ выполнения задач и прогнозирование риска срыва сроков в системе управления проектами

---

## Контекст задачи

Система управления проектами используется для координации команд, распределения задач и контроля сроков выполнения. Руководители проектов отслеживают статус задач, загрузку сотрудников и соблюдение дедлайнов.

Типичные проблемы:

* часть задач выполняется с опозданием;
* сотрудники перегружены задачами с пересекающимися сроками;
* отсутствует агрегированная оценка риска срыва проекта;
* требуется автоматическая классификация задач по уровню риска.

**Цель кейса:** реализовать RPA-процесс, который анализирует статус задач, рассчитывает показатели выполнения, прогнозирует риск нарушения сроков, формирует Excel-отчёт и выводит итоговое уведомление.

---

# Задание 1. Анализ выполнения задач Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале необходимо оценить текущее состояние задач и выявить просроченные или проблемные элементы.

---

## 1. Подготовка данных (Excel)

Создать файл `ProjectTasks_Q1.xlsx`, лист `Tasks_Q1`.

### Структура таблицы:

| Поле           | Тип    | Смысл                                           |
| -------------- | ------ | ----------------------------------------------- |
| TaskID         | string | Уникальный идентификатор задачи                 |
| ProjectCode    | string | Код проекта                                     |
| AssigneeName   | string | Ответственный сотрудник                         |
| PriorityLevel  | string | Приоритет: Low, Medium, High, Critical          |
| StartDate      | date   | Дата начала работы                              |
| DueDate        | date   | Плановая дата завершения                        |
| CompletionDate | date   | Фактическая дата завершения (может быть пустой) |
| TaskStatus     | string | Статус: New, InProgress, Completed, Blocked     |
| EstimatedHours | number | Планируемые трудозатраты в часах                |
| LoggedHours    | number | Фактически затраченные часы                     |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                                                  |
| ------------------- | ------------------ | ------------------------------------------------------ |
| total_tasks         | 0                  | Общее количество задач                                 |
| completed_tasks     | 0                  | Количество завершённых задач                           |
| inprogress_tasks    | 0                  | Количество задач в работе                              |
| blocked_tasks       | 0                  | Количество заблокированных задач                       |
| overdue_tasks       | 0                  | Количество просроченных задач                          |
| high_priority_tasks | 0                  | Количество задач с высоким или критическим приоритетом |
| total_overdue_days  | 0                  | Суммарное количество дней просрочки                    |

---

## 3. Преобразование данных

* `EstimatedHours, LoggedHours → number` — для расчёта перерасхода времени
* `StartDate, DueDate, CompletionDate → date` — для вычисления длительности
* `TaskID, ProjectCode → удалить пробелы, верхний регистр` — стандартизация идентификаторов

---

## 4. Расчёты (на каждой строке)

* `planned_duration = DueDate - StartDate`
  *плановая продолжительность задачи*

* если `CompletionDate не пусто` →
  `actual_duration = CompletionDate - StartDate`
  иначе →
  `actual_duration = CurrentDate - StartDate`

* `delay_days = actual_duration - planned_duration`
  *отклонение от плана*

* `effort_variance = LoggedHours - EstimatedHours`
  *перерасход или экономия трудозатрат*

---

## 5. Логика классификации

### Простые условия:

* если `TaskStatus = "Completed"` → `completed_tasks += 1`
* если `TaskStatus = "InProgress"` → `inprogress_tasks += 1`
* если `TaskStatus = "Blocked"` → `blocked_tasks += 1`

### Составные условия:

* если `delay_days > 0` →
  `overdue_tasks += 1`,
  `total_overdue_days += delay_days`

* если `PriorityLevel = "High" OR PriorityLevel = "Critical"` →
  `high_priority_tasks += 1`

---

## 6. Итоговый Excel

Файл `ProjectTasks_Result.xlsx`, лист `Analysis_Q1`:

| TaskID               | ProjectCode | DelayDays                  | EffortVariance      | StatusCategory     |
| -------------------- | ----------- | -------------------------- | ------------------- | ------------------ |
| Идентификатор задачи | Код проекта | Количество дней отклонения | Разница трудозатрат | Итоговая категория |

---

## 7. Итоговое уведомление

```
f"""Анализ задач Q1 завершён.

Всего задач: {total_tasks}
Завершено: {completed_tasks}
В работе: {inprogress_tasks}
Заблокировано: {blocked_tasks}

Просрочено: {overdue_tasks}
Задач высокого приоритета: {high_priority_tasks}
Суммарные дни просрочки: {total_overdue_days}

Отчёт сохранён в ProjectTasks_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ Q2 с прогнозом риска срыва сроков

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале требуется прогнозировать вероятность срыва сроков и оценивать общий риск по портфелю проектов.

---

## 1. Подготовка данных

Файл `ProjectTasks_Q2.xlsx`, лист `Tasks_Q2` (структура аналогична Q1).

---

## 2. Инициализация переменных

| Счётчик             | Начальное значение | Смысл                                             |
| ------------------- | ------------------ | ------------------------------------------------- |
| total_tasks         | 0                  | Общее количество задач                            |
| completed_tasks     | 0                  | Завершённые задачи                                |
| inprogress_tasks    | 0                  | Задачи в работе                                   |
| blocked_tasks       | 0                  | Заблокированные задачи                            |
| overdue_tasks       | 0                  | Просроченные задачи                               |
| high_risk_tasks     | 0                  | Задачи с высоким прогнозным риском                |
| critical_risk_tasks | 0                  | Задачи с критическим риском                       |
| total_overdue_days  | 0                  | Суммарные дни просрочки                           |
| total_forecast_risk | 0                  | Суммарный прогнозный индекс риска по всем задачам |

---

## 3. Цикл по строкам

На каждой итерации:

1. Привести типы данных.
2. Рассчитать:

* `planned_duration = DueDate - StartDate`
* `elapsed_time = CurrentDate - StartDate`
* `time_utilization_percent = elapsed_time / planned_duration * 100`
* `remaining_days = DueDate - CurrentDate`
* `effort_ratio = LoggedHours / EstimatedHours`
* `forecast_schedule_risk = (time_utilization_percent - 100) * effort_ratio`

---

## 4. Логика прогнозного риска

* если `TaskStatus = "Completed"` →
  `risk_category = "Closed"`

* если `TaskStatus = "InProgress"`:

  * если `forecast_schedule_risk > 50` →
    `risk_category = "Critical Risk"`,
    `critical_risk_tasks += 1`

  * если `forecast_schedule_risk > 25` →
    `risk_category = "High Risk"`,
    `high_risk_tasks += 1`

  * иначе →
    `risk_category = "On Track"`

* если `TaskStatus = "Blocked"` →
  `risk_category = "Blocked Risk"`,
  `high_risk_tasks += 1`

После классификации:

```
total_forecast_risk += forecast_schedule_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — это агрегированная оценка вероятности срыва сроков по всему портфелю задач.

Рассчитывается как:

```
total_forecast_risk = Σ forecast_schedule_risk
```

### Интерпретация:

* низкое значение → проекты выполняются стабильно;
* среднее → существует риск задержек в отдельных задачах;
* высокое → системная перегрузка и высокая вероятность срыва сроков по нескольким проектам.

Метрика позволяет руководству оценить общий уровень управленческого риска и принять меры по перераспределению ресурсов.

---

## 5. Итоговый Excel

Файл `ProjectTasks_Result.xlsx`, лист `Analysis_Q2`:

| TaskID               | ProjectCode | TimeUtilizationPercent        | RemainingDays    | ForecastScheduleRisk    | RiskCategory    |
| -------------------- | ----------- | ----------------------------- | ---------------- | ----------------------- | --------------- |
| Идентификатор задачи | Код проекта | Процент использования времени | Дней до дедлайна | Прогнозный индекс риска | Категория риска |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ задач Q2 завершён.

Всего задач: {total_tasks}
Завершено: {completed_tasks}
В работе: {inprogress_tasks}
Просрочено: {overdue_tasks}

Высокий риск: {high_risk_tasks}
Критический риск: {critical_risk_tasks}
Суммарные дни просрочки: {total_overdue_days}
Суммарный прогнозный риск по задачам: {total_forecast_risk}

Отчёт сформирован в ProjectTasks_Result.xlsx (лист Analysis_Q2).
"""
```
