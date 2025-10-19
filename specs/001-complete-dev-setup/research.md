# Исследование: Завершение настройки среды разработки

**Дата:** 2025-10-19  
**Автор:** AI Assistant  
**Статус:** Complete

## Обзор

Этот документ содержит результаты исследования инструментов и best practices для автоматизации разработки проекта Workgram. Все решения принимались с учетом требований конституции проекта и специфики стека (Rust + Tauri + TypeScript/React).

---

## 1. CI/CD Платформа

### Исследованный вопрос
Какую CI/CD платформу использовать для автоматизации проверок качества, тестирования и benchmarks?

### Решение: GitHub Actions

### Обоснование
- **Нативная интеграция:** Проект уже hosted на GitHub
- **Бесплатный tier:** 2000 минут/месяц для приватных репо, unlimited для public
- **Богатая экосистема:** Готовые actions для Rust, Node.js, Tauri
- **Простота настройки:** YAML конфигурация, хорошая документация
- **Matrix builds:** Поддержка параллельного тестирования на Linux/Windows/macOS

### Рассмотренные альтернативы

**GitLab CI:**
- ✅ Более гибкая конфигурация
- ✅ Встроенный container registry
- ❌ Требует миграцию с GitHub
- ❌ Меньше готовых интеграций для Rust/Tauri
- **Вердикт:** Избыточно для текущих нужд

**Travis CI:**
- ✅ Популярен в Rust community
- ❌ Ограниченный бесплатный tier
- ❌ Меньше активного развития
- **Вердикт:** GitHub Actions более современная альтернатива

**Jenkins:**
- ✅ Полный контроль
- ❌ Требует self-hosting
- ❌ Сложная настройка
- ❌ Дополнительные операционные затраты
- **Вердикт:** Overkill для проекта такого масштаба

---

## 2. Rust Code Quality Tools

### Исследованный вопрос
Какие инструменты использовать для проверки качества Rust кода и как их настроить?

### Решение: rustfmt + clippy + cargo-audit

### Обоснование

**rustfmt (форматирование):**
- Официальный форматтер Rust
- Детерминированный вывод
- Конфигурация через `rustfmt.toml`
- Запуск: `cargo fmt --check` в CI

**clippy (линтинг):**
- Официальный linter от Rust team
- 550+ правил для выявления проблем
- Настройка строгости: `-D warnings` (0 warnings policy)
- Интеграция с cargo: `cargo clippy --all-targets --all-features`

**cargo-audit (security):**
- Проверка dependencies на известные уязвимости
- RustSec Advisory Database
- Запуск: `cargo audit` в CI job

### Конфигурация

**Clippy настройки (Cargo.toml):**
```toml
[lints.clippy]
# Принцип 1: Качество кода
all = "warn"
pedantic = "warn"
# Исключения (если нужны):
# module_name_repetitions = "allow"
```

**Rustfmt (rustfmt.toml):**
```toml
edition = "2021"
max_width = 100
tab_spaces = 4
```

### Рассмотренные альтернативы

**cargo-deny:**
- ✅ Более строгие проверки licenses и dependencies
- ❌ Может быть too strict для начала
- **Вердикт:** Рассмотреть позже

---

## 3. Test Coverage Tools для Rust

### Исследованный вопрос
Как измерять code coverage для Rust кода и достичь цели 70%?

### Решение: tarpaulin + Codecov

### Обоснование

**tarpaulin:**
- Специализирован для Rust
- Работает на Linux (native)
- Генерирует Cobertura XML для Codecov
- GitHub Action доступен: `actions-rs/tarpaulin`
- Команда: `cargo tarpaulin --workspace --timeout 300 --out Xml`

**Codecov (визуализация):**
- Бесплатен для open-source
- Красивые отчеты и badges
- Pull request comments с diff coverage
- Trend tracking
- GitHub Action: `codecov/codecov-action@v3`

### Конфигурация

**Tarpaulin (tarpaulin.toml):**
```toml
[report]
coveralls = false
out = ["Xml", "Html"]

[run]
timeout = "5m"
exclude-files = [
    "src/main.rs",
    "*/tests/*"
]
```

