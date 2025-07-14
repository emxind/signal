import streamlit as st
import yfinance as yf
import pandas as pd
import numpy as np
from scipy.stats import zscore

st.set_page_config(page_title="📈 ETF Strength Dashboard", layout="wide")

st.title("📈 ETF Strength Signal Dashboard")
st.markdown("Ranks ETFs based on 15-day return, volume momentum, and volatility.")

# --- Helper Function ---
@st.cache_data(ttl=3600)
def get_stock_data(ticker, period="1mo", interval="1d"):
    data = yf.download(ticker, period=period, interval=interval, progress=False)
    return data.dropna()

def calculate_strength(data):
    if len(data) < 15:
        return None

    data['Return'] = data['Adj Close'].pct_change()

    return_15d = data['Adj Close'].iloc[-1] / data['Adj Close'].iloc[-15] - 1
    vol_5 = data['Volume'].iloc[-5:].mean()
    vol_10 = data['Volume'].iloc[-15:-5].mean()
    volume_ratio = vol_5 / vol_10 if vol_10 > 0 else np.nan
    volatility_15d = data['Return'].iloc[-15:].std()

    return {
        'Return_15d': return_15d,
        'Volume_Ratio': volume_ratio,
        'Volatility': volatility_15d
    }

# --- Ticker Input ---
default_etfs = "GLD, SPY, QQQ, TLT, IWM"
tickers = st.text_input("Enter tickers (comma-separated):", value=default_etfs)
tickers = [t.strip().upper() for t in tickers.split(",") if t.strip()]

# --- Compute Signals ---
signal_data = []

with st.spinner("📡 Fetching data and calculating signals..."):
    for ticker in tickers:
        try:
            data = get_stock_data(ticker)
            metrics = calculate_strength(data)
            if metrics:
                signal_data.append({ "Ticker": ticker, **metrics })
        except Exception as e:
            st.warning(f"⚠️ Error processing {ticker}: {e}")

# --- Build Score & Display ---
if signal_data:
    df = pd.DataFrame(signal_data)

    # Standardize scores
    df_z = df[['Return_15d', 'Volume_Ratio', 'Volatility']].apply(zscore)
    df['Strength_Score'] = df_z['Return_15d'] + df_z['Volume_Ratio'] - df_z['Volatility']

    df = df.sort_values("Strength_Score", ascending=False).reset_index(drop=True)
    df_display = df[['Ticker', 'Strength_Score', 'Return_15d', 'Volume_Ratio', 'Volatility']]
    df_display = df_display.round(3)

    st.success("✅ Ranking complete")
    st.dataframe(df_display.style.highlight_max(axis=0, color='lightgreen')
                                  .highlight_min(axis=0, color='#ffe6e6'),
                 use_container_width=True)
else:
    st.warning("No data available to display.")
