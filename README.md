# uTPro Mail Hub

[![Umbraco 17](https://img.shields.io/badge/Umbraco-17.4%2B-blue.svg)](https://umbraco.com)
[![Umbraco 18](https://img.shields.io/badge/Umbraco-18.x-blue.svg)](https://umbraco.com)
[![.NET 10](https://img.shields.io/badge/.NET-10.0-purple.svg)](https://dotnet.microsoft.com/)
[![License: EULA](https://img.shields.io/badge/License-EULA-green.svg)](LICENSE.txt)

> Lightweight, self-contained email delivery monitoring, audit history, and analytics dashboard for **Umbraco 17 and 18**.
> Transparently intercepts Umbraco's built-in `IEmailSender` via a fail-safe decorator so every transactional and system email (member invites, password resets, shop notifications, custom forms) is automatically tracked with zero host code changes.

---

## 1. Why Mail Hub?

Umbraco's built-in `IEmailSender` dispatches messages via SMTP but provides **no delivery history, no failure diagnostics, no recipient analytics, and no backoffice visibility**.

**uTPro Mail Hub** solves this by plugging seamlessly into the email pipeline:
- **Zero code changes**: Automatically decorates `IEmailSender`. Any code calling `_emailSender.SendAsync(...)` is monitored automatically.
- **Fail-safe logging**: Telemetry failures never interrupt or fail actual email dispatch.
- **Rich Backoffice Dashboard**: Located under **Settings → Mail Hub**, presenting delivery volume trends, error spikes, success rates, recipient domain distribution, and full audit logs.
- **Search & Filter**: Find emails by recipient, domain, subject, status, or date range.
- **Resend & Retention**: Re-dispatch failed or previous emails with one click, and automatically purge expired logs.
- **Cross-Database**: Auto-creates tables via NPoco migrations across SQL Server, SQLite, and PostgreSQL.

---

## 2. Installation

Install via NuGet:

```bash
dotnet add package uTPro.Feature.MailHub
```

The package self-wires during startup (`IComposer`). No changes to `Program.cs` or database connection strings are required.

---

## 3. Configuration

Mail Hub works out of the box with default settings. You can optionally customize behavior in `appsettings.json`:

```json
{
  "uTPro": {
    "MailHub": {
      "Mode": "Log",
      "StoreBody": true,
      "RetentionDays": 30,
      "MaxRetry": 3,
      "RetryDelaySeconds": 60
    }
  }
}
```

### Options Reference

| Property | Default | Description |
|---|---|---|
| `Mode` | `"Log"` | Operating mode: `"Log"` (synchronous tracking) or `"Queue"` (queued background processing). |
| `StoreBody` | `true` | Whether to persist HTML/plain email body. Set `false` if emails contain strict confidential credentials. |
| `RetentionDays` | `30` | Number of days to retain logs before cleanup. Completed logs (`Sent`, `Error`) older than this threshold are purged. `0` disables cleanup. |
| `MaxRetry` | `3` | Maximum retry attempts for resending operations. |
| `RetryDelaySeconds` | `60` | Delay between successive retries. |

---

## 4. Backoffice Dashboard

Navigate to **Settings → Advanced Settings → Mail Hub** in the Umbraco backoffice:

1. **Overview Dashboard**:
   - **KPI Summary**: Total Delivered, Failed Deliveries, Success Rate %, and Volume in Selected Window (7 / 30 / 90 days).
   - **Delivery Volume Trend**: Interactive daily bar chart showing delivered vs failed counts.
   - **Top Recipient Domains**: Visual breakdown of destination domains (`@gmail.com`, `@company.com`) with percentage share.
   - **Recent Activity**: Quick access to recent outgoing emails with instant inspection.

2. **Email History**:
   - Filter by keyword, subject, recipient, status, and email type.
   - Paged table with recipient, subject, type, status badge, and timestamp.
   - **Email Audit Drawer**: Inspect detailed metadata, header addresses (To, Cc, Bcc), formatted/raw HTML body preview, and complete error stack traces for failed deliveries.
   - **Actions**: One-click **Resend Email** or **Delete Record**.

---

## 5. Management API

Mail Hub exposes a versioned, authenticated Management API for the backoffice (gated behind the `SectionAccessSettings` authorization policy):

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/umbraco/management/api/v1/utpro/mailhub/stats?days=30` | Returns aggregated metrics, trends, and category distribution. |
| `GET` | `/umbraco/management/api/v1/utpro/mailhub/items` | Paged query for email logs with filtering parameters. |
| `GET` | `/umbraco/management/api/v1/utpro/mailhub/items/{id}` | Fetches full email audit details including body and error trace. |
| `POST` | `/umbraco/management/api/v1/utpro/mailhub/items/{id}/resend` | Re-sends the logged email via `IEmailSender`. |
| `POST` | `/umbraco/management/api/v1/utpro/mailhub/items/{id}/delete` | Deletes a single audit log record. |
| `POST` | `/umbraco/management/api/v1/utpro/mailhub/retention/cleanup` | Manually triggers purge of expired logs. |

---

## 6. Architecture & Extensibility

```
                                  ┌───────────────────────────┐
                                  │   Application / Features  │
                                  │   (SimpleCart, Auth, etc) │
                                  └─────────────┬─────────────┘
                                                │ Calls IEmailSender
                                                ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ uTPro.Feature.MailHub                                                                  │
│                                                                                        │
│   ┌────────────────────────────────────────┐     Calls      ┌────────────────────────┐ │
│   │ MailHubEmailSender (Decorator)         ├───────────────►│ Inner IEmailSender     │ │
│   │ - Intercepts To/Cc/Subject/Body/Type   │                │ (Umbraco SMTP Core)    │ │
│   │ - Records start state                  │                └────────────────────────┘ │
│   │ - Records success / error trace        │                                           │
│   └───────────────────┬────────────────────┘                                           │
│                       │ Writes (isolated scope)                                        │
│                       ▼                                                                │
│   ┌────────────────────────────────────────┐     Reads      ┌────────────────────────┐ │
│   │ IMailHubRepository (NPoco)             │◄───────────────┤ Management API / Lit UI│ │
│   │ Table: uTProMailHubLog                 │                │ Settings -> Mail Hub   │ │
│   └────────────────────────────────────────┘                └────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

- **Decorator Registration**: Registered via `MailHubComposer` with automatic discovery.
- **Fail-Safe Design**: Log persistence runs within an isolated `IServiceScope`. If database connectivity fails, the outgoing email is still sent and the exception is logged to `ILogger`.
- **Database Migrations**: Executed on application startup (`UmbracoApplicationStartedNotification`) using Umbraco's state-keyed `MigrationPlan`.

---

## 7. Building & Testing

Build the solution:

```bash
dotnet build uTPro.Feature.MailHub.sln
```

Run unit tests:

```bash
dotnet test tests/uTPro.Feature.MailHub.Tests/uTPro.Feature.MailHub.Tests.csproj
```

Package for both Umbraco 17 and 18:

```powershell
pwsh ./pack.ps1
```

---

## 8. License

Licensed under the [uTPro End User License Agreement](LICENSE.txt). Copyright &copy; 2025 T4VN. All rights reserved.
