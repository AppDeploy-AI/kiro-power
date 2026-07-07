# AppDeploy: Full Tool Reference

Complete reference for every tool exposed by the AppDeploy MCP server.

### get_deploy_instructions

Use this first when the user wants to deploy, publish, or update a website or web app and get a public URL. You must call this tool before starting to design or generate any code. This tool returns instructions only and does not deploy anything.

**Parameters:**
  - `model`: any (optional) - Optional model name if known, for example 'gpt-5.5'. Omit or use null when unknown.
  - `intent`: any (optional) - Optional intent of this deployment. User-initiated examples: 'initial app deploy', 'bugfix - ui is too noisy'. Agent-initiated examples: 'agent fixing deployment error', 'agent retry after lint failure'. If no specific intent was given use 'app deploy'.
  - `description`: any (optional) - Optional one-line summary of the user prompt or agent task.

### get_app_template

Call get_deploy_instructions first. Then call this once you've decided app_type and frontend_template. Returns starter files, deploy contract, and next steps for a new app. The returned files are the baseline that deploy_app will diff against. Next, call get_appdeploy_sdk_reference with the same features for SDK types, rules, and code reference.

**Parameters:**
  - `app_type`: string (required) - 'frontend-only' (static/client-side apps; can use client-side auth), 'frontend+backend' (adds database, storage, server-side auth, and any other backend-required functionality.)
  - `frontend_template`: string (required) - 'html-static' (simple sites), 'react-vite' (SPAs, interactive apps, games), 'nextjs-static' (multi-page sites)
  - `features`: array (required) - SDK features to compose into the starter app (use [] when no SDK features are needed). Available features: api (HTTP transport and router basics), realtime (WebSocket live updates and subscriptions), auth (user accounts, middleware, and scopes), notifications (push notifications, topics, and enable/subscribe UX), invites (invite code lifecycle and client invite URL helpers), database (CRUD key-value tables), storage (file uploads, downloads, and signed URLs), secrets (encrypted app-scoped backend secrets), ai.generate (single LLM text or multimodal generation), ai.extract (schema-based structured extraction), ai.ocr (image OCR and transcription), ai.classify (fixed-label text or image classification), ai.scrape (AI-friendly web page scraper), ai.run (bounded generate-to-tool loops), ai.image (image generation), cron (scheduled jobs). For frontend+backend apps, backend files are composed from these features.

### get_appdeploy_sdk_reference

Returns per-feature SDK reference: types, rules, and examples for the requested platform capabilities. Treat this as the implementation contract for those features and review it before writing files that use them. Backend features resolve api automatically when router/transport infrastructure is required.

**Parameters:**
  - `features`: array (required) - Platform capabilities to get docs for. Available features: api (HTTP transport and router basics), realtime (WebSocket live updates and subscriptions), auth (user accounts, middleware, and scopes), notifications (push notifications, topics, and enable/subscribe UX), invites (invite code lifecycle and client invite URL helpers), database (CRUD key-value tables), storage (file uploads, downloads, and signed URLs), secrets (encrypted app-scoped backend secrets), ai.generate (single LLM text or multimodal generation), ai.extract (schema-based structured extraction), ai.ocr (image OCR and transcription), ai.classify (fixed-label text or image classification), ai.scrape (AI-friendly web page scraper), ai.run (bounded generate-to-tool loops), ai.image (image generation), cron (scheduled jobs). Request the exact features your app will use. Backend features resolve api automatically when router/transport infrastructure is required.

### upload_assets

Use this before deploy_app whenever you need to send binary assets/resources or many files. Prefer this whenever the source of truth already exists as local files or a local project tree, especially for multiple files or whole-file replacements. This is the required path for images, backgrounds, icons, fonts, PDFs, media, archives, and other file assets you want deployed. Prepare local files and the upload manifest first. Call upload_assets only when you can upload immediately. If the upload_url expires before use or the upload fails because it expired, call upload_assets again to get a fresh upload_url and upload_id. PUT multipart/form-data to the returned upload_url: a 'payload' part (JSON manifest with text file changes/diffs and deletePaths) plus binary files named by app-relative path. The upload manifest carries all text changes and diffs plus deletePaths. After upload succeeds, call deploy_app with upload_id only. Do not also send files[] or deletePaths[]; those changes belong in the upload manifest. Never base64-encode binary assets into deploy_app.