**Coverage threshold check (bash):**
```bash
coverage=$(grep -oP 'line-rate="\K[^"]+' coverage/cobertura.xml | head -1)
if (( $(echo "$coverage < 0.70" | bc -l) )); then
    echo "Coverage below 70%"
    exit 1
fi
```

### Рассмотренные альтернативы

**grcov (Mozilla):**
- ✅ Более точное измерение
- ❌ Сложнее настройка
- ❌ Требует nightly Rust
- **Вердикт:** Tarpaulin проще для начала

**llvm-cov:**
- ✅ Официальная поддержка в Rust 1.60+
- ❌ Относительно новое
- ❌ Меньше документации
- **Вердикт:** Рассмотреть в будущем

---

## 4. E2E Testing Framework

### Исследованный вопрос
Какой framework использовать для E2E тестирования Tauri desktop приложения?

### Решение: Playwright (начальный этап)

### Обоснование

**Playwright:**
- ✅ Зрелая экосистема, активное развитие
- ✅ Отличная документация и примеры
- ✅ Может тестировать web view внутри Tauri
- ✅ Built-in screenshots, video recording
- ✅ Параллельное выполнение тестов
- ✅ GitHub Actions integration
- ❌ Не native Tauri testing (тестирует WebView, а не window manager)

**Конфигурация:**
```typescript
// playwright.config.ts
export default {
  testDir: './tests/e2e',
  timeout: 30000,
  use: {
    headless: true,
    viewport: { width: 1280, height: 720 },
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
}
```

### Рассмотренные альтернативы

**Tauri WebDriver:**
- ✅ Native Tauri support
- ✅ Тестирует desktop-specific функции (window, menu, etc)
- ❌ Менее зрелая экосистема
- ❌ Сложнее настройка CI
- ❌ Меньше примеров и документации
- **Вердикт:** Migration path - начать с Playwright, позже добавить Tauri WebDriver

**Cypress:**
- ✅ Популярен для web apps
- ❌ Не поддерживает desktop apps
- ❌ Хуже для Electron/Tauri
- **Вердикт:** Не подходит

**Selenium:**
- ✅ Долгая история, стабильность
- ❌ Более громоздкий
- ❌ Медленнее Playwright
- **Вердикт:** Playwright современнее

### Migration Path

**Phase 1 (сейчас):** Playwright для web view тестирования
- Тестируем UI логику
- Быстрый старт
- Хорошая документация

**Phase 2 (будущее):** Добавить Tauri WebDriver
- Тестируем desktop-specific функции
- Window management
- System tray
- Native dialogs

---

## 5. Performance Benchmarking

### Исследованный вопрос
Как измерять производительность критических операций и отслеживать регрессии?

### Решение: Criterion (Rust) + github-action-benchmark

### Обоснование

**Criterion:**
- Де-факто стандарт для Rust benchmarks
- Статистический анализ (outlier detection)
- Warmup и measurement phases
- HTML отчеты с графиками
- Сравнение с baseline

**Конфигурация:**
```toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "performance_benchmarks"
harness = false
```

**Пример benchmark:**
```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn benchmark_load_chats(c: &mut Criterion) {
    c.bench_function("load 1000 chats", |b| {
        b.iter(|| {
            // Target: < 500ms (Принцип 4)
            black_box(load_chats(1000))
        });
    });
}

criterion_group!(benches, benchmark_load_chats);
criterion_main!(benches);
```

**github-action-benchmark:**
- Сохраняет результаты в gh-pages branch
- Автоматические графики трендов
- Alert на регрессии (configurable threshold: 120%)
- Comment на PR с результатами

### Критические операции для benchmark

По анализу конституции (Принцип 4) и спецификации:

1. **Database операции:**
   - Load 1000 chats (target: < 500ms)
   - Load 50 messages (target: < 300ms)
   - Insert message (target: < 50ms)
   - Search messages (target: < 1s)

2. **Telegram API interactions:**
   - Parse incoming update (target: < 10ms)
   - Serialize outgoing message (target: < 5ms)

3. **UI rendering (через Lighthouse):**
   - Cold start (target: < 3s)
   - Time to Interactive (target: < 2s)

### Рассмотренные альтернативы

