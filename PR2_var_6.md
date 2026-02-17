# Вариант 6

**Тематика:** Анализ заказов и статуса доставки в сервисе доставки еды

---

## Контекст задачи

Сервис онлайн-доставки еды обрабатывает сотни заказов ежедневно. Пользователи могут менять адрес доставки, отменять заказы, частично оплачивать их или использовать промокоды. Доставка осуществляется курьерами, при этом возможны задержки, отмены и ошибки.

Руководство сталкивается с проблемами:

* часть заказов оплачена частично или с переплатой;
* некоторые заказы задерживаются или отменяются;
* курьеры перегружены, что увеличивает риск опозданий;
* нет единой классификации уровня риска по заказам и маршрутам.

**Цель кейса:** реализовать RPA-процесс, который анализирует заказы, рассчитывает ключевые показатели по оплате и времени доставки, классифицирует заказы по уровню риска, формирует отчёт в Excel и выводит уведомление в модальном окне.

---

# Задание 1. Анализ заказов Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале сервис обработал тысячи заказов. Необходимо выявить заказы с недоплатой или переплатой, просроченные доставки, а также подготовить сводный отчёт для менеджмента по активности курьеров и финансовой нагрузке.

---

## 1. Подготовка данных (Excel)

Создать файл `FoodDelivery_Q1.xlsx` с листом `Orders_Q1`.

**Структура таблицы:**

| Поле          | Тип     | Смысл                                     |
| ------------- | ------- | ----------------------------------------- |
| OrderID       | string  | Уникальный идентификатор заказа           |
| OrderStatus   | string  | Статус заказа: New, InProgress, Delivered |
| OrderAmount   | decimal | Сумма заказа ($)                          |
| AmountPaid    | decimal | Оплаченная сумма ($)                      |
| OrderDate     | date    | Дата оформления заказа                    |
| DeliveryDate  | date    | Плановая дата доставки                    |
| CourierID     | string  | Идентификатор курьера                     |
| NumberOfItems | integer | Количество блюд в заказе                  |

**Методический комментарий:** таблица отражает исходные данные для анализа заказов и доставки. Данные генерируются через DeepSeek для разнообразия сумм, дат и количества блюд.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик                | Начальное значение | Смысл                                                 |
| ---------------------- | ------------------ | ----------------------------------------------------- |
| total_orders           | 0                  | Общее количество заказов                              |
| new_orders             | 0                  | Заказы со статусом New                                |
| inprogress_orders      | 0                  | Заказы со статусом InProgress                         |
| delivered_orders       | 0                  | Заказы со статусом Delivered                          |
| overpaid_orders        | 0                  | Заказы с переплатой                                   |
| underpaid_orders       | 0                  | Заказы с недоплатой                                   |
| delayed_deliveries     | 0                  | Заказы с просроченной датой доставки                  |
| risk_orders            | 0                  | Заказы с высоким финансовым или временным риском      |
| critical_orders        | 0                  | Заказы с критическим отклонением по оплате и доставке |
| totalOutstandingAmount | 0                  | Суммарная недоплата по активным заказам ($)           |

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `OrderAmount, AmountPaid → decimal`
* `OrderDate, DeliveryDate → date`
* `CourierID → верхний регистр`

**Методический комментарий:** корректные типы данных и форматирование обеспечивают точность вычислений и корректное сравнение чисел и дат.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

* `deviation = OrderAmount - AmountPaid`
  *разница между суммой заказа и фактической оплатой*

* `deviation_percent = (OrderAmount - AmountPaid)/OrderAmount * 100`
  *процент недоплаты*

* `remaining_days = DeliveryDate - CurrentDate`
  *количество дней до плановой доставки; отрицательное значение → просрочка*

**Методический комментарий:** эти расчёты позволяют определить заказы с недоплатой и задержки доставки.

---

## 5. Логика классификации заказов (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

* если `AmountPaid > OrderAmount` → `overpaid_orders += 1`
* если `DeliveryDate < CurrentDate AND OrderStatus != "Delivered"` → `delayed_deliveries += 1`

**Составные условия:**

* если `AmountPaid < OrderAmount AND OrderStatus = "New"` → `totalOutstandingAmount += deviation`
  *финансовая нагрузка по новым заказам*

* если `deviation_percent > 20 OR DeliveryDate < CurrentDate` → `risk_orders += 1`

**Вложенные условия:**

* если `OrderStatus = "New"`:

  * если `deviation_percent > 30` → `critical_orders += 1`
  * иначе → заказ управляемый
* если `OrderStatus = "InProgress"` → `inprogress_orders += 1`
* если `OrderStatus = "Delivered"` → `delivered_orders += 1`

