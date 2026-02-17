# Вариант 15

**Тематика:** Приложение для медитации — анализ сессий пользователей и прогноз снижения вовлеченности

---

## Контекст задачи

Приложение фиксирует данные о медитационных сессиях пользователей: длительность, тип сессии, оценки состояния после сессии и частоту пропусков.

Менеджмент сталкивается с типичными проблемами:

* отдельные пользователи пропускают сессии или проводят короткие медитации;
* часть сессий имеет низкие оценки состояния после завершения;
* необходимо выявлять пользователей с повышенным риском снижения вовлеченности;
* прогнозировать, у кого вероятно падение активности в будущем.

**Цель кейса:** реализовать RPA-процесс, который автоматически анализирует сессии пользователей, рассчитывает ключевые показатели вовлеченности, классифицирует пользователей по уровню риска снижения активности и формирует итоговый отчёт в Excel с уведомлением в модальном окне.

---

# Задание 1. Анализ медитационных сессий за март (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Вы — аналитик продукта. Необходимо оценить активность пользователей, выявить короткие сессии и низкие оценки состояния, чтобы подготовить сводку для команды по улучшению вовлеченности.

---

## 1. Подготовка данных (Excel)

Создать файл `Meditation_Sessions_March.xlsx` с листом `Sessions_March`.

**Структура таблицы:**

| Поле            | Тип     | Смысл                                    |
| --------------- | ------- | ---------------------------------------- |
| SessionID       | string  | Уникальный идентификатор сессии          |
| UserID          | string  | Идентификатор пользователя               |
| SessionDate     | date    | Дата и время сессии                      |
| SessionType     | string  | Тип сессии (Guided, Unguided, Breathing) |
| PlannedDuration | decimal | Плановая длительность сессии в минутах   |
| ActualDuration  | decimal | Фактическая длительность сессии          |
| MoodRating      | decimal | Оценка состояния пользователя 1–5        |
| Skipped         | string  | Пропуск сессии: YES/NO                   |

**Методический комментарий:** таблица отражает фактическую активность пользователей и их самочувствие после сессий.

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                                    |
| ------------------- | ------------------ | ---------------------------------------- |
| total_sessions      | 0                  | Общее количество сессий                  |
| short_sessions      | 0                  | Сессии короче плановой длительности      |
| low_mood_sessions   | 0                  | Сессии с низкой оценкой состояния (<3)   |
| skipped_sessions    | 0                  | Пропущенные сессии                       |
| total_extra_minutes | 0                  | Суммарное отклонение по времени          |
| total_forecast_risk | 0                  | Суммарный прогноз снижения вовлеченности |

**Методический комментарий:** переменная `total_forecast_risk` суммирует баллы риска пользователей:

* `Critical` → +3
* `High Risk` → +2
* `At Risk` → +1
* `Stable` → 0

---

## 3. Преобразование данных (Puzzle RPA, на каждой строке таблицы)

* `plannedduration, actualduration, moodrating → decimal`
* `sessiondate → date`
* `sessionid, userid, sessiontype, skipped → удалить пробелы / верхний регистр`

**Методический комментарий:** корректное приведение типов обеспечивает точность вычислений и корректное сравнение длительности, оценок и пропусков.

---

## 4. Расчёты (Puzzle RPA, на каждой строке таблицы)

```
duration_diff = actualduration - plannedduration
```

* `duration_diff` — отклонение длительности сессии в минутах;

---

## 5. Логика классификации сессий (Puzzle RPA, на каждой строке таблицы)

**Простые условия:**

```
if duration_diff < 0:
    short_sessions += 1
    total_extra_minutes += abs(duration_diff)
```

```
if moodrating < 3:
    low_mood_sessions += 1
```

```
if skipped == "YES":
    skipped_sessions += 1
```

**Составные условия:**

```
if duration_diff < -5 OR moodrating < 3 OR skipped == "YES":
    risk_category = "At Risk"
else:
    risk_category = "Stable"
```

**Вложенные условия:**

