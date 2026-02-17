# Вариант 5

**Тематика:** Анализ бронирований и статуса рейсов авиакомпании

---

## Контекст задачи

Сервис онлайн-бронирования авиабилетов обрабатывает сотни тысяч рейсов и бронирований ежемесячно. Пользователи могут изменять свои рейсы, оплачивать билеты частично или полностью, а рейсы подвержены задержкам, отменам и переноса дат.

Руководство сталкивается с проблемами:

* часть бронирований имеет недоплату или переплату;
* рейсы задерживаются или отменяются, что влияет на удовлетворённость клиентов;
* некоторые направления становятся высокорисковыми из-за перегрузки или сезонного спроса;
* нет единой классификации риска по бронированиям и рейсам.

**Цель кейса:** реализовать RPA-процесс, который анализирует бронирования и статус рейсов, рассчитывает финансовые показатели и прогнозы задержек, классифицирует бронирования по уровню риска, формирует отчёт в Excel и выводит уведомление в модальном окне.

---

# Задание 1. Анализ бронирований Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале сервис обработал множество бронирований на международные и внутренние рейсы. Необходимо выявить бронирования с отклонениями по оплате, просроченные рейсы и потенциальные проблемные направления, а затем подготовить сводный отчёт для менеджмента.

---

## 1. Подготовка данных (Excel)

Создать файл `AirBooking_Q1.xlsx` с листом `Bookings_Q1`.

**Структура таблицы:**

| Поле           | Тип     | Смысл                                          |
| -------------- | ------- | ---------------------------------------------- |
| BookingID      | string  | Уникальный идентификатор бронирования          |
| BookingStatus  | string  | Статус бронирования: New, Confirmed, Completed |
| TicketPrice    | decimal | Стоимость билета ($)                           |
| AmountPaid     | decimal | Оплаченная сумма ($)                           |
| BookingDate    | date    | Дата бронирования                              |
| FlightDate     | date    | Дата рейса                                     |
| FlightNumber   | string  | Номер рейса                                    |
| PassengerCount | integer | Количество пассажиров по бронированию          |

**Методический комментарий:** таблица отражает исходные данные по бронированиям и рейсам. Значения генерируются через DeepSeek для разнообразия сумм, дат и количества пассажиров.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик               | Начальное значение | Смысл                                                    |
| --------------------- | ------------------ | -------------------------------------------------------- |
| total_bookings        | 0                  | Общее количество бронирований                            |
| new_bookings          | 0                  | Бронирования со статусом New                             |
| confirmed_bookings    | 0                  | Бронирования со статусом Confirmed                       |
| completed_bookings    | 0                  | Бронирования со статусом Completed                       |
| overpaid_bookings     | 0                  | Бронирования с переплатой                                |
| underpaid_bookings    | 0                  | Бронирования с недоплатой                                |
| delayed_flights       | 0                  | Рейсы с прошедшей датой без завершения                   |
| risk_bookings         | 0                  | Бронирования с высоким финансовым или временным риском   |
| critical_bookings     | 0                  | Бронирования с критическим отклонением по оплате и рейсу |
| totalOutstandingValue | 0                  | Суммарная недоплата по активным бронированиям ($)        |

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `TicketPrice, AmountPaid → decimal`
* `BookingDate, FlightDate → date`
* `FlightNumber → верхний регистр`

**Методический комментарий:** корректные типы данных и форматирование обеспечивают точность вычислений и корректное сравнение чисел и дат.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

* `deviation = TicketPrice - AmountPaid`
  *разница между стоимостью билета и фактической оплатой*

* `deviation_percent = (TicketPrice - AmountPaid)/TicketPrice * 100`
  *процент недоплаты*

* `remaining_days = FlightDate - CurrentDate`
  *количество дней до рейса; отрицательное значение → рейс прошёл или задержан*

**Методический комментарий:** эти расчёты позволяют определить бронирования с недоплатой и рейсы с просрочкой.

---

## 5. Логика классификации бронирований (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

* если `AmountPaid > TicketPrice` → `overpaid_bookings += 1`
* если `FlightDate < CurrentDate AND BookingStatus != "Completed"` → `delayed_flights += 1`

**Составные условия:**

* если `AmountPaid < TicketPrice AND BookingStatus = "New"` → `totalOutstandingValue += deviation`
  *финансовая нагрузка по новым бронированиям*

* если `deviation_percent > 20 OR FlightDate < CurrentDate` → `risk_bookings += 1`

**Вложенные условия:**

* если `BookingStatus = "New"`:

  * если `deviation_percent > 30` → `critical_bookings += 1`
  * иначе → бронирование управляемое
