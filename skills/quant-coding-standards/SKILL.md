# Quant Coding Standards

## Description
Coding best practices for quantitative trading systems, backtesting, and financial calculations. Auto-activates for trading, backtest, strategy, signal, position, portfolio, returns, sharpe, drawdown work or when touching strategies/, backtest/, signals/, risk/ files.

---

## Workflow: Explore - Plan - Code - Validate - Commit

Never skip Explore and Plan. Skipping causes patchwork code and tech debt.

1. **Explore**: Read relevant files, map dependencies. No code yet.
2. **Plan**: If 2+ approaches exist, list pros/cons. Changes >50 lines need confirmation.
3. **Code**: TDD - write failing test first. Functions <50 lines.
4. **Validate**: Check lookahead bias, precision issues, edge cases (zero/negative/empty/overflow).
5. **Commit**: Tests pass, types check, descriptive message explaining "why."

---

## Lookahead Bias Prevention

The #1 cause of false backtests.

WRONG - uses today's signal for today's position:
```python
df['position'] = df['close'] > df['close'].rolling(20).mean()
```

CORRECT - shift signal by 1:
```python
df['signal'] = df['close'] > df['close'].rolling(20).mean()
df['position'] = df['signal'].shift(1).fillna(False).astype(int)
```

WRONG resampling:
```python
daily = close.resample('D', label='left').last()
```

CORRECT resampling:
```python
daily = close.resample('D', label='right').last()
```

Always verify:
- Economic data aligned to RELEASE date, not event date
- Rolling calculations exclude current bar
- No same-day close price for same-day entry decisions
- Prefer event-driven backtest architecture

---

## Numerical Precision

Float 0.1 = 0.100000001490116... Errors compound.

- Use Decimal for ALL monetary calculations, never float
- Type aliases: Price, Quantity, Amount = Decimal
- ROUND_HALF_EVEN for rounding
- Research speed: float64 OK but verify and never mix with Decimal

---

## Testing (TDD Required)

1. Write failing test specifying behavior
2. Implement minimum code to pass
3. Refactor keeping tests green

Required strategy tests:
- `test_no_lookahead_bias`: signals only use data available at signal time
- `test_survivorship_bias`: universe includes later-delisted securities
- `test_transaction_costs_material`: costs meaningfully impact results
- `test_parameter_stability`: +/-10% param change shouldn't swing Sharpe >0.5

Property-based tests (Hypothesis) for financial calcs:
- Position size bounded by capital
- Drawdown monotonically non-decreasing
- RSI in [0, 100]
- Quantities conserve after order matching

Never modify tests to make them pass.

---

## Code Structure

```
src/
├── domain/           # Pure logic, NO I/O
│   ├── signals/      # Stateless signal generation
│   ├── risk/         # Position sizing, limits
│   └── models/       # Dataclasses, types
├── infrastructure/   # External interfaces
│   ├── data/         # Abstract data handlers
│   ├── execution/    # Order execution
│   └── storage/      # DB, files
├── backtest/         # Event-driven engine
└── strategies/       # Strategy implementations
```

Size limits:
- Functions <50 lines
- Files <500 lines
- Prefer functions over classes unless behavior+state needed

---

## Refactoring Discipline

AI generates faster than it refactors. Counter this:
- Extract repeated logic (2+ times = function)
- Split functions >50 lines
- Consolidate duplicates
- One logical change per commit
- Run tests after EVERY change
- No big refactoring sprints - incremental only

---

## Backtest Red Flags

Question these results:
- Sharpe > 2.0 (suspicious)
- Returns > 50% with drawdown < 10% (very suspicious)
- Equity curve too smooth (unrealistic)
- Wild performance variance across similar periods
- High parameter sensitivity

---

## Self-Maintaining Documentation

After non-trivial decisions, append to docs/DECISIONS.md:
- DEC-XXX: Title, Date, Context, Options considered, Decision + why, Consequences

After backtests/experiments, append to docs/EXPERIMENTS.md:
- EXP-XXX: Hypothesis, Config (version/params/data/costs), Results table, Conclusion

Before ending long sessions, update docs/STATE.md:
- Current focus, recent changes, open questions, next session priorities

---

## Git Practices

- Commit early and often (safe revert points)
- Descriptive messages with "why" not just "what"
- One logical change per commit
- Branch per experiment: exp/new-strategy, feat/risk-limits

---

## Commands

```bash
pytest tests/ -v                    # Full suite
pytest tests/x.py::test_name -v     # Single test
ruff check --fix .                  # Lint + autofix
mypy --strict src/                  # Type check
```

---

## Style

- Python 3.11+, strict typing all functions
- 88-char lines (Black compatible)
- Pure functions preferred, isolate side effects at boundaries
