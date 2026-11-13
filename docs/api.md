> **Rendered documentation:** [docs.utpro.dev](https://docs.utpro.dev/uTPro.Feature.MailHub/) is the canonical, always-current home of this page.

# Management API Reference

uTPro Mail Hub exposes a set of authenticated Management API endpoints designed for backoffice administration and custom reporting tools.

All endpoints are rooted under:
```
/umbraco/management/api/v1/utpro/mailhub
```

Authorization: Requires an authenticated Umbraco backoffice user with access to the **Settings** section (`AuthorizationPolicies.SectionAccessSettings`).

---

## Endpoints

### 1. Get Dashboard Statistics

```http
GET /umbraco/management/api/v1/utpro/mailhub/stats?days={days}
```

#### Query Parameters
- `days` (integer, optional, default: `30`): Lookback window in days (7, 30, or 90).

#### Response (`200 OK`)
```json
{
  "totalSent": 450,
  "totalError": 12,
  "totalQueued": 0,
  "totalProcessing": 0,
  "totalAll": 462,
  "successRate": 97.4,
  "byDay": [
    { "date": "2026-11-01", "sent": 25, "error": 0 },
    { "date": "2026-11-02", "sent": 30, "error": 1 }
  ],
  "byEmailType": [
    { "name": "OrderNotification", "count": 210 },
    { "name": "PasswordReset", "count": 85 }
  ],
  "bySource": [
    { "name": "Decorator", "count": 462 }
  ],
  "byDomain": [
    { "domain": "gmail.com", "count": 180, "percentage": 39.0 },
    { "domain": "company.com", "count": 95, "percentage": 20.6 }
  ],
  "recent": [ ... ]
}
```

---

### 2. Search & Paginate Email History

```http
GET /umbraco/management/api/v1/utpro/mailhub/items?status={status}&searchTerm={term}&page={page}&pageSize={size}
```

#### Query Parameters
- `status` (string, optional): `"Sent"` | `"Error"` | `"Processing"` | `"Queued"`
- `searchTerm` (string, optional): Searches within `Subject`, `ToAddresses`, `FromAddress`, and `LastError`.
- `recipient` (string, optional): Filters recipients by email or domain.
- `emailType` (string, optional): Exact match on the `emailType` parameter passed to `SendAsync`.
- `fromDate` (ISO-8601, optional): Cutoff start time.
- `toDate` (ISO-8601, optional): Cutoff end time.
- `page` (integer, default: `1`): Page number (1-based).
- `pageSize` (integer, default: `20`, max: `100`): Items per page.

#### Response (`200 OK`)
```json
{
  "items": [
    {
      "id": "e6740683-9b22-4a0f-8c3e-cf9d8b746812",
      "fromAddress": "noreply@example.com",
      "toAddresses": "customer@acme.com",
      "toDomain": "acme.com",
      "subject": "Your order #1004 is confirmed",
      "emailType": "OrderNotification",
      "source": "Decorator",
      "status": "Sent",
      "retryCount": 0,
      "createdDate": "2026-11-10T14:30:00Z",
      "sentDate": "2026-11-10T14:30:02Z",
      "lastError": null,
      "hasError": false
    }
  ],
  "totalItems": 150,
  "page": 1,
  "pageSize": 20,
  "totalPages": 8
}
```

---

### 3. Get Single Email Details

```http
GET /umbraco/management/api/v1/utpro/mailhub/items/{id}
```

#### Response (`200 OK`)
```json
{
  "id": "e6740683-9b22-4a0f-8c3e-cf9d8b746812",
  "fromAddress": "noreply@example.com",
  "toAddresses": "customer@acme.com",
  "cc": "manager@example.com",
  "bcc": null,
  "toDomain": "acme.com",
  "subject": "Your order #1004 is confirmed",
  "body": "<html><body>Thank you for your order!</body></html>",
  "isBodyHtml": true,
  "emailType": "OrderNotification",
  "source": "Decorator",
  "status": "Sent",
  "createdDate": "2026-11-10T14:30:00Z",
  "sentDate": "2026-11-10T14:30:02Z",
  "lastError": null
}
```

---

### 4. Resend an Email

```http
POST /umbraco/management/api/v1/utpro/mailhub/items/{id}/resend
```

Re-dispatches the specified email via `IEmailSender` and records the new delivery attempt.

#### Response (`200 OK`)
```json
{
  "success": true,
  "message": "Email has been resent successfully."
}
```

---

### 5. Delete an Audit Record

```http
POST /umbraco/management/api/v1/utpro/mailhub/items/{id}/delete
```

Deletes the specified audit record.

#### Response (`200 OK`)
```json
{
  "success": true
}
```

---

### 6. Trigger Immediate Retention Cleanup

```http
POST /umbraco/management/api/v1/utpro/mailhub/retention/cleanup
```

Purges all completed logs older than the configured `RetentionDays`.

#### Response (`200 OK`)
```json
{
  "purgedCount": 42
}
```