**Bench (built-in Rust):**
- ✅ No dependencies
- ❌ Требует nightly Rust
- ❌ Нет статистического анализа
- **Вердикт:** Criterion лучше

**Iai (Cachegrind-based):**
- ✅ Deterministic (instruction count)
- ❌ Не измеряет wall-clock time
- ❌ Сложнее интерпретация
- **Вердикт:** Criterion более практичен

---

## 6. Frontend Performance Monitoring

### Исследованный вопрос
Как измерять frontend performance метрики (TTI, FCP, Lighthouse score)?

### Решение: Lighthouse CI

### Обоснование

**Lighthouse CI:**
- Официальный инструмент Google
- Измеряет Core Web Vitals
- Автоматические assertions
- GitHub Action integration
- Trend tracking

**Конфигурация (.lighthouserc.json):**
```json
{
  "ci": {
    "collect": {
      "numberOfRuns": 3,
      "startServerCommand": "pnpm preview",
      "url": ["http://localhost:1420"]
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "first-contentful-paint": ["error", {"maxNumericValue": 2000}],
        "interactive": ["error", {"maxNumericValue": 2000}]
      }
    }
  }
}
```

**Метрики (Принцип 4):**
- Performance score: >= 90
- First Contentful Paint: < 2s
- Time to Interactive: < 2s
- Speed Index: < 3s

### Рассмотренные альтернативы

**WebPageTest:**
- ✅ Более детальный анализ
- ❌ Сложнее автоматизация в CI
- **Вердикт:** Lighthouse достаточно

**Custom performance.mark:**
- ✅ Точечные измерения
- ❌ Требует инструментацию кода
- **Вердикт:** Дополняет Lighthouse, не заменяет

---

## 7. Pre-commit Hooks

### Исследованный вопрос
Как обеспечить локальную проверку качества кода перед commit?

### Решение: Husky + lint-staged

### Обоснование

**Husky:**
- Стандарт для Git hooks в npm ecosystem
- Простая настройка
- Cross-platform (Linux/Mac/Windows)
- Команда: `pnpm prepare` устанавливает hooks

**lint-staged (опционально):**
- Запускает линтеры только на staged files
- Ускоряет проверки
- Конфигурация в package.json

**Конфигурация:**

```json
// package.json
{
  "scripts": {
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.rs": ["cargo fmt --", "cargo clippy --"],
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{css,json}": ["prettier --write"]
  }
}
```

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Pre-commit checks..."

# Rust
cd src-tauri
cargo fmt -- --check || exit 1
cargo clippy --all-targets -- -D warnings || exit 1
cd ..

# Frontend
pnpm prettier --check "src/**/*.{ts,tsx,css}" || exit 1
pnpm eslint src/ --ext .ts,.tsx --max-warnings 0 || exit 1

echo "✅ All checks passed"
```

**Performance target:** < 30 секунд (NFR-1)

### Рассмотренные альтернативы

**pre-commit (Python tool):**
- ✅ Language-agnostic
- ✅ Богатая экосистема hooks
- ❌ Требует Python dependency
- ❌ Менее популярен в JS ecosystem
- **Вердикт:** Husky проще для Node.js проекта

**Git hooks вручную:**
- ✅ No dependencies
- ❌ Сложнее поддержка
- ❌ Не cross-platform
- **Вердикт:** Husky автоматизирует это

---

## 8. Documentation Generation

### Исследованный вопрос
Как генерировать и хостить документацию для Rust и TypeScript кода?

### Решение: cargo doc + docs.rs (Rust), TypeDoc (TypeScript)

### Обоснование

**cargo doc (Rust):**
- Встроенный в Rust toolchain
- Генерирует из /// doc comments
- HTML output с навигацией
- Можно хостить на GitHub Pages
- Команда: `cargo doc --no-deps --open`

**TypeDoc (TypeScript):**
- Стандарт для TS документации
- Генерирует из JSDoc comments
- HTML output
- Интеграция с CI

**Конфигурация:**

```json
// package.json
{
  "scripts": {
    "docs": "typedoc --out docs src/",
    "docs:serve": "pnpm docs && serve docs"
  },
  "devDependencies": {
    "typedoc": "^0.25.0"
  }
}
```

**Хостинг:**
- GitHub Pages (бесплатно)
- Автоматическая публикация через GitHub Action
- URL: `https://username.github.io/workgram/`

