# Quant Trading Standards

## Description
Trading and backtesting verification patterns. Auto-activates on: trading, backtest, strategy, signal, position, portfolio, pnl, sharpe, drawdown work.

---

## STOP PATTERNS (Flag These)

### Lookahead Bias
```python
# BAD - uses future data
df['position'] = df['signal']           # No shift
df.resample('D', label='left')          # Wrong label
df['x'] = df['price'].shift(-1)         # Negative shift
df.iloc[i + 1]                          # Forward indexing
future_return = ...                     # Future naming

# GOOD
df['position'] = df['signal'].shift(1).fillna(0)
df.resample('D', label='right')
```

### Data Leakage
```python
# BAD - leaks train info to test
train_test_split(X, y, shuffle=True)    # Shuffles time series
KFold(shuffle=True)                     # Shuffles folds
scaler.fit(X)                           # Fits on all data
X.fillna(X.mean())                      # Global mean

# GOOD
train_test_split(X, y, shuffle=False)
TimeSeriesSplit(n_splits=5)
scaler.fit(X_train)
X_train.fillna(X_train.mean())
```

---

## WARN PATTERNS

### Suspicious Backtest Results
- Sharpe > 2.0 - likely overfit
- Win rate > 65% - check for lookahead
- Returns > 50% with MaxDD < 10% - too good
- OOS > IS performance - data leak
- Single feature > 40% importance - fragile

### Code Quality
- Float for money (use Decimal)
- Missing transaction costs
- 100% fill rate assumption

---

## STATISTICAL VALIDITY

Based on Harvey, Liu, Zhu (2016):

| Trials Tested | Required t-stat |
|---------------|-----------------|
| 1 | > 2.0 |
| 10 | > 2.5 |
| 100 | > 3.0 |
| 1000+ | > 3.78 |

The more strategies you test, the higher the bar for significance.

Additional requirements:
- If Sharpe > 1.5: Calculate Deflated Sharpe Ratio (must be > 0.95)
- If multiple variants: Calculate Probability of Backtest Overfitting (must be < 0.05)
- Document ALL variations tested, not just the winner

---

## EXECUTION REALISM

| Asset Class | Typical Slippage |
|-------------|------------------|
| Large-cap equity | 1-5 bps |
| Crypto major | 0.05-0.25% |
| Prediction markets | 1-3% |

Always include:
- Transaction costs
- Realistic fill rates (not 100% for limits)
- Market impact for large sizes

---

## REQUIRED TESTS

### test_no_lookahead_bias
```python
def test_no_lookahead_bias():
    """Shuffle future data - signals before cutoff must be identical."""
    df = generate_test_data()
    signals = strategy.generate_signals(df)

    df_shuffled = df.copy()
    df_shuffled.loc[df.index > cutoff, 'price'] = np.random.permutation(
        df_shuffled.loc[df.index > cutoff, 'price']
    )
    signals_shuffled = strategy.generate_signals(df_shuffled)

    pd.testing.assert_series_equal(signals[:cutoff], signals_shuffled[:cutoff])
```

### test_transaction_costs_material
```python
def test_transaction_costs_matter():
    """Costs should reduce returns by at least 10%."""
    results_no_cost = backtest(strategy, costs=0)
    results_with_cost = backtest(strategy, costs=REALISTIC_COSTS)

    impact = (results_no_cost.total_return - results_with_cost.total_return) / results_no_cost.total_return
    assert impact > 0.10, "Costs too small - check assumptions"
```

### test_parameter_stability
```python
def test_parameter_stability():
    """Small param changes shouldn't destroy performance."""
    base_sharpe = backtest(strategy, threshold=10).sharpe

    for delta in [-0.1, 0.1]:
        adjusted_sharpe = backtest(strategy, threshold=10*(1+delta)).sharpe
        change = abs(adjusted_sharpe - base_sharpe)
        assert change < 0.5, f"Sharpe swung {change:.2f} on {delta*100}% param change"
```

---

## WORKFLOW

1. **Before writing**: Search for existing similar code
2. **While writing**: Keep functions small, vectorize operations
3. **After writing**: Run tests, check for bias patterns
4. **Before trusting results**: Verify statistical validity

---

## DOCUMENTATION

After backtests, log to docs/EXPERIMENTS.md:
```markdown
## EXP-001: [Description] (YYYY-MM-DD)
Config: model version, params, data range, costs
Results:
| Metric | Value |
|--------|-------|
| Sharpe | X.XX |
| Win Rate | XX% |
| Max DD | -X% |
Red flags: [any from above]
Conclusion: [what we learned]
```
