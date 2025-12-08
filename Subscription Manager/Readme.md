
## **Overview**
An automated n8n workflow that tracks subscription payments, automatically calculates renewal dates, and sends email notifications for upcoming payments. The workflow runs weekly and maintains subscription data in Google Sheets with automatic date updates.

## **n8n Workflow**

<img width="1861" height="310" alt="image" src="https://github.com/user-attachments/assets/87ce3ca3-58ac-4bfc-a30d-04ebc73f7c7d" />

## **Features**

✅ **Automatic Date Calculation** - Calculates next payment dates based on subscription type (monthly/annual)

✅ **Self-Updating** - Automatically updates payment dates in Google Sheets when payments are due

✅ **Weekly Summary Emails** - Sends formatted HTML emails with all subscription details

✅ **Urgent Annual Alerts** - Special notifications for annual subscriptions renewing within 14 days

✅ **Organized Display** - Separates monthly and annual subscriptions in email summaries

✅ **Zero Maintenance** - Fully automated with no manual intervention required

## **How It Works**

### **Workflow Schedule**

- **Runs:** Every Monday at 8:00 AM
- **Trigger:** Schedule Trigger node

### **Workflow Process**

1. **Read Subscriptions** - Fetches all subscription data from Google Sheets
2. **Calculate Next Payment Dates** - Computes next payment date based on:
    - Last payment date + 1 month (for monthly subscriptions)
    - Last payment date + 1 year (for annual subscriptions)
3. **Update Google Sheets** - Writes calculated dates back to the sheet
4. **Check for Urgent Renewals** - Identifies annual subscriptions renewing within 14 days
5. **Send Notifications**:
    - **Urgent Alert** - If annual subscriptions are due within 14 days
    - **Weekly Summary** - Regular summary of all subscriptions
  
### **Automatic Date Updates**

The workflow automatically maintains payment dates:

- **Next Payment Date**: Recalculated every week based on last payment date
- **Last Payment Date**: Automatically updated when a payment becomes due

## **Setup Instructions**

### **1. Google Sheets Setup**

Create a Google Sheet with the following columns:

| Column Name        | Type     | Description           | Example           |
|--------------------|----------|------------------------|-------------------|
| Name               | Text     | Subscription name      | Netflix           |
| Amount             | Number   | Payment amount         | 15.99             |
| Currency           | Text     | Currency code          | USD               |
| Type               | Text     | Billing frequency      | Monthly or Annual |
| last_payment_date  | Date     | Last payment date      | 2024-12-01        |
| next_payment       | Date     | Next payment date      | 2025-01-01        |
| is_active          | Boolean  | Active status          | TRUE              |
| email              | Email    | Notification email     | user@example.com  |

**mportant Notes:**

- Use `YYYY-MM-DD` format for all dates
- `Type` must be either "Monthly" or "Annual" (case-insensitive)
- All subscriptions should use the same email address in the `email` column

### **2. n8n Workflow Configuration**

### **A. Google Sheets Credentials**

1. In n8n, configure Google Sheets OAuth2 credentials
2. Update the following nodes with your Google Sheet ID:
    - **Read Subscriptions** node
    - **Update Next Payment Dates** node

### **B. Email Configuration**

1. Configure SMTP credentials in n8n
2. Update the **Send Weekly Summary** node:
    - Set `fromEmail` to your sender email address
3. Update the **Send Urgent Annual Alert** node:
    - Set `fromEmail` to your sender email address

### **C. Activate Workflow**

1. Test the workflow manually first
2. Activate the workflow to enable weekly execution

## **Email Notifications**

### **Weekly Summary Email**

Sent every Monday with:

- **Monthly Subscriptions Section** (blue header)
    - Subscription name, amount, next payment, last payment
- **Annual Subscriptions Section** (orange header)
    - Subscription name, amount, next payment, last payment

### **Urgent Annual Alert Email**

Sent when annual subscriptions are due within 14 days:

- **Red alert banner** with urgency indicator
- **Days until renewal** countdown
- **Color-coded rows**:
    - Dark red: ≤7 days until renewal
    - Light red: 8-14 days until renewal
- **Action required notice**

### **Workflow Nodes**

