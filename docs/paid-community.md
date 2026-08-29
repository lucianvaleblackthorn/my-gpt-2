# GPT-Image2 Paid Community Launch Guide

The current implementation separates the paid community from credit packs and membership payments: after a one-time Alipay payment of ¥9.90, access is tied to the Supabase user account; the protected group QR code can only be read when the server confirms the order as `PAID`.

## Local Capabilities

- Page: `/community`
- Payment redirect: `/community/result`
- Payment method: Alipay Web Payment (`alipay.trade.page.pay`)
- Fixed amount: 990 cents, CNY
- Order status: `PENDING`, `PAID`, `CLOSED`, `REFUNDED`, `REVOKED`
- Refund status: `NONE`, `PROCESSING`, `SUCCEEDED`, `FAILED`
- Access rules: Only `PAID` is valid; during refund processing, status remains `PAID`; after Alipay confirms refund success, status changes to `REFUNDED`
- QR code: Database `bytea` asset, limited to PNG/JPEG/WebP, max 2 MB; admin replaces atomically via transactional RPC

## API Endpoints

Public and current users:

- `GET /api/community/config`
- `GET /api/community/status`
- `GET /api/community/qr`

Alipay:

- `POST /api/community/alipay/checkout`
- `GET /api/community/alipay/query`
- `POST /api/community/alipay/close`
- `POST /api/community/alipay/notify`

Super admin:

- `GET /api/admin/community/orders`
- `GET|POST /api/admin/community/qr`
- `POST /api/admin/community/refund`
- `GET /api/admin/community/refund-query`
- `POST /api/admin/community/revoke`

## Environment Variables

Production deployment must start with:

```dotenv
COMMUNITY_PAYMENT_ENABLED=false
COMMUNITY_ALIPAY_NOTIFY_URL=https://gpt-image2.canghe.ai/api/community/alipay/notify
COMMUNITY_SUPPORT_TEXT=Search Canghe on WeChat
```

Alipay production variables reuse existing `ALIPAY_APP_ID`, `ALIPAY_PRIVATE_KEY`, `ALIPAY_PUBLIC_KEY`, `ALIPAY_SELLER_ID`, and production gateway configuration. Production private keys are only configured in Vercel Sensitive Environment Variables — never in the repository, documents, or conversations.

## First Deployment Sequence

1. Keep `COMMUNITY_PAYMENT_ENABLED=false`.
2. Apply `supabase/migrations/20260722090000_paid_community.sql` to the target Supabase project.
3. Deploy the same version of code; confirm `/community`, `/community/result`, and read-only status API endpoints work.
4. Have a `super_admin` upload a new community QR code that has never been publicly shared.
5. Complete website payment contract, configuration, and publishing on the Alipay open platform; confirm the public HTTPS notification address has no redirects.
6. Configure the App ID, Alipay public key, Alipay private key, and Alipay public key belonging to the same production Alipay app; Node.js uses the PKCS#1 raw private key.
7. Enable `COMMUNITY_PAYMENT_ENABLED=true`.
8. Complete a real ¥9.90 payment and a full refund acceptance test with the same production version.

## Production Acceptance

Real payments must confirm each item:

- The frontend cannot submit or override the amount.
- Independent `community_orders` are created — credit pack fulfillment is not triggered.
- Both Alipay active query and async notification can idempotently write `PAID`.
- Sync redirect only triggers server-side query — URL parameters are not trusted.
- Paid accounts can read the QR code; unpaid accounts receive a denial response.
- Admin refunds use a stable refund request number; eligibility remains valid during processing.
- After refund query returns `REFUND_SUCCESS`, status is written as `REFUNDED`, and QR code access is revoked.
- If any critical item fails, immediately set `COMMUNITY_PAYMENT_ENABLED` back to `false`.

## QR Code Rotation and Daily Operations

- When the community QR code expires, upload a new image in the admin panel; the RPC deactivates the old image and activates the new one in the same transaction.
- Do not upload protected QR codes to GitHub, README, public object storage, or frontend static directories.
- Refunds are manually reviewed via "Search Canghe on WeChat"; admins verify account, order number, and refund status before taking action.
- Manual revocation does not automatically refund; use the refund process when a refund is needed — do not use revocation as a substitute for refund.
