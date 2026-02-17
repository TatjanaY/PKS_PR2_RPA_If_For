# Вариант 18

**Тематика:** Анализ бронирований и прогнозирование риска отмен на платформе бронирования гостиниц

---

## Контекст задачи

Платформа бронирования гостиниц объединяет тысячи отелей и десятки тысяч пользователей. Клиенты оформляют бронирования, изменяют даты проживания или отменяют поездки. Отели отслеживают загрузку номеров, а руководство платформы анализирует риски отмен и недополученной выручки.

Типичные проблемы:

* часть бронирований отменяется в последний момент;
* некоторые отели показывают высокий процент незаездов (no-show);
* отсутствует единый механизм прогнозирования риска отмен;
* требуется автоматическая классификация бронирований по уровню риска.

**Цель кейса:** реализовать RPA-процесс, который анализирует бронирования, рассчитывает показатели загрузки и отмен, прогнозирует риск отмены, формирует Excel-отчёт и выводит итоговое уведомление.

---

# Задание 1. Анализ бронирований Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале необходимо оценить структуру бронирований и выявить проблемные случаи отмен.

---

## 1. Подготовка данных (Excel)

Создать файл `HotelBookings_Q1.xlsx`, лист `Bookings_Q1`.

**Структура таблицы:**

| Поле          | Тип     | Смысл                                   |
| ------------- | ------- | --------------------------------------- |
| BookingID     | string  | Уникальный номер бронирования           |
| HotelCode     | string  | Код отеля                               |
| CityName      | string  | Город                                   |
| RoomType      | string  | Тип номера                              |
| CheckInDate   | date    | Дата заезда                             |
| CheckOutDate  | date    | Дата выезда                             |
| BookingDate   | date    | Дата оформления                         |
| BookingStatus | string  | Confirmed, Cancelled, Completed, NoShow |
| TotalPrice    | number  | Общая стоимость проживания              |
| NightsBooked  | integer | Количество ночей                        |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл |
| ------------------- | ------------------ | ----- |
| total_bookings      | 0                  |       |
| confirmed_bookings  | 0                  |       |
| cancelled_bookings  | 0                  |       |
| completed_stays     | 0                  |       |
| no_show_cases       | 0                  |       |
| short_notice_cancel | 0                  |       |
| total_revenue_loss  | 0                  |       |

---

## 3. Преобразование данных

* `TotalPrice → number`
* `NightsBooked → integer`
* `CheckInDate, CheckOutDate, BookingDate → date`
* `BookingID, HotelCode → удалить пробелы, верхний регистр`

---

## 4. Расчёты (на каждой строке)

* `days_before_checkin = CheckInDate - BookingDate`
* `stay_duration = CheckOutDate - CheckInDate`
* если `BookingStatus = "Cancelled"` →
  `potential_loss = TotalPrice`
* иначе →
  `potential_loss = 0`

---

## 5. Логика классификации

**Простые условия:**

* если `BookingStatus = "Confirmed"` → `confirmed_bookings += 1`
* если `BookingStatus = "Cancelled"` → `cancelled_bookings += 1`
* если `BookingStatus = "Completed"` → `completed_stays += 1`
* если `BookingStatus = "NoShow"` → `no_show_cases += 1`

**Составные условия:**

* если `BookingStatus = "Cancelled" AND days_before_checkin <= 2` →
  `short_notice_cancel += 1`
* если `BookingStatus = "Cancelled"` →
  `total_revenue_loss += potential_loss`

---

## 6. Итоговый Excel

Файл `HotelBookings_Result.xlsx`, лист `Analysis_Q1`:

| BookingID | HotelCode | DaysBeforeCheckIn | PotentialLoss | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ бронирований Q1 завершён.

Всего бронирований: {total_bookings}
Подтверждено: {confirmed_bookings}
Отменено: {cancelled_bookings}
Завершённых проживаний: {completed_stays}
No-show случаев: {no_show_cases}

Отмены в последний момент: {short_notice_cancel}
Потенциальные потери выручки: {total_revenue_loss}

Отчёт сохранён в HotelBookings_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ Q2 с прогнозом риска отмен

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале требуется прогнозировать риск отмены и оценивать суммарный риск по портфелю бронирований.

---

## 1. Подготовка данных

Файл `HotelBookings_Q2.xlsx`, лист `Bookings_Q2` (аналогичная структура).

---

## 2. Инициализация переменных

| Счётчик             | Начальное значение | Смысл |
| ------------------- | ------------------ | ----- |
| total_bookings      | 0                  |       |
| confirmed_bookings  | 0                  |       |
| cancelled_bookings  | 0                  |       |
| completed_stays     | 0                  |       |
| no_show_cases       | 0                  |       |
| short_notice_cancel | 0                  |       |
| high_risk_bookings  | 0                  |       |
| critical_risk_cases | 0                  |       |
| total_revenue_loss  | 0                  |       |
| total_forecast_risk | 0                  |       |

---

## 3. Цикл по строкам

На каждой итерации:

1. Привести типы данных.
2. Рассчитать:

* `days_before_checkin = CheckInDate - BookingDate`
* `stay_length = CheckOutDate - CheckInDate`
* `booking_value_per_night = TotalPrice / NightsBooked`
* `cancellation_pressure_index = (3 - days_before_checkin) * booking_value_per_night`
* `season_factor = stay_length * 2`
* `forecast_cancel_risk = cancellation_pressure_index + season_factor`

---

## 4. Логика прогнозного риска

* если `BookingStatus = "Cancelled"` →
  `risk_category = "Cancelled"`,
  `cancelled_bookings += 1`

* если `BookingStatus = "Confirmed"`:

  * если `forecast_cancel_risk > 500` →
    `risk_category = "Critical Risk"`,
    `critical_risk_cases += 1`

  * если `forecast_cancel_risk > 250` →
    `risk_category = "High Risk"`,
    `high_risk_bookings += 1`

  * иначе →
    `risk_category = "Stable"`

* если `BookingStatus = "Completed"` →
  `risk_category = "Completed"`

После классификации:

```
total_forecast_risk += forecast_cancel_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — это суммарная прогнозная оценка риска отмен по всем бронированиям за период.

Он рассчитывается как накопительная сумма:

```
total_forecast_risk = Σ forecast_cancel_risk
```

### Интерпретация:

* низкое значение → стабильный портфель бронирований;
* среднее → повышенная вероятность отмен в ближайший период;
* высокое → системный риск значительных потерь выручки.

Таким образом руководство получает агрегированную метрику для оценки устойчивости загрузки отелей.

---

## 5. Итоговый Excel

Файл `HotelBookings_Result.xlsx`, лист `Analysis_Q2`:

| BookingID | HotelCode | DaysBeforeCheckIn | ForecastCancelRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ бронирований Q2 завершён.

Всего бронирований: {total_bookings}
Подтверждённых: {confirmed_bookings}
Отменённых: {cancelled_bookings}
Завершённых проживаний: {completed_stays}

Высокий риск: {high_risk_bookings}
Критический риск: {critical_risk_cases}
Потенциальные потери: {total_revenue_loss}
Суммарный прогнозный риск по бронированиям: {total_forecast_risk}

Отчёт сформирован в HotelBookings_Result.xlsx (лист Analysis_Q2).
"""
```
