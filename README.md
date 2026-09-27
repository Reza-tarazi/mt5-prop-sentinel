# 🛡️ Prop Discipline Sentinel (MT5 to Telegram)

[![Platform: MetaTrader 5](https://img.shields.io/badge/Platform-MetaTrader%205-blue.svg)](https://www.metatrader5.com)
[![Language: MQL5](https://img.shields.io/badge/Language-MQL5-orange.svg)]()
[![Prop-Firm Safe: Read--Only](https://img.shields.io/badge/Prop--Firm-Safe%20(Read--Only)-green.svg)]()
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

A robust, **read-only** MetaTrader 5 Expert Advisor designed to enforce psychological discipline, dynamic risk governance, and prevent drawdown breaches on Prop-Firm accounts via automated, real-time Telegram alerts.

---

## 📌 Architectural Overview & Key Features

- **Real-Time Stop-Loss Tampering Alerts:** Instantly tracks any manual stop-loss widening or removal via tick-level and transaction events (`OnTradeTransaction`), calculates the exact monetary risk delta in USD, and dispatches immediate warnings to Telegram.
- **Consecutive Loss Hard Stop:** Enforces a daily consecutive loss ceiling (e.g., maximum 2 consecutive losses) to neutralize tilt and overtrading cycles.
- **Mandatory Cooldown Enforcer (Revenge Alert):** Monitors trade execution timestamps and triggers instant alerts if a new position is initiated prior to the mandatory rest window (e.g., 10 minutes) following a loss.
- **Daily Drawdown & Profit Target Caps:** Continuously assesses equity fluctuation relative to the day-opening balance, delivering urgent shutdown notifications when the daily loss limit or profit target is hit.
- **Dynamic On-Chart HUD:** Displays server-aligned trading sessions (London/NY/Asia), real-time 14-period ATR volatility buffers, and precise contract-sized lot calculations for strict 0.5% risk models.
- **30-Second Pre-Trade Pop-Up Protocol:** Generates an immediate visual and acoustic pop-up prompt inside MetaTrader 5 upon order entry, demanding deliberate commitment before holding discretionary trades.

---

## 🚀 Quick Setup & Configuration

### 1. Enable WebRequests in MetaTrader 5
1. Inside MetaTrader 5, press `Ctrl + O` to open the **Options** window.
2. Navigate to the **Expert Advisors** tab.
3. Check **Allow WebRequest for listed URL**.
4. Double-click the green addition field and add:
   ```text
   [https://api.telegram.org](https://api.telegram.org)
