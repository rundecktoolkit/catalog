# AI Job Documenter (OpenAI Version)

> Automated documentation generation for Rundeck / Runbook Automation jobs using OpenAI

[![GenAI Documentation](https://img.shields.io/badge/GenAI%20Documentation-green)](https://img.shields.io/badge/GenAI%20Documentation-green)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/downloads/)

## What It Does

This Rundeck job takes any job UUID, pulls its full definition via the Rundeck API, sends it to OpenAI with a structured prompt, and writes the generated Markdown documentation back into the job's description field. Optionally, it also pushes the docs to a Confluence page.

The prompt engineering is opinionated: it enforces factual documentation only (no hallucinated values), analyses actual script content to identify prerequisites, and produces a consistent structure across all documented jobs.

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────────┐
│ Rundeck API │────>│ Job JSON     │────>│ OpenAI API  │────>│ Markdown     │
│ GET /job/   │     │ Definition   │     │ GPT-4o      │     │ Documentation│
└─────────────┘     └──────────────┘     └─────────────┘     └──────┬───────┘
                                                                     │
                                                          ┌──────────┴──────────┐
                                                          │                     │
                                                    ┌─────▼─────┐     ┌────────▼────────┐
                                                    │ Rundeck   │     │ Confluence      │
                                                    │ PUT /job/ │     │ (optional)      │
                                                    └───────────┘     └─────────────────┘
```

1. Fetches the job definition JSON from Rundeck API (v44)
2. Sends the JSON to OpenAI with system + user prompts that enforce structured output
3. Cleans up any LLM artifacts (code fences, etc.)
4. Updates the job's `description` field in Rundeck via the import API (v47)
5. If Confluence is enabled, converts the Markdown to HTML and creates a page

## Prerequisites

### Runtime Environment

The job executes as an inline Python script within Rundeck, so the node running this job needs:

| Requirement | Details |
|---|---|
| **Python 3.8+** | Uses f-strings with `=` specifier and `tuple[str, str]` type hints (3.9+ for the latter, though it'll work on 3.8 with `from __future__ import annotations`) |
| **pip packages** | `requests`, `openai`, `markdown` |
| **Network access** | Outbound HTTPS to OpenAI API (`api.openai.com`) and your Rundeck server |
| **Rundeck API token** | Needs permissions to read job definitions and import/update jobs |
| **OpenAI API key** | Any key with access to the chosen model (gpt-4o recommended) |

### Install Python Dependencies

On the Rundeck node (or in your execution environment):

```bash
pip install requests openai markdown
```

If you're running Rundeck in Docker or a locked-down environment, make sure these are baked into the image or available in a virtualenv that Rundeck's script steps can access.

### Rundeck Key Storage

The job expects two secrets stored in Rundeck Key Storage:

| Key Path | Purpose |
|---|---|
| `keys/project/Catalog/openai_key` | OpenAI API token |
| `keys/project/Catalog/local_api_key` | Rundeck API authentication token |

Create these via the Rundeck UI under Project Settings > Key Storage, or via the API.

### Confluence (Optional)

If you want to publish to Confluence, you'll also need:

| Requirement | Details |
|---|---|
| **Confluence URL** | Your instance base URL (e.g., `https://yourcompany.atlassian.net`) |
| **API Token** | Generated from Atlassian account settings > Security > API Tokens |
| **Space Key** | The short identifier for your target space (visible in the space URL) |
| **Write permissions** | The authenticated user needs permission to create pages in the target space |

## Job Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `JOB_ID` | String | -- | The Rundeck Job UUID to generate documentation for |
| `OPENAI_TOKEN` | Secure | Key Storage | API key for OpenAI or compatible LLM endpoint |
| `API_TOKEN` | Secure | Key Storage | Rundeck API authentication token |
| `RUNBOOK_SERVER` | String | `http://oracle.local:4440` | Full URL of the Rundeck server |
| `OPENAI_MODEL` | Choice | `gpt-4o` | Model selection: gpt-3.5-turbo-16k, gpt-4-turbo, gpt-4o |
| `CONFLUENCE` | Choice | `No` | Enable Confluence publishing |
| `CONFLUENCE_LOCATION` | String | -- | Confluence server base URL |
| `CONFLUENCE_USER` | String | -- | Confluence authentication username/email |
| `CONFLUENCE_TOKEN` | String | -- | Confluence API token |
| `CONFLUENCE_SPACEKEY` | String | `CD` | Target Confluence space key |

## Usage

### Basic: Document a Single Job

Run the job with just the `JOB_ID` parameter set to the UUID of the job you want to document. The other required fields (API tokens, server URL) pull from Key Storage defaults.

```
JOB_ID: f6975c35-c075-4098-94c1-9c692b4698cb
```

The job will fetch the definition, generate docs, and update the description field in-place.

### With Confluence Publishing

Set `CONFLUENCE` to `Yes` and fill in the Confluence parameters. The job creates a new page in the specified space with the job name as the title, prefixed with a metadata table linking back to the Rundeck job.

### Batch Documentation

Since the job supports multiple executions (up to 5 concurrent), you can trigger it in bulk via the Rundeck API or a wrapper job that iterates over job UUIDs:

```bash
for uuid in $(rundeck-cli jobs list --project MyProject --format '%id'); do
  rundeck-cli run --id <documenter-job-id> -- -JOB_ID "$uuid"
done
```

## Prompt Engineering Notes

The prompt uses a system/user split with specific constraints:

**System prompt** defines the role (documentation specialist) and hard rules: only document what exists in the JSON, never invent values, analyse actual script content for prerequisites.

**User prompt** provides the output template with conditional sections. The model is told to skip sections entirely if there's no data, rather than filling in placeholders.

Temperature is set to 0.3 for consistency. Max tokens is 4096 to handle complex multi-step jobs.

The output format includes a GenAI badge so it's obvious the docs were machine-generated, and a disclaimer at the bottom.

## Known Gotchas

**`CONFLUENCE_KEY` vs `CONFLUENCE_TOKEN`**: There's a mismatch in the CONFIG dict. The code references `CONFIG["confluence_key"]` but the option is named `CONFLUENCE_TOKEN`. The CONFIG dict maps `"confluence_key"` from `@option.CONFLUENCE_KEY@` which doesn't exist as an option. If you're using Confluence publishing, you'll need to either rename the option or fix the CONFIG mapping.

**API version mismatch**: The job uses API v44 for fetching definitions but v47 for importing. This is intentional (the import endpoint was added later), but be aware if you're running an older Rundeck version.

**Type hints**: The `tuple[str, str]` syntax in `get_project_info` requires Python 3.9+. On 3.8, add `from __future__ import annotations` at the top or change to `typing.Tuple[str, str]`.

**Markdown library**: The `markdown` pip package is needed for the Confluence HTML conversion. It's not used if you're only updating Rundeck descriptions.

## License

MIT License. See the script header for the full text.
