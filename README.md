import telebot
import yt_dlp
import os
from flask import Flask
from threading import Thread

# 1. إعداد سيرفر ويب وهمي لإبقاء Render مستيقظاً
app = Flask('')

@app.route('/')
def home():
    return "Bot is running!"

def run():
    # استخدام البورت الذي يوفره Render تلقائياً
    port = int(os.environ.get("PORT", 10000))
    app.run(host='0.0.0.0', port=port)

def keep_alive():
    t = Thread(target=run)
    t.start()

# 2. إعدادات البوت (التوكن الخاص بك)
API_TOKEN = '8629591404:AAElD9enpGE52EH8DNaNVeLJp14cU9eD64o'
bot = telebot.TeleBot(API_TOKEN)

# دالة التحميل مع معالجة الحجم والجودة
def download_video(url):
    file_name = 'video.mp4'
    ydl_opts = {
        # 'best[height<=720]' تضمن جودة جيدة وحجم ملف صغير لتجنب خطأ 413
        'format': 'best[height<=720]/best',
        'outtmpl': file_name,
        'quiet': True,
        'no_warnings': True,
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        ydl.download([url])
    return file_name

@bot.message_handler(commands=['start'])
def start(message):
    welcome_text = (
        f"🌟 أهلاً بك يا {message.from_user.first_name}!\n\n"
        "أرسل لي رابط فيديو من (تيك توك أو إنستغرام) "
        "وسأقوم بتحميله لك فوراً وبأفضل جودة ممكنة."
    )
    bot.reply_to(message, welcome_text)

@bot.message_handler(func=lambda m: m.text and m.text.startswith('http'))
def handle_link(message):
    status_msg = bot.reply_to(message, "⏳ جاري التحميل ومعالجة الفيديو...")
    
    try:
        # تحميل الملف
        file_path = download_video(message.text)
        
        # التأكد من حجم الملف قبل الإرسال (حد تيليجرام 50MB)
        file_size = os.path.getsize(file_path) / (1024 * 1024)
        
        if file_size > 50:
            bot.edit_message_text(f"⚠️ عذراً، الفيديو حجمه ({file_size:.1f}MB) وهو أكبر من المسموح به (50MB).", 
                                 message.chat.id, status_msg.message_id)
        else:
            with open(file_path, 'rb') as video:
                bot.send_video(message.chat.id, video, caption="✅ تم التحميل بنجاح عبر DevFlow")
            bot.delete_message(message.chat.id, status_msg.message_id)
            
        # حذف الملف من السيرفر لتوفير المساحة
        if os.path.exists(file_path):
            os.remove(file_path)
            
    except Exception as e:
        bot.edit_message_text(f"❌ حدث خطأ غير متوقع:\n{str(e)}", message.chat.id, status_msg.message_id)

# 3. تشغيل النظام المزدوج
if __name__ == "__main__":
    keep_alive() # تشغيل سيرفر الويب في الخلفية
    print("🚀 البوت يعمل الآن بكامل طاقته...")
    bot.infinity_polling()
