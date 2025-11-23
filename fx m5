import os import time import requests from telegram import Bot from telegram.ext import Updater, CommandHandler

-----------------------------

CONFIGURATION

-----------------------------

TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN", "YOUR_TELEGRAM_BOT_TOKEN_HERE") ALPHA_VANTAGE_KEY = os.getenv("ALPHA_VANTAGE_KEY", "YOUR_ALPHA_VANTAGE_KEY") PAIR = "EURUSD"  # default TIMEFRAME = "5min"

bot = Bot(token=TELEGRAM_TOKEN)

-----------------------------

GET PRICE DATA

-----------------------------

def get_candles(symbol): url = f"https://www.alphavantage.co/query?function=FX_INTRADAY&from_symbol={symbol[:3]}&to_symbol={symbol[3:]}&interval=5min&apikey={ALPHA_VANTAGE_KEY}&outputsize=compact" r = requests.get(url).json()

if "Time Series FX (5min)" not in r:
    return None

candles = r["Time Series FX (5min)"]
return list(candles.items())[:3]  # latest 3 candles

-----------------------------

BUY / SELL SIGNAL LOGIC

-----------------------------

def generate_signal(candles): if len(candles) < 2: return None

last_time, last = candles[0]
prev_time, prev = candles[1]

open_p = float(last['1. open'])
close_p = float(last['4. close'])

# Simple confirmation candlestick rules
if close_p > open_p:
    return "BUY"
elif close_p < open_p:
    return "SELL"
else:
    return None

-----------------------------

TELEGRAM COMMANDS

-----------------------------

def start(update, context): update.message.reply_text("M5 Auto Signal Bot Running\nPairs: EURUSD\nSignals every 15 minutes.")

def manual_signal(update, context): candles = get_candles(PAIR) if candles is None: update.message.reply_text("Error retrieving data.") return

sig = generate_signal(candles)
update.message.reply_text(f"Manual Check: {sig}")

-----------------------------

AUTO LOOP

-----------------------------

def auto_loop(): last_signal = "" chat_id = None

while True:
    try:
        candles = get_candles(PAIR)
        if candles:
            signal = generate_signal(candles)
            if signal and signal != last_signal:
                bot.send_message(chat_id=os.getenv("CHAT_ID"), text=f"M5 SIGNAL: {signal}")
                last_signal = signal

    except Exception as e:
        print("Error:", e)

    time.sleep(900)  # 15 minutes

-----------------------------

MAIN

-----------------------------

if name == "main": updater = Updater(TELEGRAM_TOKEN, use_context=True) dp = updater.dispatcher

dp.add_handler(CommandHandler("start", start))
dp.add_handler(CommandHandler("signal", manual_signal))

updater.start_polling()
auto_loop()
