---
name: "appdeploy"
displayName: "AppDeploy"
description: "AppDeploy lets you create and publish real web apps directly from a Kiro conversation. Describe what you want, and AppDeploy turns it into a live public app."
keywords: ["deploy", "deployment", "publish", "hosting", "appdeploy", "web app", "live url", "full-stack", "backend"]
author: "AppDeploy"
---

# AppDeploy

AppDeploy lets you create and publish real web apps directly from a Kiro conversation. Describe what you want, and AppDeploy turns it into a live public app.

Go far beyond static landing pages and websites. With AppDeploy, you can build and deploy dashboards, internal tools, collaborative workspaces, games, AI apps, and full web apps.

Built-in app features include:

- Hosting and public links
- User authentication and databases
- File uploads and storage
- Secure secrets management
- Real-time updates and notifications
- Custom domains
- AI capabilities and scheduled jobs

Ship and iterate with confidence. AppDeploy helps you test, improve, and republish your app with automated QA, visual bug reports, version history, source code access, and rollbacks.

Zero setup required. No Git, command line, hosting provider, or infrastructure setup. Just chat and publish.

## MCP Servers

- **appdeploy**: the AppDeploy deployment platform (remote server at https://api-v2.appdeploy.ai/mcp, OAuth authentication via browser on first use)

## Onboarding Instructions

No local dependencies are needed - the AppDeploy MCP server is remote.

1. Confirm the `appdeploy` MCP server from this power is connected and enabled.
2. The first tool call triggers browser authentication (sign in with Google, Apple, or continue as Guest).
3. Validate the connection with the read-only `get_apps` tool - it lists the user's deployed apps and confirms auth works.
4. If the connection or authentication fails, see https://appdeploy.ai/mcp-docs for troubleshooting.

## Steering Instructions

Load the steering file that matches the task:

- Build and deploy an app -> steering/deploy-workflow.md
- SDK capabilities (database, storage, auth, AI, realtime, cron) -> steering/sdk-features.md
- Manage deployed apps (status, versions, source, QA, deletion) -> steering/app-management.md
- Backend secrets and custom domains -> steering/secrets-and-domains.md
- Full tool reference -> steering/tools-reference.md

The golden path for "build me an app and put it online" is steering/deploy-workflow.md.

## Support & Legal

This power connects to a single MCP server, `appdeploy` (hosted at https://api-v2.appdeploy.ai/mcp). The following applies to both this power package and that server:

- **License:** MIT (SPDX: MIT) for this power package. The AppDeploy MCP server is a hosted service governed by the AppDeploy Terms of Service: https://appdeploy.ai/terms
- **Privacy Policy:** https://appdeploy.ai/privacy
- **Support:** support@appdeploy.ai or https://appdeploy.ai/mcp-docs
