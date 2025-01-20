from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackQueryHandler, CallbackContext

# Kanal ID'lari
FIRST_CHANNEL = '@comediauz'
SECOND_CHANNEL = '@comediauz'
MOVIE_CHANNEL = '@kinouchuntest'

# Bot start komandasi
def start(update: Update, context: CallbackContext) -> None:
    keyboard = [
        [InlineKeyboardButton("Obuna bo'lish - 1-kanal", url=f'https://t.me/{FIRST_CHANNEL[1:]}')],
        [InlineKeyboardButton("Obuna bo'lish - 2-kanal", url=f'https://t.me/{SECOND_CHANNEL[1:]}')],
        [InlineKeyboardButton("Obuna bo'ldim", callback_data='subscribed')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    update.message.reply_text('Iltimos, quyidagi kanallarga obuna bo\'ling:', reply_markup=reply_markup)

# Obuna bo'ldim tugmasini bosganda
def check_subscription(update: Update, context: CallbackContext) -> None:
    query = update.callback_query
    user = query.from_user

    # Foydalanuvchining kanallarga obuna bo'lganini tekshirish
    first_channel_member = context.bot.get_chat_member(FIRST_CHANNEL, user.id)
    second_channel_member = context.bot.get_chat_member(SECOND_CHANNEL, user.id)

    if first_channel_member.status == 'member' and second_channel_member.status == 'member':
        query.edit_message_text('Sizga kerakli film raqamini kiriting.')
    else:
        query.edit_message_text('Siz hali kanallarga obuna bo\'lmagansiz. Iltimos, obuna bo\'ling va qaytadan urunib ko\'ring.')

# Kodni tekshirish va filmni yuborish
def verify_code(update: Update, context: CallbackContext) -> None:
    if update.message.text == '123':
        # Kanaldagi ikkinchi postni yuborish
        posts = context.bot.get_chat(MOVIE_CHANNEL).get_posts()
        movie_post = posts[1] if len(posts) > 1 else None  # Ikkinchi post
        if movie_post:
            context.bot.forward_message(update.message.chat_id, MOVIE_CHANNEL, movie_post.message_id)
            update.message.reply_text('Kinoni yubordim!')
        else:
            update.message.reply_text('Ikkinchi post topilmadi.')
    else:
        update.message.reply_text('Noto\'g\'ri kod. Iltimos, to\'g\'ri kodni kiriting.')

def main() -> None:
    updater = Updater("YOUR_BOT_TOKEN")

    dispatcher = updater.dispatcher

    # Komandalar
    dispatcher.add_handler(CommandHandler("start", start))

    # Obuna bo'ldim va kodni tekshirish
    dispatcher.add_handler(CallbackQueryHandler(check_subscription, pattern='subscribed'))
    dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, verify_code))

    updater.start_polling()
    updater.idle()

if name == '__main__':
    main()