| Node Name               | Type              | Purpose                                         |
|-------------------------|-------------------|-------------------------------------------------|
| Weekly Schedule         | Schedule Trigger  | Runs workflow every Monday at 8 AM              |
| Read Subscriptions      | Google Sheets     | Fetches subscription data                       |
| Calculate Next Payment Dates | Code         | Calculates next payment dates                   |
| Update Next Payment Dates    | Google Sheets | Updates dates in sheet                          |
| Pass Through Data1      | Code              | Prepares data for downstream nodes              |
| Check Annual Renewals   | Code              | Identifies urgent annual renewals               |
| Route By Urgency        | Switch            | Routes to urgent or regular flow                |
| Send Urgent Annual Alert| Email Send        | Sends urgent renewal notifications              |
| Analyze Due Dates       | Code              | Generates weekly summary HTML                   |
| Check If Any Due        | IF                | Validates subscriptions exist                   |
| Send Weekly Summary     | Email Send        | Sends weekly summary email                      |

### **Data Flow**

Weekly Schedule
    ↓
Read Subscriptions (Google Sheets)
    ↓
Calculate Next Payment Dates (Code)
    ↓
Update Next Payment Dates (Google Sheets)
    ↓
Pass Through Data1 (Code)
    ↓
Check Annual Renewals (Code)
    ↓
Route By Urgency (Switch)
    ↓                    ↓
Urgent Alert      Weekly Summary

## **Customization Options**

### **Change Schedule**

Edit the **Weekly Schedule** node:

- Current: Every Monday at 8:00 AM
- Modify: `triggerAtDay`, `triggerAtHour`, `triggerAtMinute`

### **Adjust Urgent Alert Window**

Edit the **Check Annual Renewals** node:

- Current: 14 days before renewal
- Modify: `twoWeeks.setDate(twoWeeks.getDate() + 14)` (change 14 to desired days)

### **Customize Email Templates**

Edit the **Analyze Due Dates** or **Check Annual Renewals** nodes:

- Modify HTML structure
- Change colors, fonts, styling
- Add/remove table columns

### **Add More Subscription Types**

Edit the **Calculate Next Payment Dates** node:

- Add new subscription types (e.g., quarterly, bi-annual)
- Modify date calculation logic

## **Troubleshooting**

### **Dates Not Updating**

- Verify Google Sheets credentials are valid
- Check that `row_number` column exists in sheet
- Ensure `Next_payment` column name matches exactly

### **Emails Not Sending**

- Verify SMTP credentials are configured
- Check `fromEmail` address is valid
- Ensure `email` column in sheet has valid email addresses

### **Wrong Dates Calculated**

- Verify `last_payment_date` format is `YYYY-MM-DD`
- Check `Type` column values are "Monthly" or "Annual"
- Ensure dates are not empty/null

### **No Urgent Alerts**

- Verify annual subscriptions have `Type` = "Annual" or "Annually"
- Check that `next_payment_date` is within 14 days
- Ensure workflow is running on schedule

## **Maintenance**

### **Adding New Subscriptions**

1. Add new row to Google Sheets
2. Fill in all required columns
3. Set `last_payment_date` to the most recent payment
4. Leave `Next_payment` empty (will be auto-calculated)
5. Workflow will calculate and update on next run

### **Removing Subscriptions**

1. Delete row from Google Sheets, OR
2. Set `is_active` to FALSE (subscription will still be tracked but can be filtered)

### **Updating Subscription Details**

1. Edit the subscription row in Google Sheets
2. Changes will be reflected in next email summary
3. Date calculations will continue based on `last_payment_date`

## **Technical Details**

### **Date Calculation Logic**

```jsx
// Monthly: last_payment_date + 1 month
nextDate.setMonth(nextDate.getMonth() + 1)

// Annual: last_payment_date + 1 year
nextDate.setFullYear(nextDate.getFullYear() + 1)

```

### **Auto-Update Logic**

```jsx
// When today >= Next_payment date:
if (today >= nextPayment) {
  last_payment_date = Next_payment  // Update last payment
  Next_payment = calculated_next_payment  // Update next payment
}

```
## **License**

This workflow is provided as-is for personal and commercial use.

---

**Version:** 1.0

**Last Updated:** 2025

**Platform:** n8n Workflow Automation


