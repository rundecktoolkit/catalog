# SSL Certificates

Automated SSL certificate validity monitoring with PagerDuty alerting. Checks a list of external domains against a configurable expiration threshold and triggers incidents for certificates that are expiring soon or unreachable.

![Community](https://img.shields.io/badge/Tested-Community-blue)
![Commercial](https://img.shields.io/badge/Tested-Commercial-purple)

## Jobs

| Job | File | Description |
|-----|------|-------------|
| Validity Check for External SSL Certificates | `SSL-validity-checker.json` | Iterates through a domain list, checks each certificate's expiry date via OpenSSL, and sends PagerDuty events for certificates within the warning threshold or failing checks |

## How It Works

The job uses a two-step workflow. Step 1 writes a line-separated list of domains to a temporary file using the `\cp -fR` script interpreter trick (the script content *is* the domain list, and it gets copied to a file named `website`). Step 2 reads that file and loops through each domain, connecting on port 443 with `openssl s_client` to pull the certificate dates.

Each domain gets one of three outcomes:

- **Valid**: certificate expiry is beyond the threshold, logged with a green indicator
- **Warning**: certificate expires within the threshold days, triggers a PagerDuty warning event
- **Error**: connection timed out (2s limit) or certificate date couldn't be parsed, triggers a PagerDuty error event

PagerDuty events are sent via the Events API v2 (`/v2/enqueue`) with structured payloads including the domain as source, severity level, and a component label of "SSL Certificate Monitor".

A `key-value-data-multilines` log filter captures all output for downstream use in Rundeck.

## Prerequisites

| Requirement | Details |
|---|---|
| **openssl** | Used to connect and extract certificate dates. Must be available on the execution node. |
| **curl** | Used to POST events to the PagerDuty Events API. |
| **GNU date** | The script uses `date -d` for date parsing, which is GNU-specific. Won't work on macOS/BSD `date` without modification. |
| **timeout** | GNU coreutils `timeout` command, used to cap each SSL check at 2 seconds. |
| **Network access** | Outbound HTTPS (port 443) to all target domains, plus `events.pagerduty.com` for alerting. |
| **Runner tag: US-EAST** | Job is configured with a runner selector filtering on the `US-EAST` tag. Change this to match your runner infrastructure. |
| **Key Storage** | PagerDuty routing key stored at `keys/project/global-production/Travelduty_Routing_key` |

## Job Options

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `days` | Text | No | `7` | Number of days before expiry to consider a certificate "at risk". Certificates expiring within this window trigger a PagerDuty warning event. |
| `routing_key` | Secure | Yes | Key Storage | PagerDuty Events API v2 routing key. Determines which service receives the triggered incidents. Stored securely in Rundeck Key Storage. |

## Domain List

The domain list is maintained inline as the script content of Step 1. To modify the monitored domains, edit the job and update the script block in the first step. Each domain should be on its own line, bare (no protocol prefix, no trailing slash).

The current list includes 30 UK-focused domains across retail, banking, telecoms, energy, and professional services. You could also move this to SCM-managed storage as noted in the step description.

## Usage Notes

**Runner selector**: The job targets runners tagged `US-EAST`. If your runners use different tags or you want to run this locally, update the `runnerSelector` block in the JSON before importing, or change it in the job editor after import.

**Timeout tuning**: The per-domain timeout is hardcoded at 2 seconds in the bash script. For domains behind slow load balancers or in different regions from your runner, you might need to bump this. It's the `TIMEOUT` variable at the top of Step 2's script.

**GNU date dependency**: The `date -d` flag is Linux-specific. If your Rundeck nodes are macOS or BSD, you'll need to swap in `gdate` from coreutils or rewrite the date parsing.

**No schedule defined**: Despite `scheduleEnabled: true`, the `schedules` array is empty. You'll want to add a cron schedule after import, something like daily at 06:00 makes sense for certificate monitoring.

**Keepgoing is false**: If a step fails, the workflow stops. Since Step 1 just writes the domain file, this mainly matters if the file write fails. The bash loop in Step 2 handles individual domain failures internally and continues to the next domain regardless.

## PagerDuty Event Structure

Events sent to PagerDuty follow this structure for reference:

```json
{
  "routing_key": "<from option>",
  "event_action": "trigger",
  "payload": {
    "summary": "SSL certificate for example.com expires on 2025-03-15",
    "source": "example.com",
    "severity": "warning",
    "component": "SSL Certificate Monitor",
    "group": "SSL Certificates",
    "class": "Certificate Expiration",
    "custom_details": {
      "domain": "example.com",
      "severity": "warning",
      "description": "SSL certificate for example.com expires on 2025-03-15"
    }
  }
}
```
