# Kalshi Weather Bot

> Automated weather market trader for Kalshi. Monitors NWS and NOAA forecasts, compares model output against current Kalshi weather market pricing, and enters positions when forecast data diverges from market consensus.

*Last updated: April 2026*

![preview_kalshi weather market trader](https://github.com/user-attachments/assets/7acdb902-bca1-43ec-954e-766439934697)

---

## What is Kalshi Weather Bot?

Kalshi Weather Bot pulls National Weather Service and NOAA forecast data, extracts the relevant probability values for each active Kalshi weather market, and compares official forecast probability against current Kalshi market pricing. When the gap exceeds the configured minimum, the bot enters a position in the direction of the official forecast.

Weather markets on Kalshi are often mispriced relative to official forecast data. This bot finds those gaps.

---

## Download

| Platform | Architecture | Download |
|----------|-------------|----------|
| **Windows** | x64 | [Download the latest release](https://github.com/elkghost666/kalshi-weather-bot/releases) |

---

## What It Monitors

| Data Source | Content | Update Frequency |
|-------------|---------|-----------------|
| **NWS forecasts** | Temperature, precipitation probability by city | 6 hours |
| **NOAA outlooks** | Extended range forecasts and anomaly maps | 12 hours |
| **Kalshi weather markets** | All active temperature and precipitation markets | Real-time |
| **Historical accuracy** | NWS forecast accuracy by region and horizon | Static |

---

## Engine Features

* **NWS API integration** - pulls current forecasts for all US cities with active Kalshi markets
* **Market matching** - maps NWS data to specific Kalshi weather market conditions
* **Forecast vs. price gap** - calculates edge as difference between NWS probability and Kalshi price
* **Forecast horizon weighting** - higher confidence for near-term forecasts, lower for 7-day
* **Historical accuracy adjustment** - applies regional NWS accuracy discount to edge score
* **Auto-entry on threshold** - places position when edge clears minimum
* **Position exit** - closes automatically before resolution date or on target price

---

## Two Ways to Run It

| | Windows App | Python Bot |
|---|---|---|
| **Setup** | Double-click | `pip install` + config |
| **Data source** | NWS + NOAA built-in | Configurable |
| **Entry** | Auto on threshold | Full control |
| **Config** | `config.toml` | Direct code access |
| **Alerts** | Dashboard + Telegram | JSON + Telegram |

## Quick Start

```
# 1. Download from Releases
# 2. Edit config.toml - set min edge threshold and Kalshi API key
# 3. Run Kalshi Weather Bot - forecast monitoring starts immediately
```

### Python

```bash
cd kalshi-weather-bot/python
pip install -r requirements.txt
python kalshi-weather-bot-v.1.0.7.py
```

---

## How It Works

![kalshi weather forecast pipeline](https://github.com/user-attachments/assets/8d4f56b3-2eec-44c9-bb85-05d0f5b7df50)

Four stages per update cycle:

1. **Fetch forecasts** - pulls latest NWS data for all US cities with active Kalshi weather markets
2. **Match** - maps forecast values to specific Kalshi market conditions
3. **Score** - calculates edge as gap between NWS probability and current Kalshi price
4. **Enter** - places position when edge clears minimum threshold

### Config Reference

```toml
[weather]
update_interval_min = 360
min_edge_score = 0.08
position_size_usd = 25
max_open_weather_positions = 5

[confidence]
horizon_1d_weight = 1.0
horizon_3d_weight = 0.75
horizon_7d_weight = 0.45

[filters]
min_kalshi_liquidity_usd = 150

[kalshi]
api_key = ""
api_secret = ""

[telegram]
bot_token = ""
chat_id = ""

[export]
weather_log_csv = "data/weather/weather_log.csv"
```

---

## Weather Trade Log Format

```json
{
  "trade_id": "wth_20260406_003",
  "market": "nyc-above-75f-april-15-2026",
  "city": "New York, NY",
  "nws_probability_yes": 0.72,
  "kalshi_yes_price": 0.55,
  "edge_score": 0.17,
  "forecast_horizon_days": 9,
  "confidence_adjusted_edge": 0.11,
  "position_size_usd": 25.00,
  "side_entered": "YES",
  "timestamp": "2026-04-06T06:00:00Z"
}
```

---

## Verified Live

![kalshi weather bot result](https://github.com/user-attachments/assets/bc4175b7-8c26-4962-aee9-7e3368c80c31)

**Configuration used:**
* Update every 6h, min edge 0.08, $25 per position

**Weather entry:**

| | Details |
|---|---|
| Market | nyc-above-75f-april-15-2026 |
| City | New York, NY |
| NWS probability YES | 0.72 |
| Kalshi YES price | 0.55 |
| Edge | 0.17 (adjusted 0.11 for 9-day horizon) |
| Position | YES $25.00 |
| Tx hash | 0x2a7e4c9f1b5d8a30e2b5d8f1c4a7e0b3c6f9d2a5e8b1d4f7c0a3e6c9f2b5d8a1 |

---

## Frequently Asked Questions

**What is Kalshi Weather Bot?**
Kalshi Weather Bot pulls NWS and NOAA forecast data and compares official probability values against current Kalshi weather market pricing. When official forecasts diverge from Kalshi prices, the bot enters a position in the forecast direction.

**What weather markets does Kalshi offer?**
Kalshi offers temperature threshold markets (will NYC be above X degrees on Y date), precipitation markets, and extreme weather event markets for major US cities.

**Why would Kalshi prices differ from NWS forecasts?**
Kalshi weather markets are priced by traders who may not follow NWS data closely, especially for smaller cities or specific date thresholds. Forecast updates often precede price corrections by hours.

**How is forecast confidence adjusted?**
Near-term forecasts (1-3 days) get full weight. 7-day forecasts get 45% weight because NWS accuracy drops significantly at longer horizons. The adjusted edge reflects realistic forecast reliability.

**Does it only cover US markets?**
Currently yes - the bot uses NWS/NOAA data which covers US cities. International weather markets would require additional data sources.

**What happens when a weather market resolves?**
The bot automatically closes open positions before the resolution date. Configure `exit_days_before_resolution` to set how early positions are closed.

---

## Use Cases

- **Kalshi weather trader** - systematic weather market entries based on official NWS forecast data
- **Kalshi temperature markets** - trade temperature threshold markets with forecast probability edge
- **Kalshi precipitation markets** - enter rain/snow markets when NWS diverges from Kalshi pricing
- **Prediction market weather** - forecast-driven entries on all active Kalshi weather markets
- **Kalshi NWS arbitrage** - capture gaps between official forecasts and market-implied probabilities

---

## Repository Structure

```
kalshi-weather-bot/
+-- kalshi-weather-bot-v.1.0.7.exe
+-- config.toml
+-- data/
|   +-- weather/
|   +-- forecasts/
|   +-- logs/
|   +-- dll/
+-- python/
|   +-- src/
|   |   +-- nws_fetcher.py
|   |   +-- matcher.py
|   |   +-- executor.py
|   +-- requirements.txt
+-- README.md
```

---

## Requirements

```
python-dotenv, typer[all], httpx, kalshi-python, pandas
```

* Kalshi account with API access
* NWS API (free, no key required)
* Telegram bot token (for alerts)

---

*Efficiency meets accuracy.*
