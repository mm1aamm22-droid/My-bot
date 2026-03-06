import telebot
import requests
import os
from flask import Flask
from threading import Thread

# 1. سيرفر وهمي لإبقاء البوت حياً على Render
app = Flask('')
@app.route('/')
def home(): return "Bot is Alive!"

def run():
    port = int(os.environ.get("PORT", 10000))
    app.run(host='0.0.0.0', port=port)

def keep_alive():
    Thread(target=run).start()

# 2. إعدادات البوت
API_TOKEN = '8629591404:AAElD9enpGE52EH8DNaNVeLJp14cU9eD64o'
bot = telebot.TeleBot(API_TOKEN)

@bot.message_handler(commands=['start'])
def start(message):
    bot.reply_to(message, "🌟 أهلاً بك! أرسل رابط تيك توك وسأقوم بتحميله فوراً (بدون علامة مائية).")

@bot.message_handler(func=lambda m: 'tiktok.com' in m.text)
def handle_tiktok(message):
    msg = bot.reply_to(message, "⏳ جاري تجاوز الحظر وتحميل الفيديو...")
    try:
        # استخدام API وسيط لتجاوز حظر تيك توك
        api_url = f"https://api.tiklydown.eu.org/api/download?url={message.text}"
        res = requests.get(api_url).json()
        
        video_url = res['video']['noWatermark']
        
        # إرسال الفيديو مباشرة من الرابط (أسرع ولا يستهلك مساحة السيرفر)
        bot.send_video(message.chat.id, video_url, caption="✅ تم التحميل بنجاح!")
        bot.delete_message(message.chat.id, msg.message_id)
        
    except Exception as e:
        bot.edit_message_text(f"❌ عذراً، تيك توك يفرض قيوداً إضافية حالياً.\nالخطأ: {str(e)}", message.chat.id, msg.message_id)

if __name__ == "__main__":
    keep_alive()
    print("🚀 البوت يعمل الآن...")
    bot.infinity_polling()
