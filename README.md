import telebot
import requests
import acrcloud
from pydub import AudioSegment
import io

TOKEN = "7337071419:AAHUcZqDC6VRfrs01eMkeYQ5wM3m5YJLImo"  # توکن ربات تلگرام شما
ACR_ACCESS_KEY = "6697f34041d2eaf0926f8d90cd68b2b4"  # Access Key جدید شما
ACR_SECRET_KEY = "71Q0Gw7dgRyXksUW5Gc17mSAN1T2LmjIsvKM9gaM"  # Secret Key جدید شما

bot = telebot.TeleBot(TOKEN)

acr = acrcloud.ACRCloudRecognizer({
    "access_key": ACR_ACCESS_KEY,
    "access_secret": ACR_SECRET_KEY,
    "host": "identify-eu-west-1.acrcloud.com"  # آدرس سرور ACRCloud
})

# هندلر برای دستور /start
@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(message, "سلام! ربات آماده است. لطفاً یک فایل صوتی یا ویدئویی ارسال کن تا آهنگ شناسایی بشه.")

# هندلر برای محتوای صوتی و ویدئویی
@bot.message_handler(content_types=['audio', 'voice', 'video'])
def handle_media(message):
    try:
        file_info = bot.get_file(message.audio.file_id if message.audio else message.voice.file_id)
        downloaded_file = bot.download_file(file_info.file_path)
        
        # ذخیره فایل صوتی
        with open("temp_audio.mp3", 'wb') as new_file:
            new_file.write(downloaded_file)
        
        # تشخیص آهنگ
        result = acr.recognize_by_file("temp_audio.mp3")
        result_json = result["metadata"]
        
        if result_json.get("music"):
            song = result_json["music"][0]
            song_name = song["title"]
            artist = song["artists"][0]["name"]
            bot.reply_to(message, f"آهنگ شناسایی شد: {song_name} - {artist}")
        else:
            bot.reply_to(message, "آهنگ پیدا نشد. لطفاً دوباره امتحان کن.")
    except Exception as e:
        bot.reply_to(message, f"مشکلی پیش اومد: {str(e)}")

bot.polling()
# telegram-music-bot