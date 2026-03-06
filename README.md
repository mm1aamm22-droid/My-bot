import telebot
import yt_dlp
import os

API_TOKEN = '8629591404:AAElD9enpGE52EH8DNaNVeLJp14cU9eD64o'
bot = telebot.TeleBot(API_TOKEN)

def download_video(url):
    ydl_opts = {
        'format': 'best',
        'outtmpl': 'video.mp4',
        'quiet': True
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        ydl.download([url])
    return 'video.mp4'

@bot.message_handler(commands=['start'])
def start(message):
    bot.reply_to(message, "مرحباً! أرسل رابط تيك توك أو إنستغرام للتحميل.")

@bot.message_handler(func=lambda m: m.text.startswith('http'))
def handle_link(message):
    m = bot.reply_to(message, "⏳ جاري التحميل...")
    try:
        file_path = download_video(message.text)
        with open(file_path, 'rb') as v:
            bot.send_video(message.chat.id, v)
        os.remove(file_path)
        bot.delete_message(message.chat.id, m.message_id)
    except Exception as e:
        bot.edit_message_text(f"❌ خطأ: {str(e)}", message.chat.id, m.message_id)

bot.infinity_polling()
