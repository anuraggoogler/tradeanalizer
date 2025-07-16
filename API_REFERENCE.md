# Trade2Optionss API Reference

## Authentication

All API endpoints require authentication via Replit Auth. Users must be logged in to access any trading functionality.

### Authentication Headers
- Session-based authentication using secure cookies
- Automatic token refresh for expired sessions
- OpenID Connect integration with Replit identity provider

## Webhook Endpoint

### POST /api/webhook

Receives automated trading signals from external sources like TradingView.

#### Request Format
```http
POST /api/webhook
Content-Type: application/json

{
  "symbol": "EURUSD",
  "side": "BUY",
  "lotSize": 1.0,
  "entryPrice": 1.0950,
  "takeProfit": 1.1000,
  "stopLoss": 1.0900
}
```

#### Request Parameters

| Parameter | Type | Required | Description | Validation |
|-----------|------|----------|-------------|------------|
| `symbol` | string | Yes | Trading instrument | Must be from supported symbols list |
| `side` | string | Yes | Trade direction | "BUY" or "SELL" |
| `lotSize` | number | Yes | Position size | Positive number |
| `entryPrice` | number | Yes | Entry price level | Positive number |
| `takeProfit` | number | No | Profit target | Positive number |
| `stopLoss` | number | No | Stop loss level | Positive number |

#### Supported Symbols

**Forex Pairs (28 pairs)**
```
EURUSD, GBPUSD, USDJPY, USDCHF, AUDUSD, USDCAD, NZDUSD,
EURJPY, EURGBP, GBPJPY, AUDJPY, EURAUD, EURCHF, AUDNZD,
CADCHF, CADJPY, CHFJPY, EURNZD, GBPAUD, GBPCAD, GBPCHF,
GBPNZD, NZDCAD, NZDCHF, NZDJPY, AUDCAD, EURCAD, USDNOK
```

**Cryptocurrencies (7 pairs)**
```
BTCUSD, ETHUSD, ADAUSD, SOLUSD, DOTUSD, LINKUSD, LTCUSD
```

**Commodities (4 instruments)**
```
XAUUSD (Gold), XAGUSD (Silver), USOIL (Crude Oil), UKOIL (Brent Oil)
```

**Indian Stocks (8 stocks)**
```
RELIANCE, TCS, INFY, HDFCBANK, ITC, SBIN, HINDUNILVR, BHARTIARTL
```

#### Success Response
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "message": "Webhook received successfully",
  "trade": {
    "id": 123,
    "userId": "34711896",
    "symbol": "EURUSD",
    "side": "BUY",
    "lotSize": 1.0,
    "entryPrice": 1.0950,
    "takeProfit": 1.1000,
    "stopLoss": 1.0900,
    "status": "PENDING",
    "source": "WEBHOOK",
    "createdAt": "2025-07-01T12:00:00.000Z",
    "updatedAt": "2025-07-01T12:00:00.000Z"
  }
}
```

#### Error Responses

**400 Bad Request - Validation Error**
```json
{
  "error": "Validation failed",
  "details": [
    {
      "code": "invalid_enum_value",
      "expected": ["BUY", "SELL"],
      "received": "INVALID",
      "path": ["side"],
      "message": "Invalid enum value. Expected 'BUY' | 'SELL', received 'INVALID'"
    }
  ]
}
```

**400 Bad Request - Invalid Symbol**
```json
{
  "error": "Validation failed",
  "details": [
    {
      "code": "invalid_enum_value",
      "expected": ["EURUSD", "GBPUSD", "..."],
      "received": "INVALID",
      "path": ["symbol"],
      "message": "Invalid symbol. Must be one of supported trading instruments."
    }
  ]
}
```

**401 Unauthorized**
```json
{
  "message": "Unauthorized"
}
```

**500 Internal Server Error**
```json
{
  "error": "Internal server error"
}
```

## Trading Endpoints

### GET /api/trades

Retrieves all trades for the authenticated user.

#### Request
```http
GET /api/trades
```

#### Response
```json
[
  {
    "id": 123,
    "userId": "34711896",
    "symbol": "EURUSD",
    "side": "BUY",
    "lotSize": 1.0,
    "entryPrice": 1.0950,
    "takeProfit": 1.1000,
    "stopLoss": 1.0900,
    "status": "PENDING",
    "source": "WEBHOOK",
    "createdAt": "2025-07-01T12:00:00.000Z",
    "updatedAt": "2025-07-01T12:00:00.000Z"
  }
]
```

### POST /api/trades

Creates a manual trade for the authenticated user.

#### Request
```http
POST /api/trades
Content-Type: application/json

