# Alipay Web Payment

The project retains Stripe and adds Alipay web payment for one-time credit packs. Membership subscriptions are still handled by Stripe to avoid confusing Alipay single payments with auto-renewal.

## API Endpoints

| Path | Purpose | Permission |
|---|---|---|
| POST /api/billing/alipay/checkout | Create order and return Alipay POST form | Logged-in user |
| GET /api/billing/alipay/query | Query transaction and idempotently grant credits | Order owner |
| POST /api/billing/alipay/notify | Verify signature, validate order, and process async notification | Alipay server |
| POST /api/billing/alipay/refund | Initiate full refund and deduct corresponding credits | Super admin |
| GET /api/billing/alipay/refund-query | Query refund result | Super admin |
| POST /api/billing/alipay/close | Close unpaid order | Super admin |

## Configuration

Set the following environment variables for Alipay:
- ALIPAY_APP_ID - Alipay app ID
- ALIPAY_PRIVATE_KEY - Application private key
- ALIPAY_PUBLIC_KEY - Alipay public key
- ALIPAY_GATEWAY - Gateway URL (default: https://openapi.alipay.com/gateway.do)
- ALIPAY_NOTIFY_URL - Async notification callback URL
- ALIPAY_NOTIFY_ENABLED - Set to false to disable notifications (for local testing)

## Credit Packs with Alipay

Credit packs must have a configured CNY price (price_cny field) to show the Alipay payment button. Packs without a CNY price will only show Stripe.

## Refund Flow

1. Admin initiates refund via POST /api/billing/alipay/refund
2. Server pre-deducts credits from the user
3. Alipay processes the refund
4. GET /api/billing/alipay/refund-query checks the result
5. On success, the order status changes to REFUNDED
