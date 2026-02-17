# Вариант 13

**Тематика:** Мобильный банк — анализ транзакций и прогноз рисков операций

---

## Контекст задачи

Банк предоставляет клиентам мобильное приложение для управления счетами, переводов и платежей. В системе фиксируются транзакции, остатки, типы операций и время проведения.

Менеджмент сталкивается с типичными проблемами:

* часть транзакций подозрительно превышает средние суммы;
* отдельные операции совершаются в нерабочее время или из неожиданных геолокаций;
* необходимо выявлять потенциальные мошеннические операции и формировать сводку по активности клиентов;
* прогнозировать риск финансовых потерь и отклонений по портфелю транзакций.

**Цель кейса:** реализовать RPA-процесс, который автоматически анализирует транзакции в мобильном банке, рассчитывает ключевые показатели активности и риска, классифицирует операции и формирует итоговый отчёт в Excel с уведомлением в модальном окне.

---

# Задание 1. Анализ транзакций за январь (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Вы — аналитик банка. Необходимо оценить активность пользователей и выявить транзакции с отклонениями за первый месяц года, чтобы подготовить управленческую сводку.

---

## 1. Подготовка данных (Excel)

Создать файл `MobileBank_Transactions_Jan.xlsx` с листом `Transactions_Jan`.

**Структура таблицы:**

| Поле            | Тип     | Смысл                                     |
| --------------- | ------- | ----------------------------------------- |
| TransactionID   | string  | Уникальный идентификатор транзакции       |
| UserID          | string  | Уникальный идентификатор клиента          |
| TransactionDate | date    | Дата и время проведения транзакции        |
| Amount          | decimal | Сумма операции, тыс. $                    |
| TransactionType | string  | Тип операции (Transfer, Payment, Deposit) |
| AccountBalance  | decimal | Баланс на счёте после транзакции          |
| GeoLocation     | string  | Геолокация проведения операции            |
| IsInternational | string  | Да/Нет — международная операция           |

**Методический комментарий:** таблица отражает реальные операции клиентов и содержит поля для анализа отклонений и риска.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик                    | Начальное значение | Смысл                                        |
| -------------------------- | ------------------ | -------------------------------------------- |
| total_transactions         | 0                  | Общее количество транзакций                  |
| high_amount_transactions   | 0                  | Транзакции с суммой выше лимита              |
| international_transactions | 0                  | Международные операции                       |
| night_transactions         | 0                  | Транзакции вне стандартного рабочего времени |
| total_amount               | 0                  | Суммарная сумма всех транзакций              |
| total_forecast_risk        | 0                  | Суммарный прогнозный риск транзакций         |

**Методический комментарий:** переменная `total_forecast_risk` суммирует баллы риска каждой транзакции:

* `Critical` → +3
* `High Risk` → +2
* `At Risk` → +1
* `Stable` → 0

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `amount, accountbalance → decimal`
* `transactiondate → date`
* `transactiontype, userid, transactionid, geolocation, isinternational → удалить пробелы / верхний регистр`

**Методический комментарий:** корректное приведение типов обеспечивает точность вычислений и корректное сравнение сумм, дат и строк.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

```
hour_of_day = transactiondate.hour
is_night = hour_of_day < 6 OR hour_of_day > 22
```

* `hour_of_day` — час проведения транзакции;
* `is_night` — логическая переменная для выявления операций в ночное время.

---

## 5. Логика классификации транзакций (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

```
if amount > 100:
    high_amount_transactions += 1
```

```
if isinternational == "YES":
    international_transactions += 1
```

```
if is_night:
    night_transactions += 1
```

**Составные условия:**

```
if amount > 100 OR isinternational == "YES" OR is_night:
    risk_category = "At Risk"
else:
    risk_category = "Stable"
```

**Вложенные условия:**