{
  "symbol": "GBPUSD",
  "side": "SELL",
  "lotSize": 0.5,
  "entryPrice": 1.2650,
  "takeProfit": 1.2600,
  "stopLoss": 1.2700
}
```

#### Response
```json
{
  "id": 124,
  "userId": "34711896",
  "symbol": "GBPUSD",
  "side": "SELL",
  "lotSize": 0.5,
  "entryPrice": 1.2650,
  "takeProfit": 1.2600,
  "stopLoss": 1.2700,
  "status": "PENDING",
  "source": "MANUAL",
  "createdAt": "2025-07-01T12:05:00.000Z",
  "updatedAt": "2025-07-01T12:05:00.000Z"
}
```

### GET /api/stats

Retrieves trading statistics for the authenticated user.

#### Request
```http
GET /api/stats
```

#### Response
```json
{
  "totalTrades": 15,
  "successfulTrades": 8,
  "pendingTrades": 3,
  "winRate": 53.33
}
```

## Authentication Endpoints

### GET /api/auth/user

Retrieves the current authenticated user information.

#### Request
```http
GET /api/auth/user
```

#### Response
```json
{
  "id": "34711896",
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "profileImageUrl": "https://replit.com/public/images/avatar.png",
  "createdAt": "2025-01-01T00:00:00.000Z",
  "updatedAt": "2025-07-01T12:00:00.000Z"
}
```

### GET /api/login

Initiates the authentication flow with Replit Auth.

#### Request
```http
GET /api/login
```

#### Response
Redirects to Replit authentication page.

### GET /api/logout

Logs out the current user and ends the session.

#### Request
```http
GET /api/logout
```

#### Response
Redirects to landing page after logout.

### GET /api/callback

OAuth callback endpoint for Replit Auth (internal use).

## Rate Limiting

Currently, no rate limiting is implemented, but consider these guidelines:

- Webhook endpoint: Recommended max 60 requests per minute
- Trading endpoints: Reasonable usage for manual trading
- Authentication endpoints: Standard OAuth flow limits

## Error Handling

### Standard Error Response Format
```json
{
  "error": "Error type",
  "message": "Human-readable error message",
  "details": "Additional error details (optional)"
}
```

### Common Error Codes

| HTTP Code | Description | Common Causes |
|-----------|-------------|---------------|
| 400 | Bad Request | Invalid JSON, missing fields, validation errors |
| 401 | Unauthorized | Not logged in, expired session |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Invalid endpoint |
| 500 | Internal Server Error | Database errors, server issues |

## Data Types

### Trade Object
```typescript
interface Trade {
  id: number;
  userId: string;
  symbol: string;
  side: "BUY" | "SELL";
  lotSize: number;
  entryPrice: number;
  takeProfit?: number;
  stopLoss?: number;
  status: "PENDING" | "EXECUTED" | "CANCELLED" | "COMPLETED";
  source: "MANUAL" | "WEBHOOK";
  createdAt: string; // ISO 8601 datetime
  updatedAt: string; // ISO 8601 datetime
}
```

### User Object
```typescript
interface User {
  id: string;
  email?: string;
  firstName?: string;
  lastName?: string;
  profileImageUrl?: string;
  createdAt: string; // ISO 8601 datetime
  updatedAt: string; // ISO 8601 datetime
}
```

### Statistics Object
```typescript
interface Stats {
  totalTrades: number;
  successfulTrades: number;
  pendingTrades: number;
  winRate: number; // Percentage
}
```

## Examples

### TradingView Alert Webhook
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

### cURL Example
```bash
curl -X POST https://your-domain.replit.app/api/webhook \
  -H "Content-Type: application/json" \
  -H "Cookie: your-session-cookie" \
  -d '{
    "symbol": "BTCUSD",
    "side": "BUY",
    "lotSize": 0.1,
    "entryPrice": 45000,
    "takeProfit": 46000,
    "stopLoss": 44000
  }'
```

### JavaScript Fetch Example
```javascript
const response = await fetch('/api/webhook', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    symbol: 'EURUSD',
    side: 'SELL',
    lotSize: 2.0,
    entryPrice: 1.0950,
    takeProfit: 1.0900,
    stopLoss: 1.1000
  })
});

const result = await response.json();
console.log(result);
```

## Security Considerations

### Authentication
- All endpoints require valid user session
- Automatic token refresh prevents session expiration
- Secure session storage in PostgreSQL

### Data Validation
- All input validated using Zod schemas
- SQL injection prevention via parameterized queries
- Type-safe database operations

### User Isolation
- All trades linked to authenticated user
- No cross-user data access possible
- User-specific API responses

## Support

For additional help:
1. Check the User Guide (`WEBHOOK_DOCUMENTATION.md`)
2. Review Technical Integration (`TECHNICAL_INTEGRATION.md`)
3. Test using the webhook demo component
4. Monitor browser console for errors