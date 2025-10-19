# План разработки: Завершение настройки среды разработки

**Дата создания:** 2025-10-19  
**Автор:** AI Assistant  
**Статус:** Review

## Обзор

Этот план описывает техническую реализацию полной настройки инфраструктуры разработки для проекта Workgram, включая CI/CD пайплайны, автоматизированное тестирование, performance benchmarking, pre-commit hooks и инструменты качества кода. Цель - обеспечить автоматический контроль соблюдения всех принципов конституции проекта.

## Цели

- **Бизнес-цель 1:** Повысить качество релизов через автоматизированную проверку кода
- **Бизнес-цель 2:** Сократить время обнаружения дефектов через раннюю обратную связь
- **Бизнес-цель 3:** Улучшить onboarding новых разработчиков через четкие стандарты и документацию
- **Техническая цель 1:** Достичь 70%+ покрытия критической логики тестами
- **Техническая цель 2:** Автоматизировать контроль производительности через benchmarks
- **Техническая цель 3:** Обеспечить 0 предупреждений clippy/ESLint в production коде

## Контекст

Согласно конституции проекта (v1.0.0), существует 4 принципа, которые должны соблюдаться:
1. Качество кода и поддерживаемость
2. Стандарты тестирования и покрытие
3. Согласованность пользовательского опыта
4. Требования к производительности

В настоящий момент:
- ❌ Отсутствует CI/CD пайплайн для автоматической проверки
- ❌ Нет мониторинга покрытия тестами (цель: 70%+)
- ❌ Отсутствуют performance benchmarks для критических операций
- ❌ Не настроены pre-commit hooks для локальных проверок
- ❌ Нет E2E тестов для критических user flows
- ❌ Отсутствует автоматическая генерация документации

Это создает риски:
- Дефекты попадают в production
- Деградация производительности не обнаруживается вовремя
- Несогласованность стиля кода между разработчиками
- Отсутствие актуальной документации

## Предлагаемое решение

### Архитектура

```
┌─────────────────────────────────────────────────────────────┐
│                     Developer Machine                        │
├─────────────────────────────────────────────────────────────┤
│  git commit                                                   │
│     ↓                                                         │
│  Pre-commit hooks (Husky)                                     │
│     ├─ cargo fmt --check                                      │
│     ├─ cargo clippy                                           │
│     ├─ prettier --check                                       │
│     └─ eslint --max-warnings 0                                │
└─────────────────────────────────────────────────────────────┘
                    ↓ git push
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions CI/CD                       │
├─────────────────────────────────────────────────────────────┤
│  Workflow 1: ci.yml (на каждый PR/push)                      │
│    ├─ Job: code-quality                                       │
│    │   ├─ rustfmt check (Rust)                               │
│    │   ├─ clippy -D warnings (Rust)                          │
│    │   ├─ tsc --noEmit (TypeScript)                          │
│    │   ├─ eslint --max-warnings 0                            │
│    │   └─ prettier --check                                    │
│    │                                                           │
│    ├─ Job: testing                                            │
│    │   ├─ cargo test (unit + integration)                    │
│    │   ├─ tarpaulin (coverage report)                        │
│    │   ├─ coverage check >= 70%                              │
│    │   ├─ pnpm test (frontend unit tests)                    │
│    │   └─ Upload to Codecov                                   │
│    │                                                           │
│    ├─ Job: e2e-tests                                          │
│    │   ├─ Build Tauri app                                     │
│    │   ├─ Playwright E2E tests                               │
│    │   └─ Upload screenshots on failure                       │
│    │                                                           │
│    ├─ Job: security-audit                                     │
│    │   ├─ cargo audit                                         │
│    │   └─ pnpm audit                                          │
│    │                                                           │
│    └─ Job: build                                              │
│        ├─ Build release (Linux/Win/Mac)                       │
│        └─ Check bundle size < 50MB                            │
│                                                                │
│  Workflow 2: performance.yml (на PR + weekly)                │
│    ├─ Job: benchmark                                          │
│    │   ├─ cargo bench (criterion)                            │
│    │   ├─ Compare vs baseline                                │
│    │   ├─ Fail if regression > 20%                           │
│    │   ├─ Measure cold start < 3s                            │
│    │   └─ Comment results on PR                               │
│    │                                                           │
│    └─ Job: lighthouse                                         │
│        ├─ Build frontend                                      │
│        ├─ Run Lighthouse CI                                   │
│        └─ Performance score >= 90                             │
└─────────────────────────────────────────────────────────────┘
                    ↓ on success
┌─────────────────────────────────────────────────────────────┐
│                    GitHub (merge allowed)                     │
│  ✅ All checks passed → можно мержить в main                 │
└─────────────────────────────────────────────────────────────┘
```

