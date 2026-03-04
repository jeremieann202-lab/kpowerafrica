# Kpower Africa Bot - Python Telegram Bot
# 🔹 Copie ce code dans GitHub main.py
# 🔹 Token et ADMIN_ID déjà configurés pour toi

from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater, CommandHandler, CallbackQueryHandler, CallbackContext
import sqlite3
from datetime import datetime

# ---------- CONFIGURATION ----------
TOKEN = "8756174682:AAGp6wePhV1C-n3sPCuSk35l89chqEQEcHA"
ADMIN_ID = 1919559774

POINTS_PER_CLICK = 50
MAX_CLICKS_PER_DAY = 100
POINTS_TO_FCFA = 500 / 25  # 500 points = 25 FCFA
MIN_WITHDRAWAL_FCFA = 1000

# ---------- DATABASE ----------
conn = sqlite3.connect('kpowerafrica.db', check_same_thread=False)
c = conn.cursor()
c.execute('''
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    points INTEGER DEFAULT 0,
    clicks_today INTEGER DEFAULT 0,
    last_click DATE,
    referrals TEXT
)
''')
conn.commit()

# ---------- HELPERS ----------
def get_user(user_id):
    c.execute("SELECT * FROM users WHERE id=?", (user_id,))
    user = c.fetchone()
    if not user:
        c.execute("INSERT INTO users (id, points, clicks_today, last_click, referrals) VALUES (?, 0, 0, ?, '')", (user_id, datetime.now().date()))
        conn.commit()
        return get_user(user_id)
    return user

def update_clicks(user_id):
    user = get_user(user_id)
    last_click = datetime.strptime(str(user[3]), "%Y-%m-%d").date() if user[3] else datetime.now().date()
    today = datetime.now().date()
    clicks_today = user[2] if last_click == today else 0
    if clicks_today >= MAX_CLICKS_PER_DAY:
        return False, user[1], clicks_today
    clicks_today += 1
    points = user[1] + POINTS_PER_CLICK
    c.execute("UPDATE users SET points=?, clicks_today=?, last_click=? WHERE id=?", (points, clicks_today, today, user_id))
    conn.commit()
    return True, points, clicks_today

# ---------- MENU BUTTONS ----------
def main_menu():
    keyboard = [
        [InlineKeyboardButton("⛏ Mining", callback_data='mining')],
        [InlineKeyboardButton("💰 Plus d'argent", callback_data='more_money')],
        [InlineKeyboardButton("👥 Parrainer un ami", callback_data='referral')],
        [InlineKeyboardButton("👤 Profil", callback_data='profile')],
        [InlineKeyboardButton("🔄 Échange KCU", callback_data='exchange')],
        [InlineKeyboardButton("💳 Retrait d'argent", callback_data='withdraw')],
        [InlineKeyboardButton("📊 Statistiques", callback_data='stats')]
    ]
    return InlineKeyboardMarkup(keyboard)

# ---------- COMMANDS ----------
def start(update: Update, context: CallbackContext):
    update.message.reply_text("Bienvenue sur Kpower Africa! Sélectionnez un menu :", reply_markup=main_menu())

def button(update: Update, context: CallbackContext):
    query = update.callback_query
    user_id = query.from_user.id
    query.answer()

    if query.data == "mining":
        mining_keyboard = [
            [InlineKeyboardButton("Clique", callback_data='click')],
            [InlineKeyboardButton("Menu principal", callback_data='menu')]
        ]
        query.edit_message_text("Mining: Cliquez pour gagner des points!", reply_markup=InlineKeyboardMarkup(mining_keyboard))

    elif query.data == "click":
        success, points, clicks_today = update_clicks(user_id)
        if success:
            query.edit_message_text(f"Vous avez {points} points. Clics aujourd'hui: {clicks_today}/{MAX_CLICKS_PER_DAY}", reply_markup=main_menu())
        else:
            query.edit_message_text(f"Limite quotidienne atteinte! Points: {points}.", reply_markup=main_menu())

    elif query.data == "menu":
        query.edit_message_text("Menu principal :", reply_markup=main_menu())

    elif query.data == "more_money":
        query.edit_message_text("Abonnez-vous aux canaux Telegram et envoyez les preuves à l'admin (ID: {}).".format(ADMIN_ID), reply_markup=main_menu())

    elif query.data == "referral":
        query.edit_message_text("Partage ce lien pour parrainer un ami: https://t.me/KpowerAfricaBot?start={}".format(user_id), reply_markup=main_menu())

    elif query.data == "profile":
        user = get_user(user_id)
        query.edit_message_text(f"ID: {user_id}\nPoints: {user[1]}", reply_markup=main_menu())

    elif query.data == "exchange":
        user = get_user(user_id)
        fcfa = user[1] * 25 / 500
        query.edit_message_text(f"Vous pouvez échanger vos points. Total points: {user[1]} → {fcfa} FCFA", reply_markup=main_menu())

    elif query.data == "withdraw":
        user = get_user(user_id)
        fcfa = user[1] * 25 / 500
        if fcfa >= MIN_WITHDRAWAL_FCFA:
            query.edit_message_text(f"Retrait accepté ! Envoyez demande à l'admin (ID: {ADMIN_ID}). Total: {fcfa} FCFA", reply_markup=main_menu())
            context.bot.send_message(ADMIN_ID, f"Utilisateur {user_id} demande un retrait de {fcfa} FCFA")
        else:
            query.edit_message_text(f"Retrait minimum: {MIN_WITHDRAWAL_FCFA} FCFA. Vous avez {fcfa} FCFA", reply_markup=main_menu())

    elif query.data == "stats":
        c.execute("SELECT COUNT(*), SUM(points) FROM users")
        total_users, total_points = c.fetchone()
        query.edit_message_text(f"Total utilisateurs: {total_users}\nTotal points: {total_points}", reply_markup=main_menu())

# ---------- MAIN ----------
def main():
    updater = Updater(TOKEN)
    dp = updater.dispatcher
    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CallbackQueryHandler(button))
    updater.start_polling()
    print("Bot démarré...")
    updater.idle()

if __name__ == "__main__":
    main()
