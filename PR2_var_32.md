# Вариант 32

**Тематика:** Анализ бронирований и прогнозирование риска отмен в сервисе для бронирования спортивных залов

---

## Контекст задачи

Сервис бронирования спортивных залов объединяет десятки спортивных объектов и тысячи пользователей. Клиенты бронируют тренировки, групповые занятия и аренду залов. Администраторы залов отслеживают загрузку помещений, а руководство сервиса анализирует риск отмен и недополученной выручки.

Типичные проблемы:

* часть бронирований отменяется в последний момент;
* некоторые залы показывают высокий процент неиспользованных бронирований (no-show);
* отсутствует единый механизм прогнозирования риска отмен;
* требуется автоматическая классификация бронирований по уровню риска.

Необходимо реализовать RPA-процесс, который:

* анализирует данные о бронированиях за период;
* рассчитывает показатели загрузки и отмен;
* прогнозирует риск отмены;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ бронирований Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `GymBookings_Q1.xlsx`, лист `Bookings_Q1`.

### Структура таблицы:

| Поле          | Тип    | Смысл                                              |
| ------------- | ------ | -------------------------------------------------- |
| BookingID     | string | Уникальный номер бронирования                      |
| GymCode       | string | Код спортивного зала                               |
| CityName      | string | Город расположения зала                            |
| RoomType      | string | Тип помещения (тренажёрный зал, студия йоги и др.) |
| BookingDate   | date   | Дата бронирования                                  |
| SessionDate   | date   | Дата занятия                                       |
| BookingStatus | string | Confirmed, Cancelled, Completed, NoShow            |
| TotalPrice    | number | Стоимость бронирования                             |
| DurationHours | number | Продолжительность бронирования в часах             |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                         |
| ------------------- | ------------------ | ----------------------------- |
| total_bookings      | 0                  | Общее количество бронирований |
| confirmed_bookings  | 0                  | Подтверждённые бронирования   |
| cancelled_bookings  | 0                  | Отменённые бронирования       |
| completed_sessions  | 0                  | Завершённые занятия           |
| no_show_cases       | 0                  | Случаи неявки                 |
| short_notice_cancel | 0                  | Отмены в последний момент     |
| total_revenue_loss  | 0                  | Потенциальные потери выручки  |

---

## 3. Преобразование данных

* TotalPrice → number
* DurationHours → number
* BookingDate, SessionDate → date
* BookingID, GymCode → удалить пробелы, верхний регистр

---

## 4. Расчёты (по каждой строке)

* `days_before_session = SessionDate - BookingDate`
* если `BookingStatus = "Cancelled"` → `potential_loss = TotalPrice`
* иначе → `potential_loss = 0`

---

## 5. Логика классификации

**Простые условия:**

* если `BookingStatus = "Confirmed"` → `confirmed_bookings += 1`
* если `BookingStatus = "Cancelled"` → `cancelled_bookings += 1`
* если `BookingStatus = "Completed"` → `completed_sessions += 1`
* если `BookingStatus = "NoShow"` → `no_show_cases += 1`

**Составные условия:**

* если `BookingStatus = "Cancelled" AND days_before_session <= 1` → `short_notice_cancel += 1`
* если `BookingStatus = "Cancelled"` → `total_revenue_loss += potential_loss`

---

## 6. Итоговый Excel

Файл `GymBookings_Result.xlsx`, лист `Analysis_Q1`:

| BookingID | GymCode | DaysBeforeSession | PotentialLoss | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ бронирований Q1 завершён.

Всего бронирований: {total_bookings}
Подтверждено: {confirmed_bookings}
Отменено: {cancelled_bookings}
Завершённых занятий: {completed_sessions}
No-show случаев: {no_show_cases}

Отмены в последний момент: {short_notice_cancel}
Потенциальные потери выручки: {total_revenue_loss}

Отчёт сохранён в GymBookings_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ Q2 с прогнозом риска отмен

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `GymBookings_Q2.xlsx`, лист `Bookings_Q2` (аналогичная структура).

---

## 2. Инициализация переменных

| Счётчик             | Начальное значение | Смысл                                 |
| ------------------- | ------------------ | ------------------------------------- |
| total_bookings      | 0                  | Общее количество бронирований         |
| confirmed_bookings  | 0                  | Подтверждённые бронирования           |
| cancelled_bookings  | 0                  | Отменённые бронирования               |
| completed_sessions  | 0                  | Завершённые занятия                   |
| no_show_cases       | 0                  | Случаи неявки                         |
| short_notice_cancel | 0                  | Отмены в последний момент             |
| high_risk_bookings  | 0                  | Высокий риск отмены                   |
| critical_risk_cases | 0                  | Критический риск отмены               |
| total_revenue_loss  | 0                  | Потери выручки                        |
| total_forecast_risk | 0                  | Прогнозный риск по всем бронированиям |

---

## 3. Цикл по строкам

Для каждой записи:

* Рассчитать `days_before_session = SessionDate - BookingDate`
* `booking_value_per_hour = TotalPrice / DurationHours`
* `cancellation_pressure_index = (2 - days_before_session) * booking_value_per_hour`
* `season_factor = DurationHours * 1.5`
* `forecast_cancel_risk = cancellation_pressure_index + season_factor`

---

## 4. Логика прогнозного риска

* если `BookingStatus = "Cancelled"` →
  `risk_category = "Cancelled"`
  `cancelled_bookings += 1`

* если `BookingStatus = "Confirmed"`:

  * если `forecast_cancel_risk > 300` →
    `risk_category = "Critical Risk"`
    `critical_risk_cases += 1`
  * если `forecast_cancel_risk > 150` →
    `risk_category = "High Risk"`
    `high_risk_bookings += 1`
  * иначе → `risk_category = "Stable"`

Накопление:

```
total_forecast_risk += forecast_cancel_risk
```

---

## 5. Итоговый Excel

Файл `GymBookings_Result.xlsx`, лист `Analysis_Q2`:

| BookingID | GymCode | DaysBeforeSession | ForecastCancelRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ бронирований Q2 завершён.

Всего бронирований: {total_bookings}
Высокий риск: {high_risk_bookings}
Критический риск: {critical_risk_cases}
Потенциальные потери: {total_revenue_loss}
Суммарный прогнозный риск по бронированиям: {total_forecast_risk}

Отчёт сформирован в GymBookings_Result.xlsx (лист Analysis_Q2).
"""
```
