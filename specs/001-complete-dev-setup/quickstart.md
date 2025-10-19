# Quickstart: Разработка с новой CI/CD инфраструктурой

**Дата:** 2025-10-19  
**Версия:** 1.0.0

## Обзор

Это руководство описывает, как работать с новой инфраструктурой разработки Workgram после внедрения автоматизированных проверок качества, тестирования и performance benchmarking.

---

## Для новых разработчиков

### 1. Начальная настройка окружения

#### Требования

- **Git:** >= 2.30
- **Rust:** >= 1.70 (рекомендуется stable latest)
- **Node.js:** >= 18
- **pnpm:** >= 8

#### Установка зависимостей

```bash
# 1. Клонировать репозиторий
git clone https://github.com/your-org/workgram.git
cd workgram

# 2. Установить Rust components
rustup component add rustfmt clippy

# 3. Установить Node.js зависимости
pnpm install

# 4. Установить git hooks (автоматически через prepare script)
# Это уже произошло при pnpm install, но можно проверить:
pnpm husky install

# 5. Проверить настройку
pnpm run typecheck  # Проверка TypeScript
cd src-tauri && cargo clippy  # Проверка Rust
```

**Время настройки:** ~15-20 минут (первый раз)

---

### 2. Проверка работоспособности

```bash
# Запустить все локальные проверки
pnpm lint              # ESLint
pnpm format:check      # Prettier check
pnpm typecheck         # TypeScript check

cd src-tauri
cargo fmt -- --check   # Rust formatting
cargo clippy           # Rust linting
cargo test             # Rust tests

# Если все прошло успешно → окружение готово! ✅
```

---

## Ежедневный workflow

### 1. Создание новой ветки

```bash
# Всегда начинать с актуального main
git checkout main
git pull origin main

# Создать feature branch
git checkout -b feature/my-awesome-feature
```

---

### 2. Разработка и коммиты

#### Написать код

```bash
# Редактировать файлы...
# src/components/MyComponent.tsx
# src-tauri/src/commands/my_command.rs
```

#### Перед коммитом (рекомендуется)

```bash
# Автоформатирование
pnpm format                    # Frontend
cd src-tauri && cargo fmt      # Rust

# Проверить, что нет ошибок
pnpm lint                      # Frontend
cd src-tauri && cargo clippy   # Rust
```

#### Коммит

```bash
git add .
git commit -m "feat: add awesome feature"

# 🔍 Pre-commit hooks запустятся автоматически!
# Проверят:
#  - Rust formatting (cargo fmt --check)
#  - Clippy warnings (cargo clippy -D warnings)
#  - Frontend formatting (prettier --check)
#  - ESLint (max-warnings 0)
#
# Если что-то не так → commit будет заблокирован
# Исправить проблемы и повторить commit
```

**⏱️ Pre-commit checks время:** ~10 секунд

---

### 3. Исправление pre-commit ошибок

#### Rust formatting errors

```bash
cd src-tauri
cargo fmt
git add .
git commit -m "..."
```

#### Clippy warnings

```bash
# Исправить предупреждения вручную
# Clippy подскажет, что исправить

cd src-tauri
cargo clippy  # Проверить снова
```

#### Frontend formatting errors

```bash
pnpm format   # Автоматическое форматирование
git add .
git commit -m "..."
```

#### ESLint errors

```bash
pnpm lint:fix  # Автоматическое исправление
# Некоторые ошибки требуют ручного исправления
git add .
git commit -m "..."
```

---

### 4. Отправка в remote

```bash
git push origin feature/my-awesome-feature
```

**После push:**
- 🤖 GitHub Actions автоматически запустит CI pipeline
- ⏱️ Результаты через ~5-15 минут
- 📧 Notification по email при failure

---

### 5. Создание Pull Request

#### Через GitHub Web UI

1. Перейти в репозиторий на GitHub
2. Нажать "Compare & pull request"
3. Заполнить описание PR
4. Нажать "Create pull request"

#### Что происходит автоматически

**CI Checks запускаются:**
- ✅ Code quality (rustfmt, clippy, ESLint, Prettier)
- ✅ Tests (unit, integration, E2E)
- ✅ Coverage check (>= 70%)
- ✅ Security audit
- ✅ Build verification (all platforms)
- ✅ Performance benchmarks

**Время выполнения:** ~10-15 минут

---

### 6. Проверка CI результатов

#### Все проверки прошли ✅

```
✅ code-quality        5 min
✅ testing            7 min
✅ integration-tests  8 min
✅ e2e-tests          9 min
✅ security-audit     2 min
✅ build             12 min
✅ benchmark          4 min
```

