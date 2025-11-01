import asyncio
import logging
import os
import random

from aiogram import Bot, Dispatcher,Router,types
from aiogram.enums import ParseMode
from aiogram.types import Message
from aiogram.utils.markdown import hbold
from aiogram.filters import CommandStart, Command
from aiogram.client.session.aiohttp import AiohttpSession
from aiogram.utils.keyboard import InlineKeyboardBuilder


BOT_TOKEN = "8312871309:AAE95ryzpj65Dk9G-PFwlKYuXOIDBh4OgSA"

if not BOT_TOKEN:
    exit("Error: No token provided")

logging.basicConfig(level=logging.INFO)
bot = Bot(token=BOT_TOKEN)
dp = Dispatcher()

# ������� � ������
questions = [
    {
        "question": "Выберите а",
        "options": ["а", "б", "в", "г"],
        "correct_answer": "а",
    },
    {
        "question": "Выберите б",
        "options": ["а", "б", "в", "г"],
        "correct_answer": "б",
    },
    {
        "question": "Ввыберите г",
        "options": ["а", "б", "в", "г"],
        "correct_answer": "г",
    },
    {
        "question": "Выберите в",
        "options": ["а", "б", "в", "г"],
        "correct_answer": "в",
    },
    {
        "question": "выберите д",
        "options": ["а", "д", "в", "г"],
        "correct_answer": "д",
    },
    {
        "question": "выберите ь",
        "options": ["а", "б", "ь", "в"],
        "correct_answer": "����",
    },
    {
        "question": "выберитеа",
        "options": ["f", "а", "п", "н"],
        "correct_answer": "а",
    },
        {
        "question": "выберите б",
        "options": ["б", "нр", "ывап", "вапа"],
        "correct_answer": "б",
    },
    {
        "question": "выберите г",
        "options": ["ыва", "г", "ыва", "а"],
        "correct_answer": "г",
    },    
    {
        "question": "конец?",
        "options": ["конец", "нет.", "нет", "н48735435634853475е734643535т345345"],
        "correct_answer": "конец",
    }
]

# ��������� ������������
user_states = {}


async def send_question(chat_id, question_data, question_number):
    builder = InlineKeyboardBuilder()
    for option in question_data["options"]:
        builder.button(
            text=option, callback_data=f"answer:{question_data['correct_answer']}:{option}"
        )
    builder.adjust_cols(2)

    await bot.send_message(
        chat_id,
        f"������ {question_number}/{len(questions)}: {question_data['question']}",
        reply_markup=builder.as_markup(),
    )


@dp.message(CommandStart())
async def start_command(message: types.Message):
    user_id = message.from_user.id
    user_states[user_id] = {
        "score": 0,
        "question_number": 0,
        "questions": random.sample(questions, 10),  # �������� 10 ��������� ��������
    }
    await message.answer("Привет! Нажми /quiz!")
    await send_question(
        message.chat.id, user_states[user_id]["questions"][0], 1
    )  # ���������� ������ ������


@dp.callback_query(lambda c: c.data.startswith("answer:"))
async def answer_callback(callback_query: types.CallbackQuery):
    user_id = callback_query.from_user.id
    _, correct_answer, user_answer = callback_query.data.split(":")
    user_states[user_id]["question_number"] += 1
    question_number = user_states[user_id]["question_number"]
    chat_id = callback_query.message.chat.id

    if user_answer == correct_answer:
        user_states[user_id]["score"] += 1
        await bot.send_message(chat_id, "верно!")
    else:
        await bot.send_message(
            chat_id, f"неверно. правильный ответ: {correct_answer}"
        )

    if question_number < len(user_states[user_id]["questions"]):
        await send_question(
            chat_id,
            user_states[user_id]["questions"][question_number],
            question_number + 1,
        )
    else:
        score = user_states[user_id]["score"]
        await bot.send_message(
            chat_id,
            f"ваш результат: {score}/{len(user_states[user_id]['questions'])}",
        )
        del user_states[user_id]  # ������� ��������� ������������


async def main() -> None:
    session = AiohttpSession(proxy="http://proxy.server:3128")
    bot = Bot(BOT_TOKEN, parse_mode=ParseMode.HTML, session=session)
    await dp.start.polling(bot)
    

if __name__ == "__main__":
    try:
        asyncio.run(main())
    except (KeyboardInterrupt, SystemExit):
        print("бот закончил работать")