### Технический стек

**CI/CD:**
- GitHub Actions (workflows)
- Codecov (coverage reporting)
- github-action-benchmark (performance tracking)

**Качество кода:**
- **Rust:** rustfmt, clippy, cargo-audit
- **TypeScript:** ESLint, Prettier, tsc
- **Pre-commit:** Husky (git hooks)

**Тестирование:**
- **Unit (Rust):** cargo test, criterion (benchmarks)
- **Coverage (Rust):** tarpaulin
- **Unit (Frontend):** Vitest
- **E2E:** Playwright
- **Performance:** Lighthouse CI

**Документация:**
- cargo doc (Rust API docs)
- TypeDoc (TypeScript API docs)

### Изменения в кодовой базе

#### Новые файлы конфигурации:

1. **`.github/workflows/ci.yml`** - Основной CI пайплайн
   - Code quality checks (Principle 1)
   - Testing & coverage (Principle 2)
   - E2E tests (Principle 3)
   - Security audit
   - Build verification

2. **`.github/workflows/performance.yml`** - Performance benchmarks
   - Cargo bench с criterion
   - Baseline tracking
   - Regression detection (Principle 4)
   - Lighthouse CI

3. **`.husky/pre-commit`** - Pre-commit hooks
   - Локальные проверки перед commit
   - Быстрая обратная связь разработчику

4. **`.lighthouserc.json`** - Lighthouse конфигурация
   - Performance thresholds
   - TTI < 2s, FCP < 2s

5. **`.eslintrc.json`** - ESLint конфигурация
   - max-warnings: 0
   - Strict TypeScript rules

6. **`.prettierrc.json`** - Prettier конфигурация
   - Единый стиль форматирования

7. **`src-tauri/benches/performance_benchmarks.rs`** - Performance benchmarks
   - Critical operations benchmarking
   - Baseline metrics

8. **`tests/e2e/`** - E2E тестовые сценарии
   - Playwright тесты для критических user flows

#### Обновления существующих файлов:

9. **`package.json`** - Добавление scripts
   - test, test:coverage, test:e2e
   - lint, format, typecheck
   - prepare (husky install)

10. **`src-tauri/Cargo.toml`** - Dev dependencies
    - criterion для benchmarks
    - Конфигурация bench targets

11. **`README.md`** - Status badges
    - CI/CD status
    - Code coverage
    - Performance benchmarks

## Проверка соответствия конституции

### ✅ Принцип 1: Качество кода и поддерживаемость

- [x] **Автоматизация rustfmt/ESLint:** CI проверяет форматирование
- [x] **Clippy с warn level:** Настроен `-D warnings` (0 предупреждений)
- [x] **Code review:** Branch protection с required reviews
- [x] **Документация API:** cargo doc генерируется автоматически
- [x] **Модульная архитектура:** Не изменяется (вне scope)
- [x] **Pre-commit hooks:** Локальная проверка перед commit

**Реализация:**
- GitHub Actions job `code-quality` проверяет все требования
- Pre-commit hooks дублируют проверки локально
- Отклонения блокируют merge

### ✅ Принцип 2: Стандарты тестирования

- [x] **70% coverage критической логики:** Tarpaulin с threshold check
- [x] **Unit-тесты для публичных функций:** Проверяется coverage report
- [x] **Integration тесты для Tauri commands:** Cargo test --test '*'
- [x] **Регрессионные тесты:** Процесс документирован
- [x] **Автоматический запуск в CI:** Job `testing` на каждый PR
- [x] **E2E тесты критических сценариев:** Playwright тесты (3-5 scenarios)
- [x] **Mock для внешних зависимостей:** Best practices в документации
- [x] **Быстрые тесты:** Timeout checks в CI

**Реализация:**
- Codecov интеграция для визуализации coverage
- E2E тесты с Playwright на изолированных данных
- Coverage gate блокирует merge при < 70%

### ✅ Принцип 3: Согласованность UX

- [x] **Единая дизайн-система:** Не изменяется (вне scope)
- [x] **UI отклик < 100ms:** Измеряется Lighthouse
- [x] **Индикаторы загрузки:** E2E тесты проверяют наличие
- [x] **Оптимистичные обновления:** Не изменяется (вне scope)
- [x] **Graceful error handling:** E2E тесты проверяют error states
- [x] **Keyboard shortcuts:** Документированы (не изменяется)
- [x] **Состояние между сессиями:** E2E тесты проверяют persistence
- [x] **60 FPS анимации:** Lighthouse frame rate check
- [x] **Тестирование на разных разрешениях:** Playwright viewport tests
- [x] **Локализация:** Не изменяется (вне scope)

