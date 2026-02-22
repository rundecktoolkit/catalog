# Rundeck Job Catalog

A curated collection of reusable Rundeck job definitions for enterprise automation, incident response, cloud operations, and infrastructure management. Each job is exported as JSON and organized by domain with documentation covering prerequisites, options, and compatibility.

![Rundeck](https://img.shields.io/badge/Rundeck-4.x+-blue?logo=rundeck&logoColor=white)
![License](https://img.shields.io/badge/License-Apache%202.0-green)
![Jobs](https://img.shields.io/badge/Jobs-1+-orange)
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
├── first-steps/
│   ├── README.md
│   └── *.json
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
├── ssl-certificates/
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
| [`active-directory/`](active-directory/) | User/group management, reporting, break glass access with auto-expiry, PowerShell prerequisite checks | ![Commercial](https://img.shields.io/badge/Tested-Commercial-purple) |

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

- Required plugins
- Credential / key storage entries
- Node source requirements
- Network access needed

## Job Options

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `option_name` | Text | Yes | - | What it controls |

## Usage Notes

Gotchas, tips, important context.
```

## Common Prerequisites

Most jobs have specific prerequisites in their folder README. These come up across multiple categories:

- Rundeck 4.x+ (Community or Commercial)
- `rd` CLI tool for bulk imports
- Node sources configured for your target infrastructure
- Key storage entries for credentials (API tokens, passwords, SSH keys)

## Shared Environment Variables

| Variable | Used By | Purpose |
|----------|---------|---------|
| `PAGERDUTY_API_TOKEN` | incident-response, operations-team | PagerDuty API access |
| `SLACK_WEBHOOK_URL` | chatops, incident-response | Notification delivery |
| `AZURE_DEVOPS_PAT` | azure-devops | Azure DevOps authentication |
| `AWS_DEFAULT_REGION` | aws, multicloud-provisioning | Default AWS region |
| `KUBECONFIG` | kubernetes, container-management | Cluster access |

## Contributing

Export your job as JSON, drop it in the right folder with a README following the template above, and open a PR. Include which edition you tested on so the badges are accurate. See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide.

## License

Apache 2.0

---

