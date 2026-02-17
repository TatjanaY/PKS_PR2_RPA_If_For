# Вариант 23

**Тематика:** Анализ продаж и прогнозирование коммерческого риска в онлайн-магазине одежды

---

## Контекст задачи

Онлайн-магазин одежды реализует товары разных категорий (верхняя одежда, обувь, аксессуары и т.д.), обрабатывает заказы, возвраты и скидки.

Типовые бизнес-риски:

* высокий процент возвратов;
* продажи с чрезмерными скидками;
* низкая маржинальность отдельных категорий;
* перегрузка склада по популярным товарам;
* снижение чистой прибыли в период акций.

Необходимо реализовать RPA-процесс, который:

* анализирует заказы за период;
* рассчитывает маржинальность и уровень возвратов;
* прогнозирует риск снижения прибыли;
* формирует Excel-отчёт;
* рассчитывает агрегированную метрику `total_forecast_risk`.

---

# Задание 1. Анализ заказов за месяц (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `ClothingStore_Month1.xlsx`, лист `Orders`.

**Структура таблицы:**

| Поле            | Тип    | Смысл                                |
| --------------- | ------ | ------------------------------------ |
| OrderID         | string | ID заказа                            |
| CustomerID      | string | ID клиента                           |
| ProductID       | string | ID товара                            |
| Category        | string | Jackets, Shoes, Dresses, Accessories |
| OrderDate       | date   | Дата заказа                          |
| Quantity        | number | Количество                           |
| PricePerUnit    | number | Цена за единицу                      |
| DiscountPercent | number | Скидка (%)                           |
| CostPrice       | number | Себестоимость                        |
| ReturnStatus    | string | Returned / NotReturned               |

---

## 2. Инициализация переменных

| Переменная           | Начальное значение |
| -------------------- | ------------------ |
| total_orders         | 0                  |
| total_revenue        | 0                  |
| total_profit         | 0                  |
| returned_orders      | 0                  |
| high_discount_orders | 0                  |
| low_margin_orders    | 0                  |

---

## 3. Преобразование данных

* Quantity, PricePerUnit, DiscountPercent, CostPrice → number
* OrderDate → date
* OrderID, CustomerID, ProductID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (на каждой строке)

```
gross_amount = Quantity * PricePerUnit
discount_amount = gross_amount * (DiscountPercent / 100)
net_revenue = gross_amount - discount_amount
total_cost = Quantity * CostPrice
profit = net_revenue - total_cost
margin_percent = (profit / net_revenue) * 100
```

---

## 5. Логика анализа

* если `ReturnStatus = "Returned"` →
  `returned_orders += 1`

* если `DiscountPercent > 40` →
  `high_discount_orders += 1`

* если `margin_percent < 15` →
  `low_margin_orders += 1`

Накопление показателей:

```
total_revenue += net_revenue
total_profit += profit
```

---

## 6. Итоговый Excel

Файл `ClothingStore_Result.xlsx`, лист `Month1_Analysis`:

| OrderID | NetRevenue | Profit | MarginPercent | OperationalCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ продаж завершён.

Всего заказов: {total_orders}
Возвратов: {returned_orders}
Заказов с высокой скидкой: {high_discount_orders}
Заказов с низкой маржой: {low_margin_orders}

Общая выручка: {total_revenue}
Общая прибыль: {total_profit}

Отчёт сохранён в ClothingStore_Result.xlsx (лист Month1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ с прогнозом коммерческого риска

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `ClothingStore_Month2.xlsx`, лист `Orders_Month2`.

---

## 2. Инициализация переменных

| Переменная           | Начальное значение |
| -------------------- | ------------------ |
| total_orders         | 0                  |
| returned_orders      | 0                  |
| high_risk_orders     | 0                  |
| critical_risk_orders | 0                  |
| total_revenue        | 0                  |
| total_profit         | 0                  |
| total_forecast_risk  | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой строки:

```
gross_amount = Quantity * PricePerUnit
discount_amount = gross_amount * (DiscountPercent / 100)
net_revenue = gross_amount - discount_amount
total_cost = Quantity * CostPrice
profit = net_revenue - total_cost
margin_percent = (profit / net_revenue) * 100

discount_factor = DiscountPercent * 50
return_factor = 
    если ReturnStatus = "Returned" → 1000
    иначе → 0

margin_penalty = 
    если margin_percent < 10 → 2000
    если margin_percent < 20 → 1000
    иначе → 0

forecast_commercial_risk = discount_factor + return_factor + margin_penalty
```

---

## 4. Логика классификации риска

* если `forecast_commercial_risk > 4000` →
  `risk_category = "Critical Risk"`
  `critical_risk_orders += 1`

* если `forecast_commercial_risk > 2000` →
  `risk_category = "High Risk"`
  `high_risk_orders += 1`

* иначе →
  `risk_category = "Stable"`

После классификации:

```
total_forecast_risk += forecast_commercial_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная оценка коммерческого риска за период.

```
total_forecast_risk = Σ forecast_commercial_risk
```

### Интерпретация:

* низкое значение → устойчивые продажи и контролируемая маржа;
* среднее → повышенная нагрузка из-за скидок и возвратов;
* высокое → системная угроза прибыльности (частые возвраты, демпинг, низкая маржа).

Метрика используется для:

* корректировки ценовой политики;
* ограничения глубины скидок;
* анализа проблемных категорий товаров;
* принятия решений по управлению ассортиментом.

---

## 5. Итоговый Excel

Файл `ClothingStore_Result.xlsx`, лист `Month2_Analysis`:

| OrderID | NetRevenue | Profit | ForecastCommercialRisk | RiskCategory |

---

## 6. Итоговое уведомление

```
f"""Расширенный анализ продаж завершён.

Всего заказов: {total_orders}
Высокий риск: {high_risk_orders}
Критический риск: {critical_risk_orders}

Суммарный прогнозный коммерческий риск: {total_forecast_risk}

Отчёт сформирован в ClothingStore_Result.xlsx (лист Month2_Analysis).
"""
```
