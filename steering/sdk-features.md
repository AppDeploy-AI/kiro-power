# AppDeploy: SDK Features

Deployed apps use the injected `@appdeploy/sdk` (backend) and `@appdeploy/client` (frontend) packages - never add them to package.json.

**Always call `get_appdeploy_sdk_reference` before writing SDK code.** Request only the features you need; the tool's input schema below lists the available features (database, storage, auth, realtime, AI operations, cron, and more). The returned reference is the source of truth for method signatures and usage rules.

## Tools

### get_appdeploy_sdk_reference

Returns per-feature SDK reference: types, rules, and examples for the requested platform capabilities. Treat this as the implementation contract for those features and review it before writing files that use them. Backend features resolve api automatically when router/transport infrastructure is required.

**Parameters:**
  - `features`: array (required) - Platform capabilities to get docs for. Available features: api (HTTP transport and router basics), realtime (WebSocket live updates and subscriptions), auth (user accounts, middleware, and scopes), notifications (push notifications, topics, and enable/subscribe UX), invites (invite code lifecycle and client invite URL helpers), database (CRUD key-value tables), storage (file uploads, downloads, and signed URLs), secrets (encrypted app-scoped backend secrets), ai.generate (single LLM text or multimodal generation), ai.extract (schema-based structured extraction), ai.ocr (image OCR and transcription), ai.classify (fixed-label text or image classification), ai.scrape (AI-friendly web page scraper), ai.run (bounded generate-to-tool loops), ai.image (image generation), cron (scheduled jobs). Request the exact features your app will use. Backend features resolve api automatically when router/transport infrastructure is required.
