# Вариант 24

**Тематика:** Анализ учебной активности и прогнозирование риска снижения вовлечённости в приложении для изучения языков

---

## Контекст задачи

Мобильное приложение для изучения иностранных языков фиксирует ежедневную активность пользователей: прохождение уроков, выполнение упражнений, количество ошибок, длительность сессий и непрерывность обучения (streak).

Платформа анализирует поведение пользователей и выявляет:

* снижение регулярности занятий;
* высокий процент ошибок;
* короткие учебные сессии;
* прерывание серии занятий;
* риск отказа от подписки.

Необходимо реализовать RPA-процесс, который:

* анализирует данные об активности за период;
* рассчитывает показатели вовлечённости;
* прогнозирует риск снижения учебной активности;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ активности за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `LanguageApp_Week1.xlsx`, лист `ActivityLog`.

**Структура таблицы:**

| Поле               | Тип    | Смысл                         |
| ------------------ | ------ | ----------------------------- |
| UserID             | string | ID пользователя               |
| ActivityDate       | date   | Дата активности               |
| LessonsCompleted   | number | Количество завершённых уроков |
| ExercisesCompleted | number | Количество упражнений         |
| ErrorsCount        | number | Количество ошибок             |
| SessionMinutes     | number | Длительность сессии           |
| CurrentStreakDays  | number | Текущая серия дней            |
| PlanType           | string | Free, Plus, Premium           |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик             | Начальное значение |
| ------------------- | ------------------ |
| total_records       | 0                  |
| total_lessons       | 0                  |
| total_exercises     | 0                  |
| high_error_days     | 0                  |
| short_sessions      | 0                  |
| broken_streak_cases | 0                  |

---

## 3. Преобразование данных

* LessonsCompleted, ExercisesCompleted, ErrorsCount, SessionMinutes, CurrentStreakDays → number
* ActivityDate → date
* UserID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (по каждой строке)

```
error_rate = ErrorsCount / ExercisesCompleted
activity_intensity = LessonsCompleted / SessionMinutes
```

Если `ExercisesCompleted = 0`, считать `error_rate = 0`.

---

## 5. Классификация

* если `error_rate > 0.4` →
  `high_error_days += 1`

* если `SessionMinutes < 10` →
  `short_sessions += 1`

* если `CurrentStreakDays = 0` →
  `broken_streak_cases += 1`

Накопление:

```
total_lessons += LessonsCompleted
total_exercises += ExercisesCompleted
```

---

## 6. Итоговый Excel

Файл `LanguageApp_Result.xlsx`, лист `Week1_Analysis`:

| UserID | ActivityDate | ErrorRate | ActivityIntensity | Status |

---

## 7. Итоговое уведомление

```
f"""Анализ учебной активности завершён.

Всего записей: {total_records}
Дней с высоким уровнем ошибок: {high_error_days}
Коротких сессий: {short_sessions}
Случаев прерывания серии: {broken_streak_cases}

Всего уроков: {total_lessons}
Всего упражнений: {total_exercises}

Отчёт сохранён в LanguageApp_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска снижения вовлечённости

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `LanguageApp_Month.xlsx`, лист `ActivityLog_Month`.

---

## 2. Инициализация переменных

| Переменная          | Начальное значение |
| ------------------- | ------------------ |
| total_records       | 0                  |
| high_risk_days      | 0                  |
| critical_risk_days  | 0                  |
| total_forecast_risk | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

1. Рассчитать:

```
error_rate = ErrorsCount / ExercisesCompleted
engagement_score = (LessonsCompleted * 10) + (ExercisesCompleted * 3)
streak_penalty = (14 - CurrentStreakDays) * 50
session_penalty = 
    если SessionMinutes < 15 → 300
    иначе → 0

error_penalty = error_rate * 800

plan_factor =
    если PlanType = "Free" → 400
    иначе → 150
```

2. Рассчитать итоговый риск:

```
forecast_engagement_risk = streak_penalty + session_penalty + error_penalty + plan_factor - engagement_score
```

---

## 4. Логика классификации риска

* если `forecast_engagement_risk > 3000` →
  `risk_category = "Critical Risk"`
  `critical_risk_days += 1`

* если `forecast_engagement_risk > 1500` →
  `risk_category = "High Risk"`
  `high_risk_days += 1`

* иначе →
  `risk_category = "Stable"`

После классификации:

```
total_forecast_risk += forecast_engagement_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная метрика суммарного риска снижения вовлечённости пользователей за период.

```
total_forecast_risk = Σ forecast_engagement_risk
```

### Интерпретация:

* низкое значение → стабильная учебная активность;
* среднее → часть пользователей демонстрирует признаки снижения интереса;
* высокое → высокий риск оттока и отказа от подписки.

Метрика позволяет:

* запускать автоматические напоминания;
* предлагать бонусы для восстановления streak;
* сегментировать пользователей по уровню риска;
* корректировать стратегию удержания аудитории.

---

## 5. Итоговый Excel

Файл `LanguageApp_Result.xlsx`, лист `Month_Analysis`:

| UserID | ActivityDate | ForecastEngagementRisk | RiskCategory |

---

## 6. Итоговое уведомление

```
f"""Расширенный анализ учебной активности завершён.

Всего записей: {total_records}
Высокий риск: {high_risk_days}
Критический риск: {critical_risk_days}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в LanguageApp_Result.xlsx (лист Month_Analysis).
"""
```
