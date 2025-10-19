# Интеграция CI/CD с конституцией проекта

Этот документ описывает, как настроить CI/CD pipeline для автоматической проверки соответствия принципам конституции.

## Обзор

Конституция проекта Omegram определяет четыре принципа, которые могут быть частично или полностью автоматизированы через CI/CD:

- ✅ **Принцип 1 (Качество кода):** Автоматизируется через линтеры и форматтеры
- ✅ **Принцип 2 (Тестирование):** Автоматизируется через test runners и coverage tools
- ⚠️ **Принцип 3 (UX):** Частично автоматизируется через E2E тесты и performance monitoring
- ✅ **Принцип 4 (Производительность):** Автоматизируется через benchmarks и metrics

## GitHub Actions Workflow (Рекомендуемый)

### Структура файлов

```
.github/
└── workflows/
    ├── ci.yml                 # Основной CI pipeline
    ├── performance.yml        # Performance benchmarks
    └── release.yml            # Release pipeline с полной проверкой
```

## 1. Основной CI Pipeline

**Файл:** `.github/workflows/ci.yml`

```yaml
name: CI - Constitution Compliance

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  CARGO_TERM_COLOR: always
  RUST_BACKTRACE: 1

jobs:
  # Принцип 1: Качество кода
  code-quality:
    name: Code Quality (Principle 1)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          components: rustfmt, clippy
          override: true
      
      - name: Cache cargo registry
        uses: actions/cache@v3
        with:
          path: ~/.cargo/registry
          key: ${{ runner.os }}-cargo-registry-${{ hashFiles('**/Cargo.lock') }}
      
      - name: Check Rust formatting
        run: cargo fmt -- --check
        working-directory: src-tauri
      
      - name: Run Clippy (no warnings allowed)
        run: cargo clippy --all-targets --all-features -- -D warnings
        working-directory: src-tauri
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install frontend dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Check TypeScript types
        run: pnpm tsc --noEmit
      
      - name: Run ESLint
        run: pnpm eslint src/ --ext .ts,.tsx --max-warnings 0
      
      - name: Check Prettier formatting
        run: pnpm prettier --check "src/**/*.{ts,tsx,css}"

  # Принцип 2: Тестирование
  testing:
    name: Testing (Principle 2)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          override: true
      
      - name: Run Rust unit tests
        run: cargo test --all --verbose
        working-directory: src-tauri
      
      - name: Generate coverage report
        uses: actions-rs/tarpaulin@v0.1
        with:
          version: '0.22.0'
          args: '--workspace --timeout 300 --out Xml --output-dir coverage/'
          working-directory: src-tauri
      
      - name: Check coverage threshold (70%)
        run: |
          coverage=$(grep -oP 'line-rate="\K[^"]+' coverage/cobertura.xml | head -1)
          threshold=0.70
          if (( $(echo "$coverage < $threshold" | bc -l) )); then
            echo "Coverage $coverage is below threshold $threshold"
            exit 1
          fi
        working-directory: src-tauri
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: src-tauri/coverage/cobertura.xml
          flags: rust
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run frontend unit tests
        run: pnpm test --coverage
      
      - name: Upload frontend coverage
        uses: codecov/codecov-action@v3
        with:
          files: coverage/coverage-final.json
          flags: frontend
  
  # Принцип 2: Integration тесты
  integration-tests:
    name: Integration Tests (Principle 2)
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          override: true
      
      - name: Run integration tests
        run: cargo test --test '*' -- --test-threads=1
        working-directory: src-tauri
        timeout-minutes: 10

  # Принцип 3: E2E тесты (UX проверка)
  e2e-tests:
    name: E2E Tests (Principle 3)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Install Playwright
        run: pnpm exec playwright install --with-deps
      
      - name: Build Tauri app
        run: pnpm tauri build --debug
      
      - name: Run E2E tests
        run: pnpm test:e2e
      
      - name: Upload test artifacts
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: e2e-screenshots
          path: tests/e2e/screenshots/

  # Проверка безопасности
  security-audit:
    name: Security Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Audit Rust dependencies
        run: |
          cargo install cargo-audit
          cargo audit
        working-directory: src-tauri
      
      - name: Audit npm dependencies
        run: pnpm audit --audit-level=moderate

  # Проверка сборки
  build:
    name: Build Check
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          override: true
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build Tauri app (release)
        run: pnpm tauri build
      
      - name: Check bundle size (< 50 MB)
        if: runner.os == 'Linux'
        run: |
          bundle_size=$(du -m src-tauri/target/release/bundle/appimage/*.AppImage | cut -f1)
          if [ "$bundle_size" -gt 50 ]; then
            echo "Bundle size ${bundle_size}MB exceeds 50MB limit"
            exit 1
          fi
```

## 2. Performance Benchmarks Pipeline

**Файл:** `.github/workflows/performance.yml`

```yaml
name: Performance Benchmarks (Principle 4)

on:
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday

jobs:
  benchmark:
    name: Run Performance Benchmarks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          override: true
      
      - name: Run Cargo bench
        run: cargo bench --no-fail-fast -- --output-format bencher | tee output.txt
        working-directory: src-tauri
      
      - name: Store benchmark result
        uses: benchmark-action/github-action-benchmark@v1
        with:
          tool: 'cargo'
          output-file-path: src-tauri/output.txt
          github-token: ${{ secrets.GITHUB_TOKEN }}
          auto-push: true
          alert-threshold: '120%'
          comment-on-alert: true
          fail-on-alert: true
      
      - name: Run startup time test
        run: |
          # Build release binary
          cargo build --release
          
          # Measure cold start time (должно быть < 3s)
          start_time=$(date +%s%N)
          timeout 5s ./target/release/omegram --version || true
          end_time=$(date +%s%N)
          duration=$(( (end_time - start_time) / 1000000 ))
          
          echo "Cold start time: ${duration}ms"
          if [ "$duration" -gt 3000 ]; then
            echo "Cold start time ${duration}ms exceeds 3000ms limit"
            exit 1
          fi
        working-directory: src-tauri
      
      - name: Memory usage test
        run: |
          # TODO: Implement memory profiling
          # Should measure idle memory < 150 MB
          echo "Memory profiling not yet implemented"

  lighthouse:
    name: Lighthouse Performance
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build frontend
        run: pnpm build
      
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            http://localhost:1420
          budgetPath: .lighthouserc.json
          uploadArtifacts: true
```

