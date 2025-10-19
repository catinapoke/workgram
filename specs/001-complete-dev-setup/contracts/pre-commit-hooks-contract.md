# Pre-commit Hooks Contract

**Файл:** `.husky/pre-commit`  
**Версия:** 1.0.0  
**Дата:** 2025-10-19

## Обзор

Локальные Git hooks, выполняемые перед каждым commit. Обеспечивают быструю обратную связь разработчику до отправки кода в репозиторий.

---

## Триггер

**Event:** `git commit`  
**Before:** Commit создается  
**Location:** `.husky/pre-commit` (устанавливается через `husky install`)

---

## Цель

- ✅ Ранняя обратная связь (до push в remote)
- ✅ Экономия времени CI (меньше failed builds)
- ✅ Соответствие Принципу 1 (качество кода)
- ✅ Локальная проверка перед code review

---

## Проверки

### 1. Rust Formatting Check

**Команда:**
```bash
cd src-tauri
cargo fmt -- --check
```

**Назначение:** Проверка соответствия rustfmt стандартам

**Success:** Exit code 0  
**Failure:** Exit code 1, показывает несформатированные файлы

**Исправление:**
```bash
cd src-tauri
cargo fmt
```

**Estimated Time:** 1-2 секунды

---

### 2. Rust Clippy

**Команда:**
```bash
cd src-tauri
cargo clippy --all-targets -- -D warnings
```

**Назначение:** Статический анализ Rust кода (0 warnings policy)

**Success:** Exit code 0, 0 warnings  
**Failure:** Exit code 1, выводит warnings/errors

**Исправление:** Исправить warnings вручную

**Estimated Time:** 3-5 секунд (incremental)

**Примечание:** Полная проверка только измененных файлов (cargo incremental compilation)

---

### 3. Frontend Formatting Check

**Команда:**
```bash
pnpm prettier --check "src/**/*.{ts,tsx,css}"
```

**Назначение:** Проверка форматирования TypeScript/React/CSS

**Success:** Exit code 0  
**Failure:** Exit code 1, показывает несформатированные файлы

**Исправление:**
```bash
pnpm prettier --write "src/**/*.{ts,tsx,css}"
# или
pnpm format
```

**Estimated Time:** 1-2 секунды

---

### 4. ESLint

**Команда:**
```bash
pnpm eslint src/ --ext .ts,.tsx --max-warnings 0
```

**Назначение:** Линтинг TypeScript/React (0 warnings policy)

**Success:** Exit code 0, 0 warnings  
**Failure:** Exit code 1, выводит errors/warnings

**Исправление:**
```bash
pnpm eslint src/ --ext .ts,.tsx --fix
# или
pnpm lint:fix
```

**Estimated Time:** 2-3 секунды

---

## Полный Script

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."
echo ""

# Track failures
FAILED=0

# ============================================
# RUST CHECKS
# ============================================

echo "📝 Checking Rust formatting..."
cd src-tauri
if ! cargo fmt -- --check 2>&1; then
  echo "❌ Rust formatting check failed!"
  echo "   Run: cd src-tauri && cargo fmt"
  FAILED=1
else
  echo "✅ Rust formatting OK"
fi

echo ""
echo "🔍 Running Clippy..."
if ! cargo clippy --all-targets -- -D warnings 2>&1; then
  echo "❌ Clippy found issues!"
  echo "   Fix warnings before committing"
  FAILED=1
else
  echo "✅ Clippy OK"
fi

cd ..

# ============================================
# FRONTEND CHECKS
# ============================================

echo ""
echo "📝 Checking frontend formatting..."
if ! pnpm prettier --check "src/**/*.{ts,tsx,css}" 2>&1; then
  echo "❌ Frontend formatting check failed!"
  echo "   Run: pnpm format"
  FAILED=1
else
  echo "✅ Frontend formatting OK"
fi

echo ""
echo "🔍 Running ESLint..."
if ! pnpm eslint src/ --ext .ts,.tsx --max-warnings 0 2>&1; then
  echo "❌ ESLint found issues!"
  echo "   Run: pnpm lint:fix"
  FAILED=1
else
  echo "✅ ESLint OK"
fi

# ============================================
# RESULT
# ============================================

