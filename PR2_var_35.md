# Вариант 35

**Тематика:** Анализ активности пользователей и прогнозирование риска низкой вовлечённости в сервисе для совместного редактирования документов

---

## Контекст задачи

Сервис совместного редактирования документов позволяет пользователям работать над проектами в реальном времени, отслеживать изменения и комментировать. Аналитика платформы направлена на:

* выявление документов с низкой активностью участников;
* определение пользователей с низкой вовлечённостью;
* прогнозирование риска несвоевременного завершения проектов;
* автоматическую классификацию документов и пользователей по уровню активности.

Необходимо реализовать RPA-процесс, который:

* собирает данные по пользователям и документам;
* рассчитывает показатели активности и вовлечённости;
* прогнозирует риск задержек;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ активности за неделю (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `CollabDocs_Week1.xlsx`, лист `DocumentActivity`.

### Структура таблицы:

| Поле               | Тип     | Смысл                                   |
| ------------------ | ------- | --------------------------------------- |
| DocumentID         | string  | Уникальный идентификатор документа      |
| UserID             | string  | Идентификатор пользователя              |
| ProjectID          | string  | Идентификатор проекта                   |
| Date               | date    | Дата активности                         |
| EditsCount         | integer | Количество изменений за день            |
| CommentsCount      | integer | Количество комментариев за день         |
| SharesCount        | integer | Количество совместных доступов          |
| LastEditDate       | date    | Дата последнего редактирования          |
| TargetEditsPerWeek | integer | Плановое количество изменений за неделю |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик                | Начальное значение | Смысл                                          |
| ---------------------- | ------------------ | ---------------------------------------------- |
| total_records          | 0                  | Всего записей активности                       |
| low_activity_documents | 0                  | Документы с низкой активностью                 |
| inactive_users         | 0                  | Пользователи без активности                    |
| total_edits            | 0                  | Суммарное количество изменений                 |
| total_comments         | 0                  | Суммарное количество комментариев              |
| total_forecast_risk    | 0                  | Суммарный прогнозный риск низкой вовлечённости |

---

## 3. Преобразование данных

* EditsCount, CommentsCount, SharesCount, TargetEditsPerWeek → integer
* Date, LastEditDate → date
* DocumentID, UserID, ProjectID → удалить пробелы, верхний регистр

---

## 4. Расчёты (по каждой строке)

* `activity_ratio = EditsCount / max(TargetEditsPerWeek, 1) * 100`
* `engagement_score = EditsCount + CommentsCount + SharesCount`
* если `activity_ratio < 50` → `low_activity_documents += 1`
* если `EditsCount = 0 AND CommentsCount = 0` → `inactive_users += 1`
* `total_edits += EditsCount`
* `total_comments += CommentsCount`
* `forecast_risk = max(0, 100 - activity_ratio)`
* `total_forecast_risk += forecast_risk`

---

## 5. Классификация

* если `forecast_risk > 70` → статус `"Critical Risk"`
* если `forecast_risk > 40` → статус `"High Risk"`
* иначе → `"On Track"`

---

## 6. Итоговый Excel

Файл `CollabDocs_Result.xlsx`, лист `Week1_Analysis`:

| DocumentID | UserID | ProjectID | ActivityRatio | EngagementScore | ForecastRisk | Status |

---

## 7. Итоговое уведомление

```
f"""Анализ активности за неделю завершён.

Всего записей: {total_records}
Документов с низкой активностью: {low_activity_documents}
Неактивных пользователей: {inactive_users}
Суммарные изменения: {total_edits}
Суммарные комментарии: {total_comments}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сохранён в CollabDocs_Result.xlsx (лист Week1_Analysis).
"""
```

---

# Задание 2. Расширенный анализ месяца с прогнозом риска вовлечённости

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `CollabDocs_Month.xlsx`, лист `DocumentActivity_Month`.

---

## 2. Инициализация переменных

| Переменная              | Начальное значение | Смысл                             |
| ----------------------- | ------------------ | --------------------------------- |
| total_records           | 0                  | Всего записей активности          |
| low_activity_documents  | 0                  | Документы с низкой активностью    |
| inactive_users          | 0                  | Пользователи без активности       |
| total_edits             | 0                  | Суммарное количество изменений    |
| total_comments          | 0                  | Суммарное количество комментариев |
| high_risk_documents     | 0                  | Документы с высоким риском        |
| critical_risk_documents | 0                  | Документы с критическим риском    |
| total_forecast_risk     | 0                  | Суммарный прогнозный риск         |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
activity_ratio = EditsCount / max(TargetEditsPerWeek, 1) * 100
engagement_score = EditsCount + CommentsCount + SharesCount
forecast_risk = max(0, 100 - activity_ratio + (SharesCount * 0.5))
total_forecast_risk += forecast_risk
```

---

## 4. Логика классификации риска

* если `forecast_risk > 70` → `risk_category = "Critical Risk"`, `critical_risk_documents += 1`
* если `forecast_risk > 40` → `risk_category = "High Risk"`, `high_risk_documents += 1`
* иначе → `risk_category = "On Track"`

---

## 5. Итоговый Excel

Файл `CollabDocs_Result.xlsx`, лист `Month_Analysis`:

| DocumentID | UserID | ProjectID | ActivityRatio | EngagementScore | ForecastRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

```
f"""Расширенный анализ активности за месяц завершён.

Всего записей: {total_records}
Документы с высоким риском: {high_risk_documents}
Документы с критическим риском: {critical_risk_documents}
Суммарный прогнозный риск вовлечённости: {total_forecast_risk}

Отчёт сформирован в CollabDocs_Result.xlsx (лист Month_Analysis).
"""
```
