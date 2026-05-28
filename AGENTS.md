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

# Git Workflow Rules

1. **Notion task required**: Every commit must have a corresponding Notion task. Before starting work, verify the task exists in Notion.

2. **Branch naming**: `#{task_number}_{short_description}` — e.g. `#5_add-refresh-token`. Always branch off `master`.

3. **No direct commits to master**: All changes go through a branch and PR.

4. **Commit message format**: `#<number> <service-name>: <short description>` — e.g. `#5 user-service: add refresh token endpoint`.

5. **One logical change per commit**: Do not mix refactoring with new features in a single commit.

6. **Pull before starting work**: Always run `git pull` before creating a new branch.

7. **Pre-commit confirmation required**: When the user says they are ready to commit, before running `git commit`:
   - List all files that will be included in the commit
   - Ask the user to review the changes
   - Only proceed after the user explicitly confirms (e.g. "go ahead" / "looks good" / "approved")

8. **PR self-review**: Before opening a PR, the user must review their own diff.
