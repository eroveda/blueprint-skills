# User Guide — Marketing Automation Platform

## Welcome

The Marketing Automation Platform helps your agency manage contacts, send email campaigns, set up automated sequences, and track how your emails perform. Whether you are an Admin managing the account, a Marketer running campaigns, or a Viewer checking reports, this guide walks you through everything you need to know.

## Getting Started

### First Login

1. You will receive an invitation email from your agency's Admin. Click the link in that email.
2. Set your password on the registration page. Choose something strong -- at least 8 characters with a mix of letters and numbers.
3. Log in with your email and new password.
4. You will land on the Dashboard. What you see depends on your role:
   - **Admins** see account settings, billing info, and team management alongside campaign data.
   - **Marketers** see contacts, campaigns, automations, and analytics.
   - **Viewers** see campaign reports and contact lists (read-only).
5. You are ready to start working.

### Setting Up Your Account

**Update your profile**: Click your avatar in the top-right corner, then "Profile." You can change your name and password here.

**Invite teammates** (Admins only): Go to Settings > Users > "Invite User." Enter their email and choose their role (Marketer or Viewer). They will get an invitation email.

**Configure billing** (Admins only): Go to Settings > Billing. Select a plan that matches your contact list size and email volume. Add a payment method to activate your plan.

## Daily Usage

### For Admins

#### Managing Users

1. Go to Settings > Users.
2. You will see a list of everyone on your account with their role.
3. To invite someone new, click "Invite User," enter their email, and pick a role.
4. To remove someone, click the menu next to their name and select "Remove."
5. To change someone's role, click the menu and select "Change Role."

#### Managing Billing

1. Go to Settings > Billing.
2. You will see your current plan, how many contacts you are using, and how many emails you have sent this billing cycle.
3. If you are approaching your limits, you will see a warning banner.
4. To change your plan, click "Change Plan," select a new one, and confirm.
5. To update your payment method, click "Payment Method" and enter new card details.

#### Viewing All Analytics

1. Go to Analytics from the main menu.
2. You will see aggregate numbers across all campaigns: total sent, open rate, click rate, bounce rate.
3. Use the date picker to focus on a specific time range.
4. Click on any campaign name to see its individual performance.

### For Marketers

#### Adding Contacts

**One at a time**:
1. Go to Contacts and click "Add Contact."
2. Fill in the email address (required), first name, last name, and any tags.
3. Choose which lists to add them to.
4. Click "Save."

**Importing from a file**:
1. Go to Contacts and click "Import."
2. Upload a CSV or Excel file. Make sure it has an "email" column.
3. Review the preview: the system will show you how many contacts will be added, how many are duplicates, and how many have errors.
4. Click "Confirm Import."
5. You will see a summary when it is done.

#### Creating Lists and Segments

**Lists** are static groups you manually manage:
1. Go to Contacts > Lists and click "New List."
2. Give it a name (e.g., "Newsletter Subscribers") and a description.
3. Add contacts to it individually or in bulk.

**Segments** are dynamic groups based on rules:
1. Go to Contacts > Segments and click "New Segment."
2. Set your filter rules (e.g., "Tag equals VIP" and "Created in the last 30 days").
3. The system will show you how many contacts currently match.
4. Save the segment. It will automatically update as contacts change.

#### Creating a Campaign

1. Go to Campaigns and click "New Campaign."
2. Fill in the basics: campaign name, email subject line, sender name, and sender email.
3. Pick a template from your library, or start from scratch.
4. Select which lists or segments should receive this campaign.
5. Click "Preview" to see how the email will look with real contact data.
6. When you are happy:
   - Click "Send Now" to send immediately, or
   - Click "Schedule" and pick a date and time.
7. The campaign will move through these statuses: Draft > Scheduled > Sending > Sent.

Remember: you can only edit a campaign while it is in "Draft" status. Once you schedule it, the content is locked.

#### Setting Up an Automation

1. Go to Automations and click "New Automation."
2. Choose what triggers it:
   - A contact is added to a specific list
   - A tag is applied to a contact
   - A date field on a contact is reached (e.g., birthday)
3. Add steps to the sequence:
   - **Send an email**: pick a template
   - **Wait**: set a delay (e.g., 3 days)
   - **Add a tag**: apply a tag to the contact
   - **Move to list**: move the contact to another list
4. Review the whole flow and click "Activate."

**Pausing and resuming**: Click "Pause" on any active automation to stop it temporarily. Contacts already in the sequence will stay at their current step. Click "Resume" to pick up where everything left off.

#### Checking Campaign Results

1. Go to Campaigns and click on a sent campaign.
2. You will see: total sent, delivered, opens, clicks, bounces, unsubscribes, and conversions.
3. Percentages (open rate, click-through rate, bounce rate) are calculated for you.
4. Note: these numbers may take up to 5 minutes to update after actual engagement.

### For Viewers

#### Viewing Reports

1. Log in and go to Reports.
2. Select a campaign to see its performance: sent, opens, clicks, bounces, conversions.
3. Use the date filter to narrow the time range.

#### Exporting Data

1. Open any report.
2. Click "Export" and choose CSV (for spreadsheets) or PDF (for presentations).
3. The file downloads to your computer.

#### Browsing Contacts

1. Go to Contacts to see the full list.
2. Use the search bar to find specific people by name or email.
3. Filter by list to see a specific group.
4. Remember: you can look but not make changes.

## Understanding the System

### Statuses and What They Mean

**Campaign statuses**:
- **Draft** -- The campaign is being created or edited. You can change anything at this stage.
- **Scheduled** -- The campaign is locked and waiting to send at the chosen date and time. You can cancel the schedule to return it to Draft.
- **Sending** -- The system is actively delivering emails. This happens in the background, so you can keep using the platform.
- **Sent** -- All emails have been delivered (or attempted). This is the final status.