**Реализация:**
- E2E тесты покрывают критические user flows
- Lighthouse CI проверяет performance метрики
- Playwright тесты на различных viewport размерах

### ✅ Принцип 4: Производительность

- [x] **Холодный старт < 3s:** Измеряется в performance.yml
- [x] **TTI < 2s:** Lighthouse CI assertion
- [x] **Idle память < 150MB:** Документирован процесс измерения (manual)
- [x] **Загрузка чатов < 500ms:** Benchmark в criterion
- [x] **История сообщений < 300ms:** Benchmark в criterion
- [x] **60 FPS scroll:** Lighthouse performance score
- [x] **Бинарный размер < 50MB:** Build job проверяет bundle size
- [x] **Виртуализация списков:** Не изменяется (вне scope)
- [x] **Lazy loading медиа:** Не изменяется (вне scope)
- [x] **SQLite кэширование:** Не изменяется (вне scope)
- [x] **Профилирование:** Benchmarks + baseline tracking
- [x] **LTO оптимизация:** Проверяется в release build

**Реализация:**
- Criterion benchmarks для критических операций
- Baseline tracking с автоматическим alert при регрессии > 20%
- Lighthouse CI для frontend performance
- Bundle size check в CI

## Альтернативные варианты

### Вариант 1: GitLab CI вместо GitHub Actions

**Плюсы:**
- Встроенный container registry
- Более гибкая конфигурация пайплайнов

**Минусы:**
- Требует миграции с GitHub
- Меньше готовых actions/интеграций
- Дополнительная инфраструктура

**Решение:** Не выбран - проект уже на GitHub, миграция неоправданна

### Вариант 2: Jenkins для CI/CD

**Плюсы:**
- Полный контроль над инфраструктурой
- Богатая экосистема плагинов

**Минусы:**
- Требует self-hosting и обслуживания
- Сложнее настройка
- Дополнительные затраты на инфраструктуру

**Решение:** Не выбран - GitHub Actions проще и бесплатен для open-source

### Вариант 3: Tauri webdriver вместо Playwright

**Плюсы:**
- Нативная поддержка Tauri
- Тестирование именно desktop app

**Минусы:**
- Менее зрелая экосистема
- Сложнее настройка CI
- Меньше документации

**Решение:** Рассмотреть позже - начать с Playwright для быстрого старта

### Вариант 4: Manual performance testing

**Плюсы:**
- Не требует автоматизации
- Гибкость в выборе метрик

**Минусы:**
- Не масштабируется
- Легко пропустить регрессии
- Противоречит принципам конституции

**Решение:** Не выбран - автоматизация критична

## Риски и смягчение

| Риск | Вероятность | Влияние | Смягчение |
| --- | --- | --- | --- |
| Медленный CI блокирует разработку (>15 мин) | High | High | Кэширование dependencies, параллельные jobs, оптимизация тестов |
| Ложные срабатывания тестов (flaky tests) | Medium | High | Retry mechanism, изоляция тестов, review flaky tests |
| Превышение лимитов GitHub Actions (2000 мин/мес) | Medium | Medium | Оптимизация workflows, запуск полных проверок только на PR |
| Сложность настройки для новых разработчиков | Medium | Medium | Подробная документация, автоматизация setup через scripts |
| Coverage 70% недостижим для legacy кода | Low | Medium | Фокус на критической логике, постепенное улучшение |
| Performance benchmarks нестабильны в CI | Medium | Low | Multiple runs, статистический анализ, dedicated runners |

## План реализации

### Фаза 0: Исследование и подготовка (1 день)

**Длительность:** 1 день

- [x] Изучить существующую структуру проекта
- [x] Определить критические операции для benchmarks
- [x] Идентифицировать критические user flows для E2E
- [x] Выбрать инструменты (см. research.md)

### Фаза 1: Базовая инфраструктура качества кода (2-3 дня)

**Длительность:** 2-3 дня

**Priority: Critical** (FR-1, FR-7)

- [ ] **Task 1.1:** Создать `.github/workflows/ci.yml` с job `code-quality`
  - rustfmt check
  - clippy -D warnings
  - ESLint --max-warnings 0
  - Prettier check
  - TypeScript typecheck

- [ ] **Task 1.2:** Настроить pre-commit hooks с Husky
  - Создать `.husky/pre-commit`
  - Добавить скрипты в package.json
  - Протестировать локально