**Parameters:**


### deploy_app

Deploy or update a website or web app to get a public URL. Text files only in files[]. files[] must be a JSON array, even for one file. Example: files: [{"filename":"src/App.tsx","content":"..."}]. Never pass a bare string or a single file object. Use files[] for inline text edits and diffs, not for copying large existing local file contents into tool params. Never inline or base64-encode binary assets/resources in files[]; use upload_assets first for images, fonts, media, PDFs, archives, and other client-supplied file assets, then pass upload_id. Inline deploy_app text payloads MUST be compact. For JavaScript/TypeScript/JSX/TSX string literals, use single quotes wherever valid. Keep inline HTML/CSS/JS/TS diff from/to values single-line wherever valid; do not include newline characters unless required for valid syntax. Template files from get_app_template are auto-included as the baseline — use diffs[] to modify them, content only for entirely new files. New apps: set app_id to null, provide app_name, description, app_type, frontend_template, and features. Updates: provide existing app_id, features, and either changed files/deletePaths or upload_id. If upload_id is provided, do not also send files[] or deletePaths[]; the upload manifest owns all text changes, diffs, and delete operations. Rules: do not add @appdeploy/client or @appdeploy/sdk to package.json (platform-injected). SPAs must use HashRouter. Frontend must never import @appdeploy/sdk; backend must never import @appdeploy/client. Frontend must use api from @appdeploy/client for backend calls, never fetch() or axios. If frontend realtime is used, @appdeploy/client websocket usage is ws.connect() only; do not call ws.subscribe/ws.publish/ws.send directly on ws. After deploy, poll get_app_status every 5s until status is 'ready' or 'failed'. If get_app_status returns QA/e2e/runtime errors, attempt automatic fixes and redeploy up to 3 times before asking the user for guidance.

**Parameters:**
  - `app_id`: any (optional) - MANDATORY: existing app id to update, or null for new app
  - `app_type`: string (required) - app architecture: frontend-only or frontend+backend
  - `app_name`: string (required) - short display name
  - `description`: any (optional) - short description of what the app does
  - `frontend_template`: any (optional) - REQUIRED when app_id is null. One of: 'html-static' (simple sites), 'react-vite' (SPAs, games), 'nextjs-static' (multi-page). Template files auto-included.
  - `features`: any (optional) - SDK features used (from get_appdeploy_sdk_reference). Pass on every deploy so the server applies diffs against the correct feature-composed baseline.
  - `deletePaths`: any (optional) - Array of relative paths to delete. MUST be a JSON array, even for one path. Example value: ["src/old-file.ts"]. If upload_id is provided, do not also send files[] or deletePaths[]; the upload manifest owns all text changes, diffs, and delete operations.
  - `model`: any (optional) - MANDATORY: LLM model name generating this deploy (e.g. 'claude-sonnet-4', 'gpt-4o'). If unavailable, use 'chat'.
  - `intent`: any (optional) - MANDATORY: one-line summary of what changed (e.g. 'Add dark mode toggle', 'Fix login redirect bug'). If no specific intent was given, use 'app deploy'.
  - `initiator`: any (optional) - Optional request initiator. Use 'user' when this call directly follows a user request. Use 'agent' when this call is part of autonomous agent work, such as retrying, fixing, or continuing without a new user request.
  - `type`: any (optional) - Optional deploy change type, for example 'fix', 'feature', 'refactor', 'performance', 'test', 'docs', 'chore', or 'style'.
  - `upload_id`: any (optional) - Opaque upload ID from upload_assets. When provided, omit files[] and deletePaths[]; the upload manifest carries all text changes, diffs, and delete operations.
  - `resource_requirements`: any (optional) - Optional widget upload slot definitions for ChatGPT web/Claude web. Use this when users will provide large resources (images/PDF/media/fonts) after code generation. Code should reference these target_path placeholders so uploaded files resolve without further code edits.
  - `secret_entry_ids`: any (optional) - Optional one-time secret entry ids to bind during a new-app deploy. Allowed only when app_id is omitted. For existing apps, use set_app_secrets instead.
  - `files`: any (optional) - Array of text file edit objects. MUST be a JSON array, even for one file. Never pass a bare string or a single object. Example value: [{"filename":"src/App.tsx","content":"..."}]. Each item must include filename plus either content or diffs. Use files[] for inline text edits and diffs. If the source of truth already exists as local files or a local project tree, prefer upload_assets instead of copying those file contents into tool params. If upload_id is provided, do not also send files[] or deletePaths[]; the upload manifest owns all text changes, diffs, and delete operations. Use upload_assets for binary assets/resources or other client-supplied file assets.

