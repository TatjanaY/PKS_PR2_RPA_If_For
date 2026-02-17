# Вариант 31

**Тематика:** Анализ активности игроков и прогнозирование риска снижения вовлечённости на онлайн-игровой платформе

---

## Контекст задачи

Онлайн-игровая платформа объединяет сотни тысяч игроков и десятки игровых проектов. Пользователи проводят время в играх, выполняют задания, участвуют в событиях и покупают внутриигровые товары.

Типичные проблемы:

* часть игроков теряет интерес и перестаёт заходить в игру;
* низкая активность в новых событиях или миссиях;
* высокие показатели незавершённых квестов;
* требуется прогнозирование риска снижения вовлечённости и оттока игроков.

Необходимо реализовать RPA-процесс, который:

* собирает данные о действиях игроков за период;
* рассчитывает показатели вовлечённости;
* прогнозирует риск снижения активности;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ активности игроков за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `GamePlatform_Week1.xlsx`, лист `PlayerActivity`.

### Структура таблицы:

| Поле            | Тип     | Смысл                          |
| --------------- | ------- | ------------------------------ |
| PlayerID        | string  | Идентификатор игрока           |
| LogDate         | date    | Дата активности                |
| GameID          | string  | Идентификатор игры             |
| SessionsPlayed  | integer | Количество игровых сессий      |
| QuestsCompleted | integer | Завершённые квесты             |
| InGamePurchases | number  | Потрачено внутриигровой валюты |
| PlayTimeMinutes | number  | Время в игре (минуты)          |

---

## 2. Инициализация переменных

| Счётчик             | Начальное значение |
| ------------------- | ------------------ |
| total_logs          | 0                  |
| low_activity_days   | 0                  |
| no_quests_days      | 0                  |
| zero_spending_days  | 0                  |
| total_playtime      | 0                  |
| total_forecast_risk | 0                  |

---

## 3. Преобразование данных

* SessionsPlayed, QuestsCompleted, InGamePurchases, PlayTimeMinutes → number
* LogDate → date
* PlayerID, GameID → удалить пробелы, верхний регистр

---

## 4. Расчёты (по каждой строке)

```
activity_deficit = max(0, 2 - SessionsPlayed)
quest_deficit = max(0, 1 - QuestsCompleted)
spending_penalty = 0 if InGamePurchases > 0 else 10
playtime_deficit = max(0, 60 - PlayTimeMinutes)

forecast_engagement_risk = activity_deficit*10 + quest_deficit*15 + spending_penalty + playtime_deficit*0.5
```

Если SessionsPlayed < 2 → `low_activity_days += 1`
Если QuestsCompleted = 0 → `no_quests_days += 1`
Если InGamePurchases = 0 → `zero_spending_days += 1`

Накопление:

```
total_playtime += PlayTimeMinutes
total_logs += 1
total_forecast_risk += forecast_engagement_risk
```

---

## 5. Классификация

* если `forecast_engagement_risk > 50` → `"High Risk"`
* если `forecast_engagement_risk > 25` → `"Moderate Risk"`
* иначе → `"Stable"`

---

## 6. Итоговый Excel

Файл `GamePlatform_Result.xlsx`, лист `Week1_Analysis`:

| PlayerID | LogDate | SessionsPlayed | QuestsCompleted | InGamePurchases | PlayTimeMinutes | RiskCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ активности игроков за неделю завершён.

Всего записей: {total_logs}
Дней с низкой активностью: {low_activity_days}
Дней без квестов: {no_quests_days}
Дней без внутриигровых покупок: {zero_spending_days}

Суммарное время в игре: {total_playtime}
Суммарный прогнозный риск снижения вовлечённости: {total_forecast_risk}

Отчёт сохранён в GamePlatform_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска оттока игроков

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `GamePlatform_Month.xlsx`, лист `PlayerActivity_Month`.

---

## 2. Инициализация переменных

| Переменная          | Начальное значение |
| ------------------- | ------------------ |
| total_logs          | 0                  |
| high_risk_days      | 0                  |
| moderate_risk_days  | 0                  |
| total_playtime      | 0                  |
| total_forecast_risk | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
activity_deficit = max(0, 2 - SessionsPlayed)
quest_deficit = max(0, 1 - QuestsCompleted)
spending_penalty = 0 if InGamePurchases > 0 else 15
playtime_deficit = max(0, 60 - PlayTimeMinutes)
engagement_trend_factor = (SessionsPlayed + QuestsCompleted)/2

forecast_engagement_risk = activity_deficit*10 + quest_deficit*15 + spending_penalty + playtime_deficit*0.5 - engagement_trend_factor*2
```

---

## 4. Логика классификации риска

* если `forecast_engagement_risk > 50` →
  `risk_category = "High Risk"`
  `high_risk_days += 1`

* если `forecast_engagement_risk > 25` →
  `risk_category = "Moderate Risk"`
  `moderate_risk_days += 1`

* иначе →
  `risk_category = "Stable"`

Накопление:

```
total_forecast_risk += forecast_engagement_risk
total_playtime += PlayTimeMinutes
total_logs += 1
```

---

## 5. Итоговый Excel

Файл `GamePlatform_Result.xlsx`, лист `Month_Analysis`:

| PlayerID | LogDate | SessionsPlayed | QuestsCompleted | InGamePurchases | PlayTimeMinutes | ForecastEngagementRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ активности игроков завершён.

Всего записей: {total_logs}
Дней с умеренным риском: {moderate_risk_days}
Дней с высоким риском: {high_risk_days}

Суммарное время в игре: {total_playtime}
Суммарный прогнозный риск оттока игроков: {total_forecast_risk}

Отчёт сформирован в GamePlatform_Result.xlsx (лист Month_Analysis).
"""
```
