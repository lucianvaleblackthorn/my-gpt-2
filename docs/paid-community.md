# GPT-Image2 Paid Community Launch Guide

The current implementation separates the paid community from credit packs and membership payments: after a one-time Alipay payment of 9.90 CNY, access is tied to the Supabase user account; the protected group QR code can only be read when the server confirms the order as PAID.

## Local Capabilities

- Page: /community
- Payment redirect: /community/result
- Payment method: Alipay web payment (alipay.trade.page.pay)
- Fixed amount: 990 cents, CNY
- Order status: PENDING, PAID, CLOSED, REFUNDED, REVOKED
- Refund status: NONE, PROCESSING, SUCCEEDED, FAILED
- Access rules: Only PAID is valid; during refund processing, status remains PAID; after Alipay confirms refund success, it changes to REFUNDED

## API Endpoints

| Path | Purpose | Permission |
|---|---|---|
| POST /api/community/alipay/checkout | Create order and return Alipay POST form | Logged-in user |
| GET /api/community/alipay/query | Query transaction and idempotently grant access | Order owner |
| POST /api/community/alipay/notify | Verify signature, validate order, and process async notification | Alipay server |
| POST /api/community/alipay/refund | Initiate full refund and revoke access | Super admin |
| GET /api/community/alipay/refund-query | Query refund result | Super admin |
| POST /api/community/alipay/close | Close unpaid order | Super admin |
| GET /api/community/status | Check current user community access | Logged-in user |
| GET /api/community/config | Check if community payments are enabled | Public |
| GET /api/community/qr | Get protected QR code (requires PAID status) | Paid user |
| POST /api/admin/community/orders | List all community orders | Super admin |
| POST /api/admin/community/revoke | Revoke a user community access | Super admin |
| POST /api/admin/community/refund | Initiate manual refund | Super admin |
| GET /api/admin/community/refund-query | Query refund status | Super admin |

## Configuration

Production deployments must start with COMMUNITY_PAYMENT_ENABLED=false.

Set the following environment variables:
- ALIPAY_APP_ID - Alipay app ID
- ALIPAY_PRIVATE_KEY - Alipay private key
- ALIPAY_PUBLIC_KEY - Alipay public key
- COMMUNITY_ALIPAY_NOTIFY_URL - Callback URL for community payments
- COMMUNITY_PAYMENT_ENABLED - Set to true to enable payments
