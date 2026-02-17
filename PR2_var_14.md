# Вариант 14

**Тематика:** Платформа для заказов такси — анализ поездок и прогноз рисков обслуживания

---

## Контекст задачи

Таксомоторная платформа фиксирует все поездки клиентов: данные о водителях, пассажирах, маршрутах, стоимости и времени поездки.

Менеджмент сталкивается с типичными проблемами:

* отдельные поездки длительные или превышают прогнозируемую стоимость;
* часть водителей имеет высокий процент отмен или опозданий;
* необходимо выявлять потенциально проблемные поездки и формировать сводку по качеству сервиса;
* прогнозировать риск превышения стоимости и времени для улучшения работы платформы.

**Цель кейса:** реализовать RPA-процесс, который автоматически анализирует поездки, рассчитывает ключевые показатели эффективности и риска, классифицирует поездки и формирует итоговый отчёт в Excel с уведомлением в модальном окне.

---

# Задание 1. Анализ поездок за март (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Вы — аналитик службы качества сервиса. Необходимо оценить активность водителей и пассажиров, выявить поездки с отклонениями по времени и стоимости, чтобы подготовить управленческую сводку.

---

## 1. Подготовка данных (Excel)

Создать файл `Taxi_Trips_March.xlsx` с листом `Trips_March`.

**Структура таблицы:**

| Поле            | Тип     | Смысл                               |
| --------------- | ------- | ----------------------------------- |
| TripID          | string  | Уникальный идентификатор поездки    |
| DriverID        | string  | Идентификатор водителя              |
| PassengerID     | string  | Идентификатор пассажира             |
| TripDate        | date    | Дата и время поездки                |
| PlannedDuration | decimal | Плановое время поездки в минутах    |
| ActualDuration  | decimal | Фактическое время поездки в минутах |
| PlannedCost     | decimal | Прогнозируемая стоимость поездки, $ |
| ActualCost      | decimal | Фактическая стоимость поездки, $    |
| Rating          | decimal | Оценка поездки пассажиром 1–5       |
| Cancellation    | string  | Отмена поездки: YES/NO              |

**Методический комментарий:** таблица отражает реальные поездки, стоимость и оценки.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                                    |
| ------------------- | ------------------ | ---------------------------------------- |
| total_trips         | 0                  | Общее количество поездок                 |
| long_trips          | 0                  | Поездки с превышением планового времени  |
| overcost_trips      | 0                  | Поездки с превышением плановой стоимости |
| low_rating_trips    | 0                  | Поездки с оценкой < 3                    |
| cancelled_trips     | 0                  | Отменённые поездки                       |
| total_extra_minutes | 0                  | Суммарное превышение времени поездок     |
| total_extra_cost    | 0                  | Суммарное превышение стоимости           |
| total_forecast_risk | 0                  | Суммарный прогнозный риск поездок        |

**Методический комментарий:** переменная `total_forecast_risk` суммирует баллы риска каждой поездки:

* `Critical` → +3
* `High Risk` → +2
* `At Risk` → +1
* `Stable` → 0

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `plannedduration, actualduration, plannedcost, actualcost, rating → decimal`
* `tripdate → date`
* `tripid, driverid, passengerid, cancellation → удалить пробелы / верхний регистр`

**Методический комментарий:** корректное приведение типов обеспечивает точность вычислений и корректное сравнение времени, стоимости и оценок.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

```
duration_diff = actualduration - plannedduration
cost_diff = actualcost - plannedcost
```

* `duration_diff` — превышение времени поездки в минутах;
* `cost_diff` — превышение стоимости поездки в $.

---

## 5. Логика классификации поездок (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

```
if duration_diff > 0:
    long_trips += 1
    total_extra_minutes += duration_diff
```

```
if cost_diff > 0:
    overcost_trips += 1
    total_extra_cost += cost_diff
```

```
if rating < 3:
    low_rating_trips += 1
```

```
if cancellation == "YES":
    cancelled_trips += 1
```

**Составные условия:**

```
if duration_diff > 10 OR cost_diff > 10 OR rating < 3:
    risk_category = "At Risk"
else:
    risk_category = "Stable"
```

**Вложенные условия:**

