# CI Workflow Contract

**Файл:** `.github/workflows/ci.yml`  
**Версия:** 1.0.0  
**Дата:** 2025-10-19

## Обзор

Основной CI пайплайн для проверки качества кода, запуска тестов и верификации сборки. Запускается автоматически на каждый push в `main`/`develop` и каждый Pull Request.

---

## Триггеры

```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
```

**Поведение:**
- Push в `main` или `develop` → полная проверка
- Pull Request → полная проверка + комментарии с результатами
- Другие ветки → проверка только при создании PR

---

## Jobs

### Job 1: `code-quality`

**Назначение:** Проверка стиля и качества кода (Принцип 1)

**Runner:** `ubuntu-latest`

**Шаги:**
1. Checkout code
2. Setup Rust (stable, rustfmt, clippy)
3. Cache cargo registry
4. Check Rust formatting: `cargo fmt -- --check`
5. Run Clippy: `cargo clippy --all-targets --all-features -- -D warnings`
6. Setup Node.js 18
7. Install pnpm
8. Install frontend dependencies
9. Check TypeScript types: `pnpm tsc --noEmit`
10. Run ESLint: `pnpm eslint src/ --ext .ts,.tsx --max-warnings 0`
11. Check Prettier: `pnpm prettier --check "src/**/*.{ts,tsx,css}"`

**Success Criteria:**
- ✅ All checks return exit code 0
- ✅ 0 warnings from clippy
- ✅ 0 warnings from ESLint
- ✅ All files properly formatted

**Failure Behavior:**
- ❌ Job fails → блокирует merge
- 📝 GitHub annotations показывают конкретные проблемы

**Estimated Duration:** 3-5 минут (с кэшированием)

---

### Job 2: `testing`

**Назначение:** Запуск unit и integration тестов, проверка coverage (Принцип 2)

**Runner:** `ubuntu-latest`

**Шаги:**
1. Checkout code
2. Setup Rust (stable)
3. Run Rust unit tests: `cargo test --all --verbose`
4. Generate coverage with tarpaulin
5. Check coverage threshold (>= 70%)
6. Upload coverage to Codecov (Rust)
7. Setup Node.js + pnpm
8. Install dependencies
9. Run frontend unit tests: `pnpm test --coverage`
10. Upload frontend coverage to Codecov

**Success Criteria:**
- ✅ All tests pass (exit code 0)
- ✅ Coverage >= 70% для критических модулей
- ✅ Codecov upload успешен

**Failure Behavior:**
- ❌ Test failures → job fails, блокирует merge
- ❌ Coverage < 70% → job fails
- 📊 Coverage report available in Codecov

**Estimated Duration:** 5-8 минут

**Outputs:**
- `coverage/cobertura.xml` (Rust)
- `coverage/coverage-final.json` (Frontend)

---

### Job 3: `integration-tests`

**Назначение:** Integration тесты на разных платформах (Принцип 2)

**Runner:** Matrix strategy
- `ubuntu-latest`
- `windows-latest`
- `macos-latest`

**Шаги:**
1. Checkout code
2. Setup Rust (stable)
3. Run integration tests: `cargo test --test '*' -- --test-threads=1`

**Success Criteria:**
- ✅ Tests pass on all platforms
- ✅ Duration < 10 минут

**Failure Behavior:**
- ❌ Platform-specific failures показываются отдельно
- 🔄 Retry при flaky tests (опционально)

**Estimated Duration:** 5-10 минут per platform (параллельно)

---

### Job 4: `e2e-tests`

**Назначение:** End-to-End тесты критических user flows (Принцип 3)

**Runner:** `ubuntu-latest`

**Шаги:**
1. Checkout code
2. Setup Node.js + pnpm
3. Install dependencies
4. Install Playwright: `pnpm exec playwright install --with-deps`
5. Build Tauri app (debug mode)
6. Run E2E tests: `pnpm test:e2e`
7. Upload screenshots on failure

**Success Criteria:**
- ✅ All E2E scenarios pass
- ✅ Duration < 10 минут
- ✅ No flaky tests

**Failure Behavior:**
- ❌ Scenario failures → job fails
- 📸 Screenshots uploaded as artifacts
- 🎥 Video recording (optional)

