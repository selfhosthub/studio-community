# Stripe Provider

Accept payments, manage customers, and automate billing workflows with the Stripe payment platform.

## Authentication

Stripe uses **API Key** authentication. You'll need your secret API key from the Stripe Dashboard.

### Setup Overview

1. Create or log into your Stripe account
2. Get your API keys from the Dashboard
3. Add credential in Studio with your secret key
4. Start accepting payments

### Step 1: Get API Keys

1. Go to [Stripe Dashboard](https://dashboard.stripe.com/)
2. Click **Developers** in the top menu
3. Click **API keys** in the left sidebar
4. You'll see two types of keys:
   - **Publishable key** (starts with `pk_`) - for client-side code
   - **Secret key** (starts with `sk_`) - for server-side (Studio uses this)

**Important:**
- Use **Test Mode** keys for development (toggleable in Dashboard)
- Use **Live Mode** keys for production
- Never expose secret keys publicly

### Step 2: Add Credential in Studio

1. Go to **Providers > Stripe > Credentials**
2. Click **Add Credential**
3. Paste your **Secret Key** (starts with `sk_test_` or `sk_live_`)
4. Test the connection

## Available Services

### Customer Management

| Service | Description |
|---------|-------------|
| **List Customers** | Retrieve and search customers by email or date |
| **Create Customer** | Create new customer records with email, name, and metadata |
| **Get Customer** | Retrieve customer details by ID |

### Payments

| Service | Description |
|---------|-------------|
| **List Payment Intents** | Track payment status and history |
| **Create Payment Intent** | Initiate a payment for custom integrations |
| **List Charges** | View completed payments and refunds |

### Checkout

| Service | Description |
|---------|-------------|
| **Create Checkout Session** | Create hosted payment page for easy one-time or subscription payments |

## Common Use Cases

### 1. Form Submission → Create Customer

Automatically create Stripe customers from form submissions:

**Workflow:**
1. **Trigger**: Typeform/Google Forms submission
2. **Create Customer** in Stripe with form data
3. **Send Email** confirmation with customer ID

**Example:**
```json
{
  "email": "customer@example.com",
  "name": "John Doe",
  "metadata": {
    "source": "website_signup",
    "plan": "premium"
  }
}
```

### 2. Create Checkout Session → Send Link

Generate payment links on-demand:

**Workflow:**
1. **Trigger**: Slack command or webhook
2. **Create Checkout Session** with product details
3. **Send Message** to Slack/Telegram with checkout URL

**Result:** Customer receives unique payment link

### 3. Payment Success → Update Database

Track successful payments in your database:

**Workflow:**
1. **Trigger**: Stripe webhook (checkout.session.completed)
2. **Get Customer** details from Stripe
3. **Update** Airtable/Notion with payment confirmation
4. **Send Email** receipt

### 4. Failed Payment → Alert Team

Get notified when payments fail:

**Workflow:**
1. **Trigger**: Stripe webhook (payment_intent.payment_failed)
2. **Get Customer** information
3. **Send Message** to Slack with customer details
4. **Create Task** in project management tool

### 5. Subscription Billing → Invoice

Automate subscription billing:

**Workflow:**
1. **Create Customer** (if new)
2. **Create Checkout Session** with `mode: "subscription"`
3. **Send Email** with subscription details
4. Customer completes checkout → Stripe handles recurring billing

### 6. Daily Revenue Report

Schedule daily revenue summaries:

**Workflow:**
1. **Trigger**: Scheduled daily at 9 AM
2. **List Charges** for last 24 hours
3. **Calculate** total revenue
4. **Send Message** to Telegram/Slack with summary

## Service Details

### Create Customer

Create a customer in Stripe:

```json
{
  "email": "customer@example.com",
  "name": "John Doe",
  "phone": "+1-555-0123",
  "metadata": {
    "user_id": "12345",
    "plan": "premium"
  },
  "address": {
    "line1": "123 Main St",
    "city": "San Francisco",
    "state": "CA",
    "postal_code": "94102",
    "country": "US"
  }
}
```

**Returns:**
- Customer ID (e.g., `cus_ABC123def`)
- All customer details
- Created timestamp

**Use metadata for:**
- Linking to your database records
- Storing plan information
- Custom categorization

### List Customers

Search and filter customers:

```json
{
  "email": "customer@example.com",
  "limit": 10
}
```

**Filter by creation date:**
```json
{
  "created": {
    "gte": 1640995200,
    "lte": 1672531199
  },
  "limit": 100
}
```

**Pagination:**
```json
{
  "starting_after": "cus_ABC123",
  "limit": 10
}
```

### Create Payment Intent

Create a payment for custom integrations:

```json
{
  "amount": 2000,
  "currency": "usd",
  "customer": "cus_ABC123",
  "description": "Premium Plan - Monthly",
  "metadata": {
    "order_id": "12345"
  },
  "automatic_payment_methods": {
    "enabled": true
  }
}
```

**Amount is in cents:**
- $10.00 = `1000`
- $99.99 = `9999`
- $1,234.56 = `123456`

**Supported currencies:**
- `usd` - US Dollar
- `eur` - Euro
- `gbp` - British Pound
- `cad` - Canadian Dollar
- `aud` - Australian Dollar
- `jpy` - Japanese Yen

**Payment Intent Status:**
- `requires_payment_method` - Waiting for payment method
- `requires_confirmation` - Ready to confirm
- `requires_action` - Needs customer authentication (3D Secure)
- `processing` - Payment being processed
- `succeeded` - Payment complete
- `canceled` - Payment canceled

### Create Checkout Session

Create a hosted payment page:

**Simple one-time payment:**
```json
{
  "success_url": "https://example.com/success",
  "cancel_url": "https://example.com/cancel",
  "mode": "payment",
  "line_items": [
    {
      "price_data": {
        "currency": "usd",
        "product_data": {
          "name": "Premium Plan",
          "description": "1 month access"
        },
        "unit_amount": 2000
      },
      "quantity": 1
    }
  ],
  "customer_email": "customer@example.com"
}
```

**Subscription:**
```json
{
  "success_url": "https://example.com/success?session_id={CHECKOUT_SESSION_ID}",
  "cancel_url": "https://example.com/cancel",
  "mode": "subscription",
  "line_items": [
    {
      "price_data": {
        "currency": "usd",
        "product_data": {
          "name": "Pro Plan"
        },
        "unit_amount": 2000,
        "recurring": {
          "interval": "month"
        }
      },
      "quantity": 1
    }
  ],
  "customer": "cus_ABC123"
}
```

**Multiple items:**
```json
{
  "success_url": "https://example.com/success",
  "cancel_url": "https://example.com/cancel",
  "mode": "payment",
  "line_items": [
    {
      "price_data": {
        "currency": "usd",
        "product_data": {
          "name": "T-Shirt",
          "images": ["https://example.com/shirt.jpg"]
        },
        "unit_amount": 2000
      },
      "quantity": 2
    },
    {
      "price_data": {
        "currency": "usd",
        "product_data": {
          "name": "Hat"
        },
        "unit_amount": 1500
      },
      "quantity": 1
    }
  ]
}
```

**Returns:**
- `url` - Redirect customer to this URL to complete checkout
- `id` - Session ID (for tracking)
- `status` - Session status (`open`, `complete`, `expired`)

### List Charges

View completed payments:

```json
{
  "customer": "cus_ABC123",
  "limit": 10
}
```

**Filter by date range:**
```json
{
  "created": {
    "gte": 1704067200,
    "lt": 1704153600
  },
  "limit": 100
}
```

**Returns:**
- Charge ID
- Amount and currency
- Status (`succeeded`, `pending`, `failed`)
- Receipt URL
- Customer ID

## Webhooks

Stripe sends webhooks for payment events. Common events to listen for:

| Event | When it fires |
|-------|--------------|
| `checkout.session.completed` | Checkout payment successful |
| `payment_intent.succeeded` | Payment completed |
| `payment_intent.payment_failed` | Payment failed |
| `customer.created` | New customer created |
| `customer.updated` | Customer details changed |
| `charge.succeeded` | Charge successful |
| `charge.refunded` | Charge refunded |

**Setup webhooks:**
1. Go to **Developers > Webhooks** in Stripe Dashboard
2. Click **Add endpoint**
3. Enter your Studio webhook URL
4. Select events to listen for
5. Copy **Signing secret** for verification

## Currency and Amounts

**Zero-decimal currencies** (e.g., JPY):
- JPY 1,000 = `1000` (not `100000`)
- No decimal places

**Two-decimal currencies** (most currencies):
- $10.00 = `1000`
- €50.50 = `5050`

**Three-decimal currencies** (rare, e.g., BHD, KWD):
- Check Stripe documentation

## Metadata

Use metadata to link Stripe data to your systems:

```json
{
  "metadata": {
    "user_id": "12345",
    "order_number": "ORD-2024-001",
    "source": "mobile_app",
    "plan_type": "annual"
  }
}
```

**Best practices:**
- Max 50 keys per object
- Keys max 40 characters
- Values max 500 characters
- Use for linking to your database IDs
- Store reference data, not sensitive info

## Test Mode

**Test with these card numbers:**

| Card Number | Scenario |
|-------------|----------|
| `4242 4242 4242 4242` | Successful payment |
| `4000 0000 0000 0002` | Charge declined |
| `4000 0000 0000 9995` | Declined with insufficient funds |
| `4000 0025 0000 3155` | Requires authentication (3D Secure) |

**Any:**
- Expiry: Any future date
- CVC: Any 3 digits
- ZIP: Any valid ZIP code

## Rate Limits

Stripe API limits:
- **100 requests per second** (read operations)
- **25 requests per second** (write operations)

**Best practices:**
- Implement retry logic with exponential backoff
- Use pagination for large lists
- Cache customer data when possible

## Error Handling

### Common Errors

**"No such customer"**
- **Cause**: Invalid customer ID
- **Solution**: Verify customer ID exists and is correct

**"Your card was declined"**
- **Cause**: Payment method declined by bank
- **Solution**: Ask customer to try different payment method

**"This value must be greater than or equal to 50"**
- **Cause**: Amount too small (minimum $0.50 USD)
- **Solution**: Increase amount to at least 50 cents

**"Invalid currency"**
- **Cause**: Unsupported currency code
- **Solution**: Use supported currency (usd, eur, gbp, etc.)

## Best Practices

1. **Use Metadata**: Link Stripe records to your database with metadata
2. **Test Mode First**: Always test workflows with test API keys before going live
3. **Handle Webhooks**: Listen for Stripe events instead of polling
4. **Idempotency**: Use idempotency keys for critical operations
5. **Error Handling**: Implement retry logic for network issues
6. **Secure Keys**: Never expose secret keys in client-side code or public repos
7. **Customer Records**: Create customer records before charging for easier refunds/support
8. **Checkout Sessions**: Use Checkout for simplest payment flows (Stripe handles UI)

## Security

1. **API Keys:**
   - Never commit secret keys to version control
   - Use test keys for development
   - Rotate keys if compromised

2. **Webhook Signatures:**
   - Always verify webhook signatures
   - Use signing secret from Dashboard
   - Prevents fake webhook attacks

3. **PCI Compliance:**
   - Never handle raw card data
   - Use Stripe.js for card collection
   - Let Stripe handle card storage

4. **Customer Data:**
   - Don't store full card numbers
   - Use customer IDs for tracking
   - Follow data retention policies

## Troubleshooting

### Checkout session not redirecting

**Solution:**
- Check `success_url` and `cancel_url` are valid HTTPS URLs
- Include `{CHECKOUT_SESSION_ID}` placeholder in success_url to retrieve session data

### Payment Intent stuck in "requires_action"

**Solution:**
- Customer needs to complete 3D Secure authentication
- Redirect the customer to the `next_action.redirect_to_url.url` on the Payment Intent; read it from Stripe, the step result stops at `client_secret`
- Or use Stripe.js to handle authentication

### Customer not receiving receipts

**Solution:**
- Ensure customer has email address on file
- Check **Settings > Emails** in Stripe Dashboard
- Enable automatic receipt emails

## Pricing

Stripe fees (US):
- **2.9% + 30¢** per successful card charge
- **No monthly fees** or setup costs
- **Instant payouts**: 1% fee (optional)
- **International cards**: Additional 1%

Check [Stripe Pricing](https://stripe.com/pricing) for your region.

## Additional Resources

- [Stripe API Documentation](https://stripe.com/docs/api)
- [Stripe Checkout Guide](https://stripe.com/docs/payments/checkout)
- [Payment Intents Guide](https://stripe.com/docs/payments/payment-intents)
- [Webhooks Guide](https://stripe.com/docs/webhooks)
- [Testing Guide](https://stripe.com/docs/testing)
- [Dashboard](https://dashboard.stripe.com/)

## Terms

Your use of Stripe is governed by Stripe's own terms, not by Studio's: [https://stripe.com/legal/ssa](https://stripe.com/legal/ssa). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
