import os
import threading
from fastapi import FastAPI
from dotenv import load_dotenv
import openai

from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, filters, ContextTypes

# بارگذاری متغیرهای محیطی از .env
load_dotenv()
openai.api_key = os.getenv("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")
telegram_token = os.getenv("7790106717:AAEdewcCz5YRkOHirXFFasLJbx_26MdV4hM")

app = FastAPI()

# تابع ارسال پیام به ChatGPT
async def ask_gpt(message: str) -> str:
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": message}
        ]
    )
    return response["choices"][0]["message"]["content"]

# (اختیاری) endpoint برای تست مستقیم با Postman
@app.post("/chat")
async def chat_with_gpt(req: dict):
    reply = await ask_gpt(req["message"])
    return {"reply": reply}

# هندلر پیام تلگرام
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_message = update.message.text
    reply = await ask_gpt(user_message)
    await update.message.reply_text(reply)

# استارت ربات تلگرام در یک Thread جدا
def start_telegram_bot(@Asli6bot):
    application = ApplicationBuilder().token(7790106717:AAEdewcCz5YRkOHirXFFasLJbx_26MdV4hM).build()
    application.add_handler(MessageHandler(filters.TEXT & (~filters.COMMAND), handle_message))
    application.run_polling()

threading.Thread(target=start_telegram_bot, daemon=True).start()
