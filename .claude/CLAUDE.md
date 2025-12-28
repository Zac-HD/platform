# Repository Exploration Notes

Surprising and noteworthy discoveries from exploring this trading bootcamp platform.

## 🌟 Standout Features

### Time Travel Debugging
**Most impressive discovery:** The UI can replay market history at any transaction ID. There's a slider that lets you scrub through historical states of the order book and trades.

- Implementation: `frontend/src/lib/components/marketDataUtils.ts` - `ordersAtTransaction()` and `tradesAtTransaction()`
- Orders maintain size history at each transaction
- Full audit trail and replay capability
- This is **exceptional** for an educational platform - most real trading platforms don't offer this granularity

### Puzzle Hunt Easter Egg 🍃
Markets with names starting with "ASCII" get a **leaf background image**:
- Check `frontend/src/lib/components/marketDataUtils.ts:110-117`
- Check `frontend/src/lib/components/market.svelte:359-381`
- Suggests hidden puzzle hunt or special educational markets

## 🎨 Clever Frontend Patterns

### Container Queries Over Media Queries
**No traditional media queries!** Uses CSS Container Queries throughout:
- Market component switches between tabbed (mobile) and side-by-side (desktop) based on container width
- Order book columns use `cqi` units for smooth scaling
- See `frontend/src/lib/components/market.svelte:327-358`

### Responsive Banner with Measurement
Sophisticated header that measures content width dynamically:
- `frontend/src/routes/+layout.svelte:48-98`
- Renders hidden measurement elements to determine available space
- Uses ResizeObserver to choose "Available Balance" vs "Available" vs minimal mode
- Prevents overflow without arbitrary breakpoints

### FlexNumber Component
Micro-optimization for aligning decimal numbers in the order book:
- `frontend/src/lib/components/flexNumber.svelte`
- Splits numbers into `[integer, '.', decimal]` parts
- Uses CSS Grid to perfectly align decimal points
- Small detail but shows attention to polish

### Batched Notification System
Smart notification batching prevents toast spam during high-frequency trading:
- `frontend/src/lib/notifications.ts`
- Batches order creations, fills, and cancellations by market
- 500ms debounce with immediate flush if > 500ms elapsed
- Critical for bot trading scenarios

## 🏗️ Architecture Surprises

### Patch-Based State Management
Frontend uses **incremental state updates** via WebSocket patches:
- `frontend/src/lib/api.svelte.ts` (470 lines) - **most important file**
- Server sends only changes, not full state
- Client aggregates patches into coherent reactive state
- Transaction IDs ensure ordering
- Handles order fills, cancellations, new orders, trades, settlements

### Request-Response Over WebSocket
- Each message has optional `request_id`
- Client sends request with ID, waits for response with matching ID
- Python client makes this look synchronous (see `python-client/src/metagame/trading_client.py`)

### Separate Team Allocation App
- `team-allocation/` is a **completely different stack**: TanStack Router + React
- Main frontend is SvelteKit
- Suggests this was built for specific bootcamp logistics needs
- Interesting architectural decision to keep it separate

## 🔧 Technical Choices

### Svelte 5 Throughout
Uses cutting-edge Svelte 5 runes extensively:
- `$state`, `$derived`, `$derived.by()`, `$effect`
- `SvelteMap` for reactive maps instead of regular Maps
- Snippets for component composition
- Very modern, clean reactive code

### Protobuf Schema Organization
**33 separate .proto files** with clean separation:
- `schema/client-message.proto` - all client requests
- `schema/server-message.proto` - all server responses
- Modular imports for each message type
- Generated bindings in `schema-js` workspace package

### SQLite with Sophisticated Setup
Backend uses SQLite with:
- Write-Ahead Logging (WAL)
- Custom checkpointing task (512 page limit)
- Connection pooling (8-64 connections)
- **5017 lines** in `backend/src/db.rs` - substantial business logic
- Migration system with squashing support

## 🎓 Educational Features

### Clean Python Client API
Extremely student-friendly:
```python
with TradingClient(api_url, jwt, act_as) as client:
    client.create_order(market_id, price=50.0, size=1.0, side=Side.BID)
    client.out(market_id)  # Cancel all orders
    market = client.state().markets[market_id]
```

### Example Trading Bots
Multiple example strategies:
- `naive_bot` - basic market interactions
- `market_maker` - provides liquidity
- `TWAP` - Time-Weighted Average Price execution

### Admin "Act As" Functionality
- Admins can impersonate other accounts for support
- "Sudo mode" toggle prevents accidental admin actions
- Persists to localStorage
- Smart validation prevents getting locked out

## 🎯 Advanced Market Features

Beyond basic order books:
- **Auctions** - separate from continuous markets, with BIN prices
- **Redeemable markets** - like ETF creation/redemption with constituent baskets
- **Market types and groups** - organizational hierarchy
- **Market status** - OPEN, PAUSED, SEMI_PAUSED
- **Visibility controls** - markets can be restricted to specific accounts
- **Account hiding** - anonymize order book for some markets
- **Pinned markets** - featured markets

## 📊 Notable Dependencies

**Frontend:**
- `layerchart` - Svelte charting library (not Chart.js/Recharts)
- `reconnecting-websocket` - robust WebSocket client
- `mode-watcher` - dark mode management
- `svelte-sonner` - toast notifications
- `@tanstack/svelte-virtual` - ready for virtual scrolling

**Python:**
- `betterproto` - Python protobuf library
- `websockets` - sync WebSocket client
- `typer` - CLI framework for bots

## 💡 Code Quality Notes

**Strengths:**
- Modern, idiomatic Svelte 5 patterns
- Type-safe throughout (TypeScript + Protobuf)
- Excellent separation of concerns
- Well-documented (DESIGN.md, CLAUDE.md, multiple READMEs)
- Production-ready error handling
- Comprehensive admin tooling

**This is reference-quality code** for a simulated exchange platform. The combination of real-time WebSocket state management, time-travel debugging, clean student-facing API, and modern frontend patterns shows this is battle-tested in actual bootcamp use.

The thoughtful UX touches (notification batching, responsive banner, sudo mode, take order button) only come from real-world usage and iteration.
