# Trade2Optionss Auto Trading Platform

## Overview

Trade2Optionss is a comprehensive Groww-style trading platform built for multi-asset trading across Stocks, Forex, Commodities, and Futures & Options. The platform features professional-grade Dhan API integration for real trading execution, TradingView webhook automation, advanced portfolio management, and a mobile-first design inspired by India's leading investment platforms.

## System Architecture

### Frontend Architecture
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite for fast development and optimized builds
- **UI Library**: Radix UI components with shadcn/ui system
- **Styling**: Tailwind CSS with custom trading-themed colors
- **State Management**: TanStack Query for server state management
- **Routing**: Wouter for lightweight client-side routing
- **Form Handling**: React Hook Form with Zod validation

### Backend Architecture
- **Runtime**: Node.js with TypeScript
- **Framework**: Express.js for HTTP server
- **Database**: PostgreSQL with Drizzle ORM (configured but using in-memory storage currently)
- **Validation**: Zod schemas for type-safe data validation
- **API Design**: RESTful endpoints with proper error handling

### Development Setup
- **Hot Reload**: Vite dev server with HMR
- **TypeScript**: Strict type checking across the stack
- **Path Aliases**: Configured for clean imports (@/, @shared/, etc.)
- **Development Tools**: Replit-specific plugins for enhanced development experience

## Key Components

### Trading Interface
- **Trade Form**: Manual trade entry with symbol selection, lot size, entry price, take profit, stop loss, and buy/sell options
- **Trade History**: Real-time display of all trades with filtering by status
- **Market Status**: Live market data display for major trading pairs
- **Statistics Dashboard**: Trading performance metrics including win rate and trade counts

### Data Management
- **Trade Entity**: Core trading data with fields for symbol, side, lot size, prices, status, and source
- **Validation Layer**: Comprehensive input validation using Zod schemas
- **Storage Interface**: Abstracted storage layer supporting multiple backends

### API Endpoints
- `GET /api/trades` - Retrieve all trades
- `POST /api/trades` - Create manual trade
- `POST /api/webhook` - TradingView webhook endpoint
- `GET /api/stats` - Trading statistics

## Data Flow

1. **Manual Trading**: User fills trade form → React Hook Form validation → API call → Backend validation → Storage → UI update
2. **Webhook Trading**: TradingView alert → Webhook endpoint → Backend validation → Storage → Real-time UI update
3. **Data Retrieval**: Component mount → TanStack Query fetch → API endpoint → Database query → JSON response → UI render

## External Dependencies

### Production Dependencies
- **UI Components**: Extensive Radix UI ecosystem for accessible components
- **Database**: Neon Database serverless PostgreSQL
- **ORM**: Drizzle with PostgreSQL dialect
- **Validation**: Zod for runtime type checking
- **HTTP Client**: Fetch API with custom wrapper
- **Date Handling**: date-fns for date manipulation
- **Styling**: clsx and class-variance-authority for conditional classes

### Development Tools
- **Build**: ESBuild for backend bundling
- **Database**: Drizzle Kit for schema management
- **Type Checking**: TypeScript compiler
- **CSS Processing**: PostCSS with Tailwind CSS

## Deployment Strategy

### Build Process
1. **Frontend**: Vite builds React app to `dist/public`
2. **Backend**: ESBuild bundles Node.js server to `dist/index.js`
3. **Database**: Drizzle pushes schema changes to PostgreSQL

### Environment Configuration
- `DATABASE_URL` - PostgreSQL connection string (required)
- `NODE_ENV` - Environment mode (development/production)
- Hot reload and development tools disabled in production

### Hosting Requirements
- Node.js runtime environment
- PostgreSQL database (Neon Database recommended)
- Static file serving for frontend assets
- Environment variable support

## Recent Changes

