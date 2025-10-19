# Модель данных: Завершение настройки среды разработки

**Дата:** 2025-10-19  
**Статус:** Complete

## Обзор

Эта функция является infrastructure-ориентированной и не вводит новых бизнес-сущностей в основное приложение. Однако, для работы CI/CD системы существуют метаданные, которые генерируются и хранятся.

## Сущности CI/CD метаданных

### 1. Build Artifact

**Описание:** Результаты сборки приложения

**Атрибуты:**
- `build_id`: String - Уникальный идентификатор сборки
- `commit_sha`: String - SHA коммита
- `branch`: String - Название ветки
- `platform`: Enum - linux | windows | macos
- `bundle_path`: String - Путь к собранному бинарнику
- `bundle_size_mb`: Float - Размер bundle в MB
- `build_duration_sec`: Int - Время сборки в секундах
- `timestamp`: DateTime - Дата и время сборки
- `status`: Enum - success | failure

**Валидация:**
- `bundle_size_mb` < 50 (требование конституции)
- `commit_sha` должен существовать в репозитории

**Хранение:** GitHub Actions artifacts (автоматическое управление)

---

### 2. Test Run

**Описание:** Результаты запуска тестов

**Атрибуты:**
- `run_id`: String - ID запуска
- `test_type`: Enum - unit | integration | e2e
- `total_tests`: Int - Общее количество тестов
- `passed`: Int - Успешные тесты
- `failed`: Int - Проваленные тесты
- `skipped`: Int - Пропущенные тесты
- `duration_sec`: Float - Время выполнения
- `timestamp`: DateTime
- `commit_sha`: String
- `branch`: String

**Валидация:**
- `total_tests` = `passed` + `failed` + `skipped`
- `duration_sec` < 600 для unit tests (10 минут max)

**Хранение:** GitHub Actions logs, JUnit XML artifacts

---

### 3. Coverage Report

**Описание:** Отчет о покрытии кода тестами

**Атрибуты:**
- `report_id`: String
- `coverage_percent`: Float - Общий процент покрытия
- `lines_covered`: Int - Покрытые строки
- `lines_total`: Int - Всего строк
- `branches_covered`: Int - Покрытые ветки
- `branches_total`: Int - Всего веток
- `module_coverage`: Map<String, Float> - Покрытие по модулям
- `timestamp`: DateTime
- `commit_sha`: String
- `language`: Enum - rust | typescript

**Валидация:**
- `coverage_percent` >= 70 для критических модулей
- `coverage_percent` = (`lines_covered` / `lines_total`) * 100

**Хранение:** Codecov (external service), cobertura.xml artifacts

---

### 4. Benchmark Result

**Описание:** Результаты performance benchmarks

**Атрибуты:**
- `benchmark_id`: String
- `operation_name`: String - Название операции (e.g., "load_1000_chats")
- `mean_time_ms`: Float - Среднее время выполнения
- `std_dev_ms`: Float - Стандартное отклонение
- `baseline_time_ms`: Float - Baseline для сравнения
- `regression_percent`: Float - Процент регрессии/улучшения
- `timestamp`: DateTime
- `commit_sha`: String
- `environment`: String - Информация о железе

**Валидация:**
- `regression_percent` < 20 (alert threshold)
- Specific targets (из конституции):
  - `load_1000_chats`: mean_time_ms < 500
  - `load_50_messages`: mean_time_ms < 300

**Хранение:** GitHub Pages (gh-pages branch), JSON format

---

### 5. Lighthouse Metrics

**Описание:** Frontend performance метрики

**Атрибуты:**
- `report_id`: String
- `performance_score`: Float (0-100) - Общий performance score
- `first_contentful_paint_ms`: Int - FCP метрика
- `time_to_interactive_ms`: Int - TTI метрика
- `speed_index_ms`: Int - Speed Index
- `largest_contentful_paint_ms`: Int - LCP
- `cumulative_layout_shift`: Float - CLS
- `timestamp`: DateTime
- `commit_sha`: String

