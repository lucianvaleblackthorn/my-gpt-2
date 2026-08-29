# Alipay Web Payment

The project retains Stripe and adds Alipay web payment for one-time credit packs. Membership subscriptions are still handled by Stripe to avoid confusing Alipay single payments with auto-renewal.

## API Endpoints

| Path | Purpose | Permission |
| --- | --- | --- |
| `POST /api/billing/alipay/checkout` | Create order and return Alipay POST form | Logged-in user |
| `GET /api/billing/alipay/query` | Query transaction and idempotently grant credits | Order owner |
| `POST /api/billing/alipay/notify` | Verify signature, validate order, and process async notification | Alipay server |
| `POST /api/billing/alipay/refund` | Initiate full refund and deduct corresponding credits | Super admin |
| `GET /api/billing/alipay/refund-query` | Query refund result | Super admin |
| `POST /api/billing/alipay/close` | Close unpaid order | Super admin |

Payment results are only accepted via signature-verified async notifications or active `alipay.trade.query` queries. The browser sync redirect only opens the result page and does not directly determine payment success.

## Database and CNY Pricing

First apply migration `supabase/migrations/20260721090000_alipay_webpay.sql`. The migration adds payment channels, Alipay transaction IDs, refund status, and atomic credit grant/refund functions.

Each credit pack that needs Alipay purchase enabled must have a business-confirmed CNY amount in `credit_packs.alipay_amount_cents`. Credit packs without a standalone CNY price will not show the Alipay button; the code does not treat existing USD prices as CNY prices at 1:1.

## Local Sandbox

Local code reads directly from `.alipay-sandbox.json` in the project root directory, created and verified by the Alipay AI payment skill:

- Node.js uses `appIds[0].appPrivatePkcsKey` (PKCS#1).
- Gateway is fixed to `https://openapi-sandbox.dl.alipaydev.com/gateway.do`.
- The config file must remain Git-ignored and readable/writable only by the current user.
- When there is no public HTTPS address locally, `notify_url` is not sent; payment results are confirmed by transaction query. Notification handling code is still retained.

## Production Configuration

Production environment is configured via server-side environment variables:

- `ALIPAY_APP_ID`
- `ALIPAY_PRIVATE_KEY`
- `ALIPAY_PUBLIC_KEY`
- `ALIPAY_SELLER_ID`
- `ALIPAY_GATEWAY`
- `ALIPAY_NOTIFY_URL`
- `ALIPAY_NOTIFY_ENABLED`

`ALIPAY_APP_ID`, Alipay public key, and Alipay private key must belong to the same production Alipay key set. Node.js uses the PKCS#1 raw private key string without PEM headers/footers, and never stores keys in logs, frontend, or the repository.

Production deployment must use a public HTTPS `notify_url`, complete signature verification, idempotent processing, `app_id`, `seller_id`, order number, and amount validation, and retain active query as a compensation path.
