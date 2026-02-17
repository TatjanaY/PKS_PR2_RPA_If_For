# Вариант 21

**Тематика:** Анализ онлайн-консультаций врачей и прогнозирование операционных рисков платформы

---

## Контекст задачи

Платформа онлайн-консультаций объединяет врачей разных специализаций и пациентов из различных регионов. Пользователи записываются на приём, оплачивают консультацию, проходят видеосеанс или отменяют запись.

Типовые проблемы платформы:

* отмены консультаций в последний момент;
* неявки пациентов (no-show);
* перегрузка отдельных врачей;
* высокий риск возвратов денежных средств;
* отсутствие агрегированной метрики операционного риска.

Необходимо реализовать RPA-процесс, который:

* анализирует записи на консультации;
* рассчитывает показатели отмен и неявок;
* прогнозирует риск срыва консультации;
* формирует Excel-отчёт;
* рассчитывает агрегированный показатель `total_forecast_risk`.

---

# Задание 1. Анализ консультаций за месяц (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `Telemedicine_Month1.xlsx`, лист `Consultations`.

**Структура таблицы:**

| Поле               | Тип     | Смысл                                   |
| ------------------ | ------- | --------------------------------------- |
| ConsultationID     | string  | ID консультации                         |
| PatientID          | string  | ID пациента                             |
| DoctorID           | string  | ID врача                                |
| Specialty          | string  | Специализация                           |
| AppointmentDate    | date    | Дата консультации                       |
| BookingDate        | date    | Дата записи                             |
| ConsultationStatus | string  | Scheduled, Completed, Cancelled, NoShow |
| ConsultationFee    | number  | Стоимость                               |
| DurationMinutes    | integer | Длительность                            |

---

## 2. Инициализация переменных

| Переменная          | Начальное значение |
| ------------------- | ------------------ |
| total_consultations | 0                  |
| completed_sessions  | 0                  |
| cancelled_sessions  | 0                  |
| no_show_cases       | 0                  |
| short_notice_cancel | 0                  |
| total_refund_risk   | 0                  |

---

## 3. Преобразование данных

* ConsultationFee → number
* DurationMinutes → integer
* AppointmentDate, BookingDate → date
* ConsultationID, PatientID, DoctorID → удалить пробелы, верхний регистр

---

## 4. Расчёты (на каждой строке)

```
days_before_appointment = AppointmentDate - BookingDate
revenue_per_minute = ConsultationFee / DurationMinutes
```

Если `ConsultationStatus = "Cancelled"` →
`potential_refund = ConsultationFee`
Иначе → `potential_refund = 0`

---

## 5. Логика анализа

* если `ConsultationStatus = "Completed"` → `completed_sessions += 1`
* если `ConsultationStatus = "Cancelled"` → `cancelled_sessions += 1`
* если `ConsultationStatus = "NoShow"` → `no_show_cases += 1`
* если `ConsultationStatus = "Cancelled" AND days_before_appointment <= 1` →
  `short_notice_cancel += 1`
* если `ConsultationStatus = "Cancelled"` →
  `total_refund_risk += potential_refund`

---

## 6. Итоговый Excel

Файл `Telemedicine_Result.xlsx`, лист `Month1_Analysis`:

| ConsultationID | DoctorID | DaysBeforeAppointment | PotentialRefund | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ консультаций завершён.

Всего консультаций: {total_consultations}
Завершённых: {completed_sessions}
Отменённых: {cancelled_sessions}
Неявок: {no_show_cases}
Отмен в последний момент: {short_notice_cancel}

Потенциальный риск возвратов: {total_refund_risk}

Отчёт сохранён в Telemedicine_Result.xlsx (лист Month1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ с прогнозом риска срыва консультаций

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `Telemedicine_Month2.xlsx`, лист `Consultations_Month2`.

---

## 2. Инициализация переменных

| Переменная              | Начальное значение |
| ----------------------- | ------------------ |
| total_consultations     | 0                  |
| completed_sessions      | 0                  |
| cancelled_sessions      | 0                  |
| no_show_cases           | 0                  |
| high_risk_consultations | 0                  |
| critical_risk_cases     | 0                  |
| total_refund_risk       | 0                  |
| total_forecast_risk     | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой строки:

```
days_before_appointment = AppointmentDate - BookingDate
revenue_per_minute = ConsultationFee / DurationMinutes

urgency_factor = (2 - days_before_appointment) * 100
financial_factor = revenue_per_minute * 10
duration_factor = DurationMinutes * 2

forecast_cancel_risk = urgency_factor + financial_factor + duration_factor
```

---

## 4. Логика классификации риска

Если `ConsultationStatus = "Cancelled"` →
`risk_category = "Cancelled"`
`cancelled_sessions += 1`

Если `ConsultationStatus = "Scheduled"`:

* если `forecast_cancel_risk > 800` →
  `risk_category = "Critical Risk"`
  `critical_risk_cases += 1`

* если `forecast_cancel_risk > 400` →
  `risk_category = "High Risk"`
  `high_risk_consultations += 1`

* иначе →
  `risk_category = "Stable"`

Если `ConsultationStatus = "Completed"` →
`risk_category = "Completed"`

После классификации:

```
total_forecast_risk += forecast_cancel_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — это суммарная прогнозная оценка риска срыва консультаций за период.

```
total_forecast_risk = Σ forecast_cancel_risk
```

### Интерпретация:

* низкое значение → стабильная работа платформы;
* среднее → возможны локальные перегрузки или рост отмен;
* высокое → системный операционный риск (рост возвратов, неявок, потери выручки).

Метрика используется для:

* балансировки расписания врачей;
* автоматического напоминания пациентам;
* оценки финансовой устойчивости платформы.

---

## 5. Итоговый Excel

Файл `Telemedicine_Result.xlsx`, лист `Month2_Analysis`:

| ConsultationID | DoctorID | ForecastCancelRisk | RiskCategory |

---

## 6. Итоговое уведомление

```
f"""Расширенный анализ консультаций завершён.

Всего консультаций: {total_consultations}
Завершённых: {completed_sessions}
Отменённых: {cancelled_sessions}
Неявок: {no_show_cases}

Высокий риск: {high_risk_consultations}
Критический риск: {critical_risk_cases}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в Telemedicine_Result.xlsx (лист Month2_Analysis).
"""
```