**Automation statuses**:
- **Draft** -- The automation is being configured. It will not trigger for anyone yet.
- **Active** -- The automation is live. Any contact that meets the trigger condition will enter the sequence.
- **Paused** -- The automation is temporarily stopped. No new contacts will enter, and contacts mid-sequence are frozen in place until you resume.

**Contact subscription statuses**:
- **Subscribed** -- The contact will receive emails from campaigns and automations.
- **Unsubscribed (list)** -- The contact opted out of a specific list but may still be on other lists.
- **Unsubscribed (global)** -- The contact opted out of all emails. They will not receive anything.

### Permissions

- **Admins** can: manage users, configure billing, view all analytics, and do everything a Marketer can do.
- **Marketers** can: manage contacts, lists, and tags; create templates; build and send campaigns; configure automations; view analytics.
- **Viewers** can: view campaign reports, browse contact lists, and export reports.
- **Marketers** cannot: manage users, change billing, or access account settings.
- **Viewers** cannot: create, edit, or delete anything.

## Common Scenarios

### Scenario: Launching a Weekly Newsletter

You want to send a newsletter every week to your subscriber list. Here is how:
1. Go to Templates and create a newsletter template with your standard layout.
2. Each week, go to Campaigns and click "New Campaign."
3. Use your newsletter template, write the subject line, and select your "Newsletter Subscribers" list.
4. Preview it, then schedule it for your preferred send time.
5. After it sends, check the analytics to see your open and click rates.

### Scenario: Setting Up a Welcome Sequence for New Signups

You want every new contact to receive a series of welcome emails over their first week:
1. Go to Automations and click "New Automation."
2. Set the trigger to "Contact added to list" and select your "New Signups" list.
3. Add Step 1: Send the "Welcome" email template.
4. Add Step 2: Wait 2 days.
5. Add Step 3: Send the "Getting Started Tips" email template.
6. Add Step 4: Wait 3 days.
7. Add Step 5: Send the "Feature Highlights" email template.
8. Activate the automation. Every new contact on the list will get this sequence automatically.

### Scenario: Cleaning Up After a Campaign with High Bounces

Your last campaign had a 5% bounce rate and you want to clean your list:
1. Go to the sent campaign and look at the bounce details.
2. Hard bounces (invalid addresses) should be removed. The system automatically flags these contacts.
3. Go to Contacts, filter by "hard bounce" status, and review the list.
4. Remove or suppress these contacts to keep your list healthy and protect your sender reputation.

### Scenario: Preparing a Report for a Client

A client wants to see how their campaigns performed this quarter:
1. If the client has a Viewer account, share the login credentials. They can view reports directly.
2. Alternatively, go to any campaign report and click "Export" to download a PDF.
3. You can export multiple campaigns and combine them into a presentation.

## Troubleshooting

### "I can't edit my campaign"

This happens when the campaign is no longer in Draft status. Once you schedule or send a campaign, its content is locked.
**To fix**: If the campaign is Scheduled, cancel the schedule first. This returns it to Draft. If the campaign is Sending or Sent, you will need to create a new campaign.

### "My import says I've exceeded my contact limit"

Your billing plan has a maximum number of contacts. The import would push you over that limit.
**To fix**: Ask your Admin to upgrade the billing plan, or remove contacts you no longer need before importing.

### "I don't see the Send button on my campaign"

A few things could be missing:
- You have not selected any recipient lists. Add at least one list to the campaign.
- The subject line or sender email is empty. Fill in all required fields.
- Your email quota for this billing cycle has been reached. Ask your Admin to check the billing page.

### "My analytics numbers seem outdated"

Engagement data (opens, clicks, bounces) can take up to 5 minutes to appear after the actual event.
**To fix**: Wait a few minutes and refresh the page. If data is still missing after 15 minutes, contact support.

### "A contact says they unsubscribed but I still see them in my list"

Check the contact's subscription status. If they unsubscribed from a specific list, they are removed from that list but may still appear on other lists. If they unsubscribed globally, they will appear in the system but will be marked as "Unsubscribed (global)" and will not receive any emails.

## FAQs

### Can I undo sending a campaign?

No. Once a campaign starts sending, it cannot be stopped or recalled. Always use the Preview feature and send a test to yourself before scheduling a real send.

### What happens if I hit my email limit mid-campaign?

The system checks your remaining quota before it starts sending. If the campaign would exceed your limit, it blocks the entire send and asks you to upgrade. It will not send to some contacts and skip others.

### Can I reuse a template across multiple campaigns?

Yes. Templates are reusable. Create a template once, then select it in any campaign. Editing the template does not affect already-sent campaigns.

### How do automations handle contacts who are already mid-sequence when I pause?

They stay exactly where they are. If a contact was waiting at Step 3 when you paused, they will continue from Step 3 when you resume. No steps are skipped or repeated.

### Can a contact be in multiple automations at the same time?

Yes. A contact can be enrolled in as many automations as they qualify for. Each automation tracks their progress independently.

### What is the difference between a list and a segment?

A list is a static group -- you manually add and remove contacts. A segment is dynamic -- you set rules (like "tagged VIP and joined this month") and the system automatically includes any contacts that match.

### Are all times in UTC?

Yes. All dates and times in the platform are in UTC. Make sure to account for your local time zone when scheduling campaigns.

## Getting Help

- **Email**: support@your-platform-domain.com
- **Response time**: We aim to respond within 4 business hours.
- **When contacting support**, please include:
  - Your agency name
  - The page or feature where you encountered the issue
  - A screenshot if possible
  - The steps you took before the problem occurred
