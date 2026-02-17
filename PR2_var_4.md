# Вариант 4

**Тематика:** Анализ продаж и складских запасов интернет-магазина электроники

---

## Контекст задачи

Интернет-магазин продаёт широкий ассортимент электроники: смартфоны, ноутбуки, бытовую технику и аксессуары. Каждый месяц обрабатывается несколько тысяч заказов, одновременно управляются складские остатки на нескольких складах.

Руководство сталкивается с типичными проблемами:

* часть товаров продаётся с опозданием по срокам доставки;
* некоторые заказы оплачены частично или с превышением суммы;
* на складе появляются дефицитные или избыточные позиции;
* нет единой классификации риска по заказам и остаткам на складах.

**Цель кейса:** реализовать RPA-процесс, который анализирует заказы и остатки на складе, рассчитывает финансовые показатели и прогнозы дефицита, классифицирует заказы по уровню риска, формирует отчёт в Excel и выводит уведомление в модальном окне.

---

# Задание 1. Анализ заказов Q1 (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** В первом квартале интернет-магазин обработал тысячи заказов. Необходимо определить, какие заказы имеют отклонения по сумме и срокам, выявить товары с дефицитом на складе, и подготовить сводный отчёт для руководства.

---

## 1. Подготовка данных (Excel)

Создать файл `Eshop_Orders_Q1.xlsx` с листом `Orders_Q1`.

**Структура таблицы:**

| Поле          | Тип     | Смысл                                  |
| ------------- | ------- | -------------------------------------- |
| OrderID       | string  | Уникальный идентификатор заказа        |
| OrderStatus   | string  | Статус заказа: New, Shipped, Completed |
| OrderValue    | decimal | Сумма заказа ($)                       |
| AmountPaid    | decimal | Оплаченная сумма ($)                   |
| OrderDate     | date    | Дата оформления заказа                 |
| DeliveryDate  | date    | Планируемая дата доставки              |
| ProductSKU    | string  | Артикул товара                         |
| StockQuantity | integer | Количество товара на складе            |

**Методический комментарий:** таблица отражает исходные данные по заказам и запасам. Значения генерируются через DeepSeek для разнообразия сумм, дат и остатков.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик               | Начальное значение | Смысл                                                 |
| --------------------- | ------------------ | ----------------------------------------------------- |
| total_orders          | 0                  | Общее количество заказов                              |
| new_orders            | 0                  | Заказы со статусом New                                |
| shipped_orders        | 0                  | Заказы со статусом Shipped                            |
| completed_orders      | 0                  | Заказы со статусом Completed                          |
| overpaid_orders       | 0                  | Заказы с переплатой                                   |
| delayed_orders        | 0                  | Заказы с просроченной датой доставки                  |
| low_stock_products    | 0                  | Товары с количеством на складе меньше порогового      |
| risk_orders           | 0                  | Заказы с высоким финансовым или временным риском      |
| critical_orders       | 0                  | Заказы с критическим отклонением по оплате и доставке |
| totalOutstandingValue | 0                  | Суммарная недоплата по активным заказам ($)           |

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `OrderValue, AmountPaid → decimal`
* `OrderDate, DeliveryDate → date`
* `ProductSKU → верхний регистр`

**Методический комментарий:** корректные типы данных и форматирование обеспечивают точность вычислений и корректное сравнение чисел и дат.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

* `deviation = OrderValue - AmountPaid`
  *разница между суммой заказа и фактической оплатой*

* `deviation_percent = (OrderValue - AmountPaid)/OrderValue * 100`
  *процент недоплаты по заказу*

* `remaining_days = DeliveryDate - CurrentDate`
  *количество дней до плановой доставки; отрицательное значение → просрочка*

**Методический комментарий:** эти расчёты позволяют определить заказы с недоплатой и задержкой по доставке.

---

## 5. Логика классификации заказов (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

* если `AmountPaid > OrderValue` → `overpaid_orders += 1`
* если `DeliveryDate < CurrentDate` → `delayed_orders += 1`

**Составные условия:**

* если `AmountPaid < OrderValue AND OrderStatus = "New"` → `totalOutstandingValue += deviation`
  *финансовая нагрузка по новым заказам*

* если `deviation_percent > 20 OR DeliveryDate < CurrentDate` → `risk_orders += 1`

**Вложенные условия:**

* если `OrderStatus = "New"`:

  * если `deviation_percent > 30` → `critical_orders += 1`
  * иначе → заказ управляемый
* если `OrderStatus = "Shipped"` → `shipped_orders += 1`
* если `OrderStatus = "Completed"` → `completed_orders += 1`
* если `StockQuantity < 5` → `low_stock_products += 1`

