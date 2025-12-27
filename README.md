# claude-quant-standards

A Claude Code skill for quantitative trading and backtesting workflows. Prevents common pitfalls like lookahead bias, enforces TDD, and maintains code quality standards.

## What it does

- **Catches lookahead bias** - The #1 cause of false backtests. Enforces signal shifting before position calculation.
- **Enforces TDD** - Write failing tests first. Required test patterns for strategies.
- **Numerical precision** - Decimal for money, never float. Catches compounding errors.
- **Flags suspicious results** - Sharpe > 2? Returns > 50% with < 10% drawdown? Gets flagged.
- **Auto-scaffolds projects** - Creates standard directory structure silently.
- **Documents decisions** - Maintains DECISIONS.md, EXPERIMENTS.md, STATE.md automatically.

## Installation

### Option 1: Copy the skill folder

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/claude-quant-standards.git

# Copy to your Claude skills directory
cp -r claude-quant-standards/skills/quant-coding-standards ~/.claude/skills/
```

### Option 2: Add to your CLAUDE.md

Append the contents of `CLAUDE.md.append` to your `~/.claude/CLAUDE.md` file.

## Auto-activation

The skill auto-activates when Claude detects:
- Keywords: trading, backtest, strategy, signal, position, portfolio, returns, sharpe, drawdown
- Directories: strategies/, backtest/, signals/, risk/

No manual invocation needed.

## What it enforces

### Lookahead Bias Prevention

```python
# WRONG - uses today's signal for today's position
df['position'] = df['close'] > df['close'].rolling(20).mean()

# CORRECT - shift signal by 1
df['signal'] = df['close'] > df['close'].rolling(20).mean()
df['position'] = df['signal'].shift(1).fillna(False).astype(int)
```

### Required Strategy Tests

Every strategy should have:
- `test_no_lookahead_bias` - signals only use data available at signal time
- `test_survivorship_bias` - universe includes delisted securities
- `test_transaction_costs_material` - costs meaningfully impact results
- `test_parameter_stability` - +/-10% param change doesn't swing Sharpe > 0.5

### Backtest Red Flags

Results that get questioned:
- Sharpe > 2.0
- Returns > 50% with drawdown < 10%
- Equity curve suspiciously smooth
- High parameter sensitivity

### Project Structure

```
src/
├── domain/           # Pure logic, NO I/O
│   ├── signals/      # Stateless signal generation
│   ├── risk/         # Position sizing, limits
│   └── models/       # Dataclasses, types
├── infrastructure/   # External interfaces
├── backtest/         # Event-driven engine
└── strategies/       # Strategy implementations
```

## Workflow

```
Explore → Plan → Code → Validate → Commit
```

Never skip Explore and Plan. Skipping causes patchwork code and tech debt.

## License

MIT
