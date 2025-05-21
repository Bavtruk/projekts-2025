import random
import asyncio
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes

words = [f"слово{i}" for i in range(1, 501)]

scores = {"Команда 1": 0, "Команда 2": 0}
current_team = 0
round_number = 1
main_message_id = None
info_message_id = None
game_chat_id = None
current_word = None
game_active = False
team_names = ["Команда 1", "Команда 2"]

async def update_main_message(context: ContextTypes.DEFAULT_TYPE):
    global main_message_id, game_chat_id
    text = f"Раунд {round_number}\n"
    for team in team_names:
        text += f"{team}: {scores[team]} баллов\n"
    if main_message_id:
        await context.bot.edit_message_text(chat_id=game_chat_id, message_id=main_message_id, text=text)
    else:
        msg = await context.bot.send_message(chat_id=game_chat_id, text=text)
        main_message_id = msg.message_id

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global info_message_id, game_chat_id, round_number, current_team
    game_chat_id = update.effective_chat.id
    current_team = 0
    round_number = 1
    await update_main_message(context)
    text = f"{team_names[current_team]}, нажмите 'Начать', чтобы начать ход"
    keyboard = [[InlineKeyboardButton("Начать", callback_data="start_turn")]]
    msg = await update.message.reply_text(text=text, reply_markup=InlineKeyboardMarkup(keyboard))
    info_message_id = msg.message_id

async def start_turn(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global game_active, current_word, info_message_id
    if game_active:
        return
    game_active = True
    await context.bot.edit_message_text(chat_id=game_chat_id, message_id=info_message_id, text=f"{team_names[current_team]} играет...")

    await send_new_word(context)

    await time.sleep(60)
    await context.bot.delete_message(chat_id=game_chat_id, message_id=context.chat_data["word_message_id"])
    switch_team()
    await update_main_message(context)
    keyboard = [[InlineKeyboardButton("Начать", callback_data="start_turn")]]
    new_text = f"{team_names[current_team]}, нажмите 'Начать', чтобы начать ход"
    msg = await context.bot.send_message(chat_id=game_chat_id, text=new_text, reply_markup=InlineKeyboardMarkup(keyboard))
    context.chat_data["info_message_id"] = msg.message_id
    info_message_id = msg.message_id
    game_active = False

def switch_team():
    global current_team, round_number
    current_team = 1 - current_team
    if current_team == 0:
        round_number += 1

async def send_new_word(context: ContextTypes.DEFAULT_TYPE):
    global current_word
    current_word = random.choice(words)
    keyboard = [
        [InlineKeyboardButton("Угадал", callback_data="guessed")],
        [InlineKeyboardButton("Не угадал", callback_data="not_guessed")]
    ]
    msg = await context.bot.send_message(chat_id=game_chat_id, text=current_word, reply_markup=InlineKeyboardMarkup(keyboard))
    context.chat_data["word_message_id"] = msg.message_id

async def handle_guess(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global current_word
    query = update.callback_query
    await query.answer()

    if query.data == "guessed":
        scores[team_names[current_team]] += 1
    elif query.data == "not_guessed":
        scores[team_names[current_team]] -= 1

    await update_main_message(context)
    await context.bot.edit_message_text(chat_id=game_chat_id, message_id=query.message.message_id, text=f"{current_word} — {'Угадано' if query.data == 'guessed' else 'Не угадано'}")
    await send_new_word(context)

def main():
    app = ApplicationBuilder().token("7759529423:AAGfwcr39oFfrKZWxPeYfllQA079xMZxuIM").build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CallbackQueryHandler(start_turn, pattern="^start_turn$"))
    app.add_handler(CallbackQueryHandler(handle_guess, pattern="^(guessed|not_guessed)$"))

    print("Бот запущен...")
    app.run_polling()

if __name__ == '__main__':
    main()
