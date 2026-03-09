"""
Delivery Dash - Order Profitability Calculator

Run:
    python app.py

The app listens on:
    http://127.0.0.1:5000
"""

from __future__ import annotations

from dataclasses import dataclass
from html import escape
from typing import Dict, Tuple

from flask import Flask, request

app = Flask(__name__)


@dataclass
class OrderInputs:
    """Validated order input values used for calculations."""

    vehicle_name: str
    fuel_type: str
    mpg: float
    gas_price: float
    payout: float
    distance_miles: float
    time_minutes: float


def calculate_order_metrics(
    distance_miles: float,
    mpg: float,
    gas_price: float,
    payout: float,
    time_minutes: float,
) -> Dict[str, float]:
    """Calculate fuel cost, net pay, and hourly earnings for a delivery order."""
    fuel_cost = (distance_miles / mpg) * gas_price
    net_pay = payout - fuel_cost
    hours = time_minutes / 60
    pay_per_hour_gross = payout / hours
    pay_per_hour_net = net_pay / hours
    return {
        "fuel_cost": fuel_cost,
        "net_pay": net_pay,
        "pay_per_hour_gross": pay_per_hour_gross,
        "pay_per_hour_net": pay_per_hour_net,
    }


def parse_and_validate(form_data: Dict[str, str]) -> Tuple[OrderInputs | None, str | None]:
    """Validate submitted form data and return structured inputs or an error message."""
    vehicle_name = (form_data.get("vehicle_name") or "").strip()
    fuel_type = (form_data.get("fuel_type") or "").strip().lower()

    if not vehicle_name:
        return None, "Please enter a vehicle name."
    if fuel_type != "gas":
        return None, "Fuel type must be gas for now."

    def parse_number(field_name: str, label: str, min_exclusive: bool = False) -> float:
        raw = (form_data.get(field_name) or "").strip()
        if raw == "":
            raise ValueError(f"Please enter {label}.")
        try:
            value = float(raw)
        except ValueError as exc:
            raise ValueError(f"{label} must be a valid number.") from exc
        if min_exclusive and value <= 0:
            raise ValueError(f"{label} must be greater than 0.")
        if not min_exclusive and value < 0:
            raise ValueError(f"{label} cannot be negative.")
        return value

    try:
        mpg = parse_number("mpg", "MPG", min_exclusive=True)
        gas_price = parse_number("gas_price", "Gas price per gallon")
        payout = parse_number("payout", "Estimated payout")
        distance_miles = parse_number("distance_miles", "Estimated trip distance")
        time_minutes = parse_number("time_minutes", "Estimated total time", min_exclusive=True)
    except ValueError as err:
        return None, str(err)

    return (
        OrderInputs(
            vehicle_name=vehicle_name,
            fuel_type=fuel_type,
            mpg=mpg,
            gas_price=gas_price,
            payout=payout,
            distance_miles=distance_miles,
            time_minutes=time_minutes,
        ),
        None,
    )


def as_currency(value: float) -> str:
    """Format a number as USD currency."""
    return f"${value:,.2f}"


