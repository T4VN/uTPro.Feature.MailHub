> **Rendered documentation:** [docs.utpro.dev](https://docs.utpro.dev/uTPro.Feature.MailHub/) is the canonical, always-current home of this page.

# Extensibility Guide

uTPro Mail Hub provides extension points for developers looking to inject custom logging rules, integrate external mail webhooks, or query delivery logs programmatically.

---

## 1. Programmatic Access via `IMailHubService`

You can inject `IMailHubService` into your own Umbraco controllers or background jobs to inspect delivery metrics or query history:

```csharp
using uTPro.Feature.MailHub.Models;
using uTPro.Feature.MailHub.Services;

public class CustomDashboardService(IMailHubService mailHubService)
{
    public async Task<int> GetFailedCountTodayAsync()
    {
        var stats = await mailHubService.GetStatsAsync(days: 1);
        return (int)stats.TotalError;
    }

    public async Task<PagedResult<MailLogItemDto>> GetCustomerEmailsAsync(string customerEmail)
    {
        return await mailHubService.GetItemsAsync(new MailLogQueryParameters
        {
            Recipient = customerEmail,
            Page = 1,
            PageSize = 20
        });
    }
}
```

---

## 2. Low-Level Database Access via `IMailHubRepository`

For advanced cross-database queries or custom reporting:

```csharp
using uTPro.Feature.MailHub.Persistence;

public class EmailAuditReport(IMailHubRepository repository)
{
    public async Task ProcessReportAsync(Guid emailId)
    {
        var log = await repository.GetByIdAsync(emailId);
        if (log != null && log.Status == "Error")
        {
            // Send alert to Discord / Slack / Teams
        }
    }
}
```

---

## 3. Customizing the Decorator Pipeline

If you want to execute custom actions before or after each email delivery (e.g., dispatching webhooks to Slack or an external SIEM tool), you can register a custom notification handler or decorate `IEmailSender` further in your own `IComposer`:

```csharp
[ComposeAfter(typeof(MailHubComposer))]
public class CustomEmailComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        // Custom pipeline additions
    }
}
```
