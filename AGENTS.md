# Notion Tasks Rules

1. **Language**: All task names, descriptions, and content must be in English.
2. **Approval required**: Before creating or editing any tasks, always present the plan to the user first. Only create/edit after the user explicitly says "go ahead" / "create it" / "approved".
3. **Naming Convention**: Use the format: `#<number> <ServiceName>: <TaskName>`.
   - The number must be auto-incremented based on the highest number currently in the database.
   - Example: `#1 user-service: CI setup`
4. **Metadata**:
   - `Date Added`: Current date.
   - `Service`: Select/Multi-select (`user-service`, `api-gateway`, `config-server`, `graphql-service`, `eureka-server`, `commons`).
   - `Type`: Select (`chore`, `task`, `bug`).
5. **Auto-update parent status**: When all subtasks of a parent task are marked "Done", automatically update the parent task status to "Done" as well.
