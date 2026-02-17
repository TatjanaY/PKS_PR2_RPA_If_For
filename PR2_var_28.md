# Вариант 28

**Тематика:** Анализ активности и прогнозирование риска отставания на платформе онлайн-курсов по дизайну

---

## Контекст задачи

Платформа онлайн-курсов по дизайну предлагает широкий спектр образовательных программ: UX/UI, графический дизайн, моушн-графика и др. Пользователи проходят уроки, выполняют практические задания и получают оценки от преподавателей.

Платформа сталкивается с типичными проблемами:

* часть студентов просрочивает выполнение заданий;
* наблюдается высокий процент незавершённых уроков;
* ряд пользователей резко снижает активность к концу модуля;
* отсутствует агрегированная оценка риска отставания.

Необходимо реализовать RPA-процесс, который:

* собирает данные об активности студентов;
* рассчитывает ключевые показатели прогресса;
* прогнозирует риск отставания;
* формирует Excel-отчёт;
* выводит итоговое уведомление с агрегированной метрикой риска (`total_forecast_risk`).

---

# Задание 1. Анализ активности студентов по модулям (базовый уровень)

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных (Excel)

Создать файл `DesignCourses_Q1.xlsx`, лист `StudentActivity_Q1`.

**Структура таблицы:**

| Поле             | Тип    | Смысл                             |
| ---------------- | ------ | --------------------------------- |
| StudentID        | string | Идентификатор студента            |
| CourseID         | string | Идентификатор курса               |
| ModuleID         | string | Идентификатор модуля              |
| ActivityDate     | date   | Дата активности                   |
| LessonsCompleted | number | Количество завершённых уроков     |
| AssignmentsDone  | number | Количество выполненных заданий    |
| TotalAssignments | number | Общее количество заданий в модуле |
| QuizScore        | number | Средний балл за тесты (0–100)     |
| SessionMinutes   | number | Длительность сессии               |
| ModuleDueDate    | date   | Дата конца модуля                 |

---

## 2. Инициализация переменных (Puzzle RPA)

| Счётчик                | Начальное значение |
| ---------------------- | ------------------ |
| total_records          | 0                  |
| incomplete_modules     | 0                  |
| low_quiz_score_days    | 0                  |
| short_sessions         | 0                  |
| total_lessons          | 0                  |
| total_assignments_done | 0                  |

---

## 3. Преобразование данных

* LessonsCompleted, AssignmentsDone, TotalAssignments, SessionMinutes, QuizScore → number
* ActivityDate, ModuleDueDate → date
* StudentID, CourseID, ModuleID → удалить пробелы, привести к верхнему регистру

---

## 4. Расчёты (по каждой строке)

```
assignment_completion_rate = AssignmentsDone / TotalAssignments
days_to_due = ModuleDueDate - ActivityDate
```

Если `TotalAssignments = 0` → считать `assignment_completion_rate = 1`.

---

## 5. Логика классификации

* если `assignment_completion_rate < 0.5` →
  `incomplete_modules += 1`

* если `QuizScore < 60` →
  `low_quiz_score_days += 1`

* если `SessionMinutes < 15` →
  `short_sessions += 1`

Накопление:

```
total_lessons += LessonsCompleted
total_assignments_done += AssignmentsDone
```

---

## 6. Итоговый Excel

Файл `DesignCourses_Result.xlsx`, лист `Analysis_Q1`:

| StudentID | CourseID | ModuleID | AssignmentRate | QuizScore | SessionMinutes | StatusCategory |

---

## 7. Итоговое уведомление

```
f"""Анализ активности студентов Q1 завершён.

Всего записей: {total_records}
Незавершённых модулей: {incomplete_modules}
Дней с низким баллом тестов: {low_quiz_score_days}
Коротких сессий: {short_sessions}

Всего завершённых уроков: {total_lessons}
Всего выполненных заданий: {total_assignments_done}

Отчёт сохранён в DesignCourses_Result.xlsx (лист Analysis_Q1).
"""
```

---

# Задание 2. Расширенный анализ с прогнозом риска отставания

**Выполняется в Puzzle RPA и Excel.**

---

## 1. Подготовка данных

Файл `DesignCourses_Q2.xlsx`, лист `StudentActivity_Q2`.

---

## 2. Инициализация переменных

| Переменная             | Начальное значение |
| ---------------------- | ------------------ |
| total_records          | 0                  |
| high_risk_students     | 0                  |
| critical_risk_students | 0                  |
| total_forecast_risk    | 0                  |

---

## 3. Расчёт прогнозного риска

Для каждой записи:

```
assignment_completion_rate = AssignmentsDone / TotalAssignments
progress_score = (LessonsCompleted * 10) + (assignment_completion_rate * 100) + QuizScore
time_pressure = (ModuleDueDate - ActivityDate) * 5
session_penalty = 
    если SessionMinutes < 20 → 200
    иначе → 0

completion_penalty = 
    если assignment_completion_rate < 0.5 → (0.5 - assignment_completion_rate) * 500
    иначе → 0

quiz_penalty = (60 - QuizScore) * 10
```

Итоговый прогнозный риск:

```
forecast_designer_risk = time_pressure + session_penalty + completion_penalty + quiz_penalty
```

Если значения отрицательные → риск = 0.

---

## 4. Логика классификации риска

* если `forecast_designer_risk > 4000` →
  `risk_category = "Critical Risk"`
  `critical_risk_students += 1`

* если `forecast_designer_risk > 2000` →
  `risk_category = "High Risk"`
  `high_risk_students += 1`

* иначе →
  `risk_category = "Stable"`

После классификации:

```
total_forecast_risk += forecast_designer_risk
```

---

## Что означает total_forecast_risk?

`total_forecast_risk` — агрегированная оценка вероятности отставания студентов по курсам за период.

```
total_forecast_risk = Σ forecast_designer_risk
```

### Интерпретация:

* низкое значение → большинство студентов проходят модули стабильно;
* среднее → часть студентов демонстрирует замедление прогресса;
* высокое → высокий риск массового отставания.

Метрика позволяет:

* выявлять студентов группы риска;
* запускать автоматические уведомления;
* корректировать сложность модулей;
* планировать работу с преподавателями и наставниками.

---

## 5. Итоговый Excel

Файл `DesignCourses_Result.xlsx`, лист `Analysis_Q2`:

| StudentID | CourseID | ModuleID | ForecastDesignerRisk | RiskCategory |

---

## 6. Итоговое модальное уведомление

````
f"""Расширенный анализ студентов Q2 завершён.

Всего записей: {total_records}
Высокий риск: {high_risk_students}
Критический риск: {critical_risk_students}
Суммарный прогнозный риск: {total_forecast_risk}

Отчёт сформирован в DesignCourses_Result.xlsx (лист Analysis_Q2).
"""
```---

Если нужно — оформлю **Вариант 29**!
````
