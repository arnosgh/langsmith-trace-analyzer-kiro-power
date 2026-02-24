# LangSmith Trace Analyzer

A [Kiro Power](https://kiro.dev/powers/) for deep analysis of LangSmith traces with systematic node inspection, latency profiling, cost analysis, and optional report generation.

## Install as a Kiro Power

You can install this Power into Kiro by using the one-click option and importing this URL:

```
https://github.com/arnosgh/langsmith-trace-analyzer-kiro-power
```

Then ask something like:

```
analyze this LangSmith trace: https://smith.langchain.com/public/YOUR_TRACE_ID/r
```

## What It Does

- Navigates LangSmith's web UI via Playwright browser automation
- Systematically inspects every node in the trace tree
- Reads actual payloads and responses (never infers)
- Identifies latency bottlenecks, token waste, and architectural issues
- Generates formal reports in Markdown or PDF format (on request)

## Prerequisites

- Node.js (for Playwright MCP server)

## Structure

```
├── POWER.md          # Power documentation and agent instructions
├── mcp.json          # Playwright MCP server configuration
└── steering/
    ├── trace-analysis.md      # Deep-dive analysis workflow
    └── report-generation.md   # Report generation workflow (Markdown/PDF)
```

## Settings

Once installed, the Playwright MCP server runs automatically. No additional configuration needed.

## License

MIT