- [ ] **Task 1.3:** Создать конфигурационные файлы
  - `.eslintrc.json` (max-warnings: 0)
  - `.prettierrc.json`
  - Обновить `rustfmt.toml` если нужно

- [ ] **Task 1.4:** Привести существующий код к стандартам
  - Запустить `cargo fmt`
  - Исправить clippy warnings
  - Запустить `prettier --write`
  - Исправить ESLint errors

**Acceptance Criteria:**
- ✅ CI job `code-quality` проходит успешно
- ✅ Pre-commit hooks работают локально
- ✅ 0 warnings в clippy/ESLint

### Фаза 2: Тестирование и покрытие (3-4 дня)

**Длительность:** 3-4 дня

**Priority: High** (FR-2, FR-4)

- [ ] **Task 2.1:** Добавить job `testing` в ci.yml
  - cargo test --all
  - Интеграция tarpaulin для coverage
  - Проверка threshold >= 70%
  - Upload to Codecov

- [ ] **Task 2.2:** Настроить Codecov
  - Создать аккаунт/интеграцию
  - Добавить codecov.yml конфигурацию
  - Настроить badge в README

- [ ] **Task 2.3:** Написать недостающие unit-тесты
  - Проанализировать текущее coverage
  - Написать тесты для критической логики
  - Достичь 70%+ coverage

- [ ] **Task 2.4:** Создать integration тесты
  - Тесты для Tauri commands
  - Тесты взаимодействия с БД
  - Mock для Telegram API

- [ ] **Task 2.5:** Настроить E2E тестирование
  - Установить Playwright
  - Создать `tests/e2e/` структуру
  - Написать 3-5 критических сценариев:
    - Запуск приложения
    - Авторизация (QR/Phone/Code)
    - Отправка сообщения
    - Загрузка истории чата
    - Переключение между чатами
  - Добавить job `e2e-tests` в ci.yml

**Acceptance Criteria:**
- ✅ Coverage >= 70% для критической логики
- ✅ Все integration тесты проходят
- ✅ 3-5 E2E тестов написаны и проходят
- ✅ Codecov badge добавлен в README

### Фаза 3: Performance benchmarking (2-3 дня)

**Длительность:** 2-3 дня

**Priority: High** (FR-3)

- [ ] **Task 3.1:** Определить критические операции для benchmarks
  - Загрузка списка чатов (target: < 500ms для 1000 чатов)
  - Отрисовка истории сообщений (target: < 300ms для 50 сообщений)
  - Отправка текстового сообщения (target: < 50ms оптимистичное)
  - Поиск по сообщениям
  - Загрузка медиа-превью
  - Database queries (CRUD операции)

- [ ] **Task 3.2:** Создать benchmark тесты
  - Добавить criterion в dev-dependencies
  - Создать `src-tauri/benches/performance_benchmarks.rs`
  - Реализовать benchmarks для критических операций
  - Установить baseline метрики

- [ ] **Task 3.3:** Создать `.github/workflows/performance.yml`
  - Job для cargo bench
  - Интеграция github-action-benchmark
  - Alert при регрессии > 20%
  - Comment результатов на PR
  - Weekly scheduled run

- [ ] **Task 3.4:** Добавить startup time test
  - Измерение холодного старта
  - Threshold < 3s
  - Fail CI при превышении

- [ ] **Task 3.5:** Настроить Lighthouse CI
  - Создать `.lighthouserc.json`
  - Добавить job `lighthouse` в performance.yml
  - Assertions: performance > 90, TTI < 2s

**Acceptance Criteria:**
- ✅ 5-10 benchmarks для критических операций
- ✅ Baseline метрики установлены
- ✅ Performance workflow работает
- ✅ Регрессии > 20% блокируют merge

### Фаза 4: Документация и polish (1-2 дня)

**Длительность:** 1-2 дня

**Priority: Medium** (FR-6)

- [ ] **Task 4.1:** Настроить автоматическую генерацию документации
  - cargo doc для Rust API
  - Добавить doc-комментарии для публичных API
  - TypeDoc для TypeScript (опционально)

- [ ] **Task 4.2:** Создать quickstart.md
  - Инструкции по setup окружения
  - Как запускать тесты локально
  - Как работают pre-commit hooks
  - Процесс CI/CD

- [ ] **Task 4.3:** Обновить README.md
  - Добавить status badges (CI, coverage, performance)
  - Ссылка на документацию
  - Описание процесса разработки

- [ ] **Task 4.4:** Создать CONTRIBUTING.md
  - Code style guide
  - Testing requirements
  - PR process
  - Ссылка на конституцию

- [ ] **Task 4.5:** Документировать CI/CD процесс
  - Описание workflows
  - Как интерпретировать результаты
  - Troubleshooting