### get_app_status

Use this when deploy_app returns, when checking deployment status, or when the app has errors or is not working as expected. Returns deployment status, e2e test status, QA snapshot, and frontend/backend error logs; treat deployed_and_testing status as non-final, always inspect QA/errors, and call get_e2e_qa_run_details if e2e tests fail.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `since`: integer (optional) - Optional timestamp in epoch milliseconds to filter errors. When provided, returns only errors since that timestamp.
  - `limit`: integer (optional) - Optional shared cap for returned logs across frontend and backend combined. Defaults to 50 when omitted.

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

### get_app_versions

List deployable versions for an existing app. Requires app_id. Returns newest-first {name, version, timestamp} items. Use 'version' with apply_app_version, treat 'name' as human-facing metadata, and convert timestamps to the user's local time when presenting them.

**Parameters:**
  - `app_id`: string (required) - Target app id

### apply_app_version

Start deploying an existing app at a specific version. Use the 'version' value from get_app_versions, then poll get_app_status until deployment reaches a terminal status.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `version`: string (required) - Version id to apply

### create_secret_entry

Create a one-time pre-authorized browser link so the user can submit a backend secret value outside chat without an extra login step. The returned secret_entry_url should be opened in a new tab. In terminal or CLI clients, present the full raw URL in plain text instead of a markdown link label.

**Parameters:**
  - `secret_name`: string (required) - Secret name in uppercase env-style format, for example STRIPE_API_KEY.
  - `app_id`: string (optional) - Optional existing app id. Omit this when preparing secrets for a brand-new app before deploy_app.

### get_secret_entry_status

Poll the lifecycle state of a one-time secret entry link after the user opens the browser page and submits the value.

**Parameters:**
  - `secret_entry_id`: string (required) - Opaque one-time secret entry id.

### set_app_secrets

Attach one or more submitted secret entry ids to an existing app. Reusing a secret name replaces the stored value for that app.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `secret_entry_ids`: array (required) - Submitted secret entry ids to bind to this existing app. Each entry contributes one secret name/value.

### list_app_secrets

List backend secret names currently configured for an existing app. Values are never returned.

**Parameters:**
  - `app_id`: string (required) - Target app id

### delete_app_secrets

Delete one or more backend secret names from an existing app. Missing names are ignored.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `secret_names`: array (required) - Secret names to delete from the app. Missing names are ignored.

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

### get_apps

List apps owned by the current user. Use this to discover app_id values and app metadata before calling get_app_status, deploy_app, delete_app, or the source snapshot tools.

**Parameters:**
  - `continuation_token`: string (optional) - Token for pagination

### get_custom_domain_instructions

Use this when you need setup guidance or to check the current custom domain status for an app. Returns configured hostnames, DNS instructions, stage proxy targets, fallback IPv4 addresses, and next steps.

**Parameters:**
  - `app_id`: string (required) - Target app id

### configure_custom_domain

Use this when you need to manage a custom domain for an existing app, including adding a hostname, verifying DNS, or deleting it.

**Parameters:**
  - `app_id`: string (required) - Target app id
  - `action`: string (required) - Lifecycle action to perform for this custom domain
  - `domain`: string (required) - Hostname to add, verify, or delete
  - `dns_mode`: string (optional) - Required only when action is add
