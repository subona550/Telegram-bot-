import telebot
import json
import os

API_TOKEN = '8378103195:AAFXxttTTclT2DoQzhmI6rtNBdzvd_v_Ob0'
ADMIN_ID = 8410960154  # আপনার নিজের Telegram ID বসান

bot = telebot.TeleBot(API_TOKEN)

DATA_FILE = 'data.json'

def load_data():
    if not os.path.exists(DATA_FILE):
        return {}
    with open(DATA_FILE, 'r') as f:
        return json.load(f)

def save_data(data):
    with open(DATA_FILE, 'w') as f:
        json.dump(data, f, indent=4)

users = load_data()

@bot.message_handler(commands=['start'])
def start(message):
    user_id = str(message.chat.id)
    args = message.text.split()

    if user_id not in users:
        users[user_id] = {"balance": 0, "referrals": [], "id": user_id}
        if len(args) > 1:
            referrer_id = args[1]
            if referrer_id != user_id and referrer_id in users:
                if user_id not in users[referrer_id]["referrals"]:
                    users[referrer_id]["referrals"].append(user_id)
                    users[referrer_id]["balance"] += 1

    save_data(users)

    markup = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.row('▶️ Watch Ad', '💰 Balance')
    markup.row('📲 Referral Link', '💸 Withdraw')

    bot.send_message(message.chat.id, "👋 স্বাগতম! আমি আপনার রিওয়ার্ড বট!", reply_markup=markup)

@bot.message_handler(func=lambda msg: msg.text == '▶️ Watch Ad')
def watch_ad(message):
    user_id = str(message.chat.id)
    users[user_id]["balance"] += 2
    save_data(users)
    bot.send_message(message.chat.id, "✅ আপনি বিজ্ঞাপন দেখেছেন এবং 2 টাকা পেয়েছেন!")

@bot.message_handler(func=lambda msg: msg.text == '💰 Balance')
def balance(message):
    user_id = str(message.chat.id)
    user = users[user_id]
    bot.send_message(message.chat.id, f"💼 ব্যালেন্স: {user['balance']} টাকা\n👥 রেফারেল: {len(user['referrals'])} জন")

@bot.message_handler(func=lambda msg: msg.text == '📲 Referral Link')
def referral_link(message):
    user_id = str(message.chat.id)
    username = "YOUR_BOT_USERNAME"  # বটের username বসাতে হবে
    link = f"https://t.me/{username}?start={user_id}"
    bot.send_message(message.chat.id, f"🔗 আপনার রেফারেল লিংক:\n{link}")

@bot.message_handler(func=lambda msg: msg.text == '💸 Withdraw')
def withdraw(message):
    user_id = str(message.chat.id)
    user = users[user_id]
    if user['balance'] >= 10:
        user['balance'] -= 10
        save_data(users)
        bot.send_message(message.chat.id, "✅ আপনার উইথড্র রিকোয়েস্ট পাঠানো হয়েছে!")
        bot.send_message(ADMIN_ID, f"💸 Withdraw Request:\nUser ID: {user_id}\nAmount: 10 টাকা")
    else:
        bot.send_message(message.chat.id, "❌ আপনার ব্যালেন্স কম।")

bot.polling()
