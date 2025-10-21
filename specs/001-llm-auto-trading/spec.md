# Feature Specification: LLM-Based Automated Trading System

**Feature Branch**: `001-llm-auto-trading`
**Created**: 2025-10-21
**Status**: Draft  
**Input**: User description: "우리 앱은 자동으로 코인을 거래해야 한다. 이떄 코인 거래에 대한 판단은 llm 이 진행할 수 있어야 한다. llm 판단은 주기적으로 호출되어야 한다. llm 프롬프트는 지속적으로 업그레이드되어야 하며, 버전이 제대로 추적되어야 한다. 각 거래들, llm 의 매 판단 근거 또한 저장되어야 한다. llm 에는 다양한 자산, 특히 다양한 코인들의 정보가 제공되어야 한다. 시계열 가격 데이터는 기본이고, 기본적으로 얻을 수 있는 다양한 기술적 지표, 희망적으로는 웹을 통해 얻을 수 있는 다양한 정보가 제공될 수 있어야 하고 이는 필요에 따라 추가될 수 있어야 한다. 다양한 모델을 사용할 수 있어야 한다. 거래소와 연결되어 자동화된 거래를 할 수 있어야 한다."

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Basic Automated Trading with LLM Decisions (Priority: P1)

The system automatically executes cryptocurrency trades based on LLM's analysis and recommendations. The trader configures trading parameters (position size, risk limits), starts the bot in paper trading mode to validate behavior, then switches to live trading mode. The system periodically analyzes market conditions, makes trading decisions via LLM, and executes trades automatically on the connected exchange.

**Why this priority**: This is the core MVP functionality. Without automated LLM-driven trading, there is no product. This story delivers immediate value by enabling hands-free trading with AI-powered decision making.

**Independent Test**: Can be fully tested by configuring the bot with test parameters, running it in paper trading mode, observing that it makes periodic trading decisions, and verifying trades are logged with LLM reasoning. No other features are required.

**Acceptance Scenarios**:

1. **Given** the bot is configured with valid Binance Futures credentials and risk limits, **When** the bot is started in paper trading mode, **Then** the system connects to the exchange, begins periodic market analysis, and simulates leveraged trades without using real money
2. **Given** the bot is running in paper trading mode, **When** an analysis cycle completes, **Then** the system queries account information (available cash, account value, total return, Sharpe ratio) and all active positions (symbol, quantity, entry price, current price, liquidation price, unrealized PnL, leverage, exit plan with invalidation conditions), constructs an LLM prompt including this comprehensive account context along with market data for all monitored cryptocurrencies, and the LLM analyzes all information to provide trading decisions for multiple coins in a single JSON response
3. **Given** the bot has 6 active leveraged positions (ETH, SOL, XRP, BTC, DOGE, BNB), **When** the LLM prompt is constructed, **Then** the prompt includes complete details for each position including liquidation price, profit/loss targets, stop loss settings, invalidation conditions (e.g., "If price closes below X on 3-minute candle"), confidence scores, risk amounts, and notional values to provide full portfolio awareness
4. **Given** the LLM analyzes positions with invalidation conditions, **When** the LLM evaluates each position, **Then** the LLM checks whether invalidation conditions are triggered (e.g., comparing current price to invalidation threshold) and returns 'hold' for positions where conditions are not triggered, or 'close_position' where conditions are triggered
5. **Given** the LLM returns decisions for 6 cryptocurrencies in JSON format, **When** the system parses the response, **Then** the system extracts action type (buy_to_enter | sell_to_enter | hold | close_position), quantity, leverage (5x-40x), profit target, stop loss, invalidation condition, and justification for each coin
6. **Given** the LLM recommends buy_to_enter action for Bitcoin with 10x leverage, **When** risk checks pass (position size within limits, sufficient margin, liquidation price >10% from current), **Then** a long futures order is placed on the exchange with specified leverage and execution details are logged
7. **Given** the LLM recommends sell_to_enter action for Ethereum with 15x leverage, **When** risk checks pass, **Then** a short futures order is placed on the exchange with specified leverage and execution details are logged
8. **Given** the LLM recommends close_position for an existing position, **When** the position exists and liquidation is not imminent, **Then** the position is closed at market price and final profit/loss is calculated and logged
9. **Given** the LLM recommends a trade that exceeds position size limits or would create liquidation risk >10%, **When** risk checks run, **Then** the trade is rejected, a warning is logged with specific reason, and no order is placed
10. **Given** the bot has been validated in paper trading mode for 7 days, **When** the operator switches to live trading mode with explicit confirmation, **Then** subsequent trades use real money and actual leveraged orders are placed on Binance Futures
11. **Given** the system encounters an LLM timeout or error, **When** attempting to get trading decisions, **Then** the system logs the error, skips the current cycle, and continues with the next scheduled analysis

