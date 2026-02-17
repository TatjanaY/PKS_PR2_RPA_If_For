# Вариант 26

**Тематика:** Анализ планов путешествий и прогнозирование риска отмен или изменений в сервисе по планированию путешествий

---

## Контекст задачи

Сервис по планированию путешествий помогает пользователям составлять маршруты, бронировать транспорт и отели, сохранять маршруты и управлять деталями поездки.

Типичные ситуации для анализа:

* часть поездок отменяется или переносится;
* пользователи часто вносят изменения в план;
* поездки с большим количеством изменений могут указывать на неустойчивые планы;
* отсутствие прогнозирования рисков ведёт к неудовлетворённости клиентов и отказу от использования сервиса.

Необходимо реализовать RPA-процесс, который:

* анализирует данные планирования поездок за период;
* рассчитывает показатели стабильности планов;
* прогнозирует риск отмен или изменений маршрута;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ планов путешествий за месяц (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `TripPlanning_Month1.xlsx`, лист `TripPlans`.

**Структура таблицы:**

| Поле            | Тип     | Смысл                                |
| --------------- | ------- | ------------------------------------ |
| TripID          | string  | Уникальный идентификатор путешествия |
| UserID          | string  | Идентификатор пользователя           |
| DestinationCity | string  | Город назначения                     |
| PlanCreatedDate | date    | Дата создания плана                  |
| StartDate       | date    | Дата начала путешествия              |
| EndDate         | date    | Дата окончания путешествия           |
| ChangesCount    | integer | Количество изменений плана           |
| BookingStatus   | string  | Planned, Confirmed, Cancelled        |
| TotalCost       | number  | Общая стоимость тура                 |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик              | Начальное значение |
| -------------------- | ------------------ |
| total_trips          | 0                  |
| confirmed_trips      | 0                  |
| cancelled_trips      | 0                  |
| high_change_trips    | 0                  |
| total_changes        | 0                  |
| total_potential_loss | 0                  |

---

## 3. Преобразование данных

* ChangesCount → integer
* TotalCost → number
* PlanCreatedDate, StartDate, EndDate → date
* TripID, UserID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (на каждой строке)

```
trip_duration = EndDate - StartDate
days_before_start = StartDate - PlanCreatedDate
```

Если `BookingStatus = "Cancelled"` →

```
potential_loss = TotalCost
```

иначе →

```
potential_loss = 0
```

---

## 5. Логика классификации

* если `BookingStatus = "Confirmed"` →
  `confirmed_trips += 1`

* если `BookingStatus = "Cancelled"` →
  `cancelled_trips += 1`

* если `ChangesCount > 3` →
  `high_change_trips += 1`

Накопление:

```
total_trips += 1
total_changes += ChangesCount
total_potential_loss += potential_loss
```

---

## 6. Итоговый Excel

Файл `TripPlanning_Result.xlsx`, лист `Month1_Analysis`:

| TripID | DestinationCity | TripDuration | ChangesCount | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ планов путешествий завершён.

Всего планов: {total_trips}
Подтверждено: {confirmed_trips}
Отменено: {cancelled_trips}
Поездок с большим количеством изменений: {high_change_trips}

Суммарные изменения планов: {total_changes}
Потенциальные потери: {total_potential_loss}

Отчёт сохранён в TripPlanning_Result.xlsx (лист Month1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ с прогнозом риска отмен и изменений

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `TripPlanning_Month2.xlsx`, лист `TripPlans_Month2`.

---

## 2. Инициализация переменных

| Переменная           | Начальное значение |
| -------------------- | ------------------ |
| total_trips          | 0                  |
| confirmed_trips      | 0                  |
| cancelled_trips      | 0                  |
| high_risk_trips      | 0                  |
| critical_risk_trips  | 0                  |
| total_changes        | 0                  |
| total_potential_loss | 0                  |
| total_forecast_risk  | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой поездки:

```
trip_duration = EndDate - StartDate
days_before_start = StartDate - PlanCreatedDate

change_factor = ChangesCount * 150
cost_factor = TotalCost * 0.5

urgency_factor = 
    если days_before_start < 7 → 300
    иначе → 100

duration_factor = trip_duration * 10

forecast_plan_risk = change_factor + cost_factor + urgency_factor + duration_factor
```

---

## 4. Логика классификации риска

* если `BookingStatus = "Cancelled"` →
  `risk_category = "Cancelled"`
  `cancelled_trips += 1`

* если `BookingStatus = "Planned"`:

  * если `forecast_plan_risk > 4000` →
    `risk_category = "Critical Risk"`
    `critical_risk_trips += 1`

  * если `forecast_plan_risk > 2000` →
    `risk_category = "High Risk"`
    `high_risk_trips += 1`

  * иначе →
    `risk_category = "Stable"`

* если `BookingStatus = "Confirmed"` →
  `risk_category = "Confirmed"`

Накопление:

```
total_trips += 1
total_changes += ChangesCount
total_potential_loss += potential_loss
total_forecast_risk += forecast_plan_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная метрика прогнозного риска отмен и изменений планов путешествий за период.

```
total_forecast_risk = Σ forecast_plan_risk
```

### Интерпретация:

* низкое значение → большинство путешествий проходят без значительных изменений;
* среднее → растущая доля нестабильных планов;
* высокое → высокий коммерческий и операционный риск сервиса.

Метрика позволяет:

* прогнозировать пики изменений;
* корректировать напоминания пользователям;
* оптимизировать предложения по страховкам и гибким тарифам;
* анализировать предпочтения путешественников.

---

## 5. Итоговый Excel

Файл `TripPlanning_Result.xlsx`, лист `Month2_Analysis`:

| TripID | DestinationCity | ChangesCount | ForecastPlanRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ планов путешествий завершён.

Всего планов: {total_trips}
Подтверждено: {confirmed_trips}
Отменено: {cancelled_trips}
Высокий риск: {high_risk_trips}
Критический риск: {critical_risk_trips}

Суммарные изменения планов: {total_changes}
Потенциальные потери: {total_potential_loss}
Суммарный прогнозный риск по планам: {total_forecast_risk}

Отчёт сформирован в TripPlanning_Result.xlsx (лист Month2_Analysis).
"""
```
