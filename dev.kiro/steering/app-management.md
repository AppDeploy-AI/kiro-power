# AppDeploy: Managing Deployed Apps

Tools for working with apps that already exist:

- `get_apps` lists the user's deployed apps (safe read-only starting point).
- `get_app_versions` / `apply_app_version` manage version history and rollback.
- `src_glob` / `src_grep` / `src_read` inspect the deployed source snapshot - use these to recover state when the local files are out of sync with what is deployed.
- `get_e2e_qa_run_details` fetches results of the automated post-deploy QA run.
- `delete_app` is destructive and irreversible - call it only when the user explicitly asks to delete an app.

## Tools

### get_apps

List apps owned by the current user. Use this to discover app_id values and app metadata before calling get_app_status, deploy_app, delete_app, or the source snapshot tools.

**Parameters:**
  - `continuation_token`: string (optional) - Token for pagination

### get_app_versions

List deployable versions for an existing app. Requires app_id. Returns newest-first {name, version, timestamp} items. Use 'version' with apply_app_version, treat 'name' as human-facing metadata, and convert timestamps to the user's local time when presenting them.

**Parameters:**
  - `app_id`: string (required) - Target app id

### apply_app_version

Start deploying an existing app at a specific version. Use the 'version' value from get_app_versions, then poll get_app_status until deployment reaches a terminal status.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `version`: string (required) - Version id to apply

### src_glob

Use this when you need to discover files in an app's source snapshot. Returns file paths matching a glob pattern (no content). Useful for exploring project structure before reading or searching files.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `version`: string (optional) - Version to inspect (defaults to applied version)
  - `path`: string (optional) - Directory path to search within
  - `glob`: string (optional) - Glob pattern to match files (default: **/*)
  - `include_dirs`: boolean (optional) - Include directory paths in results
  - `continuation_token`: string (optional) - Token from previous response for pagination

### src_grep

Use this when you need to search for patterns in an app's source code. Returns matching lines with optional context. Supports regex patterns, glob filters, and multiple output modes.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `version`: string (optional) - Version to search (defaults to applied version)
  - `pattern`: string (required) - Regex pattern to search for (max 500 chars)
  - `path`: string (optional) - Directory path to search within
  - `glob`: string (optional) - Glob pattern to filter files (e.g., '*.ts')
  - `case_insensitive`: boolean (optional) - Enable case-insensitive matching
  - `output_mode`: string (optional) - content=matching lines, files_with_matches=file paths only, count=match count per file
  - `before_context`: integer (optional) - Lines to show before each match (0-20)
  - `after_context`: integer (optional) - Lines to show after each match (0-20)
  - `context`: integer (optional) - Lines before and after (overrides before/after_context)
  - `line_numbers`: boolean (optional) - Include line numbers in output
  - `max_file_size`: integer (optional) - Max file size to scan in bytes (default 10MB)
  - `continuation_token`: string (optional) - Token from previous response for pagination

### src_read

Use this when you need to read a specific file from an app's source snapshot. Returns file content with line-based pagination (offset/limit). Handles both text and binary files.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `version`: string (optional) - Version to read from (defaults to applied version)
  - `file_path`: string (required) - Path to the file to read
  - `offset`: integer (optional) - Line offset to start reading from (0-indexed)
  - `limit`: integer (optional) - Number of lines to return (max 2000)

### get_e2e_qa_run_details

Use this when investigating a failed e2e QA run right after get_app_status reports e2e_tests.status='failed'. Returns the QA transcript, structured results, trace and replay URLs, and debugging keys_prefix.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `job_id`: string (optional) - Optional QA job id to inspect
  - `qa_run_group_id`: string (optional) - Optional QA run-group id from get_app_status

### delete_app

Use this when you want to permanently delete an app. Use only on explicit user request. This is irreversible; after deletion, status checks will return not found.

**Parameters:**
  - `app_id`: string (required) - Target app id
