import telebot
from telebot import types
import os
from flask import Flask
from threading import Thread

# 1. السيرفر الوهمي
app = Flask('')
@app.route('/')
def home(): return "Online"

def run():
    port = int(os.environ.get("PORT", 10000))
    app.run(host='0.0.0.0', port=port)

def keep_alive():
    Thread(target=run).start()

# 2. إعدادات البوت (تأكد من التوكن)
API_TOKEN = '8629591404:AAElD9enpGE52EH8DNaNVeLJp14cU9eD64o'
bot = telebot.TeleBot(API_TOKEN)

@bot.message_handler(commands=['start'])
def start(message):
    markup = types.InlineKeyboardMarkup(row_width=2)
    btn1 = types.InlineKeyboardButton("📐 الرياضيات", callback_data="m")
    btn2 = types.InlineKeyboardButton("📖 العربية", callback_data="a")
    markup.add(btn1, btn2)
    bot.reply_to(message, "📚 مرحباً بك في بوت الشهادة السودانية!\nاختر المادة:", reply_markup=markup)

@bot.callback_query_handler(func=lambda call: True)
def cb(call):
    bot.send_message(call.message.chat.id, "⏳ جاري تجهيز الملفات لهذه المادة...")

if __name__ == "__main__":
    keep_alive()
    bot.infinity_polling()
