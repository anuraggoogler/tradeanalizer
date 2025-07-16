# Technical Integration Guide - Trade2Optionss Webhook System

## Architecture Overview

The webhook system is built using a modern Node.js/Express backend with PostgreSQL database and implements the following technical stack:

### Backend Components
- **Express.js**: HTTP server handling webhook requests
- **Drizzle ORM**: Type-safe database operations with PostgreSQL
- **Zod Validation**: Runtime type checking and validation
- **Replit Auth**: OpenID Connect authentication system

### Database Schema
```sql
-- Trades table structure
CREATE TABLE trades (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR NOT NULL,
  symbol VARCHAR(20) NOT NULL,
  side VARCHAR(10) NOT NULL CHECK (side IN ('BUY', 'SELL')),
  lot_size DECIMAL(10,2) NOT NULL,
  entry_price DECIMAL(10,4) NOT NULL,
  take_profit DECIMAL(10,4),
  stop_loss DECIMAL(10,4),
  status VARCHAR(20) DEFAULT 'PENDING',
  source VARCHAR(20) DEFAULT 'MANUAL',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## API Endpoint Implementation

### Webhook Route Handler
```typescript
// POST /api/webhook
app.post("/api/webhook", isAuthenticated, async (req, res) => {
  try {
    // Validate request body using Zod schema
    const validatedData = insertTradeSchema.parse(req.body);
    
    // Get authenticated user ID
    const userId = req.user?.claims?.sub;
    
    // Create trade with webhook source
    const trade = await storage.createTrade({
      ...validatedData,
      source: "WEBHOOK"
    }, userId);
    
    res.status(201).json({
      message: "Webhook received successfully",
      trade
    });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: "Validation failed",
        details: error.errors
      });
    }
    res.status(500).json({ error: "Internal server error" });
  }
});
```

### Data Validation Schema
```typescript
// Zod schema for trade validation
export const insertTradeSchema = createInsertSchema(trades).omit({
  id: true,
  userId: true,
  status: true,
  source: true,
  createdAt: true,
  updatedAt: true,
}).extend({
  symbol: z.enum(TRADING_SYMBOLS),
  side: z.enum(["BUY", "SELL"]),
  lotSize: z.number().positive(),
  entryPrice: z.number().positive(),
  takeProfit: z.number().positive().optional(),
  stopLoss: z.number().positive().optional(),
});
```

## Authentication System

### Replit Auth Integration
```typescript
// Authentication middleware
export const isAuthenticated: RequestHandler = async (req, res, next) => {
  const user = req.user as any;

  if (!req.isAuthenticated() || !user.expires_at) {
    return res.status(401).json({ message: "Unauthorized" });
  }

  // Check token expiration
  const now = Math.floor(Date.now() / 1000);
  if (now <= user.expires_at) {
    return next();
  }

  // Refresh token if expired
  try {
    const config = await getOidcConfig();
    const tokenResponse = await client.refreshTokenGrant(config, user.refresh_token);
    updateUserSession(user, tokenResponse);
    return next();
  } catch (error) {
    res.status(401).json({ message: "Unauthorized" });
  }
};
```

### User Session Management
- Sessions stored in PostgreSQL using `connect-pg-simple`
- Automatic token refresh for expired sessions
- User-specific trade isolation using `userId` foreign key

## Database Operations

### Storage Interface
```typescript
export interface IStorage {
  // User operations
  getUser(id: string): Promise<User | undefined>;
  upsertUser(user: UpsertUser): Promise<User>;
  
  // Trade operations
  createTrade(trade: InsertTrade, userId?: string): Promise<Trade>;
  getAllTrades(userId?: string): Promise<Trade[]>;
  updateTradeStatus(id: number, status: string): Promise<Trade | undefined>;
  getTradesByStatus(status: string, userId?: string): Promise<Trade[]>;
}
```

### Trade Creation Implementation
```typescript
async createTrade(insertTrade: InsertTrade, userId?: string): Promise<Trade> {
  const [trade] = await db
    .insert(trades)
    .values({
      ...insertTrade,
      userId: userId || 'anonymous',
      status: 'PENDING',
      source: insertTrade.source || 'MANUAL'
    })
    .returning();
  return trade;
}
```

## Security Implementation

### Input Validation
- All webhook payloads validated using Zod schemas
- Type-safe database operations with Drizzle ORM
- SQL injection prevention through parameterized queries

### Authentication Security
- OpenID Connect with Replit as identity provider
- JWT token validation and automatic refresh
- Session-based authentication with secure cookies

### Data Isolation
- All trades linked to authenticated users via `userId`
- No cross-user data access possible
- API endpoints filter by user context automatically

## Error Handling

### Webhook Error Responses
```typescript
// Validation errors (400)
{
  "error": "Validation failed",
  "details": [
    {
      "code": "invalid_enum_value",
      "expected": ["BUY", "SELL"],
      "received": "INVALID",
      "path": ["side"]
    }
  ]
}

