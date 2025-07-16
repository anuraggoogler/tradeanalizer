# Trade2Optionss Webhook Integration Guide

## Overview

The Trade2Optionss platform includes a powerful webhook system that allows you to receive automated trading signals from external sources like TradingView. When a webhook is triggered, it automatically creates trades in your account without manual intervention.

## How Webhooks Work

### What is a Webhook?
A webhook is like a doorbell for your trading platform. When someone (like TradingView) rings the doorbell by sending a message, your platform automatically opens the door and processes the trading signal.

### The Flow
1. **Signal Generation**: TradingView generates a trading signal based on your strategy
2. **Webhook Trigger**: TradingView sends a message to your platform's webhook URL
3. **Automatic Processing**: Your platform receives the message and creates a trade
4. **Execution**: The trade appears in your trading dashboard instantly

## Webhook Endpoint

### URL Structure
```
https://your-replit-domain.replit.app/api/webhook
```

### Method
- **POST** request only
- Content-Type: `application/json`

## Message Format

### Required Fields
Your webhook message must include these fields:

```json
{
  "symbol": "EURUSD",
  "side": "BUY",
  "lotSize": 1.0,
  "entryPrice": 1.0950,
  "takeProfit": 1.1000,
  "stopLoss": 1.0900
}
```

### Field Descriptions

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `symbol` | String | Trading pair/instrument | "EURUSD", "BTCUSD", "XAUUSD" |
| `side` | String | Trade direction | "BUY" or "SELL" |
| `lotSize` | Number | Position size | 1.0, 0.5, 2.0 |
| `entryPrice` | Number | Entry price level | 1.0950 |
| `takeProfit` | Number | Profit target price | 1.1000 |
| `stopLoss` | Number | Stop loss price | 1.0900 |

### Supported Trading Symbols

The platform supports 40+ trading instruments across multiple asset classes:

#### Forex Pairs
- Major pairs: EURUSD, GBPUSD, USDJPY, USDCHF, AUDUSD, USDCAD, NZDUSD
- Minor pairs: EURJPY, EURGBP, GBPJPY, AUDCAD, CADJPY

#### Cryptocurrencies  
- BTCUSD, ETHUSD, ADAUSD, SOLUSD, DOTUSD, LINKUSD, LTCUSD

#### Commodities
- XAUUSD (Gold), XAGUSD (Silver), USOIL (Crude Oil), UKOIL (Brent Oil)

#### Indian Stocks
- RELIANCE, TCS, INFY, HDFCBANK, ITC, SBIN, HINDUNILVR, BHARTIARTL

## TradingView Integration

### Step 1: Create Your Alert
1. Open TradingView and go to your chart
2. Click the "Alert" button (bell icon)
3. Set your alert conditions (price levels, indicators, etc.)

### Step 2: Configure Webhook Settings
1. In the alert dialog, scroll to "Notifications"
2. Check the "Webhook URL" checkbox
3. Enter your webhook URL: `https://your-replit-domain.replit.app/api/webhook`

### Step 3: Create Alert Message
In the "Message" field, enter your JSON payload:

```json
{
  "symbol": "{{ticker}}",
  "side": "BUY",
  "lotSize": 1.0,
  "entryPrice": {{close}},
  "takeProfit": {{close}} * 1.02,
  "stopLoss": {{close}} * 0.98
}
```

