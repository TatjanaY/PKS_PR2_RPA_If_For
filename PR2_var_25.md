# Вариант 25

**Тематика:** Анализ активности и прогнозирование риска снижения вовлечённости на платформе для фрилансеров

---

## Контекст задачи

Платформа для фрилансеров объединяет независимых специалистов и заказчиков. Фрилансеры получают задания, выполняют их, получают отзывы и рейтинги, а платформа анализирует их активность, качество выполнения и удержание на платформе.

Типичные проблемы:

* часть фрилансеров работает нерегулярно;
* низкие оценки клиентов ухудшают рейтинг;
* большое количество отмененных предложений;
* снижение вовлечённости приводит к оттоку пользователей;
* нет агрегированной оценки риска снижения активности.

**Цель кейса:** реализовать RPA-процесс, который анализирует активность фрилансеров за период, рассчитывает ключевые показатели вовлечённости и качества, прогнозирует риск снижения вовлечённости, формирует Excel-отчёт и выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ активности фрилансеров за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `FreelanceActivity_Week1.xlsx`, лист `ActivityLog`.

### Структура таблицы:

| Поле            | Тип    | Смысл                               |
| --------------- | ------ | ----------------------------------- |
| FreelancerID    | string | Уникальный идентификатор фрилансера |
| ActivityDate    | date   | Дата активности                     |
| TasksCompleted  | number | Количество выполненных задач        |
| OffersMade      | number | Количество сделанных предложений    |
| OffersAccepted  | number | Количество принятых предложений     |
| CancelledOffers | number | Количество отменённых предложений   |
| TotalEarnings   | number | Общий заработок за день             |
| ClientRating    | number | Средний рейтинг от клиентов (1–5)   |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик                | Начальное значение |
| ---------------------- | ------------------ |
| total_records          | 0                  |
| total_tasks            | 0                  |
| total_offers           | 0                  |
| total_cancelled_offers | 0                  |
| low_rating_days        | 0                  |
| low_acceptance_days    | 0                  |

---

## 3. Преобразование данных

* TasksCompleted, OffersMade, OffersAccepted, CancelledOffers, TotalEarnings, ClientRating → number
* ActivityDate → date
* FreelancerID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (по каждой строке)

```
acceptance_rate = OffersAccepted / OffersMade
task_to_offer_ratio = TasksCompleted / OffersMade
```

Если `OffersMade = 0`, считать `acceptance_rate = 0`, `task_to_offer_ratio = 0`.

---

## 5. Логика классификации

* если `ClientRating < 3` →
  `low_rating_days += 1`

* если `acceptance_rate < 0.3` →
  `low_acceptance_days += 1`

Накопление:

```
total_tasks += TasksCompleted
total_offers += OffersMade
total_cancelled_offers += CancelledOffers
```

---

## 6. Итоговый Excel

Файл `FreelanceActivity_Result.xlsx`, лист `Week1_Analysis`:

| FreelancerID | ActivityDate | AcceptanceRate | TaskToOfferRatio | EngagementCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ активности фрилансеров за неделю завершён.

Всего записей: {total_records}
Дней с низким рейтингом: {low_rating_days}
Дней с низким коэффициентом принятия предложений: {low_acceptance_days}
Суммарные выполненные задачи: {total_tasks}
Суммарные предложения: {total_offers}
Суммарные отмены предложений: {total_cancelled_offers}

Отчёт сохранён в FreelanceActivity_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска снижения вовлечённости

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `FreelanceActivity_Month.xlsx`, лист `ActivityLog_Month`.

---

## 2. Инициализация переменных

| Переменная          | Начальное значение |
| ------------------- | ------------------ |
| total_records       | 0                  |
| low_risk_users      | 0                  |
| high_risk_users     | 0                  |
| critical_risk_users | 0                  |
| total_forecast_risk | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой строки:

```
acceptance_rate = OffersAccepted / OffersMade
task_to_offer_ratio = TasksCompleted / OffersMade
rating_penalty = (5 - ClientRating) * 100
offer_cancellation_penalty = CancelledOffers * 50
productivity_index = TasksCompleted * 10

forecast_engagement_risk = rating_penalty + offer_cancellation_penalty - (acceptance_rate * 200) - (task_to_offer_ratio * 100) - productivity_index
```

Если переменные отрицательные → риск = 0.

---

## 4. Логика классификации риска

* если `forecast_engagement_risk > 3000` →
  `risk_category = "Critical Risk"`
  `critical_risk_users += 1`

* если `forecast_engagement_risk > 1500` →
  `risk_category = "High Risk"`
  `high_risk_users += 1`

* иначе →
  `risk_category = "Low Risk"`
  `low_risk_users += 1`

После классификации:

```
total_forecast_risk += forecast_engagement_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная метрика риска снижения вовлечённости по всем фрилансерам за месяц.

```
total_forecast_risk = Σ forecast_engagement_risk
```

### Интерпретация:

* низкое значение → большая часть фрилансеров успешно выполняет задания;
* среднее → часть аудитории демонстрирует снижение активности;
* высокое → высокий риск оттока и снижения числа выполненных заказов.

Метрика позволяет:

* выявлять фрилансеров группы риска;
* давать рекомендации по повышению вовлечённости;
* корректировать алгоритмы подбора заказов;
* планировать маркетинговые кампании.

---

## 5. Итоговый Excel

Файл `FreelanceActivity_Result.xlsx`, лист `Month_Analysis`:

| FreelancerID | ActivityDate | ForecastEngagementRisk | RiskCategory |

---

## 6. Итоговое уведомление

```
f"""Расширенный анализ активности фрилансеров завершён.

Всего записей: {total_records}
Пользователей с низким риском: {low_risk_users}
Пользователей с высоким риском: {high_risk_users}
Пользователей с критическим риском: {critical_risk_users}
Суммарный прогнозный риск вовлечённости: {total_forecast_risk}

Отчёт сформирован в FreelanceActivity_Result.xlsx (лист Month_Analysis).
"""
```
