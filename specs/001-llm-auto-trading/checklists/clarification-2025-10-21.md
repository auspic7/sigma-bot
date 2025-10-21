# Specification Clarification: LLM Multi-Position Management & Leverage Trading

**Clarification Date**: 2025-10-21
**Feature**: LLM-Based Automated Trading System
**Update Type**: Major Enhancement - Multi-Position Management, Leverage Trading, Structured Output
**Trigger**: User question about LLM agent's complex reasoning capabilities

## Context

User provided an example of LLM reasoning that demonstrated sophisticated capabilities:

- Simultaneous management of 6 positions (ETH, SOL, XRP, BTC, DOGE, BNB)
- Direct evaluation of invalidation conditions for each position
- Structured JSON output with specific fields
- Leverage trading with liquidation prices
- Step-by-step reasoning process

The existing specification was insufficient to support these capabilities, requiring significant clarification.

## Questions & Answers

### Q1: LLM Output Structure and Action Types

**Question**: What specific action types and JSON structure should the LLM output?

**User Selection**: **Option A** - Explicitly define action types and JSON schema

**Decision**:

- **Action types defined**:

  - `buy_to_enter`: Open new long position
  - `sell_to_enter`: Open new short position (futures)
  - `hold`: Maintain existing position
  - `close_position`: Exit existing position

- **JSON structure defined** (FR-013):
  - coin (string)
  - signal (action type)
  - quantity (float)
  - profit_target (float)
  - stop_loss (float)
  - invalidation_condition (string)
  - leverage (integer, 5-40)
  - confidence (float, 0-1)
  - risk_usd (float)
  - justification (string, required for entry/exit/close, not for hold)

### Q2: Multi-Position Simultaneous Management

**Question**: Should LLM handle multiple coins in a single call?

**User Selection**: **Option A** - Multi-coin decision in one API call

**Decision**:

- LLM returns decisions for ALL monitored cryptocurrencies in single JSON response
- Output format: `{"ETH": {...}, "SOL": {...}, "BTC": {...}, ...}`
- More cost-efficient (one API call vs multiple)
- Enables portfolio-level coherent decision making
- Typical: 6-10 cryptocurrencies per call

### Q3: Invalidation Condition Evaluation Responsibility

**Question**: Who evaluates invalidation conditions - LLM or system?

**User Selection**: **Option A** - LLM evaluates directly

**Decision**:

- LLM receives invalidation condition text for each position (e.g., "If price closes below 3800 on a 3-minute candle")
- LLM receives current market price
- LLM performs the comparison and logical evaluation
- LLM decides hold vs close_position based on evaluation
- Rationale: Allows flexible natural language conditions, LLM can interpret complex conditions (e.g., "4-hour MACD crosses below -80")

### Q4: Leverage and Futures Trading Support

**Question**: Should the system support leveraged trading?

**User Selection**: **Option A** - Full leverage/futures support

**Decision**:

- Binance Futures API integration (FR-036)
- Leverage range: 5x to 40x (FR-038, FR-052)
- Support long positions (buy_to_enter) and short positions (sell_to_enter) (FR-039)
- Liquidation price monitoring mandatory (FR-041, FR-051)
- Liquidation risk limits: reject trades where liquidation price within 10% of current (RC-003)
- Default conservative leverage: 10x for new positions (RC-004)

### Q5: LLM Reasoning Process Requirements

**Question**: Should structured reasoning (Chain-of-Thought) be mandatory?

**User Selection**: **Option B** - Structured reasoning recommended but not mandatory

**Decision**:

- FR-021: Prompt templates SHOULD encourage step-by-step analysis
- Examples: "First check existing positions, then evaluate invalidation conditions, then determine actions"
- Not mandatory - flexibility for prompt engineering
- Minimum requirement: clear justification for entry/exit/close decisions (LS-003: min 10 characters)
- Allows cost optimization while maintaining quality

## Specification Changes Summary

### New/Updated Functional Requirements

**Core Trading Engine:**

- FR-005: Updated to specify 4 action types and multi-coin decision capability

**LLM Integration:**

- FR-013: Completely rewritten with detailed JSON schema specification for multi-coin responses