```
if amount > 500 OR (isinternational == "YES" AND amount > 200):
    risk_category = "Critical"
    total_forecast_risk += 3
elif amount > 200 OR isinternational == "YES" OR is_night:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif amount > 100:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** простые условия фиксируют отдельные признаки риска, составные — объединяют их, вложенные — формируют итоговую категорию транзакции с учётом суммы, времени и типа операции. Баллы суммируются в `total_forecast_risk`.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `MobileBank_Analysis_Result.xlsx`, лист `Analysis_Jan` с полями:

| TransactionID | UserID | Amount | TransactionType | AccountBalance | GeoLocation | IsInternational | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ транзакций за январь завершён.

Всего транзакций: {total_transactions}
С высокой суммой: {high_amount_transactions}
Международные: {international_transactions}
Ночные операции: {night_transactions}
Суммарная сумма транзакций: {total_amount} тыс. $
Суммарный прогнозный риск: {total_forecast_risk} баллов

Отчёт сохранён в MobileBank_Analysis_Result.xlsx (лист Analysis_Jan).
"""
```

---

# Задание 2. Прогнозная оценка транзакций за февраль (расширенный уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором месяце необходимо оценить потенциальный риск транзакций с учётом текущей активности пользователей, прогнозируемых сумм и времени операций, используя цикл по строкам таблицы.

---

## 1. Подготовка данных (Excel)

Файл `MobileBank_Transactions_Feb.xlsx`, лист `Transactions_Feb` с дополнительными полями:

| Поле            | Тип     | Смысл                                     |
| --------------- | ------- | ----------------------------------------- |
| TransactionID   | string  | Уникальный идентификатор транзакции       |
| UserID          | string  | Идентификатор клиента                     |
| TransactionDate | date    | Дата и время проведения транзакции        |
| Amount          | decimal | Сумма операции, тыс. $                    |
| TransactionType | string  | Тип операции (Transfer, Payment, Deposit) |
| AccountBalance  | decimal | Баланс после транзакции                   |
| GeoLocation     | string  | Геолокация                                |
| IsInternational | string  | Да/Нет                                    |
| ForecastAmount  | decimal | Прогнозируемая сумма по типу операции     |
| Notes           | string  | Комментарии                               |

**Методический комментарий:** прогнозируемая сумма позволяет оценить потенциальный риск превышения лимитов и подозрительных операций.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик                    | Начальное значение | Смысл                                        |
| -------------------------- | ------------------ | -------------------------------------------- |
| total_transactions         | 0                  | Общее количество транзакций                  |
| high_amount_transactions   | 0                  | Транзакции с суммой выше лимита              |
| international_transactions | 0                  | Международные операции                       |
| night_transactions         | 0                  | Транзакции вне стандартного рабочего времени |
| total_amount               | 0                  | Суммарная сумма всех транзакций              |
| total_forecast_risk        | 0                  | Суммарный прогнозный риск транзакций         |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы данных:

* `amount, accountbalance, forecastamount → decimal`
* `transactiondate → date`
* `transactiontype, userid, transactionid, geolocation, isinternational → удалить пробелы / верхний регистр`

2. Вычислить показатели:

```
hour_of_day = transactiondate.hour
is_night = hour_of_day < 6 OR hour_of_day > 22
forecast_gap = amount - forecastamount
```

* `forecast_gap` показывает отклонение факта от прогнозной суммы.

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

```
if amount > 500 OR (isinternational == "YES" AND amount > 200):
    risk_category = "Critical"
    total_forecast_risk += 3
elif amount > 200 OR isinternational == "YES" OR is_night:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif amount > 100 OR forecast_gap > 50:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** простые условия фиксируют отклонения суммы и времени, составные — международные операции и превышения прогнозов, вложенные — формируют итоговую категорию и баллы суммируются в `total_forecast_risk`.

---

## 5. Формирование результирующего Excel (Excel)

Файл `MobileBank_Analysis_Result.xlsx`, лист `Analysis_Feb` с полями:

| TransactionID | UserID | Amount | ForecastAmount | TransactionType | AccountBalance | GeoLocation | IsInternational | ForecastGap | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Прогнозный анализ транзакций за февраль завершён.

Всего транзакций: {total_transactions}
С высокой суммой: {high_amount_transactions}
Международные: {international_transactions}
Ночные операции: {night_transactions}
Суммарная сумма транзакций: {total_amount} тыс. $
Суммарный прогнозный риск: {total_forecast_risk} баллов

Отчёт сохранён в MobileBank_Analysis_Result.xlsx (лист Analysis_Feb).
"""
```

---