```
if duration_diff > 30 OR cost_diff > 30 OR rating < 2:
    risk_category = "Critical"
    total_forecast_risk += 3
elif duration_diff > 20 OR cost_diff > 20 OR rating < 3:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif duration_diff > 10 OR cost_diff > 10:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** простые условия фиксируют отдельные признаки риска (время, стоимость, оценка, отмены), составные — выявляют совокупные отклонения, вложенные — формируют итоговую категорию и суммируют баллы риска в `total_forecast_risk`.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `Taxi_Trips_Analysis_Result.xlsx`, лист `Analysis_March` с полями:

| TripID | DriverID | PassengerID | DurationDiff | CostDiff | Rating | Cancellation | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ поездок за март завершён.

Всего поездок: {total_trips}
С превышением времени: {long_trips}
С превышением стоимости: {overcost_trips}
С низкой оценкой: {low_rating_trips}
Отменённых поездок: {cancelled_trips}
Суммарное превышение времени: {total_extra_minutes} мин.
Суммарное превышение стоимости: {total_extra_cost} $
Суммарный прогнозный риск: {total_forecast_risk} баллов

Отчёт сохранён в Taxi_Trips_Analysis_Result.xlsx (лист Analysis_March).
"""
```

---

# Задание 2. Прогнозная оценка поездок за апрель (расширенный уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором месяце необходимо оценить потенциальный риск поездок с учётом текущих данных, прогнозируемых длительности и стоимости поездок, используя цикл по строкам таблицы.

---

## 1. Подготовка данных (Excel)

Файл `Taxi_Trips_April.xlsx`, лист `Trips_April` с дополнительными полями:

| Поле             | Тип     | Смысл                            |
| ---------------- | ------- | -------------------------------- |
| TripID           | string  | Уникальный идентификатор поездки |
| DriverID         | string  | Идентификатор водителя           |
| PassengerID      | string  | Идентификатор пассажира          |
| TripDate         | date    | Дата и время поездки             |
| PlannedDuration  | decimal | Плановое время поездки           |
| ActualDuration   | decimal | Фактическое время поездки        |
| PlannedCost      | decimal | Прогнозируемая стоимость поездки |
| ActualCost       | decimal | Фактическая стоимость поездки    |
| ForecastDuration | decimal | Прогнозируемое время поездки     |
| ForecastCost     | decimal | Прогнозируемая стоимость         |
| Rating           | decimal | Оценка поездки                   |
| Cancellation     | string  | Отмена поездки: YES/NO           |
| Notes            | string  | Комментарии                      |

**Методический комментарий:** прогнозируемые значения позволяют оценить потенциальные превышения времени и стоимости.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                                    |
| ------------------- | ------------------ | ---------------------------------------- |
| total_trips         | 0                  | Общее количество поездок                 |
| long_trips          | 0                  | Поездки с превышением планового времени  |
| overcost_trips      | 0                  | Поездки с превышением плановой стоимости |
| low_rating_trips    | 0                  | Поездки с оценкой < 3                    |
| cancelled_trips     | 0                  | Отменённые поездки                       |
| total_extra_minutes | 0                  | Суммарное превышение времени поездок     |
| total_extra_cost    | 0                  | Суммарное превышение стоимости           |
| total_forecast_risk | 0                  | Суммарный прогнозный риск поездок        |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы данных:

* `plannedduration, actualduration, plannedcost, actualcost, forecastduration, forecastcost, rating → decimal`
* `tripdate → date`
* `tripid, driverid, passengerid, cancellation → удалить пробелы / верхний регистр`

2. Вычислить:

```
duration_diff = actualduration - plannedduration
cost_diff = actualcost - plannedcost
forecast_duration_diff = forecastduration - actualduration
forecast_cost_diff = forecastcost - actualcost
```

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

```
if duration_diff > 30 OR cost_diff > 30 OR rating < 2:
    risk_category = "Critical"
    total_forecast_risk += 3
elif duration_diff > 20 OR cost_diff > 20 OR rating < 3:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif duration_diff > 10 OR cost_diff > 10 OR forecast_duration_diff > 10 OR forecast_cost_diff > 10:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** выполняется для каждой строки. Простые условия фиксируют превышения времени и стоимости, составные — прогнозные отклонения, вложенные — формируют итоговую категорию и суммируют баллы в `total_forecast_risk`.

---

## 5. Формирование результирующего Excel (Excel)

Файл `Taxi_Trips_Analysis_Result.xlsx`, лист `Analysis_April` с полями:

| TripID | DriverID | PassengerID | DurationDiff | CostDiff | ForecastDurationDiff | ForecastCostDiff | Rating | Cancellation | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Прогнозный анализ поездок за апрель завершён.

Всего поездок: {total_trips}
С превышением времени: {long_trips}
С превышением стоимости: {overcost_trips}
С низкой оценкой: {low_rating_trips}
Отменённых поездок: {cancelled_trips}
Суммарное превышение времени: {total_extra_minutes} мин.
Суммарное превышение стоимости: {total_extra_cost} $
Суммарный прогнозный риск: {total_forecast_risk} баллов

Отчёт сохранён в Taxi_Trips_Analysis_Result.xlsx (лист Analysis_April).
"""
```

---
