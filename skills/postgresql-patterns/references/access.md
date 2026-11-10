# Access and execution boundaries

- Honor the user's selected database tool and repository instructions. Establish intended environment/target before connecting; do not infer credentials from unrelated running databases or print .env values.
- If the user or repository instructions prescribe a database access tool or helper, use it from the documented working directory and follow its configuration and output contract.
- Inspect the selected tool's contract before composing multiple statements/transaction-local settings. Do not pass secrets through command arguments, logs or chat. Do not switch away from a prescribed access route just because another route is available.
- Check server_version_num/current_database/current_user when a live task actually requires verified server context. Configuration/image versions are evidence of intent, not proof of the running server. Do not connect solely to answer a conceptual question.
- Prefer minimal schema/catalog/plan reads. Authorization to inspect is not authorization to run ANALYZE, VACUUM, migrations, DML, extensions or configuration changes.
- Performance SELECTs may be expensive or take locks; use scoped limits and explain operational effects when relevant. READ ONLY is useful but not a general sandbox for arbitrary functions or external effects.
- Temporary SQL verification may use a new task-owned disposable database with no project mounts/credentials/network exposure. Its native driver/psql is for that isolated target, not an alternative way into project data. Remove only the test resources created by the task.
