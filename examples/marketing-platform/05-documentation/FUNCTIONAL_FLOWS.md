# Functional Flows — Marketing Automation Platform

## Actors

| Actor | Role | Permissions |
|---|---|---|
| Agency Admin | Manages the agency account, users, billing, and platform settings | Full access to all features; can invite and remove users; manages billing and payment |
| Marketer | Day-to-day campaign operator | Manages contacts and lists; creates templates, campaigns, and automations; views analytics |
| Viewer/Client | External stakeholder with read-only access | Views campaign reports and contact lists; can export reports; cannot modify any data |

## Flows by Actor

### Agency Admin

#### Flow: Invite a New User

**Goal**: Add a team member or client to the platform with the appropriate role.

**Steps**:
1. The Admin opens the Users section from the main menu.
2. The Admin clicks "Invite User" and enters the person's email address.
3. The Admin selects a role: Marketer or Viewer.
4. The system sends an invitation email to that address.
5. The invited person clicks the link, sets their password, and gains access.

**Outcome**: The new user can log in with the assigned role.

**Possible errors**:
- The email address is already registered -- the system shows "This email is already in use" and the Admin can resend the invitation or check existing users.
- The invitation link expires after 48 hours -- the Admin can send a new invitation.

#### Flow: Configure Billing Plan

**Goal**: Select or change the agency's subscription plan and payment method.

**Steps**:
1. The Admin opens the Billing section.
2. The Admin sees the current plan, contact usage, and email send usage for this billing cycle.
3. The Admin selects a new plan from the available options.
4. The system shows the price difference and what the new limits will be.
5. The Admin enters or updates the payment method.
6. The Admin confirms the change.
7. The system applies the new plan immediately and adjusts limits.

**Outcome**: The agency is on the new plan with updated contact and email limits.

**Possible errors**:
- Payment method is declined -- the system shows "Payment failed. Please try a different card." and the plan is not changed.
- Downgrading would exceed new limits -- the system warns "Your current usage exceeds the selected plan. Remove contacts or wait until next cycle."

#### Flow: View All Analytics

**Goal**: Review performance across all campaigns and team activity.

**Steps**:
1. The Admin opens the Analytics dashboard.
2. The system displays aggregate metrics: total emails sent, average open rate, average click rate, bounce rate, and conversion rate.
3. The Admin can filter by date range or specific campaigns.
4. The Admin can drill down into any campaign for detailed metrics.

**Outcome**: The Admin has a complete picture of platform performance.

### Marketer

#### Flow: Import Contacts

**Goal**: Add a batch of contacts from a spreadsheet into the platform.

**Steps**:
1. The Marketer opens the Contacts section and clicks "Import."
2. The Marketer uploads a CSV or Excel file.
3. The system validates the file: checks for required columns (email), detects duplicates, and flags invalid email addresses.
4. The system shows a preview: how many new contacts will be added, how many are duplicates, and how many have errors.
5. The Marketer confirms the import.
6. The system adds the new contacts and skips duplicates.
7. The Marketer sees a summary: "245 contacts added, 12 duplicates skipped, 3 invalid emails ignored."

**Outcome**: New contacts are available in the platform for campaigns and automations.

**Possible errors**:
- File format is not supported -- the system shows "Please upload a CSV or Excel file."
- The file has no recognizable email column -- the system shows "Could not find an email column. Please check your file."
- Import would exceed the plan's contact limit -- the system shows "This import would add 500 contacts but your plan only allows 200 more. Please upgrade or remove existing contacts."

#### Flow: Create and Send a Campaign

**Goal**: Design an email campaign, select recipients, and send or schedule it.

**Steps**:
1. The Marketer opens the Campaigns section and clicks "New Campaign."
2. The Marketer fills in the campaign name, subject line, sender name, and sender email.
3. The Marketer selects a template or creates content from scratch.
4. The Marketer selects one or more recipient lists and optionally a segment.
5. The Marketer previews the email with sample contact data.
6. The Marketer either clicks "Send Now" or selects a date and time and clicks "Schedule."
7. If sending now, the system checks that the email quota has not been exceeded, then begins sending in the background.
8. If scheduling, the campaign status changes to "Scheduled" and the system will send it at the chosen time.
9. The Marketer sees the campaign move through statuses: Draft, Scheduled, Sending, Sent.

