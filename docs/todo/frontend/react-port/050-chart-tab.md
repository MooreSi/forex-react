# 050 — Chart tab

**Money:** no **Depends on:** 040 **Layer:** `backend/src/api/routers/chart.py`, `frontend/src/components/chart/`

## Scope

`frontend/pages/chart/` is 866 lines: the candle chart, EMA/RSI overlays, and a trades panel
that draws open positions onto it. It reaches `chart_controller` for `get_active_trader`,
`get_open_trades`, `get_risk_settings`, `ema_series` and `rsi_series`.

No money: the tab displays, it does not act. That is why it goes first.

## Decision

- `GET /api/chart/candles?symbol=&timeframe=&count=` → OHLC rows.
- `GET /api/chart/overlays?...` → EMA and RSI series from the same controller functions. One
  endpoint, not two: the overlays are drawn on the same candles and two endpoints means two
  fetches that can disagree about the window.
- `GET /api/chart/trades` → open trades with entry, SL, TP levels for the overlay.
- Rendering: **lightweight-charts**. It is the library built for this, it is ~45KB, and the
  alternative is re-deriving candle rendering in SVG.

## Tests first (TDD)

- `tests/api/routers/test_chart.py` — each endpoint forwards to the named controller function
  with the arguments it was given, and the response model does not reshape numbers. Negative
  control: a sentinel controller that records the call proves the forward actually happens.
- `ChartPanel.test.tsx` — renders candles from a fixture; an empty series renders the empty
  state, not a blank box.
