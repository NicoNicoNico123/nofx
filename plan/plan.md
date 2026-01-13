# Multi-Agent Intelligent Trading Architecture Plan

This document outlines the transition of NOFX from a monolithic prompt architecture to a decentralized, tool-capable, multi-agent system.

## 1. Overview
The goal is to move away from sending all data in a single prompt. Instead, we will use a **Master Orchestrator** that coordinates specialized agents, uses tools for precise calculations, and incorporates a **Memory Agent** to learn from historical performance.

---

## 2. Architectural Evolution (The "Flow")

### 2.1 Current Flow (Monolithic "Single Brain")
1.  **Gather:** `kernel/engine.go` fetches all market, account, and indicator data upfront.
2.  **Prompt:** A single giant text block (the "wall of data") is sent to the AI.
3.  **Decide:** The AI processes everything in one go and returns a final decision.
4.  **Safety:** Hardcoded Go rules double-check the AI's math before execution.

### 2.2 Future Flow (Decentralized "Team of Experts")

```
┌─────────────────────────────────────────────────────────────────┐
│                    MASTER ORCHESTRATOR                           │
│                    (kernel/engine.go)                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1: Context Audit (The State)                              │
│  ─────────────────────────────────────────────────────────────  │
│  ├─ Fetch Account Balance                                        │
│  │  └─ Total Equity, Available Balance, Margin Used             │
│  ├─ Fetch Open Positions                                         │
│  │  └─ Symbol, Side, Entry Price, Unrealized PnL               │
│  └─ Fetch Recent Trade History                                   │
│     └─ 24 hours trade historys
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2: Reflection (Memory Agent)                              │
│  ─────────────────────────────────────────────────────────────  │
│  ├─ Log Decision Reasoning (Entry/Close Actions)                │
│  │  ├─ For each entry decision (open_long/open_short):          │
│  │  │  ├─ Log reasoning: Why enter this position?              │
│  │  │  │  ├─ Technical indicators (RSI, MACD, price action)    │
│  │  │  │  ├─ Market conditions (volatility, trend, volume)    │
│  │  │  │  ├─ Pattern recognition (breakout, reversal, etc.)   │
│  │  │  │  ├─ Risk/reward calculation                            │
│  │  │  │  └─ Confidence level and position sizing rationale     │
│  │  │  └─ Store in DecisionAction.reasoning field               │
│  │  └─ For each close decision (close_long/close_short):       │
│  │     ├─ Log reasoning: Why close this position?               │
│  │     │  ├─ Profit target reached                              │
│  │     │  ├─ Stop loss triggered                                │
│  │     │  ├─ Market condition changed (reversal signals)       │
│  │     │  ├─ Risk management (partial close, trailing stop)     │
│  │     │  └─ Time-based exit (holding too long)                 │
│  │     └─ Store in DecisionAction.reasoning field               │
│  ├─ Analyze Trade history with logged decision reasoning (last 50 trades) │
│  │  ├─ Review winning trades: Extract successful patterns       │
│  │  │  ├─ Entry conditions from decision logs (RSI, MACD, price action) │
│  │  │  ├─ Position sizing that worked                           │
│  │  │  ├─ Hold duration patterns                                │
│  │  │  ├─ Market conditions (volatility, trend)                 │
│  │  │  └─ Decision reasoning patterns that led to wins         │
│  │  └─ Review losing trades: Identify failure patterns          │
│  │     ├─ Common entry mistakes from decision logs (timing, indicators) │
│  │     ├─ Over-leveraging scenarios                             │
│  │     ├─ Premature exits or holding too long (from close logs) │
│  │     ├─ Market conditions to avoid                            │
│  │     └─ Decision reasoning patterns that led to losses       │
│  └─ Generate "Lessons Learned" (Actionable Rules)               │
│     ├─ DO: "AI-sector altcoins show 70% win rate, prioritize"   │
│     ├─ DO: "Use 2x leverage max when volatility > 5%"           │
│     ├─ DO: "Enter long positions when RSI < 30 + MACD bullish"  │
│     ├─ DON'T: "Avoid BTC trades during high volatility periods" │
│     ├─ DON'T: "Never enter when RSI > 70 (late entry trap)"     │
│     └─ DON'T: "Stop using 5x leverage on altcoins (3 losses)"   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3: Planning (The Orchestrator)                            │
│  ─────────────────────────────────────────────────────────────  │
│  ├─ Input: Context + Memory Lessons + rules                     │
│  ├─ Decision: Which pairs to research?                          │
│  │  ├─ Filter by Memory Agent recommendations                   │
│  │  │  ├─ Pair types based on lessons:                          │
│  │  │  │  ├─ If lesson: "AI-sector altcoins show 70% win rate"  │
│  │  │  │  │  └─ Prioritize: AI-related tokens (FET, RNDR, etc.) │
│  │  │  │  ├─ If lesson: "Avoid BTC during high volatility"      │
│  │  │  │  │  └─ Skip BTC if current volatility > threshold      │
│  │  │  │  └─ If lesson: "Best wins in DeFi sector"              │
│  │  │  │     └─ Focus: DeFi tokens (UNI, AAVE, etc.)            │
│  │  │  └─ Indicators based on lessons:                          │
│  │  │     ├─ If lesson: "RSI < 30 + MACD bullish = success"     │
│  │  │     │  └─ Required indicators: RSI, MACD                  │
│  │  │     ├─ If lesson: "Volume spike precedes big moves"       │
│  │  │     │  └─ Required indicators: Volume, Price action       │
│  │  │     └─ If lesson: "Bollinger Bands squeeze = breakout"    │
│  │  │        └─ Required indicators: BOLL, ATR                  │
│  │  ├─ Consider current positions (avoid duplicates)            │
│  │  └─ Prioritize based on risk/margin availability             │
│  └─ Output: Targeted Research Plan                              │
│     ├─ Required indicators per symbol (e.g., RSI, MACD, Volume) │
│     ├─ Required timeframes per symbol (e.g., 5m, 15m, 1h)       │
│     └─ Required historical data count (e.g., 200 candles per TF)│
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4: Targeted Research (Sub-Agents + Tools)                 │
│  ─────────────────────────────────────────────────────────────  │
│  Note: Patterns shown below are EXAMPLES, not strict rules.     │
│  Actual patterns discovered will vary based on historical data. │
│  ─────────────────────────────────────────────────────────────  │
│  ├─ Orchestrator delegates research tasks to Sub-Agents         │
│  │  └─ For each symbol: "Research FETUSDT, use RSI+MACD+Volume+timeframe" │
│  │                                                               │
│  ├─ Sub-Agent: Pattern Discovery Agent                         │
│  │  ├─ Receives: Symbol + Required indicators from Orchestrator │
│  │  ├─ Uses Tools (based on Orchestrator instructions):         │
│  │  │  ├─ get_kline_data(symbol, timeframe, count) [historical] │
│  │  │  ├─ calculate_indicator("RSI", symbol, period=14)         │
│  │  │  ├─ calculate_indicator("MACD", symbol)                  │
│  │  │  ├─ calculate_indicator("BOLL", symbol)                    │
│  │  │  ├─ calculate_indicator("Volume", symbol)                 │
│  │  │  └─ get_quant_data(symbol) [if enabled]                    │
│  │  ├─ Analyzes Historical Data to Find Patterns:               │
│  │  │  ├─ Scans historical price + indicator data (multi-timeframe)│
│  │  │  ├─ Identifies recurring patterns from past:    │
│  │  │  │  ├─ Example: "RSI < 30 in 5m + RSI < 30 in 1h → Bounce 85%"│
│  │  │  │  ├─ Example: "RSI < 30 + EMA20 > EMA250 → Bounce 80%"   │
│  │  │  │  ├─ Example: "RSI > 70 in 5m + RSI > 70 in 1h → Reversal 75%"│
│  │  │  │  ├─ Example: "Price broke BOLL upper + Volume 2x → Continue 70%"│
│  │  │  │  ├─ Example: "Price touched BOLL lower + EMA20 < EMA250 → Bounce 75%"│
│  │  │  │  ├─ Example: "MACD bullish cross in 5m + MACD bullish in 1h → Rise 78%"│
│  │  │  │  ├─ Example: "Volume spike 2x + RSI < 30 + EMA > 250 → Bounce 88%"│
│  │  │  │  ├─ Example: "RSI < 30 in 3m + RSI < 40 in 15m + EMA50 > EMA200 → Strong bounce 82%"│
│  │  │  │  ├─ Example: "BOLL squeeze + Volume spike + RSI neutral → Breakout 72%"│
│  │  │  │  └─ Example: "Support level held 3+ times + EMA alignment → High bounce 85%"│
│  │  │  ├─ Calculates pattern success rate (win rate) per timeframe │
│  │  │  ├─ Identifies multi-timeframe confirmations                │
│  │  │  ├─ Matches current conditions against historical patterns │
│  │  │  └─ Determines if current setup matches known patterns      │
│  │  └─ Returns Summary Report to Orchestrator (example):         │
│  │     ├─ Pair: FETUSDT                                           │
│  │     ├─ Patterns Found (use these):                            │
│  │     │  ├─ "RSI < 30 in 5m + RSI < 40 in 15m" → 82% success    │
│  │     │  ├─ "EMA20 > EMA250 alignment" → 80% success            │
│  │     │  ├─ "Volume spike 2x + RSI < 30" → 88% success          │
│  │     │  └─ "BOLL lower touch + EMA alignment" → 75% success    │
│  │     ├─ Combined Confidence: 87%                                │
│  │     ├─ Recommendation: TRADE                                  │
│  │     ├─ Entry: 0.85, SL: 0.80, TP: 0.95 (R:R 1:3.5)            │
│  │     └─ OR (if no pattern found):                               │
│  │        ├─ Pair: RNDRUSDT                                       │
│  │        ├─ Patterns Found: None                                 │
│  │        └─ Recommendation: SKIP                                 │
│  │                                                               │
│  └─ Orchestrator receives pattern feedback                      │
│     └─ Filters symbols: Keep only those with confirmed patterns │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 5: Specialist Analysis (Parallel Execution)              │
│  ─────────────────────────────────────────────────────────────  │
│  ├─ Trade Specialist                                            │
│  │  ├─ Input: Pattern Discovery reports, Indicator data         │
│  │  ├─ Analyzes: Which patterns to use for trading               │
│  │  ├─ Decisions based on indicator data:                        │
│  │  │  ├─ Order type: open_long / open_short / close / hold     │
│  │  │  ├─ Entry price: Based on current price + pattern          │
│  │  │  ├─ Stop loss: Calculated from support/resistance levels  │
│  │  │  ├─ Take profit: Based on pattern target + risk/reward     │
│  │  │  ├─ Position size: Based on confidence + account equity   │
│  │  │  ├─ Leverage: Based on risk management rules              │
│  │  │  └─ Confidence score: Based on pattern strength            │
│  │  └─ Output: Trading decision with all parameters             │
│  │                                                               │
│  └─ Risk Management Specialist                                 │
│     ├─ Input: Account state, Memory lessons, Position limits    │
│     ├─ Focus: Safety, Margin utilization, Risk/reward           │
│     └─ Output: Risk assessment (Safe/Moderate/High risk)        │
│        └─ Validates/Adjusts Trade Specialist decisions          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 6: Synthesis (The Decision)                              │
│  ─────────────────────────────────────────────────────────────  │
│  ├─ Master Agent receives all specialist reports               │
│  ├─ Integrates: Technical + Flow + Risk signals                 │
│  ├─ Applies Memory Agent lessons as filters                     │
│  └─ Generates final <decision> JSON                            │
│     ├─ Strictly validated (leverage, size, risk/reward)         │
│     └─ Ready for execution                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Step-by-Step Run Flow (The Lifecycle of a Trade)

Each time the bot "wakes up" (e.g., every 3 minutes), it follows this precise loop:

### Step 1: Context Preparation (`trader/auto_trader.go`)
*   **Risk Audit**: Checks if the trader is paused due to daily loss limits.
*   **Data Snapshot**: Fetches account equity, available balance, and all open positions from the exchange.
*   **Candidate Selection**: Filters the coin pool based on strategy settings (AI500, OI Top, etc.).
*   **Historical Context**: Pulls the last 10 trades and overall performance stats (Win Rate, Profit Factor) from the local database.

### Step 2: Intelligence Gathering (`kernel/engine.go`)
*   **Market Research**: Fetches K-line data (multi-timeframe) for all candidate coins.
*   **Smart Money Tracking**: Pulls Quantitative data (Institutional NetFlow and Open Interest rankings) via NofxOS.
*   **Prompt Engineering**: Generates a **System Prompt** (the Rules) and a **User Prompt** (the Current Situation).

### Step 3: AI Reasoning Loop (`mcp/client.go`)
*   **Analysis**: Sends the data to the AI.
*   **Thinking**: The AI generates a "Chain of Thought" (Reasoning) explaining its market view.
*   **Output**: The AI outputs a structured JSON array of decisions (Buy/Sell/Hold).

### Step 4: The Safety Guard (`kernel/engine.go`)
*   **Validation**: Every AI decision is strictly audited by Go logic:
    *   **Leverage**: Capped to safe limits (e.g., 5x for BTC, 2x for Alts).
    *   **Bet Size**: Ensures no single trade exceeds the "Position Value Ratio" (e.g., 1x equity).
    *   **Risk/Reward**: Rejects any trade where the target profit is not at least 3x the potential loss.

### Step 5: Optimized Execution (`trader/auto_trader.go`)
*   **Prioritization**: Sorts decisions to **Close** positions first (to free up margin) and **Open** new ones last.
*   **Order Execution**: Places Market orders on the exchange.
*   **Safety Nets**: Immediately sets **Stop-Loss** and **Take-Profit** orders after any successful entry.

### Step 6: Logging & UI Sync (`store/`)
*   **Persistence**: Saves the entire run—including the AI's "thoughts," the raw prompts, and the final outcome—to the database.
*   **Visualization**: Updates the Web UI so you can see exactly why the bot made its move.

---

## 4. Phase 1: Tool Registry & Execution Loop (The "Skills")
The foundation of a "Claude Code" style system is the ability for the AI to call tools for precise calculations.

1.  **Define Tool Interfaces**: Create `kernel/tools.go` to register Go functions as AI-callable tools.
    *   `get_kline_data(symbol, timeframe, count)`: Fetch raw market data.
    *   `calculate_indicator(name, params)`: Precise math for RSI, EMA, MACD, etc.
    *   `get_account_balance()`: Real-time balance and margin check.
    *   `simulate_trade_risk(size, entry, sl, tp)`: Calculate Risk/Reward and Liquidation points.
2.  **Upgrade the AI Loop**: Refactor `mcp/client.go` to support a **Reasoning Loop**.
    *   Instead of returning immediately, the client will check for `tool_calls` in the AI response.
    *   If found, the system executes the corresponding Go function, appends the result to the message history, and calls the AI again (Auto-looping).

## 4. Phase 2: Memory & Experience Agent (The "Learner")
This agent ensures the system doesn't make the same mistake twice by analyzing the `decision_records` and `trader_orders` tables.

1.  **Historical Analysis Logic**: Create `kernel/memory_agent.go`.
    *   **Function**: `GetLessonsLearned(traderID string)`.
    *   **Logic**: Fetch the last 5 losing trades and 5 winning trades. Extract their `CoTTrace` (Chain of Thought) and actual PnL results.
2.  **Reflection Prompt**: Send this data to a "Memory AI" to generate a short list of actionable insights.
    *   *Example Output: "Lesson: You tend to enter 'Long' too late when RSI is already > 70. Recommendation: Tighten entry standards for RSI."*

## 5. Phase 3: Specialized Agent Definitions (The "Experts")
Define unique `SystemPrompts` for specialized roles in a new `kernel/agents/` directory.

1.  **Technical Analyst Specialist**: Focuses purely on price action and chart patterns.
2.  **Quant & Flow Specialist**: Analyzes Open Interest (OI) and NetFlow to track institutional "smart money."
3.  **Risk Management Specialist**: Strict focus on account safety, margin utilization, and adherence to lessons from the Memory Agent.

## 6. Phase 4: Master Orchestrator (The "Manager")
Update `kernel/engine.go` to coordinate the high-level workflow.

1.  **Step 1: Planning**: The Master Agent receives the high-level strategy and the output from the **Memory Agent**. It outputs a task list.
2.  **Step 2: Delegation**: Call the specialists (Analyst, Flow, Risk) in parallel or sequence based on the plan.
3.  **Step 3: Synthesis**: The Master Agent receives the specialists' reports and generates the final strictly formatted `<decision>` JSON.

## 7. Phase 5: UI & Observability
1.  **Log Agent Interplay**: Update the `CoTTrace` in `DecisionRecord` to show which agent said what.
2.  **Agent Visualizer**: Update the **DecisionCard** in the web UI (`web/src/components/DecisionCard.tsx`) to display the different agent "thoughts" as a conversation thread rather than a single block of text.

---

## 8. Immediate Next Steps
1.  [ ] **Refactor `mcp`**: Enable the tool-execution loop in `mcp/client.go`.
2.  [ ] **Create Registry**: Implement the first 3 core tools (Market Data, Indicators, Risk).
3.  [ ] **Memory Agent Prototype**: Create the logic to pull historical trade "lessons" into the context.
