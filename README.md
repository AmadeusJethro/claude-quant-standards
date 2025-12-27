# claude-quant-standards

A Claude Code skill with **active enforcement** for quant trading workflows. Not just guidelines - it stops, warns, and auto-fixes.

## What it does

This skill actively enforces standards while you code:

**STOPS you from:**
- Lookahead bias (position without .shift(1), wrong resample labels)
- Committing with failing tests or type errors
- Hardcoded secrets
- Bare except clauses
- O(n^2) loops when O(n) is trivial

**WARNS you about:**
- Functions >50 lines
- Float used for money (should be Decimal)
- Suspicious backtest results (Sharpe >2, win rate >70%)
- Missing type hints

**AUTO-FIXES:**
- Missing .shift(1) on signal->position patterns
- Wrong resample labels (left->right)
- Unused imports
- Import ordering

**AUTO-DOCUMENTS:**
- Decisions -> docs/DECISIONS.md
- Experiments -> docs/EXPERIMENTS.md
- Session state -> docs/STATE.md

## Installation

```bash
git clone https://github.com/AmadeusJethro/claude-quant-standards.git
cp -r claude-quant-standards/skills/coding-standards ~/.claude/skills/
```

Then append `CLAUDE.md.append` contents to your `~/.claude/CLAUDE.md`.

## Auto-activation

Activates automatically on all coding work. No manual invocation needed.

## Core Rules

### Lookahead Bias Prevention

```python
# STOPPED - position uses current signal
df['position'] = df['signal']

# REQUIRED - shift by 1
df['position'] = df['signal'].shift(1).fillna(0)

# STOPPED - wrong resample
df.resample('D', label='left')

# REQUIRED - point-in-time correct
df.resample('D', label='right')
```

### Performance

```python
# STOPPED - Python loop over DataFrame
for i in range(len(df)):
    df.loc[i, 'result'] = df.loc[i, 'a'] * df.loc[i, 'b']

# REQUIRED - Vectorized
df['result'] = df['a'] * df['b']
```

### Backtest Red Flags

Results that get flagged:
- Sharpe > 2.0 - "Suspicious - verify not overfit"
- Returns > 50% with MaxDD < 10% - "Too good - check methodology"
- Win rate > 70% - "Often indicates lookahead"

### Required Strategy Tests

- `test_no_lookahead_bias`
- `test_survivorship_bias`
- `test_transaction_costs_material`
- `test_parameter_stability`

## Workflow

```
Explore -> Plan -> Code -> Test -> Review -> Commit
```

Skip Explore/Plan? Claude stops and makes you go back.

## Clean Code Enforcement

- Functions: max 50 lines
- Files: max 500 lines
- Nesting: max 3 levels
- Parameters: max 4 per function
- DRY: copying >3 lines = extract to function

## License

MIT