**Методический комментарий:** простые условия фиксируют факты (переплата, просрочка), составные — финансовую нагрузку и зону риска, вложенные — итоговую категорию заказа.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `FoodDelivery_Result.xlsx`, лист `Analysis_Q1` с полями:

| OrderID | Deviation | DeviationPercent | RemainingDays | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ заказов Q1 завершён.

Всего заказов: {total_orders}
Новых: {new_orders}
В обработке: {inprogress_orders}
Доставленных: {delivered_orders}

С переплатой: {overpaid_orders}
С недоплатой: {underpaid_orders}
Просроченных доставок: {delayed_deliveries}
Рисковых: {risk_orders}
Критических: {critical_orders}
Суммарная недоплата по новым заказам: {totalOutstandingAmount} $

Отчёт сохранён в FoodDelivery_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ заказов Q2 с прогнозной оценкой риска

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале увеличилось количество заказов, курьеров и маршрутов. Необходимо прогнозировать риск недоплаты, задержки доставки и перегрузки курьеров, используя цикл по строкам таблицы и динамические расчёты.

---

## 1. Подготовка данных (Excel)

Файл `FoodDelivery_Q2.xlsx`, лист `Orders_Q2` с той же структурой, что и Q1. Данные генерируются через DeepSeek.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик                | Начальное значение | Смысл                                                 |
| ---------------------- | ------------------ | ----------------------------------------------------- |
| total_orders           | 0                  | Общее количество заказов                              |
| new_orders             | 0                  | Заказы со статусом New                                |
| inprogress_orders      | 0                  | Заказы со статусом InProgress                         |
| delivered_orders       | 0                  | Заказы со статусом Delivered                          |
| overpaid_orders        | 0                  | Заказы с переплатой                                   |
| underpaid_orders       | 0                  | Заказы с недоплатой                                   |
| delayed_deliveries     | 0                  | Заказы с просроченной датой доставки                  |
| risk_orders            | 0                  | Заказы с высоким финансовым или временным риском      |
| critical_orders        | 0                  | Заказы с критическим отклонением по оплате и доставке |
| totalOutstandingAmount | 0                  | Суммарная недоплата по активным заказам ($)           |
| totalForecastRisk      | 0                  | Суммарный прогнозный риск по портфелю заказов         |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы и формат данных:

* `OrderAmount, AmountPaid → decimal`
* `OrderDate, DeliveryDate → date`
* `CourierID → верхний регистр`

2. Вычислить:

* `deviation = OrderAmount - AmountPaid`
* `deviation_percent = (OrderAmount - AmountPaid)/OrderAmount * 100`
* `remaining_days = DeliveryDate - CurrentDate`
* `total_duration_days = DeliveryDate - OrderDate`
* `forecast_deviation = deviation_percent * (remaining_days / total_duration_days)`

**Методический комментарий:** прогнозное отклонение отражает потенциальный риск недоплаты и задержки доставки.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

* если `OrderStatus = "New"`:

  * если `remaining_days < 0` (доставка просрочена):

    * если `deviation_percent > 30` → `risk_category = "Critical"`, `critical_orders += 1`
    * иначе → `risk_category = "Delayed"`
  * иначе:

    * если `forecast_deviation > 50` → `risk_category = "High Risk"`, `critical_orders += 1`
    * если `forecast_deviation > 30` → `risk_category = "Moderate Risk"`, `risk_orders += 1`
    * иначе → `risk_category = "On Track"`
* если `OrderStatus = "InProgress"` → `risk_category = "In Progress"`
* если `OrderStatus = "Delivered"` → `risk_category = "Delivered"`

**Методический комментарий:** выполняется для каждой строки, чтобы классифицировать заказы по финансовым и временным рискам.

---

## 5. Формирование результирующего Excel (Excel)

Файл `FoodDelivery_Result.xlsx`, лист `Analysis_Q2` с полями:

| OrderID | DeviationPercent | RemainingDays | ForecastDeviation | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Расширенный анализ заказов Q2 завершён.

Всего заказов: {total_orders}
Новых: {new_orders}
В обработке: {inprogress_orders}
Доставленных: {delivered_orders}

Критических: {critical_orders}
Рисковых: {risk_orders}
Просроченных доставок: {delayed_deliveries}
Суммарная недоплата по активным заказам: {totalOutstandingAmount} $
Суммарный прогнозный риск портфеля: {totalForecastRisk}%

Отчёт сформирован в FoodDelivery_Result.xlsx (лист Analysis_Q2).
"""
```

---

