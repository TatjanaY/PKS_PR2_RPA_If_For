# Вариант 29

**Тематика:** Анализ производительности и прогнозирование операционного риска в системе управления складскими роботами

---

## Контекст задачи

Система управления складскими роботами координирует работу автономных устройств, выполняющих перемещение товаров, сбор заказов и транспортировку грузов внутри склада.

Типичные проблемы:

* снижение производительности роботов;
* частые простои или ошибки навигации;
* превышение времени выполнения задач;
* высокий износ оборудования;
* риск срыва выполнения заказов.

Необходимо реализовать RPA-процесс, который:

* анализирует показатели работы роботов за период;
* рассчитывает ключевые метрики эффективности;
* прогнозирует операционный риск;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ работы роботов за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `WarehouseRobots_Week1.xlsx`, лист `RobotLogs`.

### Структура таблицы:

| Поле             | Тип    | Смысл                                 |
| ---------------- | ------ | ------------------------------------- |
| RobotID          | string | Идентификатор робота                  |
| LogDate          | date   | Дата смены                            |
| TasksAssigned    | number | Количество назначенных задач          |
| TasksCompleted   | number | Количество выполненных задач          |
| AvgTaskTime      | number | Среднее время выполнения задачи (мин) |
| DowntimeMinutes  | number | Время простоя (мин)                   |
| NavigationErrors | number | Количество ошибок навигации           |
| BatteryLevel     | number | Средний уровень заряда (%)            |

---

## 2. Инициализация переменных

| Счётчик               | Начальное значение |
| --------------------- | ------------------ |
| total_logs            | 0                  |
| low_productivity_days | 0                  |
| high_downtime_days    | 0                  |
| high_error_days       | 0                  |
| total_tasks_completed | 0                  |
| total_downtime        | 0                  |

---

## 3. Преобразование данных

* TasksAssigned, TasksCompleted, AvgTaskTime, DowntimeMinutes, NavigationErrors, BatteryLevel → number
* LogDate → date
* RobotID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (на каждой строке)

```
completion_rate = TasksCompleted / TasksAssigned
efficiency_index = TasksCompleted / (AvgTaskTime + 1)
```

Если `TasksAssigned = 0` → `completion_rate = 0`.

---

## 5. Логика классификации

* если `completion_rate < 0.7` →
  `low_productivity_days += 1`

* если `DowntimeMinutes > 120` →
  `high_downtime_days += 1`

* если `NavigationErrors > 5` →
  `high_error_days += 1`

Накопление:

```
total_tasks_completed += TasksCompleted
total_downtime += DowntimeMinutes
total_logs += 1
```

---

## 6. Итоговый Excel

Файл `WarehouseRobots_Result.xlsx`, лист `Week1_Analysis`:

| RobotID | LogDate | CompletionRate | EfficiencyIndex | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ работы складских роботов завершён.

Всего записей: {total_logs}
Дней с низкой производительностью: {low_productivity_days}
Дней с высоким простоем: {high_downtime_days}
Дней с высоким числом ошибок: {high_error_days}

Всего выполнено задач: {total_tasks_completed}
Суммарный простой (мин): {total_downtime}

Отчёт сохранён в WarehouseRobots_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ с прогнозом операционного риска

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `WarehouseRobots_Month.xlsx`, лист `RobotLogs_Month`.

---

## 2. Инициализация переменных

| Переменная            | Начальное значение |
| --------------------- | ------------------ |
| total_logs            | 0                  |
| high_risk_robots      | 0                  |
| critical_risk_robots  | 0                  |
| total_tasks_completed | 0                  |
| total_downtime        | 0                  |
| total_forecast_risk   | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
completion_rate = TasksCompleted / (TasksAssigned + 1)

productivity_penalty = (1 - completion_rate) * 2000
downtime_penalty = DowntimeMinutes * 5
error_penalty = NavigationErrors * 300
battery_penalty = 
    если BatteryLevel < 30 → (30 - BatteryLevel) * 50
    иначе → 0

time_penalty = AvgTaskTime * 20
```

Итоговая формула:

```
forecast_robot_risk = productivity_penalty 
                      + downtime_penalty 
                      + error_penalty 
                      + battery_penalty 
                      + time_penalty
```

Если значение отрицательное → риск = 0.

---

## 4. Логика классификации риска

* если `forecast_robot_risk > 8000` →
  `risk_category = "Critical Risk"`
  `critical_risk_robots += 1`

* если `forecast_robot_risk > 4000` →
  `risk_category = "High Risk"`
  `high_risk_robots += 1`

* иначе →
  `risk_category = "Stable"`

После классификации:

```
total_forecast_risk += forecast_robot_risk
total_logs += 1
total_tasks_completed += TasksCompleted
total_downtime += DowntimeMinutes
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная метрика операционного риска работы складских роботов за период.

```
total_forecast_risk = Σ forecast_robot_risk
```

### Интерпретация:

* низкое значение → роботы работают стабильно и эффективно;
* среднее → выявлены отдельные узкие места (простой, ошибки);
* высокое → системный риск срыва выполнения заказов.

Метрика позволяет:

* прогнозировать перегрузки склада;
* планировать техническое обслуживание;
* оптимизировать распределение задач;
* минимизировать риск сбоев логистики.

---

## 5. Итоговый Excel

Файл `WarehouseRobots_Result.xlsx`, лист `Month_Analysis`:

| RobotID | LogDate | ForecastRobotRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ складских роботов завершён.

Всего записей: {total_logs}
Высокий риск: {high_risk_robots}
Критический риск: {critical_risk_robots}

Всего выполнено задач: {total_tasks_completed}
Суммарный простой (мин): {total_downtime}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в WarehouseRobots_Result.xlsx (лист Month_Analysis).
"""
```
