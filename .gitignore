import logging
import requests
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

BOT_TOKEN = 'YOUR_BOT_TOKEN'  # Replace with your bot's token

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Send /leech <direct_url> to download and upload the file here.")

async def leech(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) == 0:
        await update.message.reply_text("Usage: /leech <direct_download_url>")
        return

    url = context.args[0]
    await update.message.reply_text(f"Downloading file from: {url}")

    try:
        r = requests.get(url, stream=True)
        filename = url.split("/")[-1]

        with open(filename, "wb") as f:
            for chunk in r.iter_content(chunk_size=8192):
                if chunk:
                    f.write(chunk)

        with open(filename, "rb") as f:
            await update.message.reply_document(f)

    except Exception as e:
        await update.message.reply_text(f"Error: {str(e)}")

if __name__ == '__main__':
    app = ApplicationBuilder().token(BOT_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("leech", leech))

    print("Bot is running...")
    app.run_polling()
pip install gdown

import gdown

async def leech_gdrive(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) == 0:
        await update.message.reply_text("Usage: /gdrive <gdrive_link>")
        return

    url = context.args[0]
    await update.message.reply_text(f"Downloading from Google Drive...")

    try:
        output = 'downloaded_file_from_gdrive'
        gdown.download(url, output, quiet=False)

        with open(output, "rb") as f:
            await update.message.reply_document(f)

    except Exception as e:
        await update.message.reply_text(f"Error: {str(e)}")

app.add_handler(CommandHandler("gdrive", leech_gdrive))

pip install python-libtorrent

import libtorrent as lt
import time
import os

async def leech_torrent(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) == 0:
        await update.message.reply_text("Usage: /torrent <magnet_link_or_torrent_url>")
        return

    magnet_link = context.args[0]
    await update.message.reply_text("Starting torrent download...")

    try:
        ses = lt.session()
        ses.listen_on(6881, 6891)
        params = {
            'save_path': './downloads/',
            'storage_mode': lt.storage_mode_t(2)
        }

        if magnet_link.startswith("magnet:"):
            handle = lt.add_magnet_uri(ses, magnet_link, params)
        else:
            r = requests.get(magnet_link)
            with open("temp.torrent", "wb") as f:
                f.write(r.content)
            info = lt.torrent_info("temp.torrent")
            handle = ses.add_torrent({'ti': info, 'save_path': './downloads/'})

        while not handle.has_metadata():
            time.sleep(1)

        while handle.status().state != lt.torrent_status.seeding:
            s = handle.status()
            print(f'Download: {s.progress * 100:.2f}%')
            time.sleep(5)

        files = handle.get_torrent_info().files()
        for i in range(files.num_files()):
            file_path = os.path.join('./downloads', files.file_path(i))
            with open(file_path, "rb") as f:
                await update.message.reply_document(f)

    except Exception as e:
        await update.message.reply_text(f"Error: {str(e)}")

app.add_handler(CommandHandler("torrent", leech_torrent))