**→ Ready для code review!**

#### Что-то провалилось ❌

**1. Кликнуть на failed job**

**2. Прочитать logs**

**3. Исправить локально:**

```bash
# Checkout PR branch
git checkout feature/my-awesome-feature

# Исправить проблему
# ...

# Commit fix
git add .
git commit -m "fix: address CI feedback"

# Push
git push origin feature/my-awesome-feature

# CI автоматически перезапустится
```

---

## Запуск тестов локально

### Unit тесты (Rust)

```bash
cd src-tauri

# Все тесты
cargo test

# Конкретный модуль
cargo test commands::sign

# С выводом println!
cargo test -- --nocapture

# Один тест
cargo test test_function_name
```

### Unit тесты (Frontend)

```bash
# Все тесты
pnpm test

# Watch mode
pnpm test --watch

# С coverage
pnpm test --coverage

# Конкретный файл
pnpm test MyComponent.test.tsx
```

### Integration тесты (Rust)

```bash
cd src-tauri

# Все integration тесты
cargo test --test '*'

# Конкретный integration test
cargo test --test database_integration
```

### E2E тесты (Playwright)

```bash
# Установить Playwright browsers (первый раз)
pnpm exec playwright install

# Запустить E2E тесты
pnpm test:e2e

# Headed mode (видеть браузер)
pnpm test:e2e --headed

# Debug mode
pnpm test:e2e --debug

# Конкретный тест
pnpm test:e2e auth.spec.ts
```

---

## Coverage локально

### Rust coverage

```bash
cd src-tauri

# Установить tarpaulin (первый раз)
cargo install cargo-tarpaulin

# Генерировать coverage
cargo tarpaulin --out Html --output-dir coverage

# Открыть отчет
# Linux/Mac:
open coverage/index.html
# Windows:
start coverage/index.html
```

### Frontend coverage

```bash
# Генерировать coverage
pnpm test --coverage

# Отчет в coverage/index.html
open coverage/index.html  # Mac
xdg-open coverage/index.html  # Linux
start coverage/index.html  # Windows
```

---

## Performance benchmarks локально

### Rust benchmarks

```bash
cd src-tauri

# Запустить все benchmarks
cargo bench

# Конкретный benchmark
cargo bench load_chats

# Результаты в target/criterion/
```

### Frontend performance (Lighthouse)

```bash
# Build production
pnpm build

# Запустить preview server
pnpm preview &

# В другом терминале:
pnpm exec lighthouse http://localhost:1420 --view
```

---

## Обход pre-commit hooks (не рекомендуется)

### Когда использовать

- ⚠️ Emergency hotfix
- ⚠️ WIP commit в feature branch
- ❌ НЕ для обхода legitimate issues

### Команда

```bash
git commit --no-verify -m "WIP: work in progress"
# или
git commit -n -m "..."
```

**⚠️ Важно:** CI все равно проверит код, поэтому это только откладывает feedback

---

## Troubleshooting

### Проблема: Pre-commit hooks не работают

**Симптом:** Commit проходит без проверок

**Решение:**
```bash
# Переустановить hooks
pnpm husky install

# Проверить permissions
chmod +x .husky/pre-commit

# Проверить Git config
git config core.hooksPath
# Должно быть: .husky
```

---

### Проблема: Clippy warnings в CI, но не локально

**Причина:** Разные версии Rust или clippy

**Решение:**
```bash
# Обновить Rust toolchain
rustup update stable

# Обновить clippy
rustup component add clippy --force

# Проверить версию
rustc --version  # Должна совпадать с CI
```

---

### Проблема: Tests проходят локально, fails в CI

**Причины:**
1. Platform-specific issue (Linux vs Windows/Mac)
2. Разные environment variables
3. Race conditions в тестах

**Решение:**
```bash
# Запустить тесты с теми же флагами что в CI
cargo test --all --verbose

# Проверить flaky tests
cargo test -- --test-threads=1

# Добавить debug output
RUST_LOG=debug cargo test
```

---

### Проблема: Coverage < 70% блокирует PR

**Решение:**

1. **Проверить coverage локально:**
   ```bash
   cargo tarpaulin --out Html --output-dir coverage
   open coverage/index.html
   ```

2. **Идентифицировать uncovered code:**
   - Красные линии = не покрыты

3. **Написать тесты:**
   ```rust
   #[cfg(test)]
   mod tests {
       #[test]
       fn test_my_function() {
           // ...
       }
   }
   ```

4. **Пересчитать coverage:**
   ```bash
   cargo tarpaulin --out Html
   ```

5. **Push новые тесты:**
   ```bash
   git add .
   git commit -m "test: add coverage for my_function"
   git push
   ```

