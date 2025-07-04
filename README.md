fastapi
uvicorn
aiogram
opencv-python
import cv2
import numpy as np
import tempfile

def analyze_chart(image_bytes):
    with tempfile.NamedTemporaryFile(suffix=".jpg", delete=False) as tmp:
        tmp.write(image_bytes)
        tmp_path = tmp.name

    image = cv2.imread(tmp_path, cv2.IMREAD_GRAYSCALE)
    blurred = cv2.GaussianBlur(image, (5, 5), 0)
    edges = cv2.Canny(blurred, 50, 150)
    color_result = cv2.cvtColor(edges, cv2.COLOR_GRAY2BGR)

    _, buffer = cv2.imencode('.jpg', color_result)
    return buffer.tobytes()
    import logging
import os
from aiogram import Bot, Dispatcher, types
from analyze_chart import analyze_chart
import io

TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")
if not TELEGRAM_TOKEN:
    raise ValueError("TELEGRAM_TOKEN environment variable not set")

bot = Bot(token=TELEGRAM_TOKEN)
dp = Dispatcher(bot)

logging.basicConfig(level=logging.INFO)

@dp.message_handler(content_types=types.ContentType.PHOTO)
async def handle_photo(message: types.Message):
    logging.info("Received photo from user")
    photo = message.photo[-1]
    photo_bytes = await photo.download(destination=io.BytesIO())
    photo_bytes.seek(0)
    result_bytes = analyze_chart(photo_bytes.read())
    result_file = io.BytesIO(result_bytes)
    result_file.name = "analyzed_chart.jpg"

    await message.reply_photo(result_file, caption="✅ تم تحليل الشارت!")

async def on_startup(_):
    logging.info("Bot is online and polling...")

def start_bot():
    from aiogram import executor
    executor.start_polling(dp, skip_updates=True, on_startup=on_startup)
    from fastapi import FastAPI
import asyncio
from bot import start_bot

app = FastAPI()

@app.on_event("startup")
async def startup_event():
    loop = asyncio.get_event_loop()
    loop.create_task(asyncio.to_thread(start_bot))

@app.get("/")
async def root():
    return {"status": "Chart Analyzer Bot is running."}
    __pycache__/
*.pyc
.env
# Chart Pandora Bot

✅ بوت تيليجرام يعمل على FastAPI + Aiogram

- يستقبل صور الشارت من المستخدم في تيليجرام.
- يحللها (Edge Detection) باستخدام OpenCV.
- يرسل النتيجة للمستخدم.

## Requirements
- fastapi
- uvicorn
- aiogram
- opencv-python

## Local Run
pip install -r requirements.txt
uvicorn main:app --reload
