# Вариант 27

**Тематика:** Анализ финансовой активности и прогнозирование операционного риска в приложении для управления финансами компаний

---

## Контекст задачи

Приложение для управления финансами компаний собирает данные о поступлениях и расходах, финансовых операциях, бюджетах подразделений и исполнении ключевых показателей (KPI). Финансовые аналитики и руководство нуждаются в автоматизированном мониторинге финансового здоровья, выявлении ненормальных операций, рисков перерасхода бюджета и прогнозировании угроз ликвидности.

Типичные проблемы:

* перерасход бюджета подразделений;
* задержки поступлений;
* частые корректировки финансовых планов;
* низкая маржинальность операций;
* риск отклонения от финансовых KPI.

**Цель кейса:** реализовать RPA-процесс, который анализирует финансовую активность за период, рассчитывает ключевые финансовые показатели, прогнозирует риск нарушения бюджета, формирует Excel-отчёт и выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ финансовой активности за квартал (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `FinanceApp_Q1.xlsx`, лист `Transactions_Q1`.

**Структура таблицы:**

| Поле            | Тип    | Смысл                                                           |
| --------------- | ------ | --------------------------------------------------------------- |
| TransactionID   | string | Уникальный идентификатор операции                               |
| CompanyID       | string | Идентификатор компании                                          |
| Department      | string | Подразделение                                                   |
| TransactionDate | date   | Дата операции                                                   |
| Amount          | number | Сумма операции (положительная — приход, отрицательная — расход) |
| Category        | string | Категория операции (Revenue, Expense, Adjustment)               |
| BudgetedAmount  | number | Бюджетная сумма для категории и подразделения                   |
| ActualBalance   | number | Текущий остаток по счетам компании                              |

---

## 2. Инициализация переменных

| Счётчик                | Начальное значение |
| ---------------------- | ------------------ |
| total_transactions     | 0                  |
| total_revenue          | 0                  |
| total_expenses         | 0                  |
| overbudget_departments | 0                  |
| negative_balance_cases | 0                  |
| total_budget_deviation | 0                  |

---

## 3. Преобразование данных

* Amount, BudgetedAmount, ActualBalance → number
* TransactionDate → date
* TransactionID, CompanyID → удалить пробелы, верхний регистр

---

## 4. Расчёты (на каждой строке)

```
budget_deviation = Amount - BudgetedAmount
```

Если `Category = "Revenue"` →

```
total_revenue += Amount
```

Если `Category = "Expense"` →

```
total_expenses += abs(Amount)
```

---

## 5. Логика классификации

* если `budget_deviation > 0 AND Category = "Expense"` →
  `overbudget_departments += 1`
  `total_budget_deviation += budget_deviation`

* если `ActualBalance < 0` →
  `negative_balance_cases += 1`

Накопление:

```
total_transactions += 1
```

---

## 6. Итоговый Excel

Файл `FinanceApp_Result.xlsx`, лист `Q1_Analysis`:

| TransactionID | Department | BudgetDeviation | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ финансовой активности Q1 завершён.

Всего операций: {total_transactions}
Суммарная выручка: {total_revenue}
Суммарные расходы: {total_expenses}

Подразделения с перерасходом: {overbudget_departments}
Случаи отрицательного баланса: {negative_balance_cases}
Суммарное отклонение бюджета: {total_budget_deviation}

Отчёт сохранён в FinanceApp_Result.xlsx (лист Q1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ с прогнозом риска нарушения бюджета

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `FinanceApp_Q2.xlsx`, лист `Transactions_Q2`.

---

## 2. Инициализация переменных

| Переменная                | Начальное значение |
| ------------------------- | ------------------ |
| total_transactions        | 0                  |
| high_risk_departments     | 0                  |
| critical_risk_departments | 0                  |
| total_revenue             | 0                  |
| total_expenses            | 0                  |
| total_budget_deviation    | 0                  |
| total_forecast_risk       | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
budget_deviation = Amount - BudgetedAmount
abs_deviation = abs(budget_deviation)
expense_ratio = abs(Amount) / (BudgetedAmount + 1)
balance_penalty = 
    если ActualBalance < 0 → abs(ActualBalance) * 2
    иначе → 0

category_factor = 
    если Category = "Revenue" → -500
    если Category = "Expense" → 500
    иначе → 0

forecast_financial_risk = (abs_deviation * expense_ratio) + balance_penalty + category_factor
```

---

## 4. Логика классификации риска

* если `forecast_financial_risk > 2000` →
  `risk_category = "Critical Risk"`
  `critical_risk_departments += 1`

* если `forecast_financial_risk > 1000` →
  `risk_category = "High Risk"`
  `high_risk_departments += 1`

* иначе →
  `risk_category = "Stable"`

После классификации:

```
total_forecast_risk += forecast_financial_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная метрика финансового риска нарушения бюджета по всем операциям за период.

```
total_forecast_risk = Σ forecast_financial_risk
```

### Интерпретация:

* низкое значение → финансовые операции укладываются в плановые рамки;
* среднее → умеренный риск перерасхода и отрицательных балансов;
* высокое → высокий системный риск нарушения бюджета.

Метрика позволяет:

* мониторить финансовую устойчивость;
* выявлять проблемные подразделения;
* корректировать бюджетные планы;
* предлагать автоматизированные рекомендации финансовым менеджерам.

---

## 5. Итоговый Excel

Файл `FinanceApp_Result.xlsx`, лист `Q2_Analysis`:

| TransactionID | Department | ForecastFinancialRisk | RiskCategory |

---

## 6. Итоговое уведомление

```
f"""Расширенный анализ финансов завершён.

Всего операций: {total_transactions}
Суммарная выручка: {total_revenue}
Суммарные расходы: {total_expenses}

Высокий риск: {high_risk_departments}
Критический риск: {critical_risk_departments}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в FinanceApp_Result.xlsx (лист Q2_Analysis).
"""
```
