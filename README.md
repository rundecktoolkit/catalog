# Rundeck Job Catalog

A curated collection of reusable Rundeck job definitions for enterprise automation, incident response, cloud operations, and infrastructure management. Each job is exported as JSON and organized by domain with documentation covering prerequisites, options, and compatibility.

![Rundeck](https://img.shields.io/badge/Rundeck-4.x+-blue?logo=rundeck&logoColor=white)
![License](https://img.shields.io/badge/License-Apache%202.0-green)
![Jobs](https://img.shields.io/badge/Jobs-2-orange)
![Maintained](https://img.shields.io/badge/Maintained-Yes-brightgreen)

---

## Repository Structure

Each category has its own folder containing:

| File | Purpose |
|------|---------|
| `README.md` | Description, prerequisites, job options, usage notes |
| `*.json` | Exported Rundeck job definition(s) ready for import |

```
rundeck-job-catalog/
├── README.md
├── CONTRIBUTING.md
├── ai/
│   ├── README.md
│   └── ai-job-documenter.json
├── ssl-certificates/
│   ├── README.md
│   └── SSL-validity-checker.json
├── active-directory/
├── ansible-factory/
├── aws/
├── azure-devops/
├── backstage-multicloud/
├── chatops/
├── container-management/
├── database-ops/
├── devops-scm/
├── diagnostics/
├── disaster-recovery/
├── etl-jobs/
├── first-steps/
├── gcp/
├── incident-response/
├── kubernetes/
├── line-of-business/
├── multicloud-provisioning/
├── network-automation/
├── operations-team/
├── remote-locations/
├── salesforce/
├── self-service/
├── terraform/
└── toolbox/
```

## Quick Start

1. Navigate to your project in Rundeck
2. Go to **Jobs > Job Actions > Upload a Job Definition**
3. Select the `.json` file from the relevant folder
4. Choose your duplicate handling preference and click **Upload**

That's it. The job will appear in the group path defined in the export.

## Edition Badges

| Badge | Meaning |
|-------|---------|
| ![Community](https://img.shields.io/badge/Tested-Community-blue) | Tested on Rundeck Community (OSS) |
| ![Commercial](https://img.shields.io/badge/Tested-Commercial-purple) | Tested on PagerDuty Process Automation (Commercial) |

Jobs marked with both badges work on either edition. Some jobs use commercial-only features (Enterprise ACLs, key storage integration, workflow strategy plugins) and are noted accordingly.

---

## Jobs

| Folder | Description | Edition |
|--------|-------------|---------|
| [`ai/`](ai/) | AI-powered documentation generation for Rundeck jobs using OpenAI. Fetches job definitions, generates structured Markdown docs, updates job descriptions in place, and optionally publishes to Confluence. | ![Community](https://img.shields.io/badge/Tested-Community-blue) ![Commercial](https://img.shields.io/badge/Tested-Commercial-purple) |
| [`ssl-certificates/`](ssl-certificates/) | Automated SSL certificate validity monitoring. Checks a list of external domains against a configurable expiration threshold and triggers PagerDuty incidents for certificates nearing expiry or failing checks. | ![Community](https://img.shields.io/badge/Tested-Community-blue) ![Commercial](https://img.shields.io/badge/Tested-Commercial-purple) |

### Planned Folders

The directory structure above includes placeholder folders for future job categories. These are empty for now but represent the intended scope of the catalog. Contributions welcome.

---

## Common Prerequisites

Most jobs have specific prerequisites in their folder README. These come up across multiple categories:

- Rundeck 4.x+ (Community or Commercial)
- `rd` CLI tool for bulk imports
- Node sources configured for your target infrastructure
- Key storage entries for credentials (API tokens, passwords, SSH keys)

## Shared Environment Variables

| Variable | Used By | Purpose |
|----------|---------|---------|
| `PAGERDUTY_INTEGRATION_KEY` | ssl-certificates | PagerDuty Events API v2 routing |
| `OPENAI_API_KEY` | ai | OpenAI chat completions for documentation generation |

## Key Storage Paths

Jobs in this catalog reference the following Key Storage entries. Paths are project-scoped and will need to be created in your Rundeck instance before running the jobs.

| Path | Used By | Purpose |
|------|---------|---------|
| `keys/project/Catalog/openai_key` | AI Job Documenter | OpenAI API token |
| `keys/project/Catalog/local_api_key` | AI Job Documenter | Rundeck API authentication token |
| `keys/project/global-production/Travelduty_Routing_key` | SSL Validity Checker | PagerDuty Events API routing key |

## Folder README Template

Each folder README should follow this structure:

```markdown
# Category Name

Brief description of what these jobs do and when you'd use them.

![Community](https://img.shields.io/badge/Tested-Community-blue)
![Commercial](https://img.shields.io/badge/Tested-Commercial-purple)

## Jobs

| Job | File | Description |
|-----|------|-------------|
| Job Name | `job-name.json` | What it does |

## Prerequisites

| Requirement | Details |
|---|---|
| **Tool/dependency** | What it's used for and where to get it |

## Job Options

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `option_name` | Text | Yes | - | What it controls |

## Usage Notes

Gotchas, tips, important context.
```

## Contributing

Export your job as JSON, drop it in the right folder with a README following the template above, and open a PR. Include which edition you tested on so the badges are accurate. See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide.

## License

Apache 2.0