// Authentication errors (401)
{
  "error": "Unauthorized",
  "message": "Valid authentication required"
}

// Server errors (500)
{
  "error": "Internal server error",
  "message": "Database connection failed"
}
```

### Client-Side Error Handling
```typescript
// React Query mutation with error handling
const mutation = useMutation({
  mutationFn: async (data) => apiRequest("/api/webhook", { method: "POST", body: data }),
  onError: (error) => {
    if (isUnauthorizedError(error)) {
      toast({ title: "Unauthorized", variant: "destructive" });
      window.location.href = "/api/login";
      return;
    }
    // Handle other errors
  }
});
```

## Performance Considerations

### Database Optimization
- Indexed columns: `user_id`, `symbol`, `status`, `created_at`
- Connection pooling with Neon Database
- Efficient queries using Drizzle ORM relations

### Caching Strategy
- React Query for client-side data caching
- Stale-while-revalidate pattern for real-time updates
- Automatic cache invalidation on mutations

### Rate Limiting
Consider implementing rate limiting for production:
```typescript
import rateLimit from "express-rate-limit";

const webhookLimiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minute
  max: 60, // 60 requests per minute
  message: "Too many webhook requests"
});

app.use("/api/webhook", webhookLimiter);
```

## Deployment Configuration

### Environment Variables
```bash
# Required for authentication
REPL_ID=your-repl-id
REPLIT_DOMAINS=your-domain.replit.app
SESSION_SECRET=your-session-secret

# Database connection
DATABASE_URL=postgresql://user:pass@host:port/db

# Optional for production
NODE_ENV=production
PORT=5000
```

### Database Migration
```bash
# Push schema changes to database
npm run db:push

# Generate migrations (if needed)
npx drizzle-kit generate:pg
```

## Testing Implementation

### Unit Testing Example
```typescript
describe("Webhook API", () => {
  test("should create trade from valid webhook", async () => {
    const tradeData = {
      symbol: "EURUSD",
      side: "BUY",
      lotSize: 1.0,
      entryPrice: 1.0950,
      takeProfit: 1.1000,
      stopLoss: 1.0900
    };

    const response = await request(app)
      .post("/api/webhook")
      .send(tradeData)
      .expect(201);

    expect(response.body.trade.symbol).toBe("EURUSD");
    expect(response.body.trade.source).toBe("WEBHOOK");
  });
});
```

### Integration Testing
```bash
# Test webhook endpoint
curl -X POST http://localhost:5000/api/webhook \
  -H "Content-Type: application/json" \
  -H "Cookie: sessionId=test-session" \
  -d '{"symbol":"EURUSD","side":"BUY","lotSize":1.0,"entryPrice":1.0950}'
```

## Monitoring and Logging

### Request Logging
```typescript
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} ${req.method} ${req.path}`);
  next();
});
```

### Error Tracking
```typescript
app.use((error, req, res, next) => {
  console.error("API Error:", error);
  // Send to error tracking service (Sentry, etc.)
  res.status(500).json({ error: "Internal server error" });
});
```

## Scaling Considerations

### Horizontal Scaling
- Stateless authentication (JWT tokens)
- Database connection pooling
- Session storage in PostgreSQL (not memory)

### Performance Monitoring
- Response time monitoring
- Database query performance
- Real-time error tracking
- User authentication success rates

### Database Scaling
- Read replicas for analytics queries
- Partitioning trades table by date
- Archive old trade data periodically

## API Documentation

### OpenAPI Specification
```yaml
openapi: 3.0.0
info:
  title: Trade2Optionss Webhook API
  version: 1.0.0
paths:
  /api/webhook:
    post:
      summary: Receive trading webhook
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [symbol, side, lotSize, entryPrice]
              properties:
                symbol:
                  type: string
                  enum: [EURUSD, GBPUSD, BTCUSD, ...]
                side:
                  type: string
                  enum: [BUY, SELL]
                lotSize:
                  type: number
                  minimum: 0.01
                entryPrice:
                  type: number
                  minimum: 0
```

This technical guide provides the complete implementation details for developers who need to understand or extend the webhook system.