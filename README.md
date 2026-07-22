# AIRspec Skills

Reusable AI-agent skills for implementing [AIRspec](https://github.com/bzalk/AIRspec).

## Available skill

### AI Safe Reporting Tool (`ai-safe-reporting-tool`)

Build AI-powered reports and dashboards without giving the AI direct access to application data, credentials, queries, or execution. The skill guides Bolt through secure AIRspec report generation, validation, the server-side Data Broker, trusted components, AIRMark rendering, React integration, interactions, and conformance testing.

## Import into Bolt

1. Open Bolt's **Skills library** or a project's **Settings → Skills**.
2. Select **Add skill → From GitHub**.
3. Enter this repository URL.
4. Select `ai-safe-reporting-tool` as the skill folder.
5. Create the skill and enable it for the project if necessary.

Bolt can apply the skill automatically when a request matches its description, or manually with:

```text
/$ai-safe-reporting-tool
```

Read every imported `SKILL.md` before enabling it. This repository's skill contains no credentials, external tool permissions, or instructions to transmit application data.

## License

MIT