### TradingView Variables
You can use these TradingView variables in your webhook message:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{ticker}}` | Current symbol | EURUSD |
| `{{close}}` | Current close price | 1.0950 |
| `{{open}}` | Current open price | 1.0945 |
| `{{high}}` | Current high price | 1.0960 |
| `{{low}}` | Current low price | 1.0940 |
| `{{volume}}` | Current volume | 1000 |

### Example Alert Messages

#### Simple Buy Signal
```json
{
  "symbol": "{{ticker}}",
  "side": "BUY",
  "lotSize": 1.0,
  "entryPrice": {{close}},
  "takeProfit": {{close}} * 1.015,
  "stopLoss": {{close}} * 0.99
}
```

#### Dynamic Position Sizing
```json
{
  "symbol": "{{ticker}}",
  "side": "SELL",
  "lotSize": 0.5,
  "entryPrice": {{close}},
  "takeProfit": {{close}} * 0.98,
  "stopLoss": {{close}} * 1.01
}
```

## Security Features

### Authentication
- All webhook requests are automatically linked to authenticated users
- Trades are isolated per user account
- No unauthorized access to other users' data

### Validation
- All incoming data is validated before processing
- Invalid requests are rejected with error messages
- Supported symbols are checked against the platform's trading list

### Error Handling
- Invalid JSON format returns 400 error
- Missing required fields return 400 error  
- Unsupported symbols return 400 error
- Database errors return 500 error

## Response Format

### Successful Request
```json
{
  "message": "Webhook received successfully",
  "trade": {
    "id": 123,
    "symbol": "EURUSD",
    "side": "BUY",
    "lotSize": 1.0,
    "entryPrice": 1.0950,
    "takeProfit": 1.1000,
    "stopLoss": 1.0900,
    "status": "PENDING",
    "source": "WEBHOOK",
    "createdAt": "2025-07-01T12:00:00.000Z"
  }
}
```

### Error Response
```json
{
  "error": "Validation failed",
  "details": "Missing required field: symbol"
}
```

## Testing Your Webhook

### Method 1: Using the Demo Component
1. Go to your trading dashboard
2. Find the "Webhook Demo" section
3. Click "Send Test Webhook"
4. Check if a new trade appears in your trade history

### Method 2: Using curl Command
```bash
curl -X POST https://your-replit-domain.replit.app/api/webhook \
  -H "Content-Type: application/json" \
  -d '{
    "symbol": "EURUSD",
    "side": "BUY", 
    "lotSize": 1.0,
    "entryPrice": 1.0950,
    "takeProfit": 1.1000,
    "stopLoss": 1.0900
  }'
```

### Method 3: Using Postman
1. Create a new POST request
2. URL: `https://your-replit-domain.replit.app/api/webhook`
3. Headers: `Content-Type: application/json`
4. Body: Raw JSON with your trade data

## Common Issues and Solutions

### Issue: "Unauthorized" Error
**Solution**: Make sure you're logged into the platform in the same browser

### Issue: "Invalid symbol" Error  
**Solution**: Check that your symbol is in the supported list (see above)

### Issue: TradingView Alert Not Triggering
**Solutions**: 
- Verify your webhook URL is correct
- Check your alert conditions are met
- Ensure your JSON format is valid

### Issue: Trades Not Appearing
**Solutions**:
- Check your browser's network tab for errors
- Verify you're logged in as the correct user
- Refresh the trading dashboard

## Best Practices

### 1. Test Before Going Live
- Always test your webhook with small position sizes first
- Use the demo component to verify connectivity
- Double-check your JSON format

### 2. Risk Management
- Set appropriate stop losses for all trades
- Use reasonable position sizes based on your account
- Don't risk more than you can afford to lose

### 3. Monitor Your Trades
- Regularly check your trading dashboard
- Set up notifications for trade execution
- Keep track of your trading performance

### 4. Backup Strategy
- Have manual trading capabilities as backup
- Don't rely solely on automated signals
- Monitor market conditions independently

## Advanced Configuration

### Multiple Strategies
You can run multiple TradingView strategies simultaneously:
- Each strategy can have its own alert
- All trades will appear in the same dashboard
- Use different symbols or timeframes to diversify

### Dynamic Stop Losses
Use TradingView's calculation features for dynamic stops:
```json
{
  "symbol": "{{ticker}}",
  "side": "BUY",
  "lotSize": 1.0,
  "entryPrice": {{close}},
  "takeProfit": {{close}} + ({{close}} * 0.02),
  "stopLoss": {{close}} - ({{close}} * 0.01)
}
```

### Time-Based Filters
Add time conditions to your TradingView alerts to trade only during specific hours or days.

## Support and Troubleshooting

If you encounter any issues:
1. Check this documentation first
2. Test with the webhook demo component
3. Verify your JSON format using a JSON validator
4. Check the browser console for error messages
5. Review the platform's trade history for processed signals

Remember: Automated trading carries risks. Always monitor your trades and use appropriate risk management strategies.