**Валидация:**
- `performance_score` >= 90
- `first_contentful_paint_ms` < 2000
- `time_to_interactive_ms` < 2000
- `speed_index_ms` < 3000

**Хранение:** Lighthouse CI temporary storage, JSON artifacts

---

### 6. Code Quality Issue

**Описание:** Проблемы качества кода, найденные линтерами

**Атрибуты:**
- `issue_id`: String
- `tool`: Enum - clippy | eslint | prettier
- `severity`: Enum - error | warning | info
- `rule_id`: String - ID правила (e.g., "clippy::needless_return")
- `file_path`: String - Путь к файлу
- `line_number`: Int - Номер строки
- `message`: String - Описание проблемы
- `suggestion`: Optional<String> - Предложение по исправлению
- `timestamp`: DateTime
- `commit_sha`: String

**Валидация:**
- Количество issues с `severity: error` ДОЛЖНО быть 0 для merge
- Количество issues с `severity: warning` ДОЛЖНО быть 0 для merge (0 warnings policy)

**Хранение:** GitHub Actions logs, annotations

---

### 7. Security Audit Result

**Описание:** Результаты проверки безопасности зависимостей

**Атрибуты:**
- `audit_id`: String
- `tool`: Enum - cargo_audit | npm_audit
- `vulnerabilities_found`: Int
- `critical_count`: Int
- `high_count`: Int
- `medium_count`: Int
- `low_count`: Int
- `vulnerable_packages`: Array<VulnerablePackage>
- `timestamp`: DateTime
- `commit_sha`: String

**VulnerablePackage:**
- `name`: String - Имя пакета
- `version`: String - Версия
- `advisory_id`: String - ID уязвимости
- `severity`: Enum - critical | high | medium | low
- `patched_versions`: String - Версии с исправлением

**Валидация:**
- `critical_count` ДОЛЖНО быть 0 для merge
- `high_count` рекомендуется 0

**Хранение:** GitHub Actions logs, SARIF format для GitHub Security

---

## Связи между сущностями

```
Commit (Git)
    ├─ Build Artifact (1:N по платформам)
    ├─ Test Run (1:N по типам)
    ├─ Coverage Report (1:N по языкам)
    ├─ Benchmark Result (1:N по операциям)
    ├─ Lighthouse Metrics (1:1)
    ├─ Code Quality Issues (1:N)
    └─ Security Audit Result (1:N по инструментам)
```

**Ключ связи:** `commit_sha` связывает все метаданные с конкретным коммитом

---

## Хранилища данных

### GitHub Actions Artifacts
**Хранит:** Build artifacts, test reports, coverage files
**Retention:** 90 дней (по умолчанию)
**Access:** GitHub API

### Codecov
**Хранит:** Coverage reports, тренды
**Retention:** Unlimited (external service)
**Access:** Codecov API, web UI

### GitHub Pages (gh-pages branch)
**Хранит:** Benchmark results (JSON), cargo doc, TypeDoc
**Retention:** Permanent (git history)
**Access:** Static hosting, git

### GitHub Actions Logs
**Хранит:** Console output, annotations
**Retention:** 90 дней
**Access:** GitHub UI, API

---

## Агрегированные метрики (для dashboard)

Хотя не существует централизованной БД, можно агрегировать:

### Project Health Score
Вычисляется из:
- Test pass rate (%) - вес 30%
- Coverage (%) - вес 25%
- Performance score (Lighthouse) - вес 20%
- Code quality (0 warnings = 100%) - вес 15%
- Security (0 vulns = 100%) - вес 10%

### Trend Metrics (по времени)
- Coverage trend (за последние 30 дней)
- Performance trend (regression/improvement)
- Test stability (flaky test rate)
- Build duration trend

---

## Примечания

1. **Нет постоянной БД:** Все данные эфемерны или хранятся в external services
2. **Git как источник истины:** `commit_sha` связывает все метаданные
3. **Статическая генерация:** Reports генерируются в CI, не в runtime приложения
4. **API доступ:** Все данные доступны через GitHub API или external service APIs

---

**Эта модель данных описывает CI/CD метаданные, а не бизнес-логику приложения.**