### Рассмотренные альтернативы

**docs.rs (для Rust):**
- ✅ Автоматический хостинг для crates.io
- ❌ Проект не публикуется как crate
- **Вердикт:** Self-hosted через GitHub Pages

**Docusaurus:**
- ✅ Красивый UI, много фич
- ❌ Overkill для API docs
- **Вердикт:** Для user docs, не API

---

## 9. Frontend Testing

### Исследованный вопрос
Какой framework использовать для unit тестирования React компонентов?

### Решение: Vitest + React Testing Library

### Обоснование

**Vitest:**
- Совместим с Vite (уже используется в проекте)
- Быстрее Jest (ESM native)
- Совместим с Jest API
- Built-in coverage (c8/istanbul)

**React Testing Library:**
- Стандарт для React тестирования
- Фокус на поведение, а не implementation
- Хорошая документация

**Конфигурация:**
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    environment: 'jsdom',
    coverage: {
      provider: 'c8',
      reporter: ['text', 'json', 'html'],
      lines: 70,
      functions: 70,
      branches: 70,
    },
  },
})
```

### Рассмотренные альтернативы

**Jest:**
- ✅ Более популярен
- ❌ Медленнее Vitest
- ❌ ESM support хуже
- **Вердикт:** Vitest современнее и быстрее

---

## 10. ESLint & Prettier Configuration

### Исследованный вопрос
Какие правила ESLint и настройки Prettier использовать для TypeScript/React?

### Решение: Strict ESLint + Prettier

### Обоснование

**ESLint config:**
```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "react/react-in-jsx-scope": "off",
    "no-console": "warn"
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

**Prettier config:**
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always"
}
```

**max-warnings: 0** - соответствует Принципу 1

---

## Итоговая таблица решений

| Область | Инструмент | Обоснование |
| --- | --- | --- |
| **CI/CD** | GitHub Actions | Нативная интеграция, бесплатно, богатая экосистема |
| **Rust Quality** | rustfmt + clippy + cargo-audit | Официальные инструменты, стандарт индустрии |
| **Rust Coverage** | tarpaulin + Codecov | Специализирован для Rust, отличная визуализация |
| **Rust Benchmarks** | Criterion + github-action-benchmark | Статистический анализ, trend tracking |
| **E2E Tests** | Playwright → Tauri WebDriver | Быстрый старт, migration path к native testing |
| **Frontend Tests** | Vitest + React Testing Library | Быстрее Jest, совместим с Vite |
| **Frontend Performance** | Lighthouse CI | Официальный инструмент, Core Web Vitals |
| **Pre-commit** | Husky + lint-staged | Стандарт npm ecosystem, cross-platform |
| **Docs** | cargo doc + TypeDoc | Встроенные инструменты, GitHub Pages hosting |
| **Frontend Quality** | ESLint + Prettier | Strict config, 0 warnings policy |

---

## Открытые вопросы

### Вопрос 1: GitHub Actions minutes limit
**Статус:** Требует мониторинга

**Контекст:** Бесплатный tier - 2000 минут/месяц для приватных репо. При активной разработке можем превысить.

**Митигация:**
- Кэширование dependencies (cargo cache, node_modules)
- Параллельные jobs (быстрее завершение)
- Conditional runs (полные проверки только на PR к main)
- Self-hosted runner (если понадобится)

### Вопрос 2: E2E test data
**Статус:** Требует решения команды

**Опции:**
- A: Mock Telegram API (безопаснее, но менее realistic)
- B: Test Telegram account (realistic, но требует credentials)
- C: Hybrid: unit tests с mocks, manual E2E на test account

**Рекомендация:** Начать с A (mocks), позже добавить B для smoke tests

### Вопрос 3: Performance baseline hardware
**Статус:** Требует определения

**Проблема:** Benchmarks нестабильны на shared CI runners

**Решение:**
- Использовать относительные метрики (% change)
- Multiple runs + median
- Scheduled benchmarks вместо per-PR (снизит noise)
- Dedicated self-hosted runner для стабильности (будущее)

---

**Дата завершения:** 2025-10-19  
**Все решения готовы к имплементации**