**Методический комментарий:** простые условия фиксируют факты (переплата, просрочка), составные — финансовую нагрузку и зоны риска, вложенные — итоговую категорию заказа и низкий запас.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `Eshop_Orders_Result.xlsx`, лист `Analysis_Q1` с полями:

| OrderID | Deviation | DeviationPercent | RemainingDays | RiskCategory | LowStock |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ заказов Q1 завершён.

Всего заказов: {total_orders}
Новых: {new_orders}
Отправленных: {shipped_orders}
Завершённых: {completed_orders}

С переплатой: {overpaid_orders}
Просроченных: {delayed_orders}
Рисковых: {risk_orders}
Критических: {critical_orders}
Товаров с низким запасом: {low_stock_products}
Суммарная недоплата по новым заказам: {totalOutstandingValue} $

Отчёт сохранён в Eshop_Orders_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ заказов Q2 с прогнозной оценкой риска

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором квартале объем заказов и товаров на складе увеличился. Необходимо спрогнозировать риск недоплаты и задержки доставки, а также выявить потенциальный дефицит товаров с использованием цикла по строкам таблицы.

---

## 1. Подготовка данных (Excel)

Файл `Eshop_Orders_Q2.xlsx`, лист `Orders_Q2` с той же структурой, что и Q1. Данные генерируются через DeepSeek.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик               | Начальное значение | Смысл                                                        |
| --------------------- | ------------------ | ------------------------------------------------------------ |
| total_orders          | 0                  | Общее количество заказов                                     |
| new_orders            | 0                  | Заказы со статусом New                                       |
| shipped_orders        | 0                  | Заказы со статусом Shipped                                   |
| completed_orders      | 0                  | Заказы со статусом Completed                                 |
| overpaid_orders       | 0                  | Заказы с переплатой                                          |
| delayed_orders        | 0                  | Заказы с просрочкой                                          |
| low_stock_products    | 0                  | Товары с количеством на складе меньше порогового             |
| risk_orders           | 0                  | Заказы с высоким финансовым или временным риском             |
| critical_orders       | 0                  | Заказы с критическим отклонением по оплате и срокам доставки |
| totalOutstandingValue | 0                  | Суммарная недоплата по активным заказам ($)                  |
| totalForecastRisk     | 0                  | Суммарный прогнозный риск по портфелю заказов                |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы и формат данных:

* `OrderValue, AmountPaid → decimal`
* `OrderDate, DeliveryDate → date`
* `ProductSKU → верхний регистр`

2. Вычислить:

* `deviation = OrderValue - AmountPaid`
* `deviation_percent = (OrderValue - AmountPaid)/OrderValue * 100`
* `remaining_days = DeliveryDate - CurrentDate`
* `total_duration_days = DeliveryDate - OrderDate`
* `forecast_deviation = deviation_percent * (remaining_days / total_duration_days)`

**Методический комментарий:** прогнозное отклонение отражает потенциальный риск недоплаты и задержки до конца заказа.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

* если `OrderStatus = "New"`:

  * если `remaining_days < 0` (заказ просрочен):

    * если `deviation_percent > 30` → `risk_category = "Critical"`, `critical_orders += 1`
    * иначе → `risk_category = "Delayed"`
  * иначе:

    * если `forecast_deviation > 50` → `risk_category = "High Risk"`, `critical_orders += 1`
    * если `forecast_deviation > 30` → `risk_category = "Moderate Risk"`, `risk_orders += 1`
    * иначе → `risk_category = "On Track"`
* если `OrderStatus = "Shipped"` → `risk_category = "Shipped"`
* если `OrderStatus = "Completed"` → `risk_category = "Completed"`
* если `StockQuantity < 5` → `LowStockFlag = True`

**Методический комментарий:** выполняется для каждой строки, чтобы классифицировать заказы по финансовым, временным и складским рискам.

---

## 5. Формирование результирующего Excel (Excel)

Файл `Eshop_Orders_Result.xlsx`, лист `Analysis_Q2` с полями:

| OrderID | DeviationPercent | RemainingDays | ForecastDeviation | RiskCategory | LowStockFlag |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Расширенный анализ заказов Q2 завершён.

Всего заказов: {total_orders}
Новых: {new_orders}
Отправленных: {shipped_orders}
Завершённых: {completed_orders}

Критических: {critical_orders}
Рисковых: {risk_orders}
Просроченных: {delayed_orders}
Товаров с низким запасом: {low_stock_products}
Суммарная недоплата по активным заказам: {totalOutstandingValue} $
Суммарный прогнозный риск портфеля: {totalForecastRisk}%

Отчёт сформирован в Eshop_Orders_Result.xlsx (лист Analysis_Q2).
"""
```

---
