import os
import telebot
from google import genai
from flask import Flask
from threading import Thread

# --- 1. إعداد سيرفر صغير لـ Render (Web Service) ---
app = Flask('')

@app.route('/')
def home():
    return "دحيح شغال وزي الفل! 🚀"

def run():
    # Render يتطلب بورت 8080 أو المتغير المسمى PORT
    port = int(os.environ.get("PORT", 8080))
    app.run(host='0.0.0.0', port=port)

def keep_alive():
    t = Thread(target=run)
    t.start()

# --- 2. جلب المفاتيح من إعدادات Render (للأمان) ---
TOKEN = os.environ.get('TELEGRAM_TOKEN')
API_KEY = os.environ.get('GEMINI_API_KEY')

bot = telebot.TeleBot(TOKEN)
client = genai.Client(api_key=API_KEY)

# شخصية دحيح
SYSTEM_PROMPT = "أنت دحيح، مساعد دراسي سوداني مرح وذكي. ساعد الطالب في حل المسائل والشرح ببساطة."

# --- 3. معالجة النصوص والصور ---
@bot.message_handler(content_types=['text', 'photo'])
def handle_messages(message):
    try:
        if message.content_type == 'text':
            response = client.models.generate_content(
                model="gemini-2.0-flash",
                contents=f"{SYSTEM_PROMPT}\nالطالب: {message.text}"
            )
        else:
            # معالجة الصور (حل المسائل)
            file_info = bot.get_file(message.photo[-1].file_id)
            downloaded_file = bot.download_file(file_info.file_path)
            response = client.models.generate_content(
                model="gemini-2.0-flash",
                contents=[SYSTEM_PROMPT, {'mime_type': 'image/jpeg', 'data': downloaded_file}]
            )
        bot.reply_to(message, response.text)
    except Exception as e:
        print(f"Error: {e}")
        bot.reply_to(message, "يا بطل حصلت مشكلة تقنية، جرب تاني بعد شوية!")

# --- 4. التشغيل ---
if __name__ == "__main__":
    keep_alive()  # تشغيل السيرفر لضمان قبول طلبات Render
    print("✅ دحيح انطلق في تليجرام!")
    bot.polling(none_stop=True)