**Prompt Management:**

- FR-021: New requirement for structured reasoning encouragement (SHOULD, not MUST)

**Market Data Collection:**

- Renumbered FR-021 → FR-022 (and subsequent +1 due to FR-021 insertion)

**Exchange Integration:**

- FR-036: Updated from "Binance" to "Binance Futures" with leverage support
- FR-037: Updated authentication (kept same)
- FR-038: New - futures market orders with configurable leverage (5x-40x)
- FR-039: New - long and short position support via futures
- FR-040: Updated from balance-only to include positions and liquidation prices
- FR-041: New - liquidation price monitoring for all positions
- FR-042: Updated API rate limiting

**Safety and Risk Management:**

- Renumbered FR-043 through FR-050 (previously FR-040 through FR-047)
- FR-051: New - liquidation risk monitoring with 10% threshold alert
- FR-052: New - leverage validation and range enforcement

### Updated Entities

**TradingDecision:**

- Updated action types: buy_to_enter | sell_to_enter | hold | close_position (was BUY/SELL/HOLD)
- Added fields: profit target, stop loss, invalidation condition, leverage, risk USD, justification

**Position:**

- Already had leverage and liquidation_price (no changes needed)

### Updated Safety Requirements

**Risk Controls:**

- RC-003: New - Liquidation risk limits (reject if liquidation within 10% of current price)
- RC-004: New - Leverage limits (5x-40x range, default 10x)
- RC-005 → RC-007: Renumbered

**LLM Safety:**

- LS-003: Completely rewritten with detailed JSON validation requirements including multi-coin structure

### Updated Success Criteria

- SC-002: New - Multi-coin batch decisions in single API call
- SC-003: Updated - Justification for entry/exit/close (was generic reasoning)
- SC-004: Updated - Includes liquidation price in context
- SC-005: New - JSON parsing validation for multi-coin responses with all fields
- SC-008: New - Leverage trading execution for long/short with 5x-40x
- SC-009: New - Liquidation price monitoring and alerting
- SC-010: Updated - Includes liquidation prevention
- SC-012: Updated - Multi-coin request timeout
- SC-018: Updated - Includes liquidation prices in refresh

Added SC-002 through SC-018 (was SC-001 through SC-014)

### Updated Assumptions

- Assumption 1: Changed from generic Binance to Binance Futures with margin balance requirement
- Assumption 5: New - Leverage trading risk acknowledgment
- Assumption 6: Updated - 6-10 major pairs for futures trading (specific examples)
- Assumption 7: Updated - 7 days paper trading with leverage validation
- Assumption 9: New - LLM capability to interpret natural language invalidation conditions
- Assumption 11: Updated - Leverage risk and liquidation risk acknowledgment
- Assumption 12: Updated - Futures trading regulatory compliance

### Updated User Story 1 Acceptance Scenarios

Expanded from 8 to 11 scenarios:

- Scenario 1: Updated for Binance Futures and leveraged trades
- Scenario 2: Updated for multi-coin prompt with invalidation conditions
- Scenario 3: Updated for 6 active positions with complete details
- Scenario 4: NEW - LLM evaluates invalidation conditions
- Scenario 5: NEW - Multi-coin JSON parsing with all fields
- Scenario 6: Updated - buy_to_enter with leverage and liquidation check
- Scenario 7: NEW - sell_to_enter (short position) with leverage
- Scenario 8: NEW - close_position scenario
- Scenario 9: Updated - Includes liquidation risk check
- Scenario 10: Updated - 7 days validation and Binance Futures
- Scenario 11: (was 8) - LLM error handling

## Impact on Implementation

### High Impact Areas

1. **LLM Prompt Engineering**: Must construct prompts with all 6-10 cryptocurrencies, their positions, and invalidation conditions
2. **JSON Schema Validation**: Complex multi-level JSON parsing for dictionary of coin decisions
3. **Binance Futures API Integration**: Different API endpoints from spot trading, margin management
4. **Liquidation Monitoring**: Real-time calculation and alerting system
5. **Risk Management**: Enhanced checks for leverage limits and liquidation proximity

### API Changes Required

