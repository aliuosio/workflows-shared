# Workflows Shared

This repository contains a collection of n8n workflows.

## Futures Analyst V1

The "Futures Analyst V1" is an automated workflow designed to provide technical analysis and a 4-hour price forecast for cryptocurrency futures. It is triggered through a chat interface and leverages a multi-tool AI agent to deliver a comprehensive market report.

### How It Works

The workflow operates in a structured, multi-step process:

1.  **Chat Input**: The workflow is initiated by a natural language query from a user, such as "Analyze the price of SOL."

2.  **Symbol Extraction**: A custom `SessionTools` node parses the input to identify the cryptocurrency symbol (e.g., `SOLUSDT`). If no symbol is found, it defaults to `BTCUSDT`.

3.  **Data Collection (Sub-workflow)**: The main workflow calls the **"Futures Analyst V1 - Data"** sub-workflow to gather real-time market data from the MEXC exchange API. The data collected includes:
    *   Current price
    *   Candlestick data (OHLCV)
    *   Current and historical funding rates
    *   Order book depth
    *   Recent trades list
    *   Contract details

4.  **Data Storage**: The retrieved market data is then stored in a MongoDB database for logging and subsequent analysis.

5.  **AI-Powered Analysis**: A sophisticated AI Agent, powered by Google's Gemini model, performs the core analysis. The agent is equipped with a suite of tools to process the market data:
    *   **MongoDB Retrieval**: Fetches the collected market data from the database.
    *   **TechnicalIndicators**: A custom tool that calculates key technical indicators, including MACD, RSI, EMAs, Bollinger Bands, and ATR.
    *   **SearXNG**: Gathers market sentiment by performing a web search for news and social media trends.
    *   **Calculator**: Performs any necessary mathematical calculations.
    *   **DateTime**: Retrieves the current timestamp.

6.  **Structured Report Generation**: Based on its analysis, the AI Agent generates a detailed report in a predefined format. The report includes:
    *   **Current Price**: The latest price of the asset.
    *   **Summary**: A brief overview of the market sentiment (bullish, bearish, or neutral).
    *   **Price Levels**: Key support and resistance levels, justified by ATR and recent price action.
    *   **Sentiment**: An indicator of the overall market sentiment.
    *   **Technical Analysis**: A breakdown of the values and interpretations of the calculated technical indicators.
    *   **Forecast**: A directional forecast for the next 4 hours, including a target price.
    *   **Trade Setup**: A complete trade plan with direction, entry price, stop loss, take profit targets, and recommended position size.

### Features

*   **Modular Architecture**: The workflow is split into a main logic workflow and a dedicated data-gathering sub-workflow, promoting reusability and maintainability.
*   **Data-Driven Analysis**: All analysis is based on real-time market data, ensuring the insights are timely and relevant.
*   **Multi-Tool AI Agent**: The use of multiple specialized tools allows the AI to perform a more thorough and accurate analysis than a single-prompt model.
*   **Secure Credential Management**: The workflow follows security best practices by referencing credentials stored securely within the n8n instance, avoiding any hardcoded secrets.
*   **Automated and On-Demand**: Provides a fully automated technical analysis report whenever triggered, allowing for quick and efficient market insights.
