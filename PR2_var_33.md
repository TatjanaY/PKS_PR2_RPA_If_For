# Вариант 33

**Тематика:** Анализ участия и прогнозирование риска невыполнения целей в приложении для фитнес-марафонов

---

## Контекст задачи

Приложение для фитнес-марафонов позволяет пользователям участвовать в программах с ежедневными тренировками и отслеживать прогресс по выполнению заданий. Платформа анализирует активность участников, выявляет отстающих и прогнозирует риск невыполнения целей.

Типичные проблемы:

* часть участников пропускает тренировки;
* наблюдается снижение мотивации к середине марафона;
* отсутствует прогнозный механизм выявления участников с высоким риском отставания;
* требуется автоматическая классификация участников по уровню риска.

Необходимо реализовать RPA-процесс, который:

* анализирует данные по активности участников;
* рассчитывает процент выполнения тренировок;
* прогнозирует риск отставания;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ активности за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `Marathon_Week1.xlsx`, лист `ParticipantLog`.

### Структура таблицы:

| Поле            | Тип     | Смысл                                        |
| --------------- | ------- | -------------------------------------------- |
| ParticipantID   | string  | Уникальный идентификатор участника           |
| LogDate         | date    | Дата записи                                  |
| TrainingType    | string  | Тип тренировки (кардио, силовая, йога и др.) |
| Completed       | boolean | Тренировка выполнена (Yes/No)                |
| PlannedDuration | number  | Плановая длительность тренировки (минуты)    |
| ActualDuration  | number  | Фактическая длительность (минуты)            |
| GoalCompletion  | number  | Процент выполнения цели на день              |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик                | Начальное значение | Смысл                              |
| ---------------------- | ------------------ | ---------------------------------- |
| total_records          | 0                  | Всего записей                      |
| total_completed        | 0                  | Выполнено тренировок               |
| missed_sessions        | 0                  | Пропущенные тренировки             |
| total_duration_planned | 0                  | Суммарная плановая длительность    |
| total_duration_actual  | 0                  | Суммарная фактическая длительность |
| high_risk_sessions     | 0                  | Сессии с риском невыполнения       |
| total_forecast_risk    | 0                  | Суммарный прогнозный риск          |

---

## 3. Преобразование данных

* Completed → boolean
* PlannedDuration, ActualDuration, GoalCompletion → number
* LogDate → date
* ParticipantID → удалить пробелы, верхний регистр

---

## 4. Расчёты (по каждой строке)

* `duration_difference = ActualDuration - PlannedDuration`
* если `Completed = No` → `missed_sessions += 1`
* если `GoalCompletion < 70` → `high_risk_sessions += 1`

Прогнозный риск для сессии:

```
forecast_risk = (100 - GoalCompletion) + max(0, PlannedDuration - ActualDuration)
total_forecast_risk += forecast_risk
```

---

## 5. Классификация

* если `GoalCompletion < 50` → статус `"Critical Risk"`
* если `GoalCompletion < 70` → статус `"High Risk"`
* иначе → `"On Track"`

---

## 6. Итоговый Excel

Файл `Marathon_Result.xlsx`, лист `Week1_Analysis`:

| ParticipantID | LogDate | GoalCompletion | DurationDifference | Status | ForecastRisk |

---

## 7. Итоговое уведомление

```
f"""Анализ активности за неделю завершён.

Всего записей: {total_records}
Пропущенные тренировки: {missed_sessions}
Сессий с высоким риском: {high_risk_sessions}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сохранён в Marathon_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска невыполнения целей

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `Marathon_Month.xlsx`, лист `ParticipantLog_Month`.

---

## 2. Инициализация переменных

| Переменная             | Начальное значение | Смысл                     |
| ---------------------- | ------------------ | ------------------------- |
| total_records          | 0                  | Всего записей             |
| missed_sessions        | 0                  | Пропущенные тренировки    |
| high_risk_sessions     | 0                  | Сессии с высоким риском   |
| critical_risk_sessions | 0                  | Критический риск          |
| total_duration_planned | 0                  | Плановая длительность     |
| total_duration_actual  | 0                  | Фактическая длительность  |
| total_forecast_risk    | 0                  | Суммарный прогнозный риск |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
duration_difference = PlannedDuration - ActualDuration
forecast_risk = (100 - GoalCompletion) + max(0, duration_difference) + (if TrainingType = "Cardio" then 10 else 0)
total_forecast_risk += forecast_risk
```

---

## 4. Логика классификации риска

* если `forecast_risk > 50` → `risk_category = "Critical Risk"`, `critical_risk_sessions += 1`
* если `forecast_risk > 30` → `risk_category = "High Risk"`, `high_risk_sessions += 1`
* иначе → `risk_category = "On Track"`

---

## 5. Итоговый Excel

Файл `Marathon_Result.xlsx`, лист `Month_Analysis`:

| ParticipantID | LogDate | GoalCompletion | DurationDifference | ForecastRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ активности месяца завершён.

Всего записей: {total_records}
Сессий с высоким риском: {high_risk_sessions}
Сессий с критическим риском: {critical_risk_sessions}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в Marathon_Result.xlsx (лист Month_Analysis).
"""
```