---

### User Story 2 - Market Data Collection and Technical Analysis (Priority: P2)

The system collects comprehensive market data including time-series price data (OHLCV), volume, and calculates technical indicators (moving averages, RSI, MACD, Bollinger Bands, etc.). This data is provided to the LLM to inform trading decisions. The data collection system is extensible, allowing new indicators and data sources to be added as needed.

**Why this priority**: Trading decisions require quality market data. While the basic system (P1) can work with minimal data, providing rich technical analysis significantly improves decision quality. This story enhances the MVP with professional-grade market intelligence.

**Independent Test**: Can be tested by configuring data collection for specific cryptocurrencies, verifying historical price data is fetched and stored, confirming technical indicators are calculated correctly, and observing that this data is included in LLM prompts. The data collection system works independently of trading execution.

**Acceptance Scenarios**:

1. **Given** a list of cryptocurrencies to monitor, **When** the data collection system starts, **Then** historical OHLCV (Open, High, Low, Close, Volume) data for the past 90 days is fetched from the exchange for each cryptocurrency
2. **Given** historical price data exists, **When** technical indicators are calculated, **Then** the system computes SMA (7, 25, 99 day), RSI (14 day), MACD, and Bollinger Bands for each cryptocurrency and stores the results
3. **Given** the bot is running, **When** each analysis cycle begins, **Then** the latest market data and technical indicators are refreshed and made available to the LLM
4. **Given** technical indicator data is available, **When** the LLM prompt is constructed, **Then** the prompt includes current price, 24h price change, volume trends, and key technical indicator values (RSI, MACD signal, moving average positions)
5. **Given** a new technical indicator is needed, **When** a developer adds a new indicator module, **Then** the indicator is automatically calculated and included in the data pipeline without modifying core trading logic

---

### User Story 3 - Prompt Version Management and Multi-Model Support (Priority: P3)

The system maintains versioned prompts for LLM trading analysis, allowing continuous prompt improvement while tracking which version produced which trading decisions. The system supports multiple LLM models (GPT-4, Claude, etc.) and allows switching between them or running multiple models for comparison.

**Why this priority**: Prompt engineering is crucial for trading performance. Version management enables A/B testing, performance analysis, and rollback if a prompt version underperforms. Multi-model support provides flexibility and reduces dependency on a single AI provider.

**Independent Test**: Can be tested by creating multiple prompt versions, configuring which version is active, executing trading cycles, and verifying each decision is tagged with the prompt version used. Multi-model support can be tested by configuring different models and comparing their outputs for the same market conditions.

**Acceptance Scenarios**:

1. **Given** multiple prompt versions exist in the system, **When** an operator selects prompt version "v2.3" as active, **Then** all subsequent LLM calls use the v2.3 prompt template
2. **Given** a trading decision is made, **When** the decision is logged, **Then** the log includes the prompt version identifier (e.g., "v2.3"), allowing later analysis of which prompt versions performed best
3. **Given** a new prompt version "v2.4" is created, **When** the operator activates it, **Then** the system switches to the new prompt without requiring restart or losing trading state
4. **Given** multiple LLM models are configured (GPT-4, Claude), **When** the operator selects "Claude-3-Opus" as the active model, **Then** subsequent trading decisions use Claude instead of GPT-4
5. **Given** two LLM models are configured for comparison mode, **When** an analysis cycle runs, **Then** both models analyze the same market data, their recommendations are logged side-by-side, and the primary model's decision is used for actual trading
6. **Given** a prompt version history exists, **When** an operator reviews performance, **Then** the system can display win rate, average return, and decision quality metrics grouped by prompt version

