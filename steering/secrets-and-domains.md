# AppDeploy: Secrets & Custom Domains

Backend secrets (API keys the deployed app's backend needs) are collected through a secure entry flow - never ask the user to paste secret values into the chat. Use `create_secret_entry` to generate a secure entry link, `get_secret_entry_status` to poll completion, and the remaining tools to manage stored secrets.

Custom domains: call `get_custom_domain_instructions` first, then `configure_custom_domain` once DNS is in place.

## Tools

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
