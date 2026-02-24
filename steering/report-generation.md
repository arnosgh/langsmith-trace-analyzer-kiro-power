# Report Generation Workflow

This steering file covers generating a formal report from a completed trace analysis. The analysis MUST be completed first (see trace-analysis steering file).

Reports are generated ONLY when the user explicitly requests one. Ask the user which format they prefer before proceeding.

---

## Supported Formats

### Markdown (.md)
- No dependencies required
- Written directly to a `.md` file
- Includes embedded screenshot references (relative paths)
- Good for version control, sharing in PRs, or further editing

### PDF (.pdf)
- Requires fpdf2: `pip3 install fpdf2`
- Embeds screenshots directly in the document
- Good for formal delivery to stakeholders or customers

---

## Report Structure (Both Formats)

Both formats follow the same structure:

### 1. Header / Cover
- Report title: "LangSmith Trace Analysis Report"
- Trace URL
- Date of analysis
- Summary metrics: total duration, total tokens, total cost, status, node count

### 2. Executive Summary
- 2-3 paragraph overview of the trace
- Architecture pattern identified
- Top findings (high-level, no detail)
- Overall assessment

### 3. Architecture Overview
- Exact call chain from trace tree with node names and latencies
- Models used at each layer
- Agent routing mechanism description

### 4. Step-by-Step Walkthrough

For every node inspected:

```
Node: [Name]
Type: [Chain/LLM/Tool/Retriever/Embedding/Prompt/Parser]
Duration: [Xs]
Model: [exact identifier, if LLM]
Status: [success/error]
Cost: [if available]
TTFT: [if available, streaming LLM only]

Input:
[Summary of what the node received - message count, key content, parameters]

Output:
[Summary of what the node produced - tool call details, text response, error message]

Analysis:
[Your interpretation, clearly separated from facts above]
```

With screenshot reference (markdown) or embedded screenshot (PDF).

### 5. Latency Waterfall Table

| # | Step | Node | Type | Duration | % Total | Cumulative |
|---|------|------|------|----------|---------|------------|

### 6. Token Usage Breakdown Table

| # | LLM Call | Model | Input Tokens | Output Tokens | Total | Context |
|---|----------|-------|-------------|---------------|-------|---------|

### 7. Cost Breakdown Table

| # | LLM Call | Model | Prompt Cost | Completion Cost | Total Cost |
|---|----------|-------|-------------|-----------------|------------|

Include a total row at the bottom. If cost data is not available from the trace, note this and optionally provide estimates based on known model pricing (cite source).

### 8. TTFT Analysis Table (if streaming data available)

| # | LLM Call | Model | TTFT | Total Duration | Generation Rate (tok/s) |
|---|----------|-------|------|----------------|------------------------|

Only include this section if `first_token_time` data was available for any LLM runs. If no streaming data exists, omit this section entirely.

### 9. Findings

Numbered findings, each with:
- **Finding N: [Title]**
- **Observation**: Facts with specific node references
- **Evidence**: Screenshot reference, exact numbers
- **Impact**: Quantified (time, tokens, cost)
- **Severity**: High / Medium / Low

### 10. Recommendations

Numbered recommendations, each with:
- **Recommendation N: [Title]**
- **Related Finding**: Reference to finding number
- **Suggestion**: Framed as opportunity
- **Expected Impact**: Conservative estimate
- **Caveats**: Quality validation needs, trade-offs

### 11. Projected Improvement
- Conservative latency reduction estimate (range)
- Conservative token savings estimate (range)
- Conservative cost savings estimate (range)
- Implementation complexity assessment

---

## Report Language Guidelines

### DO use:
- "agent loop iteration N"
- "response generation step"
- "the Supervisor produces its response to the user"
- "Finding" (not "Root Cause")
- "it may be worth evaluating" (not "you should switch to")
- Exact numbers from node metadata

### DO NOT use:
- "formatting call" or "re-generation call" (unless verified)
- "table formatting step" (unless verified)
- "the Supervisor re-generates the sub-agent's output"
- "Root Cause" (we report observations)
- Approximate numbers when exact numbers are available
- Superlatives or hyperbole

