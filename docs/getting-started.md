> **Rendered documentation:** [docs.utpro.dev](https://docs.utpro.dev/uTPro.Feature.MailHub/) is the canonical, always-current home of this page.

# Getting Started with uTPro Mail Hub

This guide walks you through installing, configuring, and verifying **uTPro Mail Hub** in your Umbraco website.

---

## Step 1: Install the Package

Add the package via .NET CLI:

```bash
dotnet add package uTPro.Feature.MailHub
```

Or via the NuGet Package Manager Console:

```powershell
Install-Package uTPro.Feature.MailHub
```

---

## Step 2: First Boot & Automatic Migration

Run your Umbraco application:

```bash
dotnet run
```

On startup, Mail Hub automatically:
1. Detects your database provider (SQL Server, SQLite, or PostgreSQL).
2. Executes the migration plan (`uTPro.MailHub`) to create table `uTProMailHubLog` with optimized query indexes.
3. Automatically decorates Umbraco's `IEmailSender` with `MailHubEmailSender`.
4. Registers the backoffice management menu and Lit web components.

No code modifications in `Program.cs` or startup classes are required.

---

## Step 3: Verify Email Monitoring

Send an email using any built-in Umbraco feature (such as inviting a backoffice user or requesting a password reset), or send one via code:

```csharp
using Umbraco.Cms.Core.Mail;
using Umbraco.Cms.Core.Models.Email;

public class ContactService(IEmailSender emailSender)
{
    public async Task SendContactAsync(string email, string subject, string message)
    {
        var mail = new EmailMessage(
            from: "noreply@example.com",
            to: [email],
            cc: [],
            bcc: [],
            replyTo: [],
            subject: subject,
            body: $"<p>{message}</p>",
            isBodyHtml: true,
            attachments: []);

        await emailSender.SendAsync(mail, emailType: "ContactForm", enableNotification: true);
    }
}
```

---

## Step 4: Inspect in Backoffice

1. Log into your Umbraco backoffice.
2. Go to the **Settings** section.
3. In the sidebar under **Advanced Settings**, click **Mail Hub**.
4. You will see:
   - Updated delivery count cards.
   - The volume trend bar chart.
   - Your email in the **Recent Outgoing Emails** list.
   - Click **View** to inspect the email details, headers, body preview, or error stack trace.