**Outcome**: The email is delivered (or will be delivered at the scheduled time) to all selected recipients.

**Possible errors**:
- No recipients selected -- the system shows "Please select at least one list."
- Email quota exceeded -- the system shows "Your plan allows 10,000 emails this month and you have 800 remaining. This campaign targets 2,000 contacts. Please upgrade your plan."
- Template has unresolved merge tags -- the system warns which tags have no matching contact fields.

#### Flow: Manage Lists and Tags

**Goal**: Organize contacts into lists and apply tags for segmentation.

**Steps**:
1. The Marketer creates a new list by giving it a name and description.
2. The Marketer adds contacts to the list individually or in bulk.
3. The Marketer applies tags to contacts (e.g., "VIP", "Trial User", "Webinar Attendee").
4. The Marketer creates a segment with filter rules (e.g., "contacts tagged VIP who joined in the last 30 days").
5. The system evaluates the segment and shows how many contacts match.

**Outcome**: Contacts are organized and ready for targeted campaigns.

#### Flow: Configure an Automation

**Goal**: Set up an automated email sequence triggered by a contact action.

**Steps**:
1. The Marketer opens the Automations section and clicks "New Automation."
2. The Marketer selects a trigger: "When a contact is added to a list," "When a tag is applied," or "When a date field is reached."
3. The Marketer configures the trigger details (which list, which tag, which date field).
4. The Marketer adds steps to the workflow:
   - Send an email (select a template)
   - Wait for a period (e.g., 3 days)
   - Add a tag
   - Move to another list
5. The Marketer reviews the workflow and clicks "Activate."
6. The automation status changes to "Active."

**Outcome**: The automation runs automatically whenever the trigger condition is met.

**Possible errors**:
- No steps configured -- the system shows "Please add at least one step to your automation."
- Selected template has been deleted -- the system shows "The template used in Step 2 no longer exists. Please select a different one."

#### Flow: Pause and Resume an Automation

**Goal**: Temporarily stop an automation without losing progress, then restart it.

**Steps**:
1. The Marketer opens an active automation and clicks "Pause."
2. The system stops processing new triggers and pauses any in-progress sequences.
3. Contacts already mid-sequence stay at their current step.
4. When ready, the Marketer clicks "Resume."
5. The system reactivates the automation: new triggers are processed and paused sequences continue from where they left off.

**Outcome**: The automation resumes without skipping steps or duplicating actions.

#### Flow: View Campaign Analytics

**Goal**: Understand how a sent campaign performed.

**Steps**:
1. The Marketer opens a sent campaign.
2. The system displays metrics: total sent, delivered, opens, unique opens, clicks, unique clicks, bounces, unsubscribes, and conversions.
3. The Marketer can see open rate, click-through rate, and bounce rate as percentages.
4. The Marketer can view a timeline showing when engagement happened.
5. Note: metrics may reflect a delay of up to 5 minutes from actual engagement.

**Outcome**: The Marketer knows how the campaign performed and can plan improvements.

### Viewer/Client

#### Flow: View Campaign Reports

**Goal**: Check the results of campaigns run by the agency.

**Steps**:
1. The Viewer logs in and sees the Reports section.
2. The Viewer selects a campaign to review.
3. The system shows the same metrics as the Marketer sees: sent, opens, clicks, bounces, conversions.
4. The Viewer can filter by date range.

**Outcome**: The Viewer understands campaign performance without needing to ask the agency.

#### Flow: Export a Report

**Goal**: Download campaign data for use in presentations or external tools.

**Steps**:
1. The Viewer opens a campaign report.
2. The Viewer clicks "Export" and selects a format: CSV or PDF.
3. The system generates the file and downloads it.

**Outcome**: The Viewer has the data in a portable format.

#### Flow: View Contact Lists

**Goal**: See the contacts managed by the agency (read-only).

**Steps**:
1. The Viewer opens the Contacts section.
2. The Viewer can browse contacts, search by name or email, and filter by list.
3. The Viewer cannot add, edit, or delete any contacts.

**Outcome**: The Viewer can review the contact database without risk of accidental changes.

## Cross-Actor Flows

### Flow: Campaign Lifecycle (Admin + Marketer + Viewer)