```
Recent Updates:
- July 01, 2025: Completed full migration to authentic Finnhub API data throughout entire platform
  * Replaced ALL simulated/stored data with live Finnhub API feeds for complete authenticity
  * Updated cleanMarketData service to use only real Finnhub API calls, eliminating fallback simulations
  * Enhanced universalSearchService to fetch live data from Finnhub/IndiaStockAPI, removed all fake data
  * Fixed data consistency issues - search results now match stock detail pages (TATAMOTORS ₹676.52)
  * Individual stock quotes (/api/stocks/:symbol) now return authentic real-time prices with proper error handling
  * Dashboard market overview displays live Finnhub data for stocks with 15-second refresh intervals
  * Enhanced StockDetailView to show real market data instead of hardcoded information
  * All TATA stocks (TATAMOTORS, TATASTEEL, TATACONSUM) now supported with live pricing
  * Search and detail pages show consistent authentic data throughout the platform
  * Removed all synthetic data fallbacks - system returns errors instead of fake data when APIs fail
- July 01, 2025: Built comprehensive universal search covering ALL trading instruments with cross-tab integration
  * Expanded search database to 270+ instruments: 150+ stocks, 33 forex pairs, 85+ commodities
  * Added complete NIFTY 50, Next 50, midcap, smallcap stocks across all sectors (Banking, IT, Pharma, Auto, etc.)
  * Comprehensive forex coverage: Major INR pairs, cross-currency pairs, emerging markets, commodity currencies
  * Extensive commodity database: Precious metals, energy, base metals, agriculture, spices, pulses, international
  * Built SearchContext provider for cross-tab data propagation and real-time search result sharing
  * Enhanced search to include detailed categories, exchanges, and instrument-specific metadata
  * Search now covers Indian stocks (NSE/BSE), international forex pairs, MCX commodities, auto-generated F&O
  * Optimized refresh rate from 3-second aggressive polling to industry-standard 15-second intervals
  * Professional search API returns live prices, changes, volumes, expiry dates, and comprehensive details
- July 01, 2025: Created fully interactive advanced charting system with professional features
  * Built InteractiveAdvancedChart component with real-time crosshair and tooltip functionality
  * Added comprehensive technical indicators (SMA, EMA, RSI, MACD, Bollinger Bands, Stochastic)
  * Implemented indicator management system with easy toggle on/off functionality
  * Added oscillator charts for RSI, MACD, and Stochastic indicators in separate panes
  * Created professional chart controls with timeframe selection and fullscreen mode
  * Enhanced volume visualization with interactive volume bars and hover effects
  * Fixed white screen issues by replacing problematic TradingView library with custom solution
  * Added crosshair functionality showing OHLCV data on mouse hover
  * Implemented color-coded indicator display with visibility controls
  * Charts now work perfectly in both Stock Detail and F&O sections
Recent Updates:
- July 01, 2025: Implemented real-time data architecture matching user's architectural diagram
  * Created comprehensive MarketDataService with real-time data streaming and technical analysis
  * Built WebSocket server architecture for live market data feeds to all clients
  * Added RealTimeMarketData component with live price updates and technical indicators
  * Integrated WebSocket client hooks for managing real-time connections and subscriptions
  * Enhanced TradingDashboard to display live streaming data for Stocks/Forex/Commodities
  * Implemented technical analysis calculations (SMA, RSI, Bollinger Bands, MACD, ATR)
  * Added market data ingestion simulation for 20+ symbols across multiple asset classes
  * Built data persistence layer storing real-time market data to database
  * Created subscription-based WebSocket messaging for efficient data distribution
  * Enhanced backend with dedicated endpoints for technical indicators and live quotes
- July 01, 2025: Transformed into comprehensive Groww-style multi-asset trading platform
  * Built professional Dhan API integration service for real trading execution
  * Created comprehensive TradingDashboard component with Stocks/Forex/Commodities/F&O tabs
  * Expanded database schema with watchlists, alerts, positions, and market data tables
  * Enhanced storage layer with 20+ new methods for portfolio and watchlist management
  * Added multi-asset market overview with live data feeds for indices, currencies, and commodities
  * Implemented user trading mode switching (SIMULATION vs LIVE) with Dhan credentials
  * Built professional portfolio summary with P&L calculations and fund management
  * Added comprehensive API endpoints for portfolio, market data, watchlists, and alerts
  * Enhanced authentication to support Dhan client integration and trading preferences
  * Maintained backward compatibility with existing webhook and manual trading features
- July 01, 2025: Created comprehensive webhook documentation and integration guides
  * Added complete user guide (WEBHOOK_DOCUMENTATION.md) with TradingView setup instructions
  * Created technical integration guide (TECHNICAL_INTEGRATION.md) for developers
  * Developed detailed API reference (API_REFERENCE.md) with all endpoints
  * Enhanced webhook demo component with direct access to all documentation
  * Documented all 40+ supported trading symbols across multiple asset classes
  * Included security features, error handling, and best practices
  * Added practical examples for TradingView alerts and API usage
- July 01, 2025: Implemented comprehensive authentication system
  * Added Replit Auth integration with OpenID Connect
  * Created secure PostgreSQL database with user and session management
  * Implemented protected routes and user-specific trade isolation
  * Added professional landing page for unauthenticated users
  * Created user profile dropdown with logout functionality
  * Enhanced security with JWT token handling and automatic session refresh
  * Integrated error handling for authentication failures and redirects
- July 01, 2025: Enhanced trading platform with comprehensive market data integration
  * Added real-time market charts for cryptocurrencies, Indian stocks, commodities, and forex
  * Integrated live data APIs (CoinGecko for crypto, metals APIs for commodities)
  * Expanded trading symbols to include 40+ instruments across multiple asset classes
  * Created tabbed market overview with interactive charts using Recharts
  * Added webhook demonstration component with TradingView integration guide
  * Enhanced UI with categorized symbol selection and improved market status display
  * Implemented fallback data handling for API failures
- July 01, 2025: Initial platform setup with core trading functionality
```

## User Preferences

```
Preferred communication style: Simple, everyday language.
```