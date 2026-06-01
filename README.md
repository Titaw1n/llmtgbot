from mistralai.client import Mistral
import os

import asyncio
import logging
from aiogram import Bot, Dispatcher, types
from aiogram.filters import Command
from aiogram.methods import DeleteWebhook
from aiogram.types import Message


api_key = os.getenv("AI_TOKEN")
TOKEN = os.getenv("TG_TOKEN")
model = "mistral-large-latest"

client = Mistral(api_key=api_key)


logging.basicConfig(level=logging.INFO)
bot = Bot(TOKEN)
dp = Dispatcher()

@dp.message(Command("start"))
async def cmd_start(message: types.Message):
    await message.answer('Привет! Я бот с подключенной нейросетью, отправь свой запрос', parse_mode = 'HTML')

@dp.message(lambda message: message.text)
async def filter_messages(message: Message):
    chat_response = client.chat.complete(
        model = model,
        messages = [
            {
                "role": "system",
                "content": "memori",
            },
            {
                "role": "user",
                "content": message.text,
            },
        ]
    )
    text = chat_response.choices[0].message.content
    await message.answer(text, parse_mode = "Markdown")


async def main():
    await bot(DeleteWebhook(drop_pending_updates=True))
    await dp.start_polling(bot)

if name == "__main__":
    asyncio.run(main())
