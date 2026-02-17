# Вариант 30

**Тематика:** Анализ качества сна и прогнозирование риска нарушения режима в приложении для отслеживания сна

---

## Контекст задачи

Мобильное приложение для отслеживания сна позволяет пользователям фиксировать продолжительность и качество сна, отслеживать циклы сна, время засыпания и пробуждения, а также анализировать ночные пробуждения.

Типичные проблемы:

* недостаточная продолжительность сна;
* частые ночные пробуждения;
* несоблюдение режима сна;
* риск хронической усталости и снижения продуктивности.

Необходимо реализовать RPA-процесс, который:

* анализирует данные о сне за период;
* рассчитывает ключевые метрики качества сна;
* прогнозирует риск нарушения режима сна;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ сна за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `SleepTracker_Week1.xlsx`, лист `SleepLogs`.

### Структура таблицы:

| Поле             | Тип    | Смысл                             |
| ---------------- | ------ | --------------------------------- |
| UserID           | string | Идентификатор пользователя        |
| LogDate          | date   | Дата записи сна                   |
| BedTime          | time   | Время засыпания                   |
| WakeTime         | time   | Время пробуждения                 |
| SleepDurationMin | number | Продолжительность сна (мин)       |
| SleepCycles      | number | Количество завершённых циклов сна |
| Awakenings       | number | Количество ночных пробуждений     |
| SleepQuality     | number | Индекс качества сна (0–100)       |

---

## 2. Инициализация переменных

| Счётчик             | Начальное значение |
| ------------------- | ------------------ |
| total_logs          | 0                  |
| short_sleep_days    | 0                  |
| low_quality_days    | 0                  |
| frequent_awakenings | 0                  |
| total_sleep_minutes | 0                  |
| total_forecast_risk | 0                  |

---

## 3. Преобразование данных

* SleepDurationMin, SleepCycles, Awakenings, SleepQuality → number
* LogDate, BedTime, WakeTime → date/time
* UserID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (по каждой строке)

```
sleep_deficit = max(0, 420 - SleepDurationMin)  # предполагаемая норма сна 7 часов = 420 мин
quality_penalty = max(0, 50 - SleepQuality)
awakenings_penalty = Awakenings * 10
```

Если SleepDurationMin < 420 → `short_sleep_days += 1`
Если SleepQuality < 50 → `low_quality_days += 1`
Если Awakenings > 3 → `frequent_awakenings += 1`

Накопление:

```
total_sleep_minutes += SleepDurationMin
total_logs += 1
```

---

## 5. Классификация

* если `sleep_deficit > 120 OR quality_penalty > 30` → статус `"High Risk"`
* если `sleep_deficit > 60 OR quality_penalty > 15` → статус `"Moderate Risk"`
* иначе → `"Stable"`

---

## 6. Итоговый Excel

Файл `SleepTracker_Result.xlsx`, лист `Week1_Analysis`:

| UserID | LogDate | SleepDurationMin | SleepQuality | Awakenings | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ сна за неделю завершён.

Всего записей: {total_logs}
Дней с недостаточным сном: {short_sleep_days}
Дней с низким качеством сна: {low_quality_days}
Дней с частыми пробуждениями: {frequent_awakenings}

Суммарные минуты сна: {total_sleep_minutes}

Отчёт сохранён в SleepTracker_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска нарушения режима сна

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `SleepTracker_Month.xlsx`, лист `SleepLogs_Month`.

---

## 2. Инициализация переменных

| Переменная          | Начальное значение |
| ------------------- | ------------------ |
| total_logs          | 0                  |
| high_risk_days      | 0                  |
| moderate_risk_days  | 0                  |
| total_sleep_minutes | 0                  |
| total_forecast_risk | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
sleep_deficit = max(0, 420 - SleepDurationMin)
quality_penalty = max(0, 50 - SleepQuality)
awakenings_penalty = Awakenings * 10
time_penalty = abs((BedTime.hour + BedTime.minute/60) - 23) * 5  # отклонение от оптимального времени засыпания 23:00

forecast_sleep_risk = sleep_deficit + quality_penalty + awakenings_penalty + time_penalty
```

---

## 4. Логика классификации риска

* если `forecast_sleep_risk > 150` →
  `risk_category = "High Risk"`
  `high_risk_days += 1`

* если `forecast_sleep_risk > 75` →
  `risk_category = "Moderate Risk"`
  `moderate_risk_days += 1`

* иначе →
  `risk_category = "Stable"`

Накопление:

```
total_forecast_risk += forecast_sleep_risk
total_sleep_minutes += SleepDurationMin
total_logs += 1
```

---

## 5. Итоговый Excel

Файл `SleepTracker_Result.xlsx`, лист `Month_Analysis`:

| UserID | LogDate | ForecastSleepRisk | RiskCategory | SleepDurationMin | SleepQuality | Awakenings |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ сна завершён.

Всего записей: {total_logs}
Дней с умеренным риском: {moderate_risk_days}
Дней с высоким риском: {high_risk_days}

Суммарные минуты сна: {total_sleep_minutes}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в SleepTracker_Result.xlsx (лист Month_Analysis).
"""
```
