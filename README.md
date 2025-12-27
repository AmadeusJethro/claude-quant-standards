# claude-quant-standards

A Claude Code skill for quant trading and backtesting work.

## What this is

Behavioral guidance for Claude. When you're working on trading/backtest code, Claude reads this skill and tries to follow the patterns.

**What it does:**
- Flags lookahead bias patterns (position without shift, wrong resample labels)
- Warns on suspicious results (Sharpe > 2, win rate > 65%)
- Lists required strategy tests with copy-paste code
- Documents statistical validity thresholds (Harvey, Liu, Zhu 2016)

**What it doesn't do:**
- Actually block anything (that requires hooks, not included)
- Guarantee Claude follows it perfectly
- Replace proper code review

It's a starting point. Modify for your needs.

## Installation

```bash
git clone https://github.com/AmadeusJethro/claude-quant-standards.git
cp -r claude-quant-standards/skills/quant ~/.claude/skills/
```

Optionally append `CLAUDE.md.append` contents to your `~/.claude/CLAUDE.md`.

## Auto-activation

The skill auto-activates when Claude detects keywords: trading, backtest, strategy, signal, position, portfolio, pnl, sharpe, drawdown.

No manual invocation needed.

## What I actually use

This is the public version. My local setup also has:
- Python hooks that actually block bad patterns before code is written
- Additional coding standards skill
- Project-specific configs

The hooks are the real enforcement. This skill is just guidance.

## Core Patterns

### Lookahead Bias (the silent killer)

```python
# BAD
df['position'] = df['signal']

# GOOD
df['position'] = df['signal'].shift(1).fillna(0)
```

### Suspicious Results

If your backtest shows:
- Sharpe > 2.0
- Win rate > 65%
- Returns > 50% with MaxDD < 10%

...investigate before celebrating.

### Required Tests

Every strategy should have:
- `test_no_lookahead_bias` - shuffle future data, signals before cutoff must be identical
- `test_transaction_costs_material` - costs should impact results by > 10%
- `test_parameter_stability` - +/-10% param change shouldn't swing Sharpe > 0.5

## License

MIT