echo ""
if [ $FAILED -eq 1 ]; then
  echo "❌ Pre-commit checks FAILED"
  echo ""
  echo "Please fix the issues above and try again."
  echo "Or use --no-verify to skip (NOT RECOMMENDED)"
  exit 1
fi

echo "✅ All pre-commit checks PASSED!"
echo ""
```

---

## Performance Targets

**Total time:** < 30 секунд (NFR-1 из спецификации)

**Breakdown:**
- Rust formatting: ~1-2s
- Clippy: ~3-5s (incremental)
- Prettier check: ~1-2s
- ESLint: ~2-3s
- **Total:** ~7-12s (nominal)

**Если > 30s → оптимизация required**

---

## Bypass Mechanism

**Команда:**
```bash
git commit --no-verify -m "message"
# или
git commit -n -m "message"
```

**Использование:**
- ⚠️ Emergency hotfixes
- ⚠️ WIP commits в feature branch
- ❌ НЕ использовать для обхода legitimate issues

**Policy:** Discouraged, должен быть exception, не правило

---

## Установка

### Первоначальная настройка

```bash
# Install husky
pnpm add -D husky

# Initialize husky
pnpm husky install

# Add prepare script (автоматическая установка для новых devs)
npm pkg set scripts.prepare="husky install"

# Create pre-commit hook
pnpm husky add .husky/pre-commit "# ... script content ..."

# Make executable
chmod +x .husky/pre-commit
```

### Для новых разработчиков

```bash
# После git clone и pnpm install
pnpm install  # автоматически запустит prepare script
# Hooks установлены! ✅
```

---

## Troubleshooting

### Проблема: Hooks не срабатывают

**Причины:**
1. Не запущен `husky install`
2. `.husky/` не имеет executable permissions
3. Git hooks disabled globally

**Решение:**
```bash
pnpm husky install
chmod +x .husky/pre-commit
git config --get core.hooksPath  # Должен быть .husky
```

---

### Проблема: Медленные проверки (> 30s)

**Причины:**
1. Cargo не использует incremental compilation
2. Checking all files вместо staged
3. Node modules не кэшированы

**Решение:**
- Используй `lint-staged` для проверки только staged files
- Убедись в incremental compilation для cargo
- Cache node_modules правильно

---

### Проблема: False positives

**Причины:**
1. Outdated tools (clippy, ESLint rules)
2. Конфликтующие конфигурации

**Решение:**
- Update tools: `rustup update`, `pnpm update`
- Review `.eslintrc.json` и `Cargo.toml` lint config
- Create exceptions если legitimate (документируй)

---

## Опциональная оптимизация: lint-staged

**Для больших проектов:** Проверять только staged files

**Конфигурация (package.json):**
```json
{
  "lint-staged": {
    "src-tauri/**/*.rs": [
      "cargo fmt --",
      "cargo clippy --"
    ],
    "src/**/*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "src/**/*.css": [
      "prettier --write"
    ]
  }
}
```

**Pre-commit hook (simplified):**
```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

pnpm lint-staged
```

**Преимущества:**
- Быстрее (только staged files)
- Автоматическое исправление (--fix, --write)

**Недостатки:**
- Не проверяет весь проект
- Может пропустить cross-file issues

---

## Integration с CI

**Guarantee:** Те же проверки, что и в CI job `code-quality`

**Sync:**
- Pre-commit hooks === CI checks
- Если проходит локально → пройдет в CI
- Если fails в CI но прошел локально → sync issue, нужен fix

---

## Соответствие конституции

### Принцип 1: Качество кода
- ✅ rustfmt check
- ✅ clippy -D warnings
- ✅ ESLint --max-warnings 0
- ✅ Prettier formatting

### Принцип 2: Тестирование
- ⚠️ Tests не запускаются в pre-commit (too slow)
- ✅ Tests запускаются в CI

### NFR-1: Performance
- ✅ Target: < 30 секунд
- ✅ Actual: ~10 секунд (nominal)

---

## Versioning

**Version:** 1.0.0 (Initial)

**Changelog:**
- 2025-10-19: Initial with Rust + Frontend checks

**Future Enhancements:**
- lint-staged integration
- Parallel execution (Rust + Frontend checks)
- Skip checks для docs-only changes
- Git LFS hooks (если нужны)

