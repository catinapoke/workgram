# Performance Workflow Contract

**Файл:** `.github/workflows/performance.yml`  
**Версия:** 1.0.0  
**Дата:** 2025-10-19

## Обзор

Performance benchmarking пайплайн для измерения и отслеживания производительности критических операций. Запускается на Pull Requests и еженедельно по расписанию.

---

## Триггеры

```yaml
on:
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0'  # Воскресенье, 00:00 UTC
```

**Поведение:**
- Pull Request → benchmark + compare vs baseline + comment на PR
- Weekly scheduled → baseline update для trending
- Manual dispatch → поддержка для ad-hoc запусков

---

## Jobs

### Job 1: `benchmark`

**Назначение:** Cargo benchmarks для Rust critical operations (Принцип 4)

**Runner:** `ubuntu-latest` (желательно dedicated runner для стабильности)

**Шаги:**
1. Checkout code
2. Setup Rust (stable)
3. Run cargo bench: `cargo bench --no-fail-fast -- --output-format bencher | tee output.txt`
4. Store benchmark result with `github-action-benchmark`
5. Compare vs baseline
6. Alert if regression > 20%
7. Comment results on PR
8. Measure cold start time
9. Verify cold start < 3s

**Success Criteria:**
- ✅ All benchmarks complete successfully
- ✅ Cold start time < 3000ms
- ✅ No regression > 20% vs baseline

**Failure Behavior:**
- ❌ Regression > 20% → job fails, blocks merge
- ⚠️ Regression 10-20% → warning, не блокирует
- 📊 Results posted as PR comment

**Benchmark Operations:**
1. `load_1000_chats` - Target: < 500ms
2. `load_50_messages` - Target: < 300ms
3. `insert_message` - Target: < 50ms
4. `search_messages` - Target: < 1000ms
5. `parse_telegram_update` - Target: < 10ms
6. `serialize_message` - Target: < 5ms

**Configuration:**

```rust
// src-tauri/benches/performance_benchmarks.rs
use criterion::{criterion_group, criterion_main, Criterion, BenchmarkId};

fn bench_load_chats(c: &mut Criterion) {
    let mut group = c.benchmark_group("database");
    
    for size in [100, 500, 1000].iter() {
        group.bench_with_input(
            BenchmarkId::new("load_chats", size), 
            size, 
            |b, &size| {
                b.iter(|| load_chats(size));
            }
        );
    }
    
    group.finish();
}

criterion_group!(benches, bench_load_chats, /* ... */);
criterion_main!(benches);
```

**Estimated Duration:** 3-5 минут

**Outputs:**
- `output.txt` - Bencher format results
- JSON data for trending (stored in gh-pages)

---

### Job 2: `startup-time`

**Назначение:** Измерение холодного старта приложения

**Runner:** `ubuntu-latest`

**Шаги:**
1. Checkout code
2. Setup Rust (stable)
3. Build release binary: `cargo build --release`
4. Measure cold start time
5. Compare vs 3s threshold
6. Upload metrics

**Measurement Script:**

```bash
# Measure cold start (холодный запуск)
start_time=$(date +%s%N)
timeout 5s ./target/release/workgram --version || true
end_time=$(date +%s%N)
duration=$(( (end_time - start_time) / 1000000 ))  # Convert to ms

echo "Cold start time: ${duration}ms"

# Threshold check (Принцип 4: < 3s)
if [ "$duration" -gt 3000 ]; then
    echo "ERROR: Cold start ${duration}ms exceeds 3000ms limit"
    exit 1
fi
```

**Success Criteria:**
- ✅ Cold start < 3000ms

**Failure Behavior:**
- ❌ > 3000ms → job fails, блокирует merge
- 📈 Trend data stored for history

**Estimated Duration:** 1-2 минуты

---

### Job 3: `memory-profiling`

**Назначение:** Измерение потребления памяти (будущее, пока manual)

**Runner:** `ubuntu-latest`

**Статус:** TODO (Phase 2)

**План:**
```bash
# Запустить приложение в фоне
./target/release/workgram &
APP_PID=$!

# Подождать idle state
sleep 5

# Измерить RSS memory
memory_kb=$(ps -o rss= -p $APP_PID)
memory_mb=$(( memory_kb / 1024 ))

echo "Idle memory usage: ${memory_mb}MB"

# Threshold (Принцип 4: < 150MB)
if [ "$memory_mb" -gt 150 ]; then
    echo "ERROR: Memory ${memory_mb}MB exceeds 150MB limit"
    exit 1
fi

kill $APP_PID
```

**Target:** < 150 MB в idle состоянии

---

### Job 4: `lighthouse`

**Назначение:** Frontend performance metrics (Принцип 4)

