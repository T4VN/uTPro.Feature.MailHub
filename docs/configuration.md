> **Rendered documentation:** [docs.utpro.dev](https://docs.utpro.dev/uTPro.Feature.MailHub/) is the canonical, always-current home of this page.

# Configuration Guide

uTPro Mail Hub provides options to control tracking behavior, sensitive data persistence, and retention lifetimes.

---

## AppSettings Schema

Configure options under the `uTPro:MailHub` key in `appsettings.json`:

```json
{
  "uTPro": {
    "MailHub": {
      "Mode": "Log",
      "StoreBody": true,
      "RetentionDays": 30,
      "MaxRetry": 3,
      "RetryDelaySeconds": 60,
      "ProcessIntervalSeconds": 30,
      "ProcessingTimeoutSeconds": 300
    }
  }
}
```

---

## Settings Reference

### `Mode`
- **Type**: `string`
- **Default**: `"Log"`
- **Values**: `"Log"` | `"Queue"`
- **Description**:
  - `"Log"`: Intercepts `IEmailSender`, dispatches immediately via SMTP, and records the delivery outcome (`Sent` or `Error`).
  - `"Queue"`: Future extension mode allowing asynchronous background queue processing.

### `StoreBody`
- **Type**: `bool`
- **Default**: `true`
- **Description**: Controls whether the email body (plain text or HTML) is saved in the database.
- **Security Note**: If your site transmits high-sensitivity credentials (e.g. temporary OTP tokens or raw reset links), set this to `false` to log only header metadata (sender, recipient, subject, time, status, errors).

### `RetentionDays`
- **Type**: `int`
- **Default**: `30`
- **Description**: Number of days completed logs (`Sent`, `Error`, `Cancelled`) are kept in the database. Logs older than this threshold are purged during cleanup. Setting to `0` disables cleanup.

### `MaxRetry`
- **Type**: `int`
- **Default**: `3`
- **Description**: Maximum number of attempts tracked when resending emails from the dashboard.