1. The **Marketer** creates a campaign, selects recipients, and schedules it for next Tuesday at 9 AM.
2. The system changes the campaign status to "Scheduled."
3. On Tuesday at 9 AM, the system begins sending. The status changes to "Sending."
4. The **Admin** receives a notification when sending completes. The status changes to "Sent."
5. Over the next few hours, engagement events (opens, clicks) arrive and update the campaign metrics.
6. The **Viewer** logs in later that day and reviews the campaign report with up-to-date metrics.

### Flow: Billing Limit Blocks a Campaign (Admin + Marketer)

1. The **Marketer** tries to send a campaign targeting 5,000 contacts.
2. The system checks the billing plan and finds only 1,000 emails remain in this cycle.
3. The system blocks the send and notifies the Marketer: "Your plan's email limit has been reached."
4. The **Admin** receives a billing alert: "Your agency has reached 100% of email quota."
5. The **Admin** upgrades the billing plan.
6. The **Marketer** retries the send, which now succeeds.

### Flow: Contact Unsubscribes (Marketer + System)

1. A contact clicks the "Unsubscribe" link in a received email.
2. The system records the unsubscribe event.
3. If the contact chose to unsubscribe from a specific list, they are removed from that list only.
4. If the contact chose to unsubscribe globally, they are removed from all lists and will not receive any future emails.
5. The **Marketer** sees the updated subscription status in the contact record and the unsubscribe count in campaign analytics.

## State Machines

### Campaign State Machine

- **Initial state**: Draft
- **Transitions**:
  - From **Draft** to **Scheduled** -- triggered when the Marketer sets a send date and clicks "Schedule"
  - From **Scheduled** to **Sending** -- triggered automatically by the system at the scheduled time, or immediately if the Marketer clicks "Send Now"
  - From **Sending** to **Sent** -- triggered automatically when all emails have been delivered (or failed and logged)
- **Forbidden transitions**:
  - Draft cannot go directly to Sending or Sent -- must pass through Scheduled
  - Scheduled cannot go back to Draft -- the Marketer must cancel the schedule first, which returns it to Draft
  - Sent cannot go back to any previous state -- it is final
- **Special rule**: A campaign can only be edited while in Draft status. Once scheduled, the content is locked.

### Automation State Machine

- **Initial state**: Draft
- **Transitions**:
  - From **Draft** to **Active** -- triggered when the Marketer clicks "Activate"
  - From **Active** to **Paused** -- triggered when the Marketer clicks "Pause"
  - From **Paused** to **Active** -- triggered when the Marketer clicks "Resume"
- **Forbidden transitions**:
  - Draft cannot go directly to Paused -- must be activated first
  - Paused does not go back to Draft -- it returns to Active when resumed
- **Special rule**: When paused, contacts currently mid-sequence keep their position. When resumed, they continue from where they stopped.

## Business Rules

1. Only Agency Admins can invite or remove users and assign roles.
2. Only Agency Admins can view and change billing plans and payment methods.
3. Marketers can create, edit, and send campaigns, but cannot manage billing or users.
4. Viewers can only view reports and contact lists; they cannot create, edit, or delete anything.
5. A campaign can only be edited while its status is "Draft." Once scheduled, its content is locked.
6. Campaign status must follow the sequence: Draft, Scheduled, Sending, Sent. No steps can be skipped.
7. If some emails in a campaign fail to send, the Marketer can retry. The system will only resend to contacts who did not receive the email, avoiding duplicates.
8. Automations can be paused and resumed without losing the progress of contacts currently in the sequence.
9. Contacts can unsubscribe from a specific list (they stay on other lists) or globally (they are removed from all lists and receive no further emails).
10. The system blocks campaign sends that would exceed the plan's email quota. It does not allow overage charges.
11. The system blocks adding contacts beyond the plan's contact limit.
12. All dates and times in the system are stored and displayed in UTC.
13. Email sending happens in the background. Large campaigns (100,000+ contacts) do not freeze or slow down the interface.
14. Analytics data (opens, clicks, bounces) may take up to 5 minutes to appear after the actual event occurs.
15. Contact email addresses must be unique within an agency. Importing a duplicate email updates the existing contact rather than creating a new one.
