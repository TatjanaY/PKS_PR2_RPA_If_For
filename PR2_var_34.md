# Вариант 34

**Тематика:** Анализ кампаний и прогнозирование эффективности в платформе онлайн-рекламы

---

## Контекст задачи

Платформа онлайн-рекламы позволяет рекламодателям создавать кампании, задавать бюджеты, таргетировать аудитории и отслеживать показатели эффективности. Аналитика платформы направлена на:

* выявление низкоэффективных кампаний;
* оптимизацию бюджета;
* прогнозирование риска недостижения целевых KPI;
* автоматическую классификацию кампаний по уровню риска.

Необходимо реализовать RPA-процесс, который:

* собирает данные по кампаниям и их показателям;
* рассчитывает показатели эффективности (CTR, конверсии, расход бюджета);
* прогнозирует риск неэффективности;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ кампаний за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `AdCampaigns_Week1.xlsx`, лист `CampaignLog`.

### Структура таблицы:

| Поле            | Тип     | Смысл                             |
| --------------- | ------- | --------------------------------- |
| CampaignID      | string  | Уникальный идентификатор кампании |
| AdvertiserID    | string  | Идентификатор рекламодателя       |
| StartDate       | date    | Дата начала кампании              |
| EndDate         | date    | Дата окончания кампании           |
| Impressions     | integer | Количество показов объявлений     |
| Clicks          | integer | Количество кликов                 |
| Conversions     | integer | Количество конверсий              |
| BudgetAllocated | number  | Выделенный бюджет                 |
| BudgetSpent     | number  | Потраченный бюджет                |
| TargetCTR       | number  | Целевой CTR (%)                   |
| TargetCPC       | number  | Целевая стоимость клика           |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик                   | Начальное значение | Смысл                                     |
| ------------------------- | ------------------ | ----------------------------------------- |
| total_campaigns           | 0                  | Всего кампаний                            |
| underperforming_campaigns | 0                  | Кампании ниже KPI                         |
| budget_overruns           | 0                  | Кампании с перерасходом бюджета           |
| total_clicks              | 0                  | Суммарное количество кликов               |
| total_conversions         | 0                  | Суммарное количество конверсий            |
| total_forecast_risk       | 0                  | Суммарный прогнозный риск неэффективности |

---

## 3. Преобразование данных

* Impressions, Clicks, Conversions → integer
* BudgetAllocated, BudgetSpent, TargetCTR, TargetCPC → number
* StartDate, EndDate → date
* CampaignID, AdvertiserID → удалить пробелы, верхний регистр

---

## 4. Расчёты (по каждой строке)

* `CTR = Clicks / Impressions * 100`
* `CPC = BudgetSpent / max(Clicks, 1)`
* `budget_usage = BudgetSpent / BudgetAllocated * 100`

Прогнозный риск неэффективности:

```
forecast_risk = max(0, TargetCTR - CTR) + max(0, CPC - TargetCPC) + max(0, budget_usage - 100)
total_forecast_risk += forecast_risk
```

---

## 5. Классификация

* если `forecast_risk > 50` → статус `"Critical Risk"`
* если `forecast_risk > 25` → статус `"High Risk"`
* иначе → `"On Track"`

Если `CTR < TargetCTR OR CPC > TargetCPC` → `underperforming_campaigns += 1`
Если `budget_usage > 100` → `budget_overruns += 1`

---

## 6. Итоговый Excel

Файл `AdCampaigns_Result.xlsx`, лист `Week1_Analysis`:

| CampaignID | AdvertiserID | CTR | CPC | BudgetUsage | ForecastRisk | Status |

---

## 7. Итоговое уведомление

```
f"""Анализ кампаний за неделю завершён.

Всего кампаний: {total_campaigns}
Кампаний ниже KPI: {underperforming_campaigns}
Кампаний с перерасходом бюджета: {budget_overruns}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сохранён в AdCampaigns_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска неэффективности

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `AdCampaigns_Month.xlsx`, лист `CampaignLog_Month`.

---

## 2. Инициализация переменных

| Переменная                | Начальное значение | Смысл                           |
| ------------------------- | ------------------ | ------------------------------- |
| total_campaigns           | 0                  | Всего кампаний                  |
| underperforming_campaigns | 0                  | Кампании ниже KPI               |
| budget_overruns           | 0                  | Кампании с перерасходом бюджета |
| total_clicks              | 0                  | Суммарное количество кликов     |
| total_conversions         | 0                  | Суммарное количество конверсий  |
| critical_risk_campaigns   | 0                  | Кампании с критическим риском   |
| high_risk_campaigns       | 0                  | Кампании с высоким риском       |
| total_forecast_risk       | 0                  | Суммарный прогнозный риск       |

---

## 3. Расчёт прогнозного риска

Для каждой кампании:

```
CTR = Clicks / Impressions * 100
CPC = BudgetSpent / max(Clicks, 1)
budget_usage = BudgetSpent / BudgetAllocated * 100
forecast_risk = max(0, TargetCTR - CTR) + max(0, CPC - TargetCPC) + max(0, budget_usage - 100)
total_forecast_risk += forecast_risk
```

---

## 4. Логика классификации риска

* если `forecast_risk > 50` → `risk_category = "Critical Risk"`, `critical_risk_campaigns += 1`
* если `forecast_risk > 25` → `risk_category = "High Risk"`, `high_risk_campaigns += 1`
* иначе → `risk_category = "On Track"`

---

## 5. Итоговый Excel

Файл `AdCampaigns_Result.xlsx`, лист `Month_Analysis`:

| CampaignID | AdvertiserID | CTR | CPC | BudgetUsage | ForecastRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ кампаний месяца завершён.

Всего кампаний: {total_campaigns}
Кампаний с высоким риском: {high_risk_campaigns}
Кампаний с критическим риском: {critical_risk_campaigns}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в AdCampaigns_Result.xlsx (лист Month_Analysis).
"""
```
