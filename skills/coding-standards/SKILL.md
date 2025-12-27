# Coding Standards (ACTIVE ENFORCEMENT)

## Description
Active enforcement of clean code, performance, and quant standards. Auto-activates on all coding work. This skill DOES things - stops, fixes, warns, refactors.

---

## CLEAN CODE (Enforce Always)

### Before Writing ANY Code:
1. Search codebase for similar functionality - reuse/extend, don't duplicate
2. Plan abstraction - will this be reused? Keep it generic
3. Identify which file this belongs in - don't just add to whatever's open

### Function Rules (Enforce Hard):
- Max 50 lines - if longer, STOP and split before continuing
- Max 3 nesting levels - deeper = extract to helper
- Max 4 parameters - more = use config dataclass
- Single responsibility - one function does ONE thing
- Pure when possible - same inputs = same outputs, no side effects

### DRY (Actively Detect & Fix):
- Before writing, search for similar patterns
- Copying >3 lines? Extract to shared function instead
- Same logic in 2+ places? Immediately refactor to single source
- Similar functions? Consolidate with parameters

### Naming (Enforce):
- Functions: verb_noun (calculate_returns, validate_signal)
- Variables: descriptive (prices not p, returns not r)
- Loop vars: i/j/k fine
- Booleans: is_, has_, can_, should_ (is_valid, has_data)
- Constants: UPPER_SNAKE

### File Organization:
- Max 500 lines - if larger, split by responsibility
- One concept per file
- Imports at top, grouped: stdlib, third-party, local
- Public functions at top, private (_prefixed) below

### Code Smells (Detect & Flag):
- God functions (does too much)
- Dead code (unused functions/variables) - remove it
- Commented-out code - remove it
- Magic numbers - extract to named constants
- Deeply nested conditionals - extract or use early returns

---

## PERFORMANCE (Enforce for Data/Quant Code)

### Vectorization (Critical):
```python
# WRONG - Python loop over DataFrame
for i in range(len(df)):
    df.loc[i, 'result'] = df.loc[i, 'a'] * df.loc[i, 'b']

# CORRECT - Vectorized
df['result'] = df['a'] * df['b']
```

When you see iterrows(), itertuples(), or for loops over DataFrame indices - STOP and vectorize.

### Memory Efficiency:
- Large data? Use generators/chunking
- Avoid unnecessary DataFrame copies
- Use appropriate dtypes (float32 vs float64, category for strings)

### Big-O Awareness:
- Flag O(n^2) when O(n) is possible
- Flag repeated calculations inside loops - cache outside
- Flag DataFrame operations inside loops - vectorize

---

## QUANT-SPECIFIC (Enforce on Strategy/Backtest Code)

### Lookahead Bias (STOP on These):
```python
# STOP - position uses current signal
df['position'] = df['signal']

# REQUIRE - shift by 1
df['position'] = df['signal'].shift(1).fillna(0)

# STOP - wrong resample label
df.resample('D', label='left')

# REQUIRE - point-in-time correct
df.resample('D', label='right')
```

Also check:
- Economic data must use RELEASE date, not event date
- Rolling calculations must exclude current bar
- No same-day close for same-day entry

### Numerical Precision (WARN):
```python
# WARN - float for money
price = 100.50

# CORRECT - Decimal
from decimal import Decimal
price = Decimal('100.50')
```

### Backtest Red Flags (Auto-Flag):
- Sharpe > 2.0 - "Suspicious - verify not overfit"
- Returns > 50% with MaxDD < 10% - "Too good - check methodology"
- Win rate > 70% - "High win rate often indicates lookahead"

---

## ERROR HANDLING (Enforce)

- No bare `except:` - always catch specific exceptions
- No silent failures - log or raise, never swallow
- Fail fast with clear errors
- Validate inputs at function boundaries

```python
# WRONG
try:
    result = risky_operation()
except:
    pass

# CORRECT
try:
    result = risky_operation()
except SpecificError as e:
    logger.error(f"Operation failed: {e}")
    raise
```

---

## TESTING (Enforce)

### TDD Flow:
1. Write failing test
2. Implement minimum code to pass
3. Refactor while keeping green

### Required Tests for Strategies:
- Lookahead bias test
- Survivorship bias test
- Transaction costs impact test
- Parameter sensitivity test

---

## SECURITY (Enforce)

### STOP on:
- Hardcoded API keys, passwords, secrets
- Credentials in code (even commented)

### REQUIRE:
- Secrets from environment variables or gitignored config

---

## MULTI-PASS REVIEW

After writing >20 lines or core logic:
1. **Critique**: Review what you wrote - any issues?
2. **Improve**: Fix issues
3. **Verify**: Run tests

---

## GIT (Enforce)

- One logical change per commit
- Message: what + why
- Before commit: tests pass, types check, lint clean
- If any fail - STOP, fix first

---

## AUTO-DOCUMENTATION

### docs/DECISIONS.md - Append when:
- Choosing between approaches
- Architectural changes
- Parameter changes

### docs/EXPERIMENTS.md - Append when:
- Backtest completes
- Parameter sweep finishes

### docs/STATE.md - Update when:
- Session ending
- Context getting long

---

## AUTO-SCAFFOLD

When starting work, create if missing:
- docs/DECISIONS.md, docs/EXPERIMENTS.md, docs/STATE.md
- src/ with subdirs
- tests/ mirroring src/
- archive/ for deprecated code

Don't announce. Just do it.

---

## STOP vs WARN vs AUTO-FIX

### STOP (Refuse to Proceed):
- Lookahead bias patterns
- Commits with failing tests/types
- Bare except clauses
- Hardcoded secrets
- O(n^2) when O(n) is trivial

### WARN (Flag But Continue):
- Functions >50 lines
- Files >500 lines
- Float for money
- Suspicious backtest results

### AUTO-FIX:
- Missing .shift(1) on signal->position
- Resample label='left' to 'right'
- Remove unused imports

---

## WORKFLOW

### Explore - Plan - Code - Test - Review - Commit

1. **Explore**: Read files. What exists? Reuse?
2. **Plan**: State approach. >50 lines? Confirm first.
3. **Code**: Test first. Then implement. Small functions.
4. **Test**: All pass?
5. **Review**: Critique. Fix issues.
6. **Commit**: Only after checks pass.

Coding without Explore/Plan? STOP and go back.