**Acceptance Criteria:**
- ✅ Все публичные API задокументированы
- ✅ quickstart.md создан
- ✅ README обновлен с badges
- ✅ CONTRIBUTING.md создан

### Фаза 5: Security и build verification (1 день)

**Длительность:** 1 день

**Priority: Medium**

- [ ] **Task 5.1:** Добавить security audit в ci.yml
  - Job `security-audit`
  - cargo audit для Rust
  - pnpm audit для npm dependencies

- [ ] **Task 5.2:** Добавить build verification
  - Job `build` для Linux/Windows/macOS
  - Проверка bundle size < 50MB
  - Smoke test собранного бинарника

- [ ] **Task 5.3:** Настроить branch protection
  - Require status checks перед merge
  - Require code review
  - Restrict force pushes

**Acceptance Criteria:**
- ✅ Security audit проходит без critical issues
- ✅ Build успешен на всех платформах
- ✅ Branch protection настроена

### Фаза 6: Валидация и rollout (1-2 дня)

**Длительность:** 1-2 дня

**Priority: Critical**

- [ ] **Task 6.1:** Полное тестирование CI/CD
  - Создать тестовый PR
  - Проверить все workflows
  - Убедиться, что все gates работают

- [ ] **Task 6.2:** Обучение команды
  - Провести walkthrough новых процессов
  - Демонстрация pre-commit hooks
  - Объяснение CI/CD результатов

- [ ] **Task 6.3:** Сбор обратной связи
  - Опросить команду о сложностях
  - Собрать предложения по улучшению
  - Документировать known issues

- [ ] **Task 6.4:** Финальные adjustments
  - Исправить выявленные проблемы
  - Оптимизировать медленные проверки
  - Обновить документацию

**Acceptance Criteria:**
- ✅ Все workflows проходят успешно
- ✅ Команда обучена
- ✅ Документация финализирована

## Метрики успеха

### Автоматизация (через 2 недели после внедрения)
- **CI pass rate:** > 95% (исключая ожидаемые failures)
- **Mean time to feedback:** < 5 минут от push до результатов
- **False positive rate:** < 5% (flaky tests)

### Качество кода (ongoing)
- **Clippy warnings:** 0 в main branch
- **ESLint errors:** 0 в main branch
- **Code review coverage:** 100% PR требуют review

### Тестирование (через 1 месяц)
- **Test coverage:** >= 70% для критической логики
- **E2E test count:** 3-5 критических сценариев
- **Integration test count:** Все Tauri commands покрыты

### Производительность (ongoing)
- **Benchmark stability:** < 5% вариация между запусками
- **Regression detection rate:** 100% регрессий > 20% обнаружены
- **Cold start time:** < 3s на reference hardware

### Developer experience (через 1 месяц)
- **Setup time для нового dev:** < 1 час
- **Developer satisfaction:** > 8/10 в опросе
- **CI/CD понимание:** 100% команды понимают процесс

## Зависимости

### Внешние сервисы
- **GitHub Actions:** Бесплатный tier (2000 мин/мес для приватных репо)
- **Codecov:** Бесплатный для open-source
- **GitHub:** Branch protection, required reviews

### Инструменты и библиотеки
- **Rust:** rustfmt, clippy, tarpaulin, criterion, cargo-audit
- **Node.js:** ESLint, Prettier, Husky, Playwright, Lighthouse CI
- **CI:** Ubuntu/Windows/macOS runners

### Доступы и права
- **GitHub:** Admin права для настройки branch protection
- **Codecov:** Интеграция с GitHub репозиторием
- **Secrets:** GitHub secrets для sensitive data (если нужно)

### Команда
- **Минимум 1 разработчик** для code reviews
- **Tech lead** для одобрения конфигураций CI/CD

## Вопросы для обсуждения

1. **CI/CD limits:** Укладываемся ли в 2000 мин/мес GitHub Actions? Может потребоваться self-hosted runner.

2. **E2E test data:** Использовать mock Telegram API или test аккаунт? Mock безопаснее, но менее realistic.

3. **Performance baseline:** На каком hardware устанавливать baseline метрики? Нужен reference environment.

4. **Coverage exceptions:** Какие модули можно исключить из 70% requirement (например, legacy код)?

5. **Rollout strategy:** Внедрять все сразу или постепенно (фаза за фазой)? Постепенное снизит риски.

---

**Одобрения:**

- [ ] Tech Lead - Одобрение архитектуры и инструментов
- [ ] Product Owner - Одобрение timeline и приоритетов
- [ ] Security Review - Не требуется (только dev infrastructure)