* если `BookingStatus = "Confirmed"` → `confirmed_bookings += 1`
* если `BookingStatus = "Completed"` → `completed_bookings += 1`

**Методический комментарий:** простые условия фиксируют факты (переплата, задержка), составные — финансовую нагрузку и зону риска, вложенные — итоговую категорию бронирования.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `AirBooking_Result.xlsx`, лист `Analysis_Q1` с полями:

| BookingID | Deviation | DeviationPercent | RemainingDays | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ бронирований Q1 завершён.

Всего бронирований: {total_bookings}
Новых: {new_bookings}
Подтверждённых: {confirmed_bookings}
Завершённых: {completed_bookings}

С переплатой: {overpaid_bookings}
С недоплатой: {underpaid_bookings}
Просроченных рейсов: {delayed_flights}
Рисковых: {risk_bookings}
Критических: {critical_bookings}
Суммарная недоплата по новым бронированиям: {totalOutstandingValue} $

Отчёт сохранён в AirBooking_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ бронирований Q2 с прогнозной оценкой риска

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале увеличилось количество бронирований, рейсов и пассажиров. Необходимо прогнозировать риск недоплаты, задержки рейсов и перегрузки по направлениям с использованием цикла по строкам таблицы.

---

## 1. Подготовка данных (Excel)

Файл `AirBooking_Q2.xlsx`, лист `Bookings_Q2` с той же структурой, что и Q1. Данные генерируются через DeepSeek.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик               | Начальное значение | Смысл                                                    |
| --------------------- | ------------------ | -------------------------------------------------------- |
| total_bookings        | 0                  | Общее количество бронирований                            |
| new_bookings          | 0                  | Бронирования со статусом New                             |
| confirmed_bookings    | 0                  | Бронирования со статусом Confirmed                       |
| completed_bookings    | 0                  | Бронирования со статусом Completed                       |
| overpaid_bookings     | 0                  | Бронирования с переплатой                                |
| underpaid_bookings    | 0                  | Бронирования с недоплатой                                |
| delayed_flights       | 0                  | Рейсы с прошедшей датой без завершения                   |
| risk_bookings         | 0                  | Бронирования с высоким финансовым или временным риском   |
| critical_bookings     | 0                  | Бронирования с критическим отклонением по оплате и рейсу |
| totalOutstandingValue | 0                  | Суммарная недоплата по активным бронированиям ($)        |
| totalForecastRisk     | 0                  | Суммарный прогнозный риск по портфелю бронирований       |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы и формат данных:

* `TicketPrice, AmountPaid → decimal`
* `BookingDate, FlightDate → date`
* `FlightNumber → верхний регистр`

2. Вычислить:

* `deviation = TicketPrice - AmountPaid`
* `deviation_percent = (TicketPrice - AmountPaid)/TicketPrice * 100`
* `remaining_days = FlightDate - CurrentDate`
* `total_duration_days = FlightDate - BookingDate`
* `forecast_deviation = deviation_percent * (remaining_days / total_duration_days)`

**Методический комментарий:** прогнозное отклонение отражает потенциальный риск недоплаты и задержки рейса.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

* если `BookingStatus = "New"`:

  * если `remaining_days < 0` (рейс прошёл или задержан):

    * если `deviation_percent > 30` → `risk_category = "Critical"`, `critical_bookings += 1`
    * иначе → `risk_category = "Delayed"`
  * иначе:

    * если `forecast_deviation > 50` → `risk_category = "High Risk"`, `critical_bookings += 1`
    * если `forecast_deviation > 30` → `risk_category = "Moderate Risk"`, `risk_bookings += 1`
    * иначе → `risk_category = "On Track"`
* если `BookingStatus = "Confirmed"` → `risk_category = "Confirmed"`
* если `BookingStatus = "Completed"` → `risk_category = "Completed"`

**Методический комментарий:** выполняется для каждой строки, чтобы классифицировать бронирования по финансовым и временным рискам.

---

## 5. Формирование результирующего Excel (Excel)

Файл `AirBooking_Result.xlsx`, лист `Analysis_Q2` с полями:

| BookingID | DeviationPercent | RemainingDays | ForecastDeviation | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Расширенный анализ бронирований Q2 завершён.

Всего бронирований: {total_bookings}
Новых: {new_bookings}
Подтверждённых: {confirmed_bookings}
Завершённых: {completed_bookings}

Критических: {critical_bookings}
Рисковых: {risk_bookings}
Просроченных рейсов: {delayed_flights}
Суммарная недоплата по активным бронированиям: {totalOutstandingValue} $
Суммарный прогнозный риск портфеля: {totalForecastRisk}%

Отчёт сформирован в AirBooking_Result.xlsx (лист Analysis_Q2).
"""
```

---
