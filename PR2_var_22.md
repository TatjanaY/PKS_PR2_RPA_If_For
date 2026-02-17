# Вариант 22

**Тематика:** Анализ операций и прогнозирование финансово-операционного риска в сервисе аренды автомобилей

---

## Контекст задачи

Сервис аренды автомобилей предоставляет краткосрочную и среднесрочную аренду транспортных средств. Платформа фиксирует бронирования, фактическое время возврата, пробег, повреждения и дополнительные услуги.

Типовые риски сервиса:

* просроченный возврат автомобиля;
* превышение лимита пробега;
* высокий риск страховых выплат;
* убытки из-за повреждений;
* перегрузка автопарка в пиковые даты.

Необходимо реализовать RPA-процесс, который:

* анализирует аренды за период;
* рассчитывает отклонения по срокам и пробегу;
* прогнозирует риск убыточности аренды;
* формирует Excel-отчёт;
* рассчитывает агрегированный показатель `total_forecast_risk`.

---

# Задание 1. Анализ аренд за месяц (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `CarRental_Month1.xlsx`, лист `Rentals`.

**Структура таблицы:**

| Поле              | Тип    | Смысл                           |
| ----------------- | ------ | ------------------------------- |
| RentalID          | string | ID аренды                       |
| CustomerID        | string | ID клиента                      |
| CarID             | string | ID автомобиля                   |
| CarCategory       | string | Economy, Standard, Premium, SUV |
| RentalStartDate   | date   | Дата начала аренды              |
| PlannedReturnDate | date   | Плановая дата возврата          |
| ActualReturnDate  | date   | Фактическая дата возврата       |
| PlannedMileage    | number | Планируемый пробег (км)         |
| ActualMileage     | number | Фактический пробег (км)         |
| RentalFee         | number | Стоимость аренды                |
| DamageReported    | string | Yes / No                        |

---

## 2. Инициализация переменных

| Переменная           | Начальное значение |
| -------------------- | ------------------ |
| total_rentals        | 0                  |
| late_returns         | 0                  |
| mileage_excess_cases | 0                  |
| damage_cases         | 0                  |
| total_late_days      | 0                  |
| total_extra_mileage  | 0                  |
| total_damage_risk    | 0                  |

---

## 3. Преобразование данных

* PlannedMileage, ActualMileage, RentalFee → number
* RentalStartDate, PlannedReturnDate, ActualReturnDate → date
* RentalID, CustomerID, CarID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (на каждой строке)

```
planned_duration = PlannedReturnDate - RentalStartDate
actual_duration = ActualReturnDate - RentalStartDate
delay_days = actual_duration - planned_duration

mileage_difference = ActualMileage - PlannedMileage
daily_revenue = RentalFee / planned_duration
```

---

## 5. Логика анализа

* если `delay_days > 0` →
  `late_returns += 1`
  `total_late_days += delay_days`

* если `mileage_difference > 0` →
  `mileage_excess_cases += 1`
  `total_extra_mileage += mileage_difference`

* если `DamageReported = "Yes"` →
  `damage_cases += 1`
  `total_damage_risk += RentalFee * 0.3`

---

## 6. Итоговый Excel

Файл `CarRental_Result.xlsx`, лист `Month1_Analysis`:

| RentalID | DelayDays | MileageDifference | DamageFlag | OperationalCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ аренд завершён.

Всего аренд: {total_rentals}
Просроченных возвратов: {late_returns}
Случаев превышения пробега: {mileage_excess_cases}
Случаев повреждений: {damage_cases}

Суммарные дни просрочки: {total_late_days}
Дополнительный пробег (км): {total_extra_mileage}
Потенциальный риск ущерба: {total_damage_risk}

Отчёт сохранён в CarRental_Result.xlsx (лист Month1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ с прогнозом финансового риска

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `CarRental_Month2.xlsx`, лист `Rentals_Month2`.

---

## 2. Инициализация переменных

| Переменная            | Начальное значение |
| --------------------- | ------------------ |
| total_rentals         | 0                  |
| late_returns          | 0                  |
| mileage_excess_cases  | 0                  |
| damage_cases          | 0                  |
| high_risk_rentals     | 0                  |
| critical_risk_rentals | 0                  |
| total_forecast_risk   | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой аренды:

```
planned_duration = PlannedReturnDate - RentalStartDate
actual_duration = ActualReturnDate - RentalStartDate
delay_days = actual_duration - planned_duration

mileage_difference = ActualMileage - PlannedMileage
delay_factor = delay_days * 150
mileage_factor = mileage_difference * 2
damage_factor = 0

если DamageReported = "Yes" →
    damage_factor = RentalFee * 0.5

category_factor = 
    если CarCategory = "Premium" → 300
    если CarCategory = "SUV" → 200
    иначе → 100

forecast_financial_risk = delay_factor + mileage_factor + damage_factor + category_factor
```

---

## 4. Логика классификации риска

* если `forecast_financial_risk > 5000` →
  `risk_category = "Critical Risk"`
  `critical_risk_rentals += 1`

* если `forecast_financial_risk > 2500` →
  `risk_category = "High Risk"`
  `high_risk_rentals += 1`

* иначе →
  `risk_category = "Normal"`

После классификации:

```
total_forecast_risk += forecast_financial_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — это суммарная прогнозная финансовая нагрузка по всем арендам за период.

```
total_forecast_risk = Σ forecast_financial_risk
```

### Интерпретация:

* низкое значение → автопарк используется эффективно, отклонения минимальны;
* среднее → есть локальные риски перерасхода и ущерба;
* высокое → системная финансовая нагрузка (просрочки, повреждения, износ автопарка).

Метрика позволяет:

* оценивать устойчивость бизнес-модели;
* корректировать штрафную политику;
* планировать обновление автопарка;
* выявлять проблемные категории автомобилей.

---

## 5. Итоговый Excel

Файл `CarRental_Result.xlsx`, лист `Month2_Analysis`:

| RentalID | DelayDays | MileageDifference | ForecastFinancialRisk | RiskCategory |

---

## 6. Итоговое уведомление

```
f"""Расширенный анализ аренд завершён.

Всего аренд: {total_rentals}
Высокий риск: {high_risk_rentals}
Критический риск: {critical_risk_rentals}

Суммарный прогнозный финансовый риск: {total_forecast_risk}

Отчёт сформирован в CarRental_Result.xlsx (лист Month2_Analysis).
"""
```