## 3. Pre-commit Hooks (Local)

**Файл:** `.husky/pre-commit`

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks (Constitution Principle 1)..."

# Rust formatting
echo "Checking Rust formatting..."
cd src-tauri && cargo fmt -- --check
if [ $? -ne 0 ]; then
  echo "❌ Rust formatting check failed. Run 'cargo fmt' to fix."
  exit 1
fi

# Clippy
echo "Running Clippy..."
cargo clippy --all-targets -- -D warnings
if [ $? -ne 0 ]; then
  echo "❌ Clippy found issues. Fix them before committing."
  exit 1
fi

cd ..

# Frontend formatting
echo "Checking frontend formatting..."
pnpm prettier --check "src/**/*.{ts,tsx,css}"
if [ $? -ne 0 ]; then
  echo "❌ Frontend formatting check failed. Run 'pnpm format' to fix."
  exit 1
fi

# ESLint
echo "Running ESLint..."
pnpm eslint src/ --ext .ts,.tsx --max-warnings 0
if [ $? -ne 0 ]; then
  echo "❌ ESLint found issues. Fix them before committing."
  exit 1
fi

echo "✅ All pre-commit checks passed!"
```

## 4. Lighthouse Configuration

**Файл:** `.lighthouserc.json`

```json
{
  "ci": {
    "collect": {
      "numberOfRuns": 3,
      "startServerCommand": "pnpm preview",
      "url": ["http://localhost:1420"]
    },
    "assert": {
      "preset": "lighthouse:recommended",
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "first-contentful-paint": ["error", {"maxNumericValue": 2000}],
        "interactive": ["error", {"maxNumericValue": 2000}],
        "speed-index": ["error", {"maxNumericValue": 3000}]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

## 5. Package.json Scripts

Добавьте в `package.json`:

```json
{
  "scripts": {
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "lint": "eslint src/ --ext .ts,.tsx",
    "lint:fix": "eslint src/ --ext .ts,.tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,css}\"",
    "typecheck": "tsc --noEmit",
    "bench": "vitest bench",
    "prepare": "husky install"
  }
}
```

## 6. Cargo.toml Additions

Добавьте в `src-tauri/Cargo.toml`:

```toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "performance_benchmarks"
harness = false
```

## 7. Создание Benchmarks

**Файл:** `src-tauri/benches/performance_benchmarks.rs`

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn benchmark_message_load(c: &mut Criterion) {
    c.bench_function("load 50 messages", |b| {
        b.iter(|| {
            // TODO: Implement benchmark
            // Should complete in < 300ms (Principle 4)
            black_box(load_messages(50))
        });
    });
}

fn benchmark_chat_list_load(c: &mut Criterion) {
    c.bench_function("load 1000 chats", |b| {
        b.iter(|| {
            // TODO: Implement benchmark
            // Should complete in < 500ms (Principle 4)
            black_box(load_chats(1000))
        });
    });
}

criterion_group!(benches, benchmark_message_load, benchmark_chat_list_load);
criterion_main!(benches);
```

## 8. Status Badge

Добавьте в `README.md`:

```markdown
[![CI](https://github.com/username/omegram/workflows/CI%20-%20Constitution%20Compliance/badge.svg)](https://github.com/username/omegram/actions)
[![codecov](https://codecov.io/gh/username/omegram/branch/main/graph/badge.svg)](https://codecov.io/gh/username/omegram)
[![Performance](https://github.com/username/omegram/workflows/Performance%20Benchmarks/badge.svg)](https://github.com/username/omegram/actions)
```

## Чек-лист внедрения

- [ ] Создать `.github/workflows/ci.yml`
- [ ] Создать `.github/workflows/performance.yml`
- [ ] Настроить pre-commit hooks с Husky
- [ ] Добавить Lighthouse конфигурацию
- [ ] Создать benchmark тесты
- [ ] Настроить Codecov
- [ ] Добавить status badges в README
- [ ] Обновить package.json scripts
- [ ] Документировать CI/CD процесс для команды

## Метрики для мониторинга

### Принцип 1: Качество кода
- ✅ Clippy warnings: 0
- ✅ Rustfmt compliance: 100%
- ✅ ESLint errors: 0
- ✅ TypeScript strict mode: enabled

### Принцип 2: Тестирование
- ✅ Code coverage: > 70%
- ✅ Test execution time: < 10 min
- ✅ Flaky test rate: < 1%

### Принцип 3: UX (частично)
- ⚠️ E2E test pass rate: 100%
- ⚠️ Lighthouse performance score: > 90

### Принцип 4: Производительность
- ✅ Cold start time: < 3s
- ✅ Benchmark regressions: None allowed
- ✅ Bundle size: < 50 MB
- ⚠️ Memory usage (manual): < 150 MB idle

---

**Примечание:** Некоторые метрики (особенно UX и memory profiling) требуют ручной проверки или специализированных инструментов. CI/CD автоматизирует максимально возможное.

