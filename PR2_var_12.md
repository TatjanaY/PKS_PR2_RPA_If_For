# Вариант 12

**Тематика:** Система управления складом — анализ запасов и прогноз потребления

---

## Контекст задачи

Компания управляет большим складом с разнообразными товарами: от комплектующих до готовой продукции. Система фиксирует поступления, отгрузки, текущие остатки и прогнозы потребления.

Менеджмент сталкивается с типичными проблемами:

* часть товаров имеет низкий оборот или избыточный запас;
* отдельные позиции часто заканчиваются раньше срока;
* необходимо выявлять критические и рисковые позиции, чтобы оптимизировать закупки и складские операции;
* прогнозировать потенциальный дефицит или переполнение склада.

**Цель кейса:** реализовать RPA-процесс, который автоматически анализирует движение товаров, рассчитывает ключевые показатели запасов, классифицирует позиции по уровню риска и формирует итоговый отчёт в Excel с уведомлением в модальном окне.

---

# Задание 1. Анализ складских остатков за январь (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Вы — аналитик отдела логистики. Необходимо оценить состояние запасов на складе за первый месяц года, выявить товары с низким оборотом или риском дефицита и составить управленческую сводку.

---

## 1. Подготовка данных (Excel)

Создать файл `Warehouse_Inventory_Jan.xlsx` с листом `Inventory_Jan`.

**Структура таблицы:**

| Поле             | Тип     | Смысл                            |
| ---------------- | ------- | -------------------------------- |
| ItemID           | string  | Уникальный идентификатор товара  |
| ItemName         | string  | Наименование товара              |
| Category         | string  | Категория товара                 |
| StockQty         | decimal | Текущий остаток                  |
| MinRequiredQty   | decimal | Минимальный необходимый остаток  |
| MaxCapacity      | decimal | Максимальная вместимость склада  |
| IncomingQty      | decimal | Количество поступлений за период |
| OutgoingQty      | decimal | Количество отгрузок за период    |
| LastMovementDate | date    | Дата последнего движения товара  |

**Методический комментарий:** таблица отражает текущие остатки, движение и плановые показатели товаров.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                                          |
| ------------------- | ------------------ | ---------------------------------------------- |
| total_items         | 0                  | Общее количество товарных позиций              |
| low_stock_items     | 0                  | Товары с запасом меньше минимального           |
| overstock_items     | 0                  | Товары с запасом выше максимальной вместимости |
| moving_items        | 0                  | Товары с движением за период                   |
| total_stock         | 0                  | Суммарный остаток                              |
| total_incoming      | 0                  | Суммарные поступления                          |
| total_outgoing      | 0                  | Суммарные отгрузки                             |
| total_forecast_risk | 0                  | Суммарный прогнозный риск по запасам           |

**Методический комментарий:** переменная `total_forecast_risk` будет суммировать баллы риска каждой позиции:

* `Critical` → +3
* `High Risk` → +2
* `At Risk` → +1
* `Stable` → 0

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `stockqty, minrequiredqty, maxcapacity, incomingqty, outgoingqty → decimal`
* `lastmovementdate → date`
* `itemname, category, itemid → удалить пробелы / верхний регистр`

**Методический комментарий:** корректное приведение типов обеспечивает точность вычислений и корректное сравнение числовых и датированных значений.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

```
net_movement = incomingqty - outgoingqty
forecast_stock = stockqty + net_movement
stock_ratio = forecast_stock / maxcapacity * 100
```

* `net_movement` — чистое изменение запасов за период;
* `forecast_stock` — прогнозируемый остаток после движения;
* `stock_ratio` — процент заполнения по отношению к вместимости склада.

---

## 5. Логика классификации запасов (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

```
if stockqty < minrequiredqty:
    low_stock_items += 1
```

```
if stockqty > maxcapacity:
    overstock_items += 1
```

```
if incomingqty > 0 OR outgoingqty > 0:
    moving_items += 1
```

**Составные условия:**

```
if stock_ratio < 20 OR stockqty < minrequiredqty:
    risk_category = "At Risk"
else:
    risk_category = "Stable"
```

**Вложенные условия:**