### Model change recommendations:
- Always caveat with quality validation
- Verify throughput/performance claims with real data when possible
- Never recommend a model change without acknowledging the quality trade-off

---

## Markdown Report Generation

Write the report directly as a `.md` file to `output/trace-analysis-report.md`.

### Structure:

```markdown
# LangSmith Trace Analysis Report

**Trace URL:** [url]
**Date:** [date]
**Total Duration:** [duration]
**Total Tokens:** [tokens]
**Total Cost:** [cost or "Not available in trace"]
**Status:** [status]

---

## Executive Summary

[2-3 paragraphs]

---

## Architecture Overview

[Call chain description, models, routing mechanism]

![Trace Overview](screenshots/01-trace-overview.png)

---

## Step-by-Step Walkthrough

### Step 1: [Node Name]

| Property | Value |
|----------|-------|
| Type | [Chain/LLM/Tool/Retriever/Embedding/Prompt/Parser] |
| Duration | [Xs] |
| Model | [identifier or N/A] |
| Status | [success/error] |
| Cost | [value or N/A] |
| TTFT | [value or N/A] |

**Input:** [description]

**Output:** [description]

**Analysis:** [interpretation]

![Node Screenshot](screenshots/02-node-name.png)

[Repeat for every node]

---

## Latency Waterfall

| # | Node | Type | Duration | % Total | Cumulative |
|---|------|------|----------|---------|------------|
| 1 | ...  | ...  | ...      | ...     | ...        |

---

## Token Usage Breakdown

| # | LLM Call | Model | Input Tokens | Output Tokens | Total | Context |
|---|----------|-------|-------------|---------------|-------|---------|
| 1 | ...      | ...   | ...         | ...           | ...   | ...     |

---

## Cost Breakdown

| # | LLM Call | Model | Prompt Cost | Completion Cost | Total Cost |
|---|----------|-------|-------------|-----------------|------------|
| 1 | ...      | ...   | ...         | ...             | ...        |
| **Total** | | | | | **$X.XX** |

---

## TTFT Analysis

_Include only if first_token_time data was available._

| # | LLM Call | Model | TTFT | Total Duration | Generation Rate |
|---|----------|-------|------|----------------|-----------------|
| 1 | ...      | ...   | ...  | ...            | ... tok/s       |

---

## Findings

### Finding 1: [Title]

**Observation:** [facts]
**Evidence:** [screenshot ref, numbers]
**Impact:** [quantified — time, tokens, cost]
**Severity:** [High/Medium/Low]

[Repeat for each finding]

---

## Recommendations

### Recommendation 1: [Title]

**Related Finding:** Finding [N]
**Suggestion:** [opportunity framing]
**Expected Impact:** [conservative estimate]
**Caveats:** [trade-offs]

[Repeat for each recommendation]

---

## Projected Improvement

[Conservative estimates with ranges for latency, tokens, and cost]
```

### Screenshot references in Markdown:
- Use relative paths: `![Caption](screenshots/filename.png)`
- Screenshots should be in `output/screenshots/` alongside the report

---

## PDF Report Generation

The PDF report uses fpdf2. The generation script is provided as a separate file at `output/generate_report.py`.

### Prerequisites:
```bash
pip3 install fpdf2
```

### Key implementation notes:

1. **fpdf2 with Helvetica does not support Unicode** — use only ASCII characters. Replace:
   - Em dashes with " - "
   - Curly quotes with straight quotes
   - Ellipsis with "..."

2. **Screenshot embedding** — use `pdf.image()` with width constrained to page width minus margins

3. **Page breaks** — add automatic page break handling before screenshots and large sections

### Generating the PDF:

1. Build `trace_data.json` from the analysis data (schema below)
2. Save to `output/trace_data.json`
3. Save the generate_report.py script to `output/generate_report.py` (template below)
4. Run:
```bash
pip3 install fpdf2
python3 output/generate_report.py
```
5. PDF saved to `output/trace-analysis-report.pdf`