---

### User Story 4 - Web-Based Intelligence and Extensible Data Sources (Priority: P4)

The system can fetch and incorporate external information from the web (news, social sentiment, on-chain metrics, etc.) into LLM trading decisions. The data source system is plugin-based, allowing new information sources to be added without modifying core logic.

**Why this priority**: Advanced traders use information beyond price charts. News events, social sentiment, and on-chain data can provide trading edge. However, this is not essential for initial functionality and can be added incrementally.

**Independent Test**: Can be tested by configuring a web scraper plugin (e.g., crypto news aggregator), verifying it fetches relevant news articles, confirming this information is included in LLM context, and observing that the LLM's decision references the external information. Works independently of trading execution.

**Acceptance Scenarios**:

1. **Given** a news aggregator plugin is configured, **When** an analysis cycle begins, **Then** recent news headlines related to the cryptocurrencies being traded are fetched and summarized
2. **Given** external information is available, **When** the LLM prompt is constructed, **Then** the prompt includes a summary of relevant news, social sentiment indicators, and other external data sources
3. **Given** a new data source plugin is installed (e.g., Twitter sentiment analyzer), **When** the plugin is activated, **Then** its data is automatically collected and included in the LLM context without code changes to the core trading system
4. **Given** multiple external data sources are active, **When** data collection fails for one source, **Then** the system continues with available data sources and logs the failure without blocking trading decisions
5. **Given** web-scraped information is expensive or rate-limited, **When** configuring a data source, **Then** the operator can set collection frequency (e.g., every 15 minutes vs every hour) independently from trading cycle frequency

---

### User Story 5 - Decision Audit Trail and Performance Analysis (Priority: P4)

Every trading decision, LLM reasoning, and trade execution result is stored in a structured audit log. The system provides analysis tools to review historical decisions, understand why trades were made, calculate performance metrics, and identify patterns in successful vs unsuccessful trades.

**Why this priority**: Audit trails are essential for compliance, debugging, and improvement. However, basic logging (included in P1) is sufficient for initial operation. This story adds comprehensive analysis and querying capabilities.

**Independent Test**: Can be tested by running the bot for multiple trading cycles, executing various trades, then using the analysis tools to query decisions, filter by outcome, view LLM reasoning for specific trades, and generate performance reports. Works with logged data independently of active trading.

**Acceptance Scenarios**:

1. **Given** the bot has executed multiple trades, **When** an operator queries the audit log, **Then** all trading decisions are retrievable with timestamp, cryptocurrency, LLM model used, prompt version, full reasoning text, and decision outcome (BUY/SELL/HOLD)
2. **Given** a specific trade resulted in a loss, **When** reviewing the audit log for that trade, **Then** the operator can see the exact market conditions, technical indicators, external information, and LLM reasoning that led to the decision
3. **Given** trading history exists, **When** generating a performance report, **Then** the system calculates win rate, average return per trade, maximum drawdown, Sharpe ratio, and total profit/loss for a specified time period
4. **Given** multiple prompt versions have been used, **When** comparing performance by prompt version, **Then** the report shows which versions had the highest win rate and best risk-adjusted returns
5. **Given** LLM reasoning is stored, **When** analyzing unsuccessful trades, **Then** the system can identify common patterns in the reasoning that correlate with losses (e.g., "overconfidence words" or "missing risk factors")
6. **Given** a regulatory audit or tax requirement, **When** exporting trade history, **Then** the system generates a complete record of all trades with timestamps, amounts, prices, and profit/loss in required formats