- Binance Futures REST API (not Spot API)
- Futures market order placement with leverage parameter
- Position query endpoints for liquidation prices
- Margin balance queries

### Data Model Implications

- TradingDecision storage must accommodate 4 action types
- Position entity already supports leverage and liquidation (good)
- Logs must track invalidation condition evaluations

### Testing Considerations

- Paper trading mode must simulate leverage and liquidation accurately
- Test multi-coin JSON parsing extensively (edge cases: missing coins, invalid fields)
- Test invalidation condition evaluation by LLM (accuracy, edge cases)
- Test liquidation risk calculations
- Test long and short position execution

## Validation Status

**Specification Completeness**: ✅ COMPLETE

All 5 clarification questions answered and integrated into specification:

- ✅ Action types and JSON structure defined (FR-005, FR-013, LS-003)
- ✅ Multi-coin decision support specified (FR-005, SC-002, SC-005)
- ✅ LLM evaluates invalidation conditions (User Story 1 Scenario 4, Assumption 9)
- ✅ Leverage/futures trading fully specified (FR-036-042, RC-003-004, SC-008-010)
- ✅ Structured reasoning recommended (FR-021)

**Requirements Testability**: ✅ ALL TESTABLE

- Multi-coin decision: Can verify JSON contains all monitored coins
- Action types: Can verify each decision has valid action type
- Invalidation evaluation: Can test with known conditions and prices
- Leverage execution: Can verify orders placed with correct leverage
- Liquidation monitoring: Can test alert triggers at 10% threshold

**No Ambiguities Remain**: ✅ CONFIRMED

All questions that arose from user example are now explicitly addressed in specification.

## Next Steps

1. ✅ Specification is complete and ready for `/speckit.plan` command
2. During planning phase, pay special attention to:
   - Binance Futures API integration complexity
   - Multi-coin prompt construction strategy (context window limits)
   - JSON schema validation library selection
   - Liquidation price calculation methodology
   - LLM prompt design for invalidation condition evaluation

## Files Modified

- `specs/001-llm-auto-trading/spec.md` - Major update
  - 52 functional requirements (was 47)
  - 7 risk controls (was 5)
  - 4 LLM safety requirements (updated LS-003)
  - 18 success criteria (was 14)
  - 12 assumptions (was 10)
  - 11 acceptance scenarios for User Story 1 (was 8)

## Commit Message

```
feat(spec): add multi-position management, leverage trading, and structured LLM output

Major enhancements based on user clarification of LLM agent capabilities:

Action Types & JSON Structure:
- Define 4 explicit action types: buy_to_enter, sell_to_enter, hold, close_position
- Specify complete JSON schema with 10 required fields per coin
- FR-013 rewritten with detailed multi-coin response format

Multi-Coin Decision Making:
- LLM returns decisions for ALL monitored coins in single API call
- Output format: {"ETH": {...}, "SOL": {...}, ...}
- More cost-efficient, enables portfolio-level coherent decisions
- SC-002, SC-005 added for multi-coin validation

Invalidation Condition Evaluation:
- LLM directly evaluates natural language invalidation conditions
- Receives condition text and current price, performs logical comparison
- User Story 1 Scenario 4 added for validation
- Assumption 9 acknowledges LLM interpretation capability

Leverage & Futures Trading:
- Binance Futures API integration (FR-036)
- Support 5x-40x leverage with long and short positions (FR-038, FR-039)
- Liquidation price monitoring and risk limits (FR-041, FR-051, RC-003)
- Default 10x leverage, reject if liquidation within 10% (RC-004)
- SC-008, SC-009, SC-010 for leverage execution and monitoring

Structured Reasoning:
- FR-021 encourages Chain-of-Thought via prompts (SHOULD, not MUST)
- Flexibility for prompt engineering and cost optimization

Requirements renumbered: FR-022+ (was FR-021+), FR-043+ (was FR-040+)
Success criteria expanded: SC-001 through SC-018 (was SC-001 through SC-014)
Assumptions updated: 12 assumptions (was 10)
User Story 1: 11 scenarios (was 8)

Ref: User clarification questions answered (5/5)
```