### generate_report.py template:

Create `output/generate_report.py` with a `TraceReportPDF` class extending `FPDF`. The script should:
- Read `output/trace_data.json`
- Generate a cover page with trace URL, date, duration, tokens, cost, status
- Generate sections for: Executive Summary, Architecture Overview, Step-by-Step Walkthrough, Latency Waterfall table, Token Usage table, Cost Breakdown table, TTFT Analysis table (if data exists), Findings, Recommendations, Projected Improvement
- Embed screenshots from `output/screenshots/`
- Output to `output/trace-analysis-report.pdf`

Key methods needed in the PDF class:
- `header()` / `footer()` — page headers and footers with page numbers
- `section_title(title)` — bold 14pt section headers
- `subsection_title(title)` — bold 11pt subsection headers
- `body_text(text)` — 10pt body text with multi_cell
- `add_screenshot(filepath, caption)` — embed image with page break handling
- `table_row(cols, widths, bold, fill)` — render a table row with borders

### trace_data.json schema:

The analysis phase should produce a `trace_data.json` file with this structure:

```json
{
  "url": "https://smith.langchain.com/public/...",
  "date": "2025-02-23",
  "total_duration": "45.2s",
  "total_tokens": "12,450",
  "total_cost": "$0.42",
  "status": "success",
  "executive_summary": "Plain ASCII text...",
  "architecture_overview": "Plain ASCII text...",
  "nodes": [
    {
      "name": "Supervisor",
      "type": "Chain",
      "duration": "42.1s",
      "model": null,
      "status": "success",
      "cost": null,
      "ttft": null,
      "input_summary": "1 human message: user query about...",
      "output_summary": "Final response with table of...",
      "analysis": "Orchestration wrapper containing...",
      "screenshot": "02-supervisor-chain.png"
    }
  ],
  "latency_waterfall": [
    {
      "node": "LLM (routing)",
      "type": "LLM",
      "duration": "3.2s",
      "percent": "7.1%",
      "cumulative": "3.2s"
    }
  ],
  "token_usage": [
    {
      "call": "Supervisor routing",
      "model": "claude-sonnet-4-20250514",
      "input_tokens": "2,100",
      "output_tokens": "150",
      "total": "2,250",
      "context": "3 messages"
    }
  ],
  "cost_breakdown": [
    {
      "call": "Supervisor routing",
      "model": "claude-sonnet-4-20250514",
      "prompt_cost": "$0.0063",
      "completion_cost": "$0.0023",
      "total_cost": "$0.0086"
    }
  ],
  "ttft_analysis": [
    {
      "call": "Sub-agent response",
      "model": "claude-sonnet-4-20250514",
      "ttft": "1.2s",
      "total_duration": "8.5s",
      "generation_rate": "42 tok/s"
    }
  ],
  "findings": [
    {
      "number": 1,
      "title": "Large tool response inflating downstream context",
      "observation": "The fetch_orders tool returned 41 fields per record...",
      "evidence": "Screenshot 07, RAW view shows 41 unique field names...",
      "impact": "Added ~3,000 tokens to every subsequent LLM call",
      "severity": "Medium"
    }
  ],
  "recommendations": [
    {
      "number": 1,
      "title": "Filter tool response fields before passing to LLM",
      "related_finding": "Finding 1",
      "suggestion": "It may be worth evaluating a field filter...",
      "expected_impact": "Estimated 20-30% reduction in downstream token usage",
      "caveats": "Requires identifying which fields the LLM actually uses"
    }
  ],
  "projected_improvement": "Based on the findings, conservative estimates suggest..."
}
```

**IMPORTANT**: All string values in trace_data.json must use only ASCII characters for PDF generation. Replace em dashes with " - ", curly quotes with straight quotes, ellipsis with "...".

**Note**: The `cost_breakdown` and `ttft_analysis` arrays may be empty if the trace did not include cost or streaming data. The PDF script should handle empty arrays gracefully by omitting those sections.
