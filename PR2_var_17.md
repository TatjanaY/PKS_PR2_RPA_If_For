# Вариант 17

**Тематика:** Анализ статусов доставки и прогнозирование риска задержек в сервисе отслеживания посылок

---

## Контекст задачи

Сервис отслеживания посылок обрабатывает тысячи отправлений ежедневно. Клиенты проверяют статусы доставки, логистические менеджеры контролируют сроки, а руководство анализирует эффективность доставки по регионам и перевозчикам.

Типичные проблемы:

* часть посылок задерживается в сортировочных центрах;
* некоторые отправления долго не меняют статус;
* отсутствует единая система прогнозирования задержек;
* требуется автоматическая классификация отправлений по уровню риска нарушения SLA.

**Цель кейса:** реализовать RPA-процесс, который анализирует статусы посылок, рассчитывает показатели своевременности доставки, прогнозирует риск задержки, формирует Excel-отчёт и выводит итоговое уведомление.

---

# Задание 1. Анализ доставки Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале необходимо оценить текущие статусы отправлений и выявить проблемные посылки.

---

## 1. Подготовка данных (Excel)

Создать файл `ParcelTracking_Q1.xlsx`, лист `ShipmentStatus_Q1`.

**Структура таблицы:**

| Поле               | Тип     | Смысл                                         |
| ------------------ | ------- | --------------------------------------------- |
| TrackingNumber     | string  | Уникальный номер отправления                  |
| CarrierCode        | string  | Код перевозчика                               |
| OriginCity         | string  | Город отправления                             |
| DestinationCity    | string  | Город назначения                              |
| DispatchDate       | date    | Дата отправки                                 |
| EstimatedDelivery  | date    | Плановая дата доставки                        |
| ActualDelivery     | date    | Фактическая дата доставки (может быть пустой) |
| CurrentStatus      | string  | Статус: InTransit, Delivered, Delayed         |
| TransitDaysPlanned | integer | Плановое количество дней доставки             |

**Методический комментарий:** данные используются для расчёта фактического времени доставки и выявления задержек.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик              | Начальное значение | Смысл                              |
| -------------------- | ------------------ | ---------------------------------- |
| total_shipments      | 0                  | Общее количество посылок           |
| delivered_shipments  | 0                  | Доставленные                       |
| delayed_shipments    | 0                  | Со статусом Delayed                |
| in_transit_shipments | 0                  | В пути                             |
| overdue_shipments    | 0                  | Просроченные по срокам             |
| high_delay_cases     | 0                  | Критические задержки               |
| totalDelayDays       | 0                  | Суммарное количество дней задержки |

---

## 3. Преобразование данных (на каждой строке)

* `TransitDaysPlanned → integer`
* `DispatchDate, EstimatedDelivery, ActualDelivery → date`
* `TrackingNumber, CarrierCode → удалить пробелы, верхний регистр`

---

## 4. Расчёты (на каждой строке)

* `planned_arrival_days = EstimatedDelivery - DispatchDate`
* `actual_arrival_days = ActualDelivery - DispatchDate`
* `delay_days = actual_arrival_days - planned_arrival_days`
* если `ActualDelivery пусто` →
  `delay_days = CurrentDate - EstimatedDelivery`

---

## 5. Логика классификации (на каждой строке)

**Простые условия:**

* если `CurrentStatus = "Delivered"` → `delivered_shipments += 1`
* если `CurrentStatus = "Delayed"` → `delayed_shipments += 1`
* если `CurrentStatus = "InTransit"` → `in_transit_shipments += 1`

**Составные условия:**

* если `delay_days > 0` → `overdue_shipments += 1`, `totalDelayDays += delay_days`
* если `delay_days > 5` → `high_delay_cases += 1`

---

## 6. Результирующий Excel

Файл `ParcelTracking_Result.xlsx`, лист `Analysis_Q1`:

| TrackingNumber | CarrierCode | DelayDays | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ доставки Q1 завершён.

Всего отправлений: {total_shipments}
Доставлено: {delivered_shipments}
В пути: {in_transit_shipments}
Со статусом Delayed: {delayed_shipments}

Просрочено: {overdue_shipments}
Критические задержки: {high_delay_cases}
Суммарные дни задержки: {totalDelayDays}

Отчёт сохранён в ParcelTracking_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ Q2 с прогнозом риска задержек

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале необходимо не только фиксировать задержки, но и прогнозировать риск нарушения сроков доставки.

---

## 1. Подготовка данных

Файл `ParcelTracking_Q2.xlsx`, лист `ShipmentStatus_Q2` (аналогичная структура).

---

## 2. Инициализация переменных

| Счётчик              | Начальное значение | Смысл |
| -------------------- | ------------------ | ----- |
| total_shipments      | 0                  |       |
| delivered_shipments  | 0                  |       |
| delayed_shipments    | 0                  |       |
| in_transit_shipments | 0                  |       |
| overdue_shipments    | 0                  |       |
| critical_cases       | 0                  |       |
| totalDelayDays       | 0                  |       |
| total_forecast_risk  | 0                  |       |

---

## 3. Цикл по строкам

На каждой итерации:

1. Привести типы данных.
2. Рассчитать:

* `planned_transit_days = EstimatedDelivery - DispatchDate`
* `elapsed_days = CurrentDate - DispatchDate`
* `delivery_progress_percent = elapsed_days / planned_transit_days * 100`
* `remaining_days = EstimatedDelivery - CurrentDate`
* `forecast_delay_index = (delivery_progress_percent - 100) * (-remaining_days / planned_transit_days)`

---

## 4. Логика прогнозного риска

* если `CurrentStatus = "Delivered"` → `risk_category = "Closed"`

* если `CurrentStatus = "InTransit"`:

  * если `forecast_delay_index > 40` →
    `risk_category = "Critical Delay"`, `critical_cases += 1`
  * если `forecast_delay_index > 20` →
    `risk_category = "High Risk"`, `delayed_shipments += 1`
  * иначе →
    `risk_category = "On Schedule"`

* если `remaining_days < 0 AND CurrentStatus != "Delivered"` →
  `overdue_shipments += 1`

После определения категории:

```
total_forecast_risk += forecast_delay_index
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — это суммарный прогнозный индекс риска по всем отправлениям за квартал.

Он формируется как накопительная сумма `forecast_delay_index` по каждой строке таблицы:

```
total_forecast_risk = Σ forecast_delay_index
```

### Интерпретация:

* низкое значение → большинство доставок укладываются в сроки;
* среднее → наблюдается рост потенциальных задержек;
* высокое → высокий системный риск нарушения SLA по портфелю отправлений.

Таким образом руководство получает агрегированную оценку логистического риска.

---

## 5. Итоговый Excel

Файл `ParcelTracking_Result.xlsx`, лист `Analysis_Q2`:

| TrackingNumber | CarrierCode | DeliveryProgressPercent | RemainingDays | ForecastDelayIndex | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ доставки Q2 завершён.

Всего отправлений: {total_shipments}
Доставлено: {delivered_shipments}
В пути: {in_transit_shipments}
Просрочено: {overdue_shipments}

Критические случаи: {critical_cases}
Суммарные дни задержки: {totalDelayDays}
Суммарный прогнозный риск по отправлениям: {total_forecast_risk}

Отчёт сформирован в ParcelTracking_Result.xlsx (лист Analysis_Q2).
"""
```