```
if stock_ratio < 10 OR stockqty == 0:
    risk_category = "Critical"
    total_forecast_risk += 3
elif stock_ratio < 30 OR stockqty < minrequiredqty:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif stock_ratio < 50:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** простые условия фиксируют низкий и избыточный запас, составные — выявляют риск дефицита, вложенные — определяют итоговую категорию с учётом прогнозного остатка. Баллы суммируются в `total_forecast_risk`.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `Warehouse_Analysis_Result.xlsx`, лист `Analysis_Jan` с полями:

| ItemID | ItemName | Category | StockQty | IncomingQty | OutgoingQty | ForecastStock | StockRatio | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ складских остатков за январь завершён.

Всего позиций: {total_items}
С низким запасом: {low_stock_items}
С превышением вместимости: {overstock_items}
С движением: {moving_items}
Суммарный остаток: {total_stock}
Суммарные поступления: {total_incoming}
Суммарные отгрузки: {total_outgoing}
Суммарный прогнозный риск запасов: {total_forecast_risk} баллов

Отчёт сохранён в Warehouse_Analysis_Result.xlsx (лист Analysis_Jan).
"""
```

---

# Задание 2. Прогнозная оценка запасов за февраль (расширенный уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором месяце необходимо оценить потенциальный дефицит и переполнение запасов на складе с учётом текущих остатков, движения товаров и прогнозируемого спроса, используя цикл по строкам таблицы.

---

## 1. Подготовка данных (Excel)

Файл `Warehouse_Inventory_Feb.xlsx`, лист `Inventory_Feb` с дополнительными полями:

| Поле             | Тип     | Смысл                           |
| ---------------- | ------- | ------------------------------- |
| ItemID           | string  | Уникальный идентификатор товара |
| ItemName         | string  | Наименование товара             |
| Category         | string  | Категория                       |
| StockQty         | decimal | Текущий остаток                 |
| MinRequiredQty   | decimal | Минимальный необходимый остаток |
| MaxCapacity      | decimal | Максимальная вместимость склада |
| IncomingQty      | decimal | Количество поступлений          |
| OutgoingQty      | decimal | Количество отгрузок             |
| ForecastDemand   | decimal | Прогнозируемый спрос на период  |
| LastMovementDate | date    | Дата последнего движения        |

**Методический комментарий:** прогнозируемый спрос позволяет оценить потенциальный риск дефицита или переполнения.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                                       |
| ------------------- | ------------------ | ------------------------------------------- |
| total_items         | 0                  | Общее количество товарных позиций           |
| low_stock_items     | 0                  | Товары с запасом < минимального             |
| overstock_items     | 0                  | Товары с запасом > максимальной вместимости |
| moving_items        | 0                  | Товары с движением за период                |
| total_stock         | 0                  | Суммарный остаток                           |
| total_incoming      | 0                  | Суммарные поступления                       |
| total_outgoing      | 0                  | Суммарные отгрузки                          |
| total_forecast_risk | 0                  | Суммарный прогнозный риск запасов           |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы данных:

* `stockqty, minrequiredqty, maxcapacity, incomingqty, outgoingqty, forecastdemand → decimal`
* `lastmovementdate → date`
* `itemname, category, itemid → удалить пробелы / верхний регистр`

2. Вычислить показатели:

```
net_movement = incomingqty - outgoingqty
forecast_stock = stockqty + net_movement
stock_ratio = forecast_stock / maxcapacity * 100
forecast_gap = forecast_stock - forecastdemand
```

* `forecast_gap` показывает, насколько прогнозируемый остаток покрывает прогнозируемый спрос.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

```
if forecast_stock < minrequiredqty OR forecast_gap < 0:
    risk_category = "Critical"
    total_forecast_risk += 3
elif forecast_stock < minrequiredqty*1.5 OR forecast_gap < forecastdemand*0.2:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif forecast_stock < maxcapacity*0.5:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** простые условия фиксируют низкий или высокий запас, составные — учитывают движение и прогнозируемый спрос, вложенные — определяют категорию риска с учётом прогноза. Баллы суммируются в `total_forecast_risk`.

---

## 5. Формирование результирующего Excel (Excel)

Файл `Warehouse_Analysis_Result.xlsx`, лист `Analysis_Feb` с полями:

| ItemID | ItemName | Category | StockQty | IncomingQty | OutgoingQty | ForecastStock | StockRatio | ForecastGap | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Прогнозный анализ запасов за февраль завершён.

Всего позиций: {total_items}
С низким запасом: {low_stock_items}
С превышением вместимости: {overstock_items}
С движением: {moving_items}
Суммарный остаток: {total_stock}
Суммарные поступления: {total_incoming}
Суммарные отгрузки: {total_outgoing}
Суммарный прогнозный риск запасов: {total_forecast_risk} баллов

Отчёт сохранён в Warehouse_Analysis_Result.xlsx (лист Analysis_Feb).
"""
```

---
