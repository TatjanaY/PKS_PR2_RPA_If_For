# Вариант 20

**Тематика:** Анализ пищевых привычек и прогнозирование риска нарушения рациона в приложении для здорового питания

---

## Контекст задачи

Мобильное приложение для здорового питания позволяет пользователям фиксировать приёмы пищи, отслеживать калорийность, баланс белков, жиров и углеводов (БЖУ), а также контролировать соблюдение дневных норм.

Платформа анализирует поведение пользователей и выявляет:

* систематическое превышение калорийности;
* дефицит белка;
* высокую долю быстрых углеводов;
* риск срыва диеты в ближайшие дни.

Необходимо реализовать RPA-процесс, который:

* анализирует данные о питании за период;
* рассчитывает показатели соблюдения норм;
* прогнозирует риск нарушения рациона;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ рациона за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `Nutrition_Week1.xlsx`, лист `FoodLog`.

**Структура таблицы:**

| Поле             | Тип    | Смысл                         |
| ---------------- | ------ | ----------------------------- |
| UserID           | string | ID пользователя               |
| LogDate          | date   | Дата записи                   |
| MealType         | string | Завтрак, Обед, Ужин, Перекус  |
| Calories         | number | Калории                       |
| ProteinGrams     | number | Белки (г)                     |
| FatGrams         | number | Жиры (г)                      |
| CarbGrams        | number | Углеводы (г)                  |
| SugarGrams       | number | Сахар (г)                     |
| DailyCalorieNorm | number | Дневная норма калорий         |
| UserGoal         | string | WeightLoss, Maintenance, Gain |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик                 | Начальное значение |
| ----------------------- | ------------------ |
| total_records           | 0                  |
| total_calories          | 0                  |
| total_protein           | 0                  |
| total_excess_days       | 0                  |
| total_low_protein_days  | 0                  |
| high_sugar_meals        | 0                  |
| total_calorie_deviation | 0                  |

---

## 3. Преобразование данных

* Calories, ProteinGrams, FatGrams, CarbGrams, SugarGrams → number
* LogDate → date
* UserID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (по каждой строке)

* `calorie_difference = Calories - DailyCalorieNorm`
* `protein_ratio = ProteinGrams / (ProteinGrams + FatGrams + CarbGrams)`
* если `SugarGrams > 25` → `high_sugar_meals += 1`

Если `calorie_difference > 0` →
`total_calorie_deviation += calorie_difference`

---

## 5. Классификация

* если `calorie_difference > 300` → статус `"Excess"`
* если `protein_ratio < 0.15` → статус `"Low Protein"`
* иначе → `"Balanced"`

---

## 6. Итоговый Excel

Файл `Nutrition_Result.xlsx`, лист `Week1_Analysis`:

| UserID | LogDate | CalorieDifference | ProteinRatio | Status |

---

## 7. Итоговое уведомление

```
f"""Анализ питания за неделю завершён.

Всего записей: {total_records}
Дней с превышением калорий: {total_excess_days}
Дней с дефицитом белка: {total_low_protein_days}
Приёмов пищи с высоким сахаром: {high_sugar_meals}
Суммарное превышение калорий: {total_calorie_deviation}

Отчёт сохранён в Nutrition_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска срыва

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `Nutrition_Month.xlsx`, лист `FoodLog_Month`.

---

## 2. Инициализация переменных

| Переменная          | Начальное значение |
| ------------------- | ------------------ |
| total_records       | 0                  |
| excess_days         | 0                  |
| low_protein_days    | 0                  |
| high_sugar_meals    | 0                  |
| high_risk_days      | 0                  |
| critical_risk_days  | 0                  |
| total_forecast_risk | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

1. Рассчитать:

```
calorie_difference = Calories - DailyCalorieNorm
protein_ratio = ProteinGrams / (ProteinGrams + FatGrams + CarbGrams)
sugar_factor = SugarGrams * 2
calorie_pressure = calorie_difference * 1.5
protein_penalty = (0.2 - protein_ratio) * 100
```

2. Рассчитать итоговый риск:

```
forecast_diet_risk = calorie_pressure + sugar_factor + protein_penalty
```

---

## 4. Логика классификации риска

* если `forecast_diet_risk > 400` →
  `risk_category = "Critical Risk"`
  `critical_risk_days += 1`

* если `forecast_diet_risk > 200` →
  `risk_category = "High Risk"`
  `high_risk_days += 1`

* иначе →
  `risk_category = "Stable"`

После классификации:

```
total_forecast_risk += forecast_diet_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная метрика суммарного риска нарушения рациона за период.

```
total_forecast_risk = Σ forecast_diet_risk
```

### Интерпретация:

* низкое значение → пользователь стабильно соблюдает рацион;
* среднее → повышенная вероятность срывов;
* высокое → системный риск нарушения режима питания и отклонения от цели.

Метрика позволяет:

* выявлять пользователей группы риска;
* запускать автоматические рекомендации;
* оценивать устойчивость пищевого поведения за период.

---

## 5. Итоговый Excel

Файл `Nutrition_Result.xlsx`, лист `Month_Analysis`:

| UserID | LogDate | ForecastDietRisk | RiskCategory |

---

## 6. Итоговое уведомление

```
f"""Расширенный анализ питания завершён.

Всего записей: {total_records}
Высокий риск: {high_risk_days}
Критический риск: {critical_risk_days}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в Nutrition_Result.xlsx (лист Month_Analysis).
"""
```
