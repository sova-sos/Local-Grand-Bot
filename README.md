import os
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes
from dotenv import load_dotenv

load_dotenv()
TOKEN = os.getenv("BOT_TOKEN")

user_data = {}

questions = [
    "Ты гражданин Ирландии?",
    "Ты официально работаешь?",
    "У тебя есть дети?",
    "Ты студент или безработный?",
    "Ты снимаешь жилье или владеешь им?",
    "Твой доход ниже среднего?"
]

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    user_data[user_id] = {"step": 0, "answers": []}
    await update.message.reply_text("Привет! Я помогу тебе узнать, на какие выплаты ты можешь претендовать.")
    await ask_next(update, user_id)

async def ask_next(update, user_id):
    step = user_data[user_id]["step"]
    if step < len(questions):
        await update.message.reply_text(questions[step])
    else:
        await result(update, user_id)

async def handle(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    answer = update.message.text.lower()
    user_data[user_id]["answers"].append(answer)
    user_data[user_id]["step"] += 1
    await ask_next(update, user_id)

async def result(update, user_id):
    answers = user_data[user_id]["answers"]
    grants = []
    if answers[0] == "да":
        if answers[2] == "да":
            grants.append("Child Benefit")
        if answers[3] == "да":
            grants.append("Jobseeker's Allowance")
        if answers[5] == "да":
            grants.append("Housing Assistance Payment")
    if grants:
        await update.message.reply_text("Ты можешь претендовать на:\n- " + "\n- ".join(grants))
    else:
        await update.message.reply_text("Похоже, нет доступных выплат по твоим ответам.")

if __name__ == "__main__":
    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle))
    app.run_polling()а
