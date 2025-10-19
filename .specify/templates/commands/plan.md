---
description: Create a development plan for a new feature or significant change using the plan template, ensuring alignment with project constitution.
---

## User Input

```text
{USER_INPUT}
```

## Outline

You are creating a development plan using the template at `.specify/templates/plan-template.md`.

Follow this execution flow:

1. **Parse user input:**
   - Extract feature name, goals, and any specific requirements mentioned
   - Identify technical constraints or preferences
   - Determine scope and complexity

2. **Analyze project context:**
   - Read relevant source files to understand current architecture
   - Review `.specify/memory/constitution.md` for compliance requirements
   - Check existing patterns and conventions in the codebase

3. **Fill the plan template:**
   - Load `.specify/templates/plan-template.md`
   - Replace placeholders with concrete information:
     * `[FEATURE_NAME]`: Clear, descriptive name
     * `[DATE]`: Current date (YYYY-MM-DD)
     * `[AUTHOR]`: User or "Development Team"
     * `[Status]`: Start with "Draft"
   - Complete all sections with specific, actionable content

4. **Constitution compliance check:**
   - For each of the 4 principles, explicitly verify compliance:
     * Principle 1 (Code Quality): Identify how code maintainability will be ensured
     * Principle 2 (Testing): Define testing strategy and coverage targets
     * Principle 3 (UX): Describe user experience considerations
     * Principle 4 (Performance): List performance metrics and optimizations
   - Check all items in the constitution checklist section

5. **Technical design:**
   - Propose concrete architectural approach
   - Identify files/modules to be created or modified
   - List new dependencies (if any) with justification
   - Create sequence diagrams or architecture sketches (in markdown) if helpful

6. **Risk analysis:**
   - Identify potential risks (technical, timeline, dependencies)
   - Propose mitigation strategies
   - Flag any unknowns that need investigation

7. **Implementation phases:**
   - Break down work into logical phases
   - Provide realistic time estimates
   - Identify dependencies between phases

8. **Save the plan:**
   - Create filename: `.specify/plans/[feature-name]-plan.md`
   - Ensure filename is kebab-case
   - Write completed plan to file

9. **Output summary:**
   - High-level overview of the plan
   - Key risks and dependencies
   - Estimated timeline
   - Next steps (e.g., "Ready for team review")
   - Suggested commit message

## Guidelines

- Be specific, not generic. Use actual file paths, function names, API endpoints.
- If information is missing, make reasonable assumptions based on project context or mark as "[TODO: clarification needed]"
- Keep plans focused. If scope is too large, suggest breaking into multiple features.
- Always consider the Rust + Tauri stack specifics (async, Tauri commands, IPC, etc.)
- Reference existing code patterns found in the project.
- Ensure performance considerations align with Principle 4 metrics.

## Stack-Specific Considerations

**Rust + Tauri:**
- Tauri commands for frontend-backend communication
- SQLite for local data persistence
- Async/await patterns for non-blocking operations
- Error handling with Result<T, E>
- Serde for serialization
- Consider bundle size impact

**Frontend (React/TypeScript):**
- Component reusability
- State management patterns
- API interaction via Tauri invoke
- Loading states and error boundaries
- Responsive design

Do not ask for confirmations; proceed with creating the plan based on available information.

