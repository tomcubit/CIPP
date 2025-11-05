# Autotask Stale Ticket Monitor (n8n)

This workflow (`n8n/workflows/autotask-stale-ticket-monitor.json`) sends a daily digest for Autotask tickets that have not received an internal response in the past two days and are not waiting on the customer.

## Prerequisites

- n8n v0.200+ (earlier versions may require adjusting node type versions)
- Autotask API credentials configured in n8n as an `Autotask API` credential
- SMTP (or compatible) credential configured for outbound email
- Accurate status IDs for your “Awaiting Customer” (or equivalent) states

## Deployment Steps

1. **Import the workflow**
   - In n8n, choose *Import from File* and select `autotask-stale-ticket-monitor.json`.

2. **Wire credentials**
   - Open the `Find Stale Candidates` node and map it to your Autotask credential.
   - Open `Send Notification Email` and assign your SMTP (or other email) credential.

3. **Configure defaults**
   - Open the `Set Config` node and update:
     - `staleAfterHours`: how many hours without an internal activity before escalation (48 = 2 days by default).
     - `serviceDeskEmail`: fallback recipient and default sender.
     - `fromEmail`: email address that appears in the notification.
     - `ticketUrlBase`: base URL that links directly to a ticket in your Autotask tenant (append ticket number or ID automatically).
     - `awaitingCustomerStatusIds`: array of status IDs that indicate the ticket is waiting on the customer.
     - `notificationRecipients`: list of recipients (comma-separated in the UI) for the digest email.

4. **Adjust the schedule**
   - The `Schedule Check` node runs daily at 07:00. Modify the cron settings to suit your support hours.

5. **Test**
   - Temporarily reduce `staleAfterHours` (e.g., to `1`) and execute from the `Schedule Check` node.
   - Confirm the email renders as expected and the ticket links are correct.

6. **Activate**
   - Once validated, toggle the workflow to *Active* in n8n.

## How It Works

- `Schedule Check` triggers the workflow on a schedule.
- `Set Config` centralises runtime settings for easy adjustment.
- `Persist Config` stores the config in workflow static data for downstream nodes.
- `Find Stale Candidates` pulls all Autotask tickets (respecting the configured fields) via the native connector.
- `Filter Tickets` excludes tickets that are awaiting the customer and enforces the “no response in X hours” rule.
- `Build Email Body` generates both HTML and plain-text summaries, including a basic call-to-action.
- `Send Notification Email` sends a single digest to the configured recipients when one or more tickets qualify. If no tickets require attention, the workflow exits before the email step.

## Extending

- **Per-assignee emails**: Split the items by `assignedResourceEmail` and send personalised notifications.
- **Latest note retrieval**: Loop through qualifying tickets and call the Autotask Ticket Note endpoint to embed the most recent technician note in the email body.
- **Escalation**: Add logic to create an Autotask workflow rule, update priority, or post to Teams/Slack alongside the email.
- **Customer notifications**: Trigger a parallel branch to send proactive customer emails when tickets age beyond another threshold.

## Troubleshooting

- Ensure the status IDs in `awaitingCustomerStatusIds` match your Autotask site. Using names instead of IDs is not supported by the connector.
- The workflow relies on `lastActivityDate`. If your Autotask instance does not update this field for certain actions, consider querying ticket notes directly for more accurate tracking.
- If the email step fails silently, check that your SMTP credential supports the configured `fromEmail`.

Let the service desk team review the digest and adjust the HTML template in `Build Email Body` as needed for branding or additional context.