---

### Edge Cases

- **What happens when the exchange API is temporarily unavailable?** The system detects API failures, pauses trading, logs the issue, and retries with exponential backoff. No trades are executed until connection is restored. Alerts are sent if downtime exceeds a configured threshold.

- **What happens when LLM response is ambiguous or unparseable?** The response is logged as invalid, the current trading cycle is skipped, and an alert is raised. The system continues with the next scheduled cycle. If multiple consecutive failures occur, the bot automatically switches to safety mode (HOLD only, no new positions).

- **What happens when a trade order is rejected by the exchange?** The rejection reason is logged, the order is not retried automatically, and an alert is sent. The system continues monitoring but does not attempt similar orders until conditions change or operator intervenes.

- **What happens when the bot owns cryptocurrency but LLM recommends buying more of the same asset?** Position sizing logic checks current holdings. If adding the new position would exceed maximum position size limits, the order is scaled down or rejected. The logic and decision are logged.

- **What happens when network latency causes stale price data?** Each data point is timestamped. If market data is older than a configured threshold (e.g., 60 seconds), the system treats it as stale, skips the trading cycle, and logs a warning. Fresh data is fetched for the next cycle.

- **What happens during high volatility when prices change rapidly between LLM analysis and order execution?** The system implements a price tolerance check. If the current market price deviates more than a configured percentage from the price used in LLM analysis, the order is aborted and the cycle is rerun with fresh data.

- **What happens if the daily loss limit is reached mid-trade?** The trade in execution is allowed to complete, but immediately after, the bot enters safety lockdown mode. No new positions are opened for the remainder of the day. HOLD-only mode is activated automatically.

- **What happens when the kill switch is activated?** All pending orders are cancelled immediately, the system stops all trading activity, and enters a frozen state. Manual operator intervention is required to restart. The kill switch can be activated via API, command line, or configuration file.

## Requirements _(mandatory)_

### Functional Requirements

**Core Trading Engine**

- **FR-001**: System MUST connect to cryptocurrency exchanges via their official APIs to execute buy and sell orders
- **FR-002**: System MUST support two operating modes: paper trading (simulation) and live trading (real money), with explicit operator confirmation required to switch from paper to live
- **FR-003**: System MUST execute a periodic trading cycle every 5 minutes by default, with the interval configurable via environment variable or configuration file to allow adjustment based on trading strategy needs (supported range: 1 minute to 24 hours)
- **FR-004**: System MUST query current market conditions (prices, volumes, order books) at the start of each trading cycle
- **FR-005**: System MUST invoke an LLM with market data and prompt to generate trading decisions for all monitored cryptocurrencies in a single call, with each decision including an action type and reasoning. Supported action types:
  - **buy_to_enter**: Open a new long position (buying cryptocurrency)
  - **sell_to_enter**: Open a new short position (selling cryptocurrency short via futures)
  - **hold**: Maintain existing position without changes
  - **close_position**: Close an existing position (exit trade)
- **FR-006**: System MUST validate LLM recommendations against safety rules (position limits, loss limits, balance checks) before execution
- **FR-007**: System MUST place actual orders on the exchange when in live trading mode and risk checks pass
- **FR-008**: System MUST log every trading decision with timestamp, cryptocurrency, action, reasoning, and execution result

**LLM Integration**

- **FR-009**: System MUST support multiple LLM models including GPT-4 and Claude, with configuration to select active model
- **FR-010**: System MUST construct prompts that include current market data, technical indicators, and comprehensive trading context including:
  - Account performance metrics (current total return percentage, Sharpe ratio)
  - Account financial status (available cash, current account value)
  - All active positions with details: symbol, quantity, entry price, current price, liquidation price, unrealized PnL, leverage, exit plan (profit target, stop loss, invalidation condition), confidence score, risk in USD, order IDs (stop loss, take profit, entry), notional value in USD
  - Portfolio-level risk metrics and exposure