---

### Проблема: E2E тесты unstable (flaky)

**Симптомы:** Проходят локально, fails в CI (или наоборот)

**Решение:**

1. **Добавить explicit waits:**
   ```typescript
   await page.waitForSelector('#element');
   await page.waitForLoadState('networkidle');
   ```

2. **Увеличить timeouts:**
   ```typescript
   test.setTimeout(60000);  // 60 seconds
   ```

3. **Изолировать тесты:**
   - Не полагаться на порядок выполнения
   - Очищать state между тестами

4. **Запустить многократно локально:**
   ```bash
   for i in {1..10}; do pnpm test:e2e; done
   ```

---

### Проблема: Benchmark regression в CI

**Симптом:** PR заблокирован due to performance regression > 20%

**Решение:**

1. **Проверить benchmark локально:**
   ```bash
   cd src-tauri
   cargo bench
   ```

2. **Profile hot paths:**
   ```bash
   # Install flamegraph
   cargo install flamegraph

   # Generate flamegraph
   cargo flamegraph --bench performance_benchmarks

   # Открыть flamegraph.svg
   ```

3. **Оптимизировать:**
   - Reduce allocations
   - Optimize database queries
   - Cache expensive operations
   - Use more efficient algorithms

4. **Verify improvement:**
   ```bash
   cargo bench
   # Compare results
   ```

5. **Если legitimate (новая функциональность):**
   - Обновить baseline (с одобрения team lead)
   - Документировать причину regression

---

## Best Practices

### Commits

✅ **DO:**
- Делать частые small commits
- Писать descriptive commit messages
- Использовать conventional commits (feat:, fix:, test:, docs:)

❌ **DON'T:**
- Коммитить неформатированный код
- Коммитить с warnings
- Делать huge commits (>500 lines changed)

### Testing

✅ **DO:**
- Писать тесты вместе с кодом
- Тестировать edge cases
- Проверять coverage локально

❌ **DON'T:**
- Пропускать тесты (#[ignore])
- Писать flaky тесты
- Полагаться только на E2E (писать unit tests)

### Performance

✅ **DO:**
- Запускать benchmarks перед оптимизацией
- Профилировать перед оптимизацией
- Документировать performance-critical code

❌ **DON'T:**
- Преждевременная оптимизация
- Оптимизировать без измерений
- Жертвовать читаемостью ради микрооптимизаций

---

## Полезные команды (шпаргалка)

```bash
# ============================================
# ЛОКАЛЬНЫЕ ПРОВЕРКИ
# ============================================

# Rust
cd src-tauri
cargo fmt                       # Форматирование
cargo clippy                    # Линтинг
cargo test                      # Тесты
cargo bench                     # Benchmarks
cargo tarpaulin --out Html      # Coverage

# Frontend
pnpm format                     # Форматирование
pnpm lint                       # Линтинг
pnpm typecheck                  # TypeScript check
pnpm test                       # Тесты
pnpm test --coverage            # Coverage
pnpm test:e2e                   # E2E тесты

# ============================================
# GIT WORKFLOW
# ============================================

git checkout -b feature/name    # Новая ветка
git add .                       # Stage changes
git commit -m "feat: ..."       # Commit (pre-commit hooks)
git push origin feature/name    # Push (triggers CI)

# ============================================
# TROUBLESHOOTING
# ============================================

# Переустановить hooks
pnpm husky install

# Обновить Rust
rustup update stable

# Очистить cargo cache
cargo clean

# Очистить node_modules
rm -rf node_modules && pnpm install

# Проверить CI локально (act)
act -l  # List workflows
```

---

## Получение помощи

### Документация
- **Конституция проекта:** `.specify/CONSTITUTION.md`
- **CI/CD интеграция:** `.specify/CI-CD-INTEGRATION.md`
- **Workflow контракты:** `specs/001-complete-dev-setup/contracts/`

### Команда
- **Вопросы:** GitHub Discussions
- **Баги CI/CD:** GitHub Issues с label `ci/cd`
- **Slack:** #workgram-dev channel

### External Resources
- [GitHub Actions Docs](https://docs.github.com/actions)
- [Rust Testing Guide](https://doc.rust-lang.org/book/ch11-00-testing.html)
- [Playwright Docs](https://playwright.dev/docs/intro)
- [Criterion Benchmarking](https://bheisler.github.io/criterion.rs/book/)

---

**Последнее обновление:** 2025-10-19  
**Версия:** 1.0.0

**Обратная связь:** Если что-то в этом руководстве неясно или неработает, пожалуйста, создайте issue или PR для улучшения документации!

