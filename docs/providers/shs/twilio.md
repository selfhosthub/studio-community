# Twilio Provider

Send SMS, WhatsApp messages, and voice calls via Twilio for automated communication workflows.

## Authentication

Twilio uses **Account SID + Auth Token** authentication (HTTP Basic Auth). Get your credentials from the Twilio Console.

### Setup

1. Sign up at [twilio.com](https://www.twilio.com/)
2. From the **Console Dashboard**, copy your **Account SID** and **Auth Token**
3. Get a Twilio phone number (or use the trial number)
4. In Studio, go to **Providers > Twilio > Credentials**
5. Click **Add Credential** and enter your Account SID, Auth Token, and From Number

### Trial Account Limitations

Twilio trial accounts have restrictions:
- Can only send messages to **verified phone numbers**
- SMS messages include a "Sent from your Twilio trial account" prefix
- Limited to one Twilio phone number
- WhatsApp only works with the Twilio Sandbox

To remove limitations, upgrade your account and add funds.

### WhatsApp Setup

To send WhatsApp messages:

1. Go to **Console > Messaging > Try it out > Send a WhatsApp message**
2. Follow the sandbox setup (send the join code from your phone)
3. For production: apply for a **WhatsApp Business Profile** through Twilio

WhatsApp numbers use the `whatsapp:` prefix (e.g., `whatsapp:+15551234567`).

## Available Services

### Messaging

| Service | Description |
|---------|-------------|
| **Send SMS** | Send text messages to any phone number (up to 1600 characters) |
| **Send WhatsApp** | Send WhatsApp messages with optional media attachments |
| **List Messages** | Retrieve sent and received messages with filters |
| **Get Message** | Get details of a specific message by SID |

### Voice

| Service | Description |
|---------|-------------|
| **Make Call** | Initiate outbound voice calls with TwiML instructions |

## Common Use Cases

### 1. AI SMS Lead Qualifier

Instant AI responses to inbound SMS:

**Workflow:**
1. **Trigger**: Webhook (Twilio inbound SMS)
2. **Lookup Contact** in Airtable
3. **Classify & Respond** with Claude (intent, sentiment, escalation)
4. **Send SMS** reply automatically
5. **Update CRM** in Airtable
6. **Escalate** via Telegram if high-value lead

### 2. Appointment Reminders

Send automated reminders before scheduled appointments:

**Workflow:**
1. **Trigger**: Schedule (daily at 8 AM)
2. **Query** Airtable for tomorrow's appointments
3. **Send SMS** to each client with appointment details

**Example Message:**
```
Hi Sarah! Reminder: Your appointment with Dr. Smith is tomorrow at 2:30 PM.
Reply CONFIRM to confirm or RESCHEDULE to change.
```

### 3. Order Status Notifications

Keep customers updated on their orders:

**Workflow:**
1. **Trigger**: Webhook (order status change)
2. **Send SMS** or **Send WhatsApp** with status update
3. **Log** notification to Airtable

### 4. Two-Factor Authentication

Send verification codes:

**Workflow:**
1. **Trigger**: API call from your app
2. **Generate** 6-digit code
3. **Send SMS** with verification code
4. **Store** code in database with expiry

### 5. Voice Call Alerts

Urgent escalation via phone call:

**Workflow:**
1. **Trigger**: Critical alert detected
2. **Make Call** with TwiML message
3. **Log** call status

**Example TwiML:**
```xml
<Response>
  <Say>Alert: Server CPU usage is above 95 percent. Please check immediately.</Say>
  <Pause length="1"/>
  <Say>Press 1 to acknowledge.</Say>
  <Gather numDigits="1" action="/handle-response"/>
</Response>
```

## Service Details

### Send SMS

Send text messages to any phone number:

```json
{
  "To": "+15551234567",
  "From": "+15559876543",
  "Body": "Your order #1234 has shipped! Track at: https://example.com/track/1234"
}
```

**Parameters:**
- `To` (required): Destination in E.164 format (+[country code][number])
- `From` (required): Your Twilio phone number
- `Body` (required): Message text, up to 1600 characters
- `StatusCallback`: URL for delivery status webhooks
- `MessagingServiceSid`: Use a Messaging Service for sender pool routing

**Message Segmentation:**
- SMS segments are 160 characters (GSM-7) or 70 characters (Unicode)
- Longer messages are split into segments and reassembled by the recipient's phone
- Each segment is billed separately

### Send WhatsApp

Send WhatsApp messages with optional media:

```json
{
  "To": "whatsapp:+15551234567",
  "From": "whatsapp:+15559876543",
  "Body": "Your invoice is attached.",
  "MediaUrl": "https://example.com/invoice.pdf"
}
```

**Key Differences from SMS:**
- Numbers must include `whatsapp:` prefix
- Supports media attachments (images, PDFs, etc.)
- Requires WhatsApp Business Profile for production
- Subject to WhatsApp message template rules for outbound messages

### Make Call

Initiate outbound voice calls:

```json
{
  "To": "+15551234567",
  "From": "+15559876543",
  "Twiml": "<Response><Say voice='Polly.Amy'>Hello! This is an automated call from Studio.</Say></Response>"
}
```

**Call Instructions (choose one):**
- `Twiml`: Inline TwiML XML for simple calls
- `Url`: URL returning TwiML for dynamic call logic

**TwiML Quick Reference:**

| Verb | Description |
|------|-------------|
| `<Say>` | Text-to-speech |
| `<Play>` | Play audio file |
| `<Gather>` | Collect DTMF input |
| `<Record>` | Record caller |
| `<Dial>` | Connect to another number |
| `<Pause>` | Wait N seconds |

**Voice Options for `<Say>`:**
- Amazon Polly voices: `Polly.Amy`, `Polly.Joanna`, `Polly.Matthew`, etc.
- Languages: English, Spanish, French, German, Italian, Japanese, and more

**Advanced Parameters:**
- `MachineDetection`: Detect voicemail (`Enable` or `DetectMessageEnd`)
- `Record`: Record the entire call
- `Timeout`: Seconds to wait for answer (default 60)
- `StatusCallbackEvent`: Get notified on call progress events

### List Messages

Retrieve message history:

```json
{
  "PageSize": 20,
  "To": "+15551234567",
  "DateSent": ">2025-01-01"
}
```

**Filters:**
- `To`: Filter by recipient
- `From`: Filter by sender
- `DateSent`: Filter by date (supports `>`, `<` prefixes for ranges)
- `PageSize`: Results per page (1-1000, default 50)

### Get Message

Look up a specific message:

```json
{
  "message_sid": "SM1234567890abcdef1234567890abcdef"
}
```

**Returns:** Full message details including delivery status, price, error codes, and segment count.

## Webhook Integration

### Receiving Inbound SMS

Configure your Twilio number's webhook URL to point to a Studio workflow webhook trigger:

1. Go to **Twilio Console > Phone Numbers > Active Numbers**
2. Click your number
3. Under **Messaging**, set the webhook URL to your Studio workflow's webhook URL
4. Twilio sends: `From`, `To`, `Body`, `MessageSid`, `NumMedia`

### Call Status Callbacks

Track call progress with status callbacks:

```json
{
  "StatusCallback": "https://your-studio.com/api/webhooks/...",
  "StatusCallbackEvent": ["initiated", "ringing", "answered", "completed"]
}
```

## Phone Number Formats

Always use E.164 format:

| Country | Format | Example |
|---------|--------|---------|
| US/Canada | +1XXXXXXXXXX | +15551234567 |
| UK | +44XXXXXXXXXX | +447911123456 |
| Australia | +61XXXXXXXXX | +61412345678 |
| Germany | +49XXXXXXXXXXX | +4915112345678 |

## Rate Limits

| Resource | Limit |
|----------|-------|
| SMS (long code) | 1 message/second per number |
| SMS (toll-free) | 3 messages/second |
| SMS (short code) | 100 messages/second |
| Voice calls | 1 call/second per number |
| API requests | 100 requests/second per account |

**Best practices:**
- Use Messaging Services for high-volume SMS (automatic sender pool rotation)
- Add delays between bulk sends
- Monitor delivery rates in Twilio Console

## Pricing

Twilio charges per message/call segment:

| Service | US Price (approx.) |
|---------|-------------------|
| SMS outbound | $0.0079/segment |
| SMS inbound | $0.0075/segment |
| WhatsApp | $0.005-0.05/message (varies by template) |
| Voice outbound | $0.014/minute |
| Phone number | $1.15/month |

Prices vary by country. Check [twilio.com/pricing](https://www.twilio.com/pricing) for current rates.

## Error Handling

### Common Errors

**Error 21211: "Invalid 'To' Phone Number"**
- **Cause**: Phone number not in E.164 format
- **Solution**: Include country code with `+` prefix

**Error 21608: "Unverified number (trial)"**
- **Cause**: Trial account can only send to verified numbers
- **Solution**: Verify the number in Console > Verified Caller IDs, or upgrade account

**Error 21610: "Message blocked (opt-out)"**
- **Cause**: Recipient replied STOP
- **Solution**: Remove number from send list, respect opt-out

**Error 21614: "Not a valid mobile number"**
- **Cause**: Trying to SMS a landline
- **Solution**: Verify number type, use voice call instead

**Error 30008: "Unknown error"**
- **Cause**: Carrier rejected the message
- **Solution**: Check message content for spam triggers, try again later

## Best Practices

1. **Use E.164 Format**: Always include country code with `+` prefix
2. **Handle Opt-Outs**: Respect STOP/UNSUBSCRIBE replies - Twilio handles this automatically
3. **Monitor Delivery**: Use StatusCallback URLs to track message delivery
4. **Use Messaging Services**: For production SMS at scale - handles sender rotation and compliance
5. **Test with Trial**: Test workflows thoroughly before going live
6. **Secure Credentials**: Never expose Account SID or Auth Token in client-side code
7. **Check Number Capabilities**: Verify your Twilio number supports SMS/WhatsApp/Voice before configuring
8. **Use TwiML Bins**: For simple call scripts, use Twilio's hosted TwiML Bins instead of self-hosting

## Troubleshooting

### SMS not delivered

**Check:**
1. Twilio Console > Messaging Logs for error codes
2. Recipient hasn't opted out (replied STOP)
3. Phone number is valid and mobile (not landline)
4. Account has sufficient balance
5. For trial: recipient number is verified

### WhatsApp messages failing

**Check:**
1. Using `whatsapp:` prefix on both To and From
2. Sandbox is active (for testing) or Business Profile approved (for production)
3. Recipient has opted in to your WhatsApp number
4. Message content complies with WhatsApp policies

### Calls not connecting

**Check:**
1. TwiML is valid XML (use Twilio's TwiML validator)
2. If using URL: endpoint returns valid TwiML with correct Content-Type
3. From number has voice capability
4. Recipient number is valid

## Additional Resources

- [Twilio Console](https://console.twilio.com/)
- [SMS API Reference](https://www.twilio.com/docs/sms/api/message-resource)
- [WhatsApp API Guide](https://www.twilio.com/docs/whatsapp/api)
- [Voice API Reference](https://www.twilio.com/docs/voice/api/call-resource)
- [TwiML Reference](https://www.twilio.com/docs/voice/twiml)
- [Error Code Dictionary](https://www.twilio.com/docs/api/errors)

## Terms

Your use of Twilio is governed by Twilio's own terms, not by Studio's: [https://www.twilio.com/en-us/legal/tos](https://www.twilio.com/en-us/legal/tos). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