```
if duration_diff < -15 OR moodrating < 2 OR skipped == "YES":
    risk_category = "Critical"
    total_forecast_risk += 3
elif duration_diff < -10 OR moodrating < 3:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif duration_diff < -5:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** простые условия фиксируют короткие сессии и пропуски, составные — совокупные отклонения, вложенные — итоговая категория риска и суммирование баллов в `total_forecast_risk`.

---

## 6. Формирование аналитической таблицы (Excel)

Файл `Meditation_Sessions_Result.xlsx`, лист `Analysis_March` с полями:

| SessionID | UserID | SessionType | DurationDiff | MoodRating | Skipped | RiskCategory |

---

## 7. Итоговое уведомление (Puzzle RPA, модальное окно)

```
f"""Анализ сессий за март завершён.

Всего сессий: {total_sessions}
Коротких сессий: {short_sessions}
С низкой оценкой состояния: {low_mood_sessions}
Пропущенных сессий: {skipped_sessions}
Суммарное отклонение длительности: {total_extra_minutes} мин.
Суммарный прогноз снижения вовлеченности: {total_forecast_risk} баллов

Отчёт сохранён в Meditation_Sessions_Result.xlsx (лист Analysis_March).
"""
```

---

# Задание 2. Прогнозная оценка сессий за апрель (расширенный уровень)

**Выполняется в Puzzle RPA и Excel.**

**История:** Во втором месяце необходимо оценить потенциальный риск снижения вовлеченности каждого пользователя с учётом текущих и прогнозируемых длительностей сессий и оценок состояния, используя цикл по строкам таблицы.

---

## 1. Подготовка данных (Excel)

Файл `Meditation_Sessions_April.xlsx`, лист `Sessions_April` с дополнительными полями:

| Поле             | Тип     | Смысл                                    |
| ---------------- | ------- | ---------------------------------------- |
| SessionID        | string  | Уникальный идентификатор сессии          |
| UserID           | string  | Идентификатор пользователя               |
| SessionDate      | date    | Дата и время сессии                      |
| SessionType      | string  | Тип сессии (Guided, Unguided, Breathing) |
| PlannedDuration  | decimal | Плановая длительность сессии             |
| ActualDuration   | decimal | Фактическая длительность сессии          |
| ForecastDuration | decimal | Прогнозируемая длительность сессии       |
| MoodRating       | decimal | Оценка состояния пользователя            |
| ForecastMood     | decimal | Прогнозируемая оценка состояния          |
| Skipped          | string  | Пропуск сессии: YES/NO                   |
| Notes            | string  | Комментарии                              |

---

## 2. Инициализация переменных и счётчиков (Puzzle RPA)

| Счётчик             | Начальное значение | Смысл                                    |
| ------------------- | ------------------ | ---------------------------------------- |
| total_sessions      | 0                  | Общее количество сессий                  |
| short_sessions      | 0                  | Сессии короче плановой длительности      |
| low_mood_sessions   | 0                  | Сессии с низкой оценкой состояния        |
| skipped_sessions    | 0                  | Пропущенные сессии                       |
| total_extra_minutes | 0                  | Суммарное отклонение по времени          |
| total_forecast_risk | 0                  | Суммарный прогноз снижения вовлеченности |

---

## 3. Цикл по строкам таблицы (Puzzle RPA)

На каждой итерации:

1. Привести типы данных:

* `plannedduration, actualduration, forecastduration, moodrating, forecastmood → decimal`
* `sessiondate → date`
* `sessionid, userid, sessiontype, skipped → удалить пробелы / верхний регистр`

2. Вычислить:

```
duration_diff = actualduration - plannedduration
forecast_duration_diff = forecastduration - actualduration
mood_diff = moodrating - forecastmood
```

---

## 4. Многоуровневая логика прогнозного риска (Puzzle RPA, на каждой строке)

```
if duration_diff < -15 OR moodrating < 2 OR forecastmood < 2 OR skipped == "YES":
    risk_category = "Critical"
    total_forecast_risk += 3
elif duration_diff < -10 OR moodrating < 3 OR forecastmood < 3:
    risk_category = "High Risk"
    total_forecast_risk += 2
elif duration_diff < -5 OR forecast_duration_diff < -5:
    risk_category = "At Risk"
    total_forecast_risk += 1
else:
    risk_category = "Stable"
```

**Методический комментарий:** выполняется для каждой строки. Простые условия фиксируют короткие сессии и пропуски, составные — прогнозные отклонения, вложенные — итоговая категория и суммирование баллов риска.

---

## 5. Формирование результирующего Excel (Excel)

Файл `Meditation_Sessions_Result.xlsx`, лист `Analysis_April` с полями:

| SessionID | UserID | SessionType | DurationDiff | ForecastDurationDiff | MoodRating | ForecastMood | Skipped | RiskCategory |

---

## 6. Итоговое модальное уведомление (Puzzle RPA)

```
f"""Прогнозный анализ сессий за апрель завершён.

Всего сессий: {total_sessions}
Коротких сессий: {short_sessions}
С низкой оценкой состояния: {low_mood_sessions}
Пропущенных сессий: {skipped_sessions}
Суммарное отклонение длительности: {total_extra_minutes} мин.
Суммарный прогноз снижения вовлеченности: {total_forecast_risk} баллов

Отчёт сохранён в Meditation_Sessions_Result.xlsx (лист Analysis_April).
"""
```

---