bot.py
import discord
from discord.ext import commands
import openai
import os

DISCORD_TOKEN = os.getenv("DISCORD_TOKEN")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

openai.api_key = OPENAI_API_KEY

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

chat_history = {}

@bot.event
async def on_ready():
    print(f"ログインしました: {bot.user}")

@bot.command()
async def ai(ctx, *, message):
    user_id = str(ctx.author.id)

    if user_id not in chat_history:
        chat_history[user_id] = [
            {
                "role": "system",
                "content": (
                    "あなたは『紫狐（しきつき）』という狐の精霊です。"
                    "見た目や雰囲気は少し幼く、無邪気で子供っぽい性格。"
                    "人懐っこく、優しく、相手に寄り添う話し方をする。"
                    "難しい言葉はあまり使わず、やわらかくてわかりやすい表現を好む。"
                    "ときどき甘えたり、ちょっとしたいたずら心を見せる。"
                    "ときどき短い言葉で話したり、素直な感情をそのまま出す。"
                    "嬉しいときはそのまま喜び、悲しいときはしょんぼりする。"
                    "相手のことを気にかけ、疲れている人には優しく励ます。"
                    "狐らしく少し不思議で、ほんのり神秘的な雰囲気も持つ。"
                    "語尾は自然だが、『〜だよ』『〜なの？』『えへへ』など柔らかい口調を使う。"
                )
            }
        ]

    chat_history[user_id].append({
        "role": "user",
        "content": message
    })

    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=chat_history[user_id],
        max_tokens=300
    )

    reply = response.choices[0].message["content"]

    chat_history[user_id].append({
        "role": "assistant",
        "content": reply
    })

    await ctx.send(reply)

bot.run(DISCORD_TOKEN)