- **FR-011**: System MUST query and refresh account information and active positions at the start of each trading cycle before constructing the LLM prompt
- **FR-012**: System MUST enforce timeout limits on LLM API calls (30 seconds maximum)
- **FR-013**: System MUST parse LLM responses to extract structured trading decisions for multiple cryptocurrencies. Expected response format is a JSON object where each key is a cryptocurrency symbol and each value contains:
  - **coin**: cryptocurrency symbol (string)
  - **signal**: action type (buy_to_enter | sell_to_enter | hold | close_position)
  - **quantity**: trade quantity (float, full current size for hold/close)
  - **profit_target**: target price for taking profit (float)
  - **stop_loss**: price for stopping loss (float)
  - **invalidation_condition**: text description of condition that invalidates the trade thesis (string)
  - **leverage**: leverage multiplier for futures trading (integer, 5-40)
  - **confidence**: confidence level in the decision (float, 0-1)
  - **risk_usd**: risk amount in USD (float)
  - **justification**: reasoning for entry/exit/close decisions (string, required for buy_to_enter, sell_to_enter, close_position; not required for hold)
- **FR-014**: System MUST handle LLM failures gracefully by skipping the current cycle and logging errors without crashing
- **FR-015**: System MUST track and enforce daily LLM API cost budgets to prevent runaway expenses

**Prompt Management**

- **FR-016**: System MUST store prompt templates with version identifiers (e.g., "v1.0", "v2.3")
- **FR-017**: System MUST allow operators to activate a specific prompt version, making it the active prompt for all subsequent trading cycles
- **FR-018**: System MUST tag every logged trading decision with the prompt version and LLM model used
- **FR-019**: System MUST support creating new prompt versions without requiring code changes or system restart
- **FR-020**: System MUST preserve historical prompt versions to enable rollback and A/B testing
- **FR-021**: Prompt templates SHOULD encourage structured reasoning from the LLM by requesting step-by-step analysis (e.g., "First check existing positions, then evaluate invalidation conditions, then determine actions"), but structured reasoning is not mandatory as long as clear justification is provided

**Market Data Collection**

- **FR-022**: System MUST fetch and store historical OHLCV (Open, High, Low, Close, Volume) data for monitored cryptocurrencies
- **FR-023**: System MUST calculate technical indicators including Simple Moving Average (SMA), Relative Strength Index (RSI), MACD, and Bollinger Bands
- **FR-024**: System MUST update market data and indicators at the start of each trading cycle to provide fresh information to the LLM
- **FR-025**: System MUST support adding new technical indicators via a plugin or module system without modifying core trading logic
- **FR-026**: System MUST store calculated indicator values with timestamps for historical analysis

**External Data Sources**

- **FR-027**: System MUST provide a plugin interface for integrating external data sources (news, social sentiment, on-chain metrics)
- **FR-028**: System MUST allow data source plugins to be enabled or disabled via configuration
- **FR-029**: System MUST gracefully handle data source failures by continuing with available data and logging warnings
- **FR-030**: System MUST include active external data source information in LLM prompts when available

**Audit Trail and Logging**

- **FR-031**: System MUST store every trading decision in a structured format including timestamp, cryptocurrency, action type (buy_to_enter/sell_to_enter/hold/close_position), quantity, price, LLM model, prompt version, and full reasoning text
- **FR-032**: System MUST store every trade execution result including order ID, execution price, filled quantity, fees, and profit/loss
- **FR-033**: System MUST provide query capabilities to retrieve decisions by date range, cryptocurrency, outcome, or prompt version
- **FR-034**: System MUST calculate and store performance metrics including win rate, average return, maximum drawdown, Sharpe ratio, and total profit/loss
- **FR-035**: System MUST support exporting audit logs in structured formats (CSV, JSON) for external analysis or compliance

**Exchange Integration**

- **FR-036**: System MUST initially support Binance Futures exchange API (selected for largest global trading volume, leverage trading support, extensive cryptocurrency pair coverage, and excellent API documentation), with architecture designed to allow additional exchange integrations in the future
- **FR-037**: System MUST authenticate with exchange APIs using API key and secret stored securely in environment variables
- **FR-038**: System MUST support futures market orders with configurable leverage (5x to 40x) for all trading actions
- **FR-039**: System MUST support both long positions (buy_to_enter) and short positions (sell_to_enter) via futures contracts
- **FR-040**: System MUST query account balance, active positions, and liquidation prices before attempting to place orders to prevent insufficient funds errors and to provide context awareness
- **FR-041**: System MUST monitor liquidation prices for all open positions and include this information in LLM prompts for risk awareness
- **FR-042**: System MUST handle exchange API rate limits by throttling requests and respecting rate limit headers

**Safety and Risk Management**

- **FR-043**: System MUST enforce maximum position size limit per cryptocurrency (defined in configuration, not to exceed $1000 per position by default)
- **FR-044**: System MUST enforce maximum loss per trade limit ($100 default) and maximum loss per day limit ($500 default)
- **FR-045**: System MUST implement a kill switch mechanism that immediately cancels all pending orders and stops all trading activity
- **FR-046**: System MUST operate in paper trading mode by default, requiring explicit configuration flag to enable live trading
- **FR-047**: System MUST validate that real money trading flag is set before executing live trades, with operator confirmation required
- **FR-048**: System MUST implement rate limiting on order placement (maximum 10 orders per minute by default) to prevent runaway execution
- **FR-049**: System MUST enter safety lockdown mode if daily loss limit is reached, preventing new positions until the next day
- **FR-050**: System MUST calculate and monitor portfolio-level exposure and risk concentration across all active positions to prevent over-exposure to any single asset or correlated assets
- **FR-051**: System MUST monitor liquidation risk for leveraged positions and alert when liquidation price is within 10% of current price
- **FR-052**: System MUST validate leverage values are within allowed range (5x-40x) and enforce lower limits for higher-risk market conditions

### Key Entities _(include if feature involves data)_

- **TradingDecision**: Represents a single LLM-generated trading decision for one cryptocurrency. Attributes: timestamp, cryptocurrency symbol, action (buy_to_enter | sell_to_enter | hold | close_position), recommended quantity, profit target price, stop loss price, invalidation condition text, leverage multiplier, confidence score, risk USD, justification text (for entry/exit/close), LLM model identifier, prompt version, market data snapshot at decision time, account snapshot at decision time.

- **TradeExecution**: Represents an actual trade executed on the exchange. Attributes: execution timestamp, cryptocurrency symbol, order type (BUY/SELL), quantity, execution price, fees, exchange order ID, profit/loss, related TradingDecision ID.

- **AccountSnapshot**: Represents the account state at a specific moment. Attributes: timestamp, total account value, available cash, current total return percentage, Sharpe ratio, list of active positions, portfolio-level risk metrics, total unrealized PnL across all positions.

- **Position**: Represents an active trading position. Attributes: symbol, quantity, entry price, current price, liquidation price, unrealized PnL, leverage, exit plan (profit target price, stop loss price, invalidation condition text), confidence score, risk amount in USD, order IDs (stop loss order, take profit order, entry order), wait for fill status, notional value in USD, position open timestamp, position status (open/closed).

- **PromptVersion**: Represents a versioned prompt template. Attributes: version identifier, prompt text template, creation timestamp, active status, metadata (author, description, intended purpose).

- **MarketData**: Represents market conditions at a point in time. Attributes: timestamp, cryptocurrency symbol, OHLCV values (open, high, low, close, volume), calculated technical indicators (SMA values, RSI, MACD, etc.), external data source information.

- **LLMModel**: Represents a configured LLM model. Attributes: model identifier, provider (OpenAI, Anthropic), model name (gpt-4, claude-3-opus), API credentials reference, timeout configuration, cost per token, daily usage tracking.

- **DataSourcePlugin**: Represents an external data source. Attributes: plugin name, data source type (news, sentiment, on-chain), collection frequency, enabled status, last successful fetch timestamp, configuration parameters.

- **TradingSession**: Represents a continuous trading operation period. Attributes: session start timestamp, end timestamp, operating mode (paper/live), total trades executed, total profit/loss, active prompt version, active LLM model, configuration snapshot.

- **RiskLimit**: Represents safety constraints. Attributes: limit type (position size, loss per trade, loss per day, order rate), current value, maximum allowed value, currency, enforcement status, override permissions.

### Safety & Risk Requirements _(mandatory if trading logic involved)_

**Risk Controls** (per Constitution Principle VI):

- **RC-001**: Position size limits - Maximum $1000 USD equivalent per cryptocurrency position, enforced at code level before order placement, configurable via environment variable but hardcoded minimum of $10 and maximum of $10,000
- **RC-002**: Loss limits - Maximum $100 loss per trade (enforced via stop-loss or position size calculation) and maximum $500 total loss per day, enforced by safety lockdown mechanism that prevents new positions when threshold reached
- **RC-003**: Liquidation risk limits - System MUST reject any trade where liquidation price would be within 10% of current market price, enforced before order placement, leverage automatically reduced if risk exceeds threshold
- **RC-004**: Leverage limits - Minimum 5x, maximum 40x leverage enforced at code level, default conservative leverage (10x) for new positions, higher leverage requires explicit justification in LLM reasoning
- **RC-005**: Kill switch - Emergency shutdown mechanism accessible via configuration file flag (`kill_switch_enabled=true` in designated file), API endpoint, or command-line signal, immediately cancels all orders and freezes trading state
- **RC-006**: Dry-run mode - System defaults to paper trading mode on first run, requiring explicit `ENABLE_LIVE_TRADING=true` environment variable AND operator confirmation command to activate real money trading, with warning messages displayed before live mode activation
- **RC-007**: Rate limiting - Maximum 10 orders per minute to prevent runaway execution, enforced by order placement throttle with exponential backoff if limit approached, configurable but hardcoded maximum of 60 orders per minute to prevent exchange API bans

**LLM Safety** (per Constitution Principle V, if LLM used):

