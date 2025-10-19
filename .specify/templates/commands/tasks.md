---
description: Generate a task list from a specification, breaking down work into actionable items aligned with constitution principles.
---

## User Input

```text
{USER_INPUT}
```

## Outline

You are creating a task list using the template at `.specify/templates/tasks-template.md`.

Follow this execution flow:

1. **Load related artifacts:**
   - If user references a spec, load `.specify/specs/[spec-name].md`
   - Read the related plan if mentioned
   - Review `.specify/memory/constitution.md` for task categorization

2. **Analyze work breakdown:**
   - Identify all deliverables from the specification
   - Break down each requirement into implementation tasks
   - Consider task dependencies and sequencing
   - Estimate effort for each task

3. **Fill the tasks template:**
   - Load `.specify/templates/tasks-template.md`
   - Link to the source specification
   - Set sprint/milestone information
   - Calculate timeline based on estimates

4. **Create categorized tasks:**
   
   Use these categories (aligned with constitution):
   
   - **🏗️ Architecture:** Design, structure, module organization
   - **💻 Development:** Feature implementation (backend/frontend)
   - **🧪 Testing:** Unit, integration, E2E tests (Principle 2)
   - **📈 Performance:** Optimizations, benchmarks (Principle 4)
   - **🎨 UX/UI:** Interface, animations, user feedback (Principle 3)
   - **📚 Documentation:** Code docs, user guides (Principle 1)
   - **🐛 Bugfix:** Known issues from README or discovered bugs
   - **🔒 Security:** Authentication, encryption, data protection
   - **♿ Accessibility:** Keyboard nav, screen readers, ARIA

5. **Task structure (for each task):**
   
   ```markdown
   #### TASK-XXX: [Clear, actionable title]
   
   **Приоритет:** P0/P1/P2/P3
   **Оценка:** [X hours/days]
   **Исполнитель:** [Name or Unassigned]
   **Статус:** 🔲 To Do | 🔄 In Progress | ✅ Done | ❌ Blocked
   
   **Описание:**
   [Detailed description of what needs to be done]
   
   **Принципы конституции:**
   - Principle X: [How this task supports the principle]
   
   **Acceptance Criteria:**
   - [ ] Criterion 1
   - [ ] Criterion 2
   
   **Технические детали:**
   - Файлы: [Specific file paths]
   - Зависимости: [TASK-YYY, external dependencies]
   
   **Definition of Done:**
   - [ ] Code complete and reviewed
   - [ ] Tests written and passing
   - [ ] Documentation updated
   - [ ] [Other DoD items specific to task category]
   ```

6. **Ensure coverage:**
   - Every FR from spec has corresponding development task(s)
   - Every NFR has verification task (test/benchmark)
   - Architecture decisions have implementation tasks
   - All constitution principles are addressed:
     * Principle 1: Documentation tasks exist
     * Principle 2: Testing tasks cover 70%+ of critical paths
     * Principle 3: UX tasks for all user-facing changes
     * Principle 4: Performance tasks with metrics

7. **Create dependency graph:**
   - Visualize task dependencies in ASCII/Markdown
   - Identify critical path
   - Flag potential bottlenecks

8. **Track progress:**
   - Initialize progress table (all "To Do")
   - Create statistics by category
   - Add constitution compliance checklist

9. **Save task list:**
   - Create filename: `.specify/tasks/[feature-name]-tasks.md`
   - Use kebab-case for filename
   - Write completed task list to file

10. **Output summary:**
    - Total number of tasks by category
    - Estimated timeline
    - Critical path tasks
    - Dependencies on external factors
    - Ready to start tasks (no blockers)
    - Suggested commit message

## Guidelines

- Tasks should be granular (4-16 hours each, not more than 2 days)
- Each task must have clear acceptance criteria
- Link related tasks (e.g., "TASK-002 must complete before TASK-003")
- Assign priorities based on:
  * P0: Blocks release, critical bugs
  * P1: Core functionality, must-haves
  * P2: Nice-to-haves, can be deferred
  * P3: Future enhancements
- Include code skeletons/templates in task descriptions
- Reference specific file paths and line numbers when relevant
- Ensure every task has a "Definition of Done" that includes testing

## Task Estimation Heuristics

- **Architecture task:** 4-8 hours (design + document)
- **Simple Tauri command:** 4-6 hours (code + tests)
- **Complex backend logic:** 8-16 hours
- **React component:** 4-8 hours (component + tests)
- **Unit tests:** 2-4 hours (per module)
- **Integration test:** 4-8 hours (per scenario)
- **E2E test:** 6-12 hours (per critical flow)
- **Performance optimization:** 8-16 hours (profile + optimize + verify)
- **Documentation:** 2-4 hours (per major component)

## Stack-Specific Task Templates

**Backend Task Example:**
```markdown
#### TASK-XXX: Implement [Feature] Tauri Command

**Описание:**
Create async Tauri command to [functionality]

**Acceptance Criteria:**
- [ ] Command accepts typed parameters (struct)
- [ ] Returns Result<T, String> with proper error handling
- [ ] Uses async/await for non-blocking operations
- [ ] Integrates with SQLite via db service
- [ ] Logs operations for debugging

**Definition of Done:**
- [ ] Clippy warnings = 0
- [ ] Unit tests coverage > 70%
- [ ] Integration test created
- [ ] Rustdoc comments complete
```

**Frontend Task Example:**
```markdown
#### TASK-XXX: Create [Component] React Component

**Описание:**
Implement UI component for [purpose]

**Acceptance Criteria:**
- [ ] TypeScript typed props interface
- [ ] Uses existing design system components
- [ ] Responsive (mobile/desktop)
- [ ] Keyboard accessible
- [ ] Loading/error states handled

**Definition of Done:**
- [ ] ESLint/Prettier passing
- [ ] Component tests (RTL) written
- [ ] Props documented (JSDoc)
- [ ] Tested on different screen sizes
```

Do not ask for confirmations; generate the complete task breakdown based on the specification.

