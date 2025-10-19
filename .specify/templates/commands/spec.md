---
description: Create a detailed technical specification from a plan, defining requirements, architecture, and acceptance criteria.
---

## User Input

```text
{USER_INPUT}
```

## Outline

You are creating a technical specification using the template at `.specify/templates/spec-template.md`.

Follow this execution flow:

1. **Load related artifacts:**
   - If user references a plan, load `.specify/plans/[plan-name].md`
   - Read `.specify/memory/constitution.md` for requirements
   - Examine relevant source code for context

2. **Parse requirements:**
   - Extract functional requirements from user input or plan
   - Identify non-functional requirements (performance, security, reliability)
   - Determine scope boundaries (what's in, what's out)

3. **Fill the spec template:**
   - Load `.specify/templates/spec-template.md`
   - Replace all placeholders with concrete values
   - Version starts at 1.0.0 for new specs
   - Link to related plan if exists

4. **Define functional requirements (FR):**
   - Create FR-1, FR-2, etc. for each distinct feature
   - Assign priority: Critical | High | Medium | Low
   - Write clear, testable acceptance criteria
   - Map each FR to relevant constitution principles

5. **Define non-functional requirements (NFR):**
   - **Performance (Principle 4):** Specific metrics with numbers
     * Response times, throughput, memory limits
     * Benchmark test examples
   - **Reliability:** Error handling, offline support, recovery
   - **Security:** Authentication, encryption, data protection
   - **Maintainability (Principle 1):** Code complexity, documentation coverage
   - **Testability (Principle 2):** Test coverage targets, test types

6. **Design UI/UX (Principle 3):**
   - Describe UI components needed
   - Define UX flow with state transitions
   - Specify timing (< 100ms feedback, loading indicators)
   - Document keyboard shortcuts
   - Consider accessibility

7. **Technical architecture:**
   - **Rust backend:** Tauri commands with type signatures
   - **Frontend:** React components with props interfaces
   - **Data model:** SQLite schema, Rust structs
   - **API integration:** Telegram API calls, error handling

8. **Testing plan (Principle 2):**
   - Unit tests: Code skeletons with coverage targets
   - Integration tests: Scenarios to cover
   - E2E tests: Critical user flows
   - Performance tests: Benchmarks with baselines

9. **Rollout strategy:**
   - Phased rollout plan (Alpha → Beta → GA)
   - Success metrics to monitor
   - Rollback strategy

10. **Save the specification:**
    - Create filename: `.specify/specs/[feature-name]-spec.md`
    - Use kebab-case for filename
    - Write completed spec to file

11. **Output summary:**
    - Key requirements overview
    - Critical NFRs highlighted
    - Testing strategy summary
    - Implementation readiness status
    - Suggested commit message

## Guidelines

- Requirements must be specific and testable (avoid "should be fast" → use "< 500ms")
- Include code examples (function signatures, schemas, test skeletons)
- Reference actual file paths in the project
- Map every requirement to constitution principles
- Flag any assumptions or open questions
- Ensure all NFRs from Principle 4 (performance) have concrete numbers

## Stack-Specific Details

**Rust/Tauri:**
```rust
// Tauri command template
#[tauri::command]
async fn command_name(
    state: tauri::State<'_, AppState>,
    param: ParamType
) -> Result<ResponseType, String> {
    // Implementation
}
```

**TypeScript/React:**
```typescript
// Component interface
interface ComponentProps {
    // Props definition
}

// Tauri invoke
import { invoke } from '@tauri-apps/api/tauri';
const result = await invoke('command_name', { param });
```

**SQLite:**
```sql
-- Schema design with indexes for performance
CREATE TABLE table_name (
    id INTEGER PRIMARY KEY,
    -- fields
);
CREATE INDEX idx_field ON table_name(field);
```

## Validation

Before finalizing:
- [ ] All FRs have acceptance criteria
- [ ] All NFRs have measurable metrics
- [ ] Constitution principles referenced
- [ ] Test plan includes all three levels (unit/integration/e2e)
- [ ] Performance benchmarks defined
- [ ] UI/UX timing requirements specified

Do not ask for confirmations; create the specification based on available information.