- **LS-001**: Timeout limits - 30 seconds maximum per LLM API call, enforced at HTTP client level, with automatic retry once if timeout occurs, then skip cycle if second attempt times out
- **LS-002**: Cost controls - $10 USD daily budget for LLM API calls by default, tracked via token usage counter, trading paused automatically if budget exceeded until next day, configurable via `LLM_DAILY_BUDGET` environment variable
- **LS-003**: Response validation - LLM responses must be parseable JSON object with cryptocurrency symbols as keys, each value must contain required fields (coin, signal, quantity, profit_target, stop_loss, invalidation_condition, leverage, confidence, risk_usd), signal must be valid action type (buy_to_enter | sell_to_enter | hold | close_position), justification text required for entry/exit/close actions (minimum 10 characters), confidence score must be between 0 and 1, leverage must be integer between 5 and 40
- **LS-004**: Fallback strategy - If LLM call fails or times out, log error and skip current trading cycle, continue with next scheduled cycle, if 3 consecutive failures occur, enter HOLD-only safety mode (no new positions, only allow closing existing positions), send alert notification to operator

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Trading decisions occur automatically at configured intervals without operator intervention (system uptime >99% excluding maintenance windows)
- **SC-002**: LLM generates decisions for multiple cryptocurrencies in a single API call (batch decision), with decisions for all monitored cryptocurrencies (typically 6-10 coins) returned in one JSON response
- **SC-003**: Every trading decision includes clear justification text for entry/exit/close actions, with justification text averaging at least 50 words for non-hold decisions
- **SC-004**: LLM prompts include complete account context (available cash, account value, total return, Sharpe ratio) and all active positions with their full details (entry price, current price, liquidation price, PnL, leverage, exit plans including invalidation conditions, risk amounts) for 100% of trading cycles
- **SC-005**: System correctly parses and validates LLM JSON responses containing multiple coin decisions with all required fields (coin, signal, quantity, profit_target, stop_loss, invalidation_condition, leverage, confidence, risk_usd, justification) for 99%+ of trading cycles
- **SC-006**: System operates in paper trading mode successfully for minimum 7 consecutive days without crashes, executing at least 100 trading cycles during validation period
- **SC-007**: All trades and decisions are logged with complete audit trail, with 100% of trading decisions retrievable from logs including timestamp, action type, reasoning, outcome, and account snapshot at decision time
- **SC-008**: Leverage trading executes correctly for both long (buy_to_enter) and short (sell_to_enter) positions with specified leverage (5x-40x), with 100% of leveraged orders placed successfully when risk checks pass
- **SC-009**: Liquidation prices are calculated and monitored for all leveraged positions, with alerts triggered when liquidation risk exceeds threshold (within 10% of current price)
- **SC-010**: Safety mechanisms prevent losses exceeding configured limits, with 0 instances of daily loss limit being exceeded undetected, and 0 instances of liquidation occurring due to insufficient monitoring
- **SC-011**: Prompt version changes take effect within one trading cycle (typically <5 minutes) without requiring system restart
- **SC-012**: LLM decision generation completes within 30 seconds for 95% of multi-coin requests, with remaining 5% handled gracefully via timeout mechanism
- **SC-013**: System survives exchange API outages by pausing trading and automatically resuming when connectivity restores, with recovery time <60 seconds after API availability confirmed
- **SC-014**: Operator can switch between different LLM models (GPT-4, Claude) and observe decision differences within a single trading session
- **SC-015**: Kill switch activation stops all trading activity within 5 seconds, with 0 orders executed after kill switch triggered
- **SC-016**: Audit logs support performance analysis queries, with ability to retrieve decisions by prompt version, time range, outcome, or action type in <2 seconds for 1000 decisions
- **SC-017**: System operates within LLM cost budget, with actual daily LLM costs not exceeding configured budget by more than 5% (accounting for request timing edge cases)
- **SC-018**: Account information refresh (querying balances, positions, and liquidation prices) completes within 5 seconds for 99% of cycles to avoid blocking trading decisions

### Assumptions

- **Assumption 1**: Operator has obtained valid API credentials from Binance Futures exchange with futures trading permissions and sufficient margin balance
- **Assumption 2**: Operator has obtained valid API keys for LLM providers (OpenAI and/or Anthropic) with sufficient quota for trading frequency
- **Assumption 3**: Trading environment has reliable internet connectivity with latency <500ms to exchange APIs
- **Assumption 4**: Operator accepts that LLM trading decisions are probabilistic and past performance does not guarantee future results
- **Assumption 5**: Operator understands leverage trading risks including liquidation risk, amplified losses, and the need for active position monitoring
- **Assumption 6**: Initial deployment will monitor 6-10 major cryptocurrency pairs that have sufficient liquidity for futures trading (e.g., BTC, ETH, BNB, SOL, XRP, DOGE)
- **Assumption 7**: Operator will monitor paper trading results for minimum 7 days and validate leverage calculations before enabling live trading mode
- **Assumption 8**: Market data from exchange APIs is sufficiently accurate and timely for trading decisions (no separate data vendor required initially)
- **Assumption 9**: LLM is capable of understanding and correctly interpreting invalidation conditions expressed in natural language (e.g., "price closes below X on 3-minute candle", "4-hour MACD crosses below Y")
- **Assumption 10**: Trading occurs during normal market conditions, not during extreme volatility events or exchange outages (circuit breakers may need manual intervention)
- **Assumption 11**: Operator understands that cryptocurrency trading with leverage involves significant financial risk and the system's safety mechanisms reduce but do not eliminate risk of loss or liquidation
- **Assumption 12**: Regulatory compliance for automated futures trading in operator's jurisdiction is the operator's responsibility, not provided by the system