**Test Scenarios:**
- App launch
- User authentication flow
- Send message
- Load chat history
- Switch between chats

**Estimated Duration:** 8-10 минут

---

### Job 5: `security-audit`

**Назначение:** Проверка безопасности dependencies

**Runner:** `ubuntu-latest`

**Шаги:**
1. Checkout code
2. Install cargo-audit
3. Run Rust audit: `cargo audit`
4. Install pnpm
5. Run npm audit: `pnpm audit --audit-level=moderate`

**Success Criteria:**
- ✅ 0 critical vulnerabilities
- ✅ 0 high vulnerabilities (желательно)

**Failure Behavior:**
- ❌ Critical/High vulns → job fails
- ⚠️ Medium vulns → warning (не блокирует)
- 📋 SARIF report uploaded to GitHub Security

**Estimated Duration:** 2-3 минуты

---

### Job 6: `build`

**Назначение:** Верификация сборки на всех платформах

**Runner:** Matrix strategy
- `ubuntu-latest`
- `windows-latest`
- `macos-latest`

**Шаги:**
1. Checkout code
2. Setup Rust (stable)
3. Setup Node.js + pnpm
4. Install dependencies
5. Build Tauri app (release): `pnpm tauri build`
6. Check bundle size < 50 MB (только Linux)

**Success Criteria:**
- ✅ Build successful на всех платформах
- ✅ Bundle size < 50 MB (Принцип 4)
- ✅ No build warnings (Rust: -D warnings)

**Failure Behavior:**
- ❌ Build failures → блокирует merge
- ❌ Bundle size > 50MB → блокирует merge

**Estimated Duration:** 10-15 минут per platform (параллельно)

**Artifacts:**
- Build logs
- Bundle size report

---

## Общие настройки

### Environment Variables

```yaml
env:
  CARGO_TERM_COLOR: always
  RUST_BACKTRACE: 1
```

### Caching Strategy

**Cargo cache:**
```yaml
- uses: actions/cache@v3
  with:
    path: ~/.cargo/registry
    key: ${{ runner.os }}-cargo-registry-${{ hashFiles('**/Cargo.lock') }}
```

**Node modules cache:**
- Автоматическое через `pnpm/action-setup`

### Timeout

**Per-job timeout:** 20 минут
**Full workflow timeout:** 45 минут

---

## Success Criteria для Merge

Для разрешения merge в `main` branch, ALL jobs ДОЛЖНЫ пройти:

- ✅ `code-quality` - PASSED
- ✅ `testing` - PASSED (coverage >= 70%)
- ✅ `integration-tests` - PASSED (all platforms)
- ✅ `e2e-tests` - PASSED
- ✅ `security-audit` - PASSED (0 critical/high)
- ✅ `build` - PASSED (all platforms, size < 50MB)

**Branch Protection Rules:**
- Require status checks to pass before merging
- Require branches to be up to date before merging
- Require code review from 1 maintainer

---

## Notifications

**On Failure:**
- 📧 Email to commit author
- 💬 GitHub PR comment with details
- 🔔 GitHub notifications

**On Success:**
- ✅ Green checkmark in PR
- 📊 Coverage report comment (Codecov bot)

---

## Performance Metrics

**Target Times:**
- `code-quality`: < 5 минут
- `testing`: < 8 минут
- `integration-tests`: < 10 минут per platform
- `e2e-tests`: < 10 минут
- `security-audit`: < 3 минуты
- `build`: < 15 минут per platform

**Total (parallelized):** < 15 минут для всего workflow

**Если превышает 15 минут → оптимизация required**

---

## Error Handling

### Flaky Tests
- Retry mechanism: 2 попытки
- Если 2+ failures → report issue

### Infrastructure Failures
- GitHub Actions outage → retry автоматически
- Self-hosted runner issues → fallback to GitHub runners

---

## Versioning

**Version:** 1.0.0 (Initial)

**Changelog:**
- 2025-10-19: Initial version with all 6 jobs

**Future Enhancements:**
- Automated rollback on failure
- Canary deployments
- Performance regression tracking in CI

