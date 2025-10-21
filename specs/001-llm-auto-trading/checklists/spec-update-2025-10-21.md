# Specification Update Log: Account Context for LLM

**Update Date**: 2025-10-21
**Feature**: LLM-Based Automated Trading System
**Update Type**: Enhancement - Account Information Context
**Updated By**: User request for comprehensive account context in LLM prompts

## Update Summary

Enhanced the specification to explicitly include comprehensive account information and position details in the LLM context for trading decisions.

## Changes Made

### Functional Requirements

**Updated Requirements:**

- **FR-010** (expanded): Now explicitly lists all account context that must be included in LLM prompts:
  - Account performance metrics (total return %, Sharpe ratio)
  - Account financial status (available cash, current account value)
  - All active positions with complete details (symbol, quantity, entry/current/liquidation price, unrealized PnL, leverage, exit plan, confidence, risk USD, order IDs, notional value)
  - Portfolio-level risk metrics and exposure

**New Requirements:**

- **FR-011**: System MUST query and refresh account information and active positions at the start of each trading cycle before constructing the LLM prompt
- **FR-033** (updated): Added Sharpe ratio to performance metrics
- **FR-038** (updated): Expanded to query both account balance and active positions for context awareness
- **FR-047** (new): System MUST calculate and monitor portfolio-level exposure and risk concentration

**Renumbered Requirements:**

Due to additions, all requirements from FR-021 onwards were renumbered (+2 from original):

- Old FR-020 → New FR-021 (Market Data Collection start)
- Old FR-045 → New FR-046 (Safety lockdown)
- Added FR-047 (Portfolio exposure monitoring)

### Key Entities

**Updated Entities:**

- **TradingDecision**: Added "account snapshot at decision time" attribute to link each decision with the account state when it was made

**New Entities:**

- **AccountSnapshot**: Represents complete account state at a moment (timestamp, total account value, available cash, total return %, Sharpe ratio, list of active positions, portfolio risk metrics, total unrealized PnL)

- **Position**: Represents an active trading position with complete details matching the example provided (symbol, quantity, entry/current/liquidation price, unrealized PnL, leverage, exit plan with profit target/stop loss/invalidation condition, confidence score, risk USD, order IDs for SL/TP/entry, wait for fill status, notional value USD, timestamp, status)

### User Stories

**User Story 1 - Updated Acceptance Scenarios:**

- **Scenario 2** (expanded): Now explicitly describes that account information and active positions are queried and included in the LLM prompt construction along with market data

- **Scenario 3** (new): Added specific scenario for when bot has multiple active positions, ensuring prompt includes complete details (liquidation price, profit/loss targets, stop loss settings, confidence scores, risk amounts, notional values) for full portfolio awareness

- Renumbered subsequent scenarios (old 3-7 became 4-8)

### Success Criteria

**Updated Criteria:**

- **SC-003** (new): LLM prompts include complete account context (available cash, account value, total return, Sharpe ratio) and all active positions with their full details for 100% of trading cycles

- **SC-005** (updated): Added "account snapshot at decision time" to audit trail requirements

- **SC-014** (new): Account information refresh (querying balances and positions) completes within 5 seconds for 99% of cycles

- Renumbered subsequent criteria due to insertions

## Validation

### Completeness Check

- ✅ All account information fields from user example are represented in FR-010
- ✅ Position details match the structure provided (symbol, quantity, prices, PnL, leverage, exit plan, confidence, risk, order IDs, notional value)
- ✅ Performance metrics included (Sharpe ratio, total return)
- ✅ Account financial status included (available cash, account value)
- ✅ Acceptance scenarios validate the behavior
- ✅ Success criteria ensure measurable outcomes
- ✅ Key entities properly model the data structures

### Requirements Traceability

| User Request Element                                                   | Specification Location                           |
| ---------------------------------------------------------------------- | ------------------------------------------------ |
| Current Total Return %                                                 | FR-010, AccountSnapshot entity, SC-003           |
| Available Cash                                                         | FR-010, AccountSnapshot entity, SC-003           |
| Current Account Value                                                  | FR-010, AccountSnapshot entity, SC-003           |
| Sharpe Ratio                                                           | FR-010, FR-033, AccountSnapshot entity, SC-003   |
| Position: symbol, quantity                                             | FR-010, Position entity                          |
| Position: entry_price, current_price                                   | FR-010, Position entity                          |
| Position: liquidation_price                                            | FR-010, Position entity, User Story 1 Scenario 3 |
| Position: unrealized_pnl                                               | FR-010, Position entity, User Story 1 Scenario 3 |
| Position: leverage                                                     | FR-010, Position entity, User Story 1 Scenario 3 |
| Position: exit_plan (profit_target, stop_loss, invalidation_condition) | FR-010, Position entity, User Story 1 Scenario 3 |
| Position: confidence                                                   | FR-010, Position entity, User Story 1 Scenario 3 |
| Position: risk_usd                                                     | FR-010, Position entity, User Story 1 Scenario 3 |
| Position: order IDs (sl_oid, tp_oid, entry_oid)                        | FR-010, Position entity                          |
| Position: wait_for_fill                                                | Position entity                                  |
| Position: notional_usd                                                 | FR-010, Position entity, User Story 1 Scenario 3 |

## Impact Assessment

### Affected Components (for future implementation)

1. **LLM Prompt Constructor**: Must be updated to query and include account context
2. **Exchange API Client**: Must support querying account balance and active positions
3. **Data Models**: New AccountSnapshot and Position entities required
4. **Logging System**: TradingDecision logs must include account snapshot
5. **Performance Metrics Calculator**: Must calculate and track Sharpe ratio
6. **Risk Monitor**: Must calculate portfolio-level exposure (FR-047)

### Backward Compatibility

- ✅ No breaking changes to existing requirements
- ✅ Additions are extensions, not modifications
- ✅ Renumbering is administrative, not semantic

## Approval Status

- ✅ User confirmed: Option 1 (update existing spec rather than create new feature)
- ✅ Specification updated and validated
- ✅ No linter errors
- ✅ All requirements testable and unambiguous
- ✅ Ready for planning phase

## Next Steps

1. Specification is complete and ready for `/speckit.plan` command
2. During planning, ensure technical design addresses:
   - Efficient account data querying (FR-011, SC-014: <5 seconds)
   - Position data structure matching exchange API response format
   - Sharpe ratio calculation methodology
   - Portfolio exposure risk calculation algorithm (FR-047)
   - Prompt construction strategy to handle variable number of positions without exceeding LLM context limits