def render_page(
    values: Dict[str, str],
    error: str | None = None,
    results: Dict[str, str] | None = None,
) -> str:
    """Render full HTML page for input form, errors, and optional results."""

    def v(name: str, default: str = "") -> str:
        return escape(values.get(name, default), quote=True)

    error_block = (
        f'<div class="error" role="alert">{escape(error)}</div>' if error else ""
    )

    results_block = ""
    if results:
        results_block = f"""
        <section class="card result-card" aria-live="polite">
            <h2>Order Results</h2>
            <div class="result-row"><span>Fuel cost</span><strong>{results['fuel_cost']}</strong></div>
            <div class="result-row"><span>Net pay</span><strong>{results['net_pay']}</strong></div>
            <div class="result-row"><span>Pay/hour before fuel</span><strong>{results['pay_per_hour_gross']}</strong></div>
            <div class="result-row highlight"><span>Pay/hour after fuel</span><strong>{results['pay_per_hour_net']}</strong></div>
        </section>
        """

    return f"""
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Delivery Dash</title>
  <style>
    :root {{
      color-scheme: light dark;
      --bg: #0f172a;
      --card: #111827;
      --text: #e5e7eb;
      --muted: #94a3b8;
      --border: #334155;
      --accent: #22c55e;
      --danger: #ef4444;
    }}
    @media (prefers-color-scheme: light) {{
      :root {{
        --bg: #f1f5f9;
        --card: #ffffff;
        --text: #0f172a;
        --muted: #475569;
        --border: #e2e8f0;
      }}
    }}
    * {{ box-sizing: border-box; }}
    body {{
      margin: 0;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
      display: flex;
      justify-content: center;
      padding: 16px;
    }}
    main {{ width: 100%; max-width: 520px; display: grid; gap: 12px; }}
    h1 {{ margin: 0 0 6px; font-size: 1.4rem; }}
    p.sub {{ margin: 0 0 10px; color: var(--muted); font-size: 0.95rem; }}
    .card {{
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 14px;
      box-shadow: 0 3px 10px rgba(0, 0, 0, 0.15);
    }}
    label {{ display: block; font-size: 0.9rem; margin-bottom: 6px; }}
    input, select, button {{
      width: 100%;
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 10px 11px;
      font-size: 1rem;
      background: transparent;
      color: var(--text);
    }}
    .grid {{ display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }}
    .field {{ margin-bottom: 10px; }}
    .full {{ grid-column: 1 / -1; }}
    button {{
      background: var(--accent);
      color: #06280f;
      font-weight: 700;
      border: none;
      margin-top: 6px;
      cursor: pointer;
    }}
    .error {{
      background: rgba(239, 68, 68, 0.14);
      border: 1px solid var(--danger);
      color: #fecaca;
      border-radius: 10px;
      padding: 10px;
      font-weight: 600;
    }}
    .result-card h2 {{ margin: 0 0 10px; font-size: 1.15rem; }}
    .result-row {{ display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px dashed var(--border); }}
    .result-row:last-child {{ border-bottom: none; }}
    .highlight {{ font-size: 1.1rem; font-weight: 800; color: var(--accent); }}
  </style>
</head>
<body>
  <main>
    <section class="card">
      <h1>Delivery Dash</h1>
      <p class="sub">Estimate true hourly pay after fuel before accepting an order.</p>
      {error_block}
      <form method="post" action="/calculate" novalidate>
        <div class="grid">
          <div class="field full">
            <label for="vehicle_name">Vehicle Name</label>
            <input id="vehicle_name" name="vehicle_name" type="text" required value="{v('vehicle_name', 'My Car')}">
          </div>

          <div class="field">
            <label for="fuel_type">Fuel Type</label>
            <select id="fuel_type" name="fuel_type" required>
              <option value="gas" {'selected' if v('fuel_type', 'gas') == 'gas' else ''}>Gas</option>
            </select>
          </div>

          <div class="field">
            <label for="mpg">MPG</label>
            <input id="mpg" name="mpg" type="number" step="0.01" min="0.01" required value="{v('mpg', '30')}">
          </div>

          <div class="field">
            <label for="gas_price">Gas Price / Gallon ($)</label>
            <input id="gas_price" name="gas_price" type="number" step="0.01" min="0" required value="{v('gas_price', '3.50')}">
          </div>

          <div class="field">
            <label for="payout">Estimated Payout ($)</label>
            <input id="payout" name="payout" type="number" step="0.01" min="0" required value="{v('payout', '8.50')}">
          </div>

          <div class="field">
            <label for="distance_miles">Trip Distance (miles)</label>
            <input id="distance_miles" name="distance_miles" type="number" step="0.01" min="0" required value="{v('distance_miles', '5')}">
          </div>

          <div class="field">
            <label for="time_minutes">Total Time (minutes)</label>
            <input id="time_minutes" name="time_minutes" type="number" step="0.01" min="0.01" required value="{v('time_minutes', '25')}">
          </div>
        </div>
        <button type="submit">Calculate True Hourly Pay</button>
      </form>
    </section>
    {results_block}
  </main>
</body>
</html>
"""


@app.get("/")
def index() -> str:
    defaults = {
        "vehicle_name": "My Car",
        "fuel_type": "gas",
        "mpg": "30",
        "gas_price": "3.50",
        "payout": "8.50",
        "distance_miles": "5",
        "time_minutes": "25",
    }
    return render_page(defaults)


@app.post("/calculate")
def calculate() -> str:
    form_values = {
        "vehicle_name": request.form.get("vehicle_name", ""),
        "fuel_type": request.form.get("fuel_type", "gas"),
        "mpg": request.form.get("mpg", ""),
        "gas_price": request.form.get("gas_price", ""),
        "payout": request.form.get("payout", ""),
        "distance_miles": request.form.get("distance_miles", ""),
        "time_minutes": request.form.get("time_minutes", ""),
    }

    parsed, error = parse_and_validate(form_values)
    if error:
        return render_page(form_values, error=error), 400

    assert parsed is not None
    metrics = calculate_order_metrics(
        distance_miles=parsed.distance_miles,
        mpg=parsed.mpg,
        gas_price=parsed.gas_price,
        payout=parsed.payout,
        time_minutes=parsed.time_minutes,
    )
    formatted = {
        "fuel_cost": as_currency(metrics["fuel_cost"]),
        "net_pay": as_currency(metrics["net_pay"]),
        "pay_per_hour_gross": as_currency(metrics["pay_per_hour_gross"]),
        "pay_per_hour_net": as_currency(metrics["pay_per_hour_net"]),
    }
    return render_page(form_values, results=formatted)


if __name__ == "__main__":
    app.run(host="127.0.0.1", port=5000, debug=False)