**Runner:** `ubuntu-latest`

**Шаги:**
1. Checkout code
2. Setup Node.js + pnpm
3. Install dependencies
4. Build frontend: `pnpm build`
5. Start preview server: `pnpm preview`
6. Run Lighthouse CI
7. Assert metrics meet thresholds
8. Upload artifacts

**Lighthouse Configuration:**

```json
// .lighthouserc.json
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
        "speed-index": ["error", {"maxNumericValue": 3000}],
        "largest-contentful-paint": ["error", {"maxNumericValue": 2500}]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

**Success Criteria:**
- ✅ Performance score >= 90
- ✅ First Contentful Paint < 2000ms
- ✅ Time to Interactive < 2000ms
- ✅ Speed Index < 3000ms

**Failure Behavior:**
- ❌ Any assertion fails → job fails
- 📊 Detailed report uploaded
- 💬 Comment on PR with scores

**Estimated Duration:** 5-7 минут

---

## Baseline Management

### Initial Baseline

**Когда:** После первого успешного запуска benchmarks

**Как:**
1. Запустить benchmark на `main` branch
2. Сохранить результаты как baseline в `gh-pages` branch
3. All future runs сравниваются с этим baseline

### Baseline Update

**Когда:** 
- Weekly scheduled run (воскресенье)
- Manual trigger при significant improvements
- После major refactoring (с одобрения)

**Процесс:**
1. Запустить benchmarks
2. Если стабильно лучше текущего baseline → update
3. Commit новый baseline в gh-pages

---

## Alert Thresholds

| Метрика | Warning | Error (блокирует) |
| --- | --- | --- |
| Regression | 10-20% | > 20% |
| Cold start | 2.5-3s | > 3s |
| Lighthouse Performance | 85-90 | < 85 |
| FCP/TTI | 1.8-2s | > 2s |

---

## Data Storage

### Benchmark Results

**Location:** `gh-pages` branch, `/benchmarks/data.json`

**Format:**
```json
{
  "lastUpdate": "2025-10-19T10:00:00Z",
  "entries": [
    {
      "commit": "abc123",
      "date": 1697712000000,
      "tool": "cargo",
      "benches": [
        {
          "name": "load_1000_chats",
          "value": 450,
          "unit": "ms",
          "range": "±15"
        }
      ]
    }
  ]
}
```

**Visualization:** Static HTML page on GitHub Pages

---

## PR Comments

**Format example:**

```markdown
## 🚀 Performance Benchmark Results

**Commit:** abc123  
**Baseline:** def456

### Cargo Benchmarks

| Operation | Current | Baseline | Change | Status |
|-----------|---------|----------|--------|--------|
| load_1000_chats | 450ms | 480ms | -6.25% 🎉 | ✅ Pass |
| load_50_messages | 310ms | 290ms | +6.90% | ⚠️ Warning |
| insert_message | 45ms | 50ms | -10% 🎉 | ✅ Pass |

### Startup Time
- **Cold start:** 2.8s (target: < 3s) ✅

### Lighthouse Scores
- **Performance:** 92/100 (target: >= 90) ✅
- **FCP:** 1850ms (target: < 2000ms) ✅
- **TTI:** 1950ms (target: < 2000ms) ✅

---

**Overall:** ✅ All performance checks passed!
```

---

## Optimization Recommendations

При обнаружении регрессий, bot может предлагать:

- 🔍 Profile с `cargo flamegraph`
- 📊 Check allocation patterns с `cargo-instruments`
- ⚡ Review database query efficiency
- 🎯 Optimize hot paths с inline hints
- 💾 Check caching strategy

---

## Manual Triggers

**Workflow dispatch inputs:**

```yaml
on:
  workflow_dispatch:
    inputs:
      baseline_update:
        description: 'Update baseline after run'
        required: false
        type: boolean
        default: false
```

**Usage:** Для forced baseline update после одобренного refactoring

---

## Success Criteria

**Workflow считается успешным, если:**
- ✅ All benchmarks complete без panic
- ✅ No regression > 20%
- ✅ Cold start < 3s
- ✅ Lighthouse score >= 90

**При успехе:**
- 💬 Comment на PR с результатами
- 📈 Data stored для trending
- ✅ Green status check

**При неудаче:**
- ❌ Блокировка merge (если > 20% regression)
- 📋 Детальный отчет в comments
- 🔔 Notification команде

---

## Versioning

**Version:** 1.0.0 (Initial)

**Changelog:**
- 2025-10-19: Initial with cargo bench + Lighthouse

**Future Enhancements:**
- Memory profiling automation
- Flamegraph generation on regression
- Historical trend charts
- Cross-platform benchmarks

