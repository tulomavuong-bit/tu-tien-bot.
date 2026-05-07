import discord
from discord.ext import commands
import json
import random
import asyncio
import os

# ================= TOKEN =================

TOKEN = os.getenv("TOKEN")

# ================= CONFIG =================

WELCOME_CHANNEL = 1491037295539781712
LEAVE_CHANNEL = 1493853835352080495

DATA_FILE = "data.json"

MAX_LEVEL = 250
BASE_XP = 50

# ================= BOT =================

intents = discord.Intents.all()

bot = commands.Bot(
    command_prefix="Q",
    intents=intents,
    help_command=None
)

# ================= DATA =================

def load_data():
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except:
        return {}

data = load_data()

def save_data():
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=4, ensure_ascii=False)

# ================= USER =================

def get_user(uid):

    uid = str(uid)

    if uid not in data:

        data[uid] = {
            "xp": 0,
            "level": 1,
            "stone": 0,
            "sect": "Nghịch Hà Tông"
        }

    return data[uid]

# ================= XP =================

def xp_need(level):
    return int(BASE_XP * (1.2 ** (level - 1)))

def add_xp(user):

    if user["level"] >= MAX_LEVEL:
        return

    user["xp"] += random.randint(8, 15)

    while user["xp"] >= xp_need(user["level"]):

        user["xp"] -= xp_need(user["level"])

        user["level"] += 1

        if user["level"] > MAX_LEVEL:
            user["level"] = MAX_LEVEL
            user["xp"] = 0

# ================= RANK =================

RANKS = [
    (1, "Phàm Nhân"),
    (10, "Luyện Khí"),
    (30, "Trúc Cơ"),
    (60, "Kim Đan"),
    (100, "Nguyên Anh"),
    (140, "Hóa Thần"),
    (180, "Luyện Hư"),
    (210, "Độ Kiếp"),
    (235, "Đại Thừa"),
    (250, "Tiên Nhân")
]

def get_rank(level):

    for req, rank in reversed(RANKS):

        if level >= req:
            return rank

    return "Phàm Nhân"

# ================= TIỂU CẢNH GIỚI =================

def get_sub_rank(level):

    percent = level / MAX_LEVEL

    if percent < 0.25:
        return "Sơ Kỳ"

    elif percent < 0.5:
        return "Trung Kỳ"

    elif percent < 0.75:
        return "Hậu Kỳ"

    else:
        return "Đại Viên Mãn"

# ================= BAR =================

def bar(current, maximum, length=20):

    filled = int(length * current / maximum)

    return "█" * filled + "░" * (length - filled)

# ================= HP BAR =================

def hp_bar(hp):

    filled = int(hp / 10)

    return (
        "🟥" * filled +
        "⬛" * (10 - filled) +
        f" {hp}/100"
    )

# ================= COOLDOWN =================

cooldown = {}

# ================= EVENTS =================

@bot.event
async def on_ready():

    print(f"✅ {bot.user} đã online")

@bot.event
async def on_message(message):

    if message.author.bot:
        return

    uid = str(message.author.id)

    user = get_user(uid)

    now = asyncio.get_event_loop().time()

    if uid not in cooldown or now - cooldown[uid] > 5:

        add_xp(user)

        cooldown[uid] = now

        save_data()

    await bot.process_commands(message)

# ================= WELCOME =================

@bot.event
async def on_member_join(member):

    ch = bot.get_channel(WELCOME_CHANNEL)

    if not ch:
        return

    embed = discord.Embed(
        description=(
            f"🌸 **Đạo hữu giáng lâm**\n\n"
            f"⚡ {member.mention} đã bước vào Tu Tiên Giới\n\n"
            f"🔥 Con đường trường sinh bắt đầu..."
        ),
        color=0xff66cc
    )

    embed.set_author(
        name=member.name,
        icon_url=member.display_avatar.url
    )

    embed.set_thumbnail(
        url=member.display_avatar.url
    )

    await ch.send(embed=embed)

# ================= LEAVE =================

@bot.event
async def on_member_remove(member):

    ch = bot.get_channel(LEAVE_CHANNEL)

    if not ch:
        return

    embed = discord.Embed(
        description=(
            f"🍃 **Đạo hữu từ biệt**\n\n"
            f"💭 {member.name} đã rời khỏi thế giới\n\n"
            f"⚰️ Hành trình đã kết thúc..."
        ),
        color=0x2f3136
    )

    embed.set_author(
        name=member.name,
        icon_url=member.display_avatar.url
    )

    embed.set_thumbnail(
        url=member.display_avatar.url
    )

    await ch.send(embed=embed)

# ================= PROFILE =================

@bot.command()
async def profile(ctx, member: discord.Member = None):

    member = member or ctx.author

    user = get_user(member.id)

    level = user["level"]

    need = xp_need(level)

    rank = get_rank(level)

    sub = get_sub_rank(level)

    embed = discord.Embed(
        title=f"👤 {member.name}",
        color=0x00ffcc
    )

    embed.set_thumbnail(
        url=member.display_avatar.url
    )

    embed.add_field(
        name="🏯 Tông Môn",
        value=user["sect"],
        inline=False
    )

    embed.add_field(
        name="🎴 Cảnh Giới",
        value=f"{rank} • {sub}",
        inline=False
    )

    embed.add_field(
        name="⭐ Level",
        value=level
    )

    embed.add_field(
        name="💎 Linh Thạch",
        value=user["stone"]
    )

    embed.add_field(
        name="XP",
        value=f"{user['xp']} / {need}\n{bar(user['xp'], need)}",
        inline=False
    )

    await ctx.send(embed=embed)

# ================= TOP =================

@bot.command()
async def top(ctx):

    if not data:
        return await ctx.send("❌ Chưa có dữ liệu")

    users = sorted(
        data.items(),
        key=lambda x: x[1]["level"],
        reverse=True
    )[:10]

    embed = discord.Embed(
        title="🏆 BXH Tu Tiên",
        description="⚡ Cường giả mạnh nhất",
        color=0xffd700
    )

    for i, (uid, u) in enumerate(users, start=1):

        member = ctx.guild.get_member(int(uid))

        if member:
            name = member.name
            avatar = member.display_avatar.url
        else:
            name = f"User {uid}"
            avatar = None

        level = u["level"]

        rank = get_rank(level)

        sub = get_sub_rank(level)

        medal = "✨"

        if i == 1:
            medal = "🥇"

        elif i == 2:
            medal = "🥈"

        elif i == 3:
            medal = "🥉"

        embed.add_field(
            name=f"{medal} TOP {i} • {name}",
            value=(
                f"🏯 {u['sect']}\n"
                f"🎴 {rank} • {sub}\n"
                f"⭐ Level: {level}"
            ),
            inline=False
        )

        if i == 1 and avatar:
            embed.set_thumbnail(url=avatar)

    await ctx.send(embed=embed)

# ================= DAILY =================

@bot.command()
async def daily(ctx):

    user = get_user(ctx.author.id)

    reward = random.randint(50, 120)

    user["stone"] += reward

    save_data()

    await ctx.send(
        f"💎 {ctx.author.mention} nhận {reward} linh thạch!"
    )

# ================= SLOT =================

@bot.command(aliases=["slots"])
async def slot(ctx, amount: int):

    user = get_user(ctx.author.id)

    if amount <= 0:
        return await ctx.send("❌ Số tiền không hợp lệ")

    if user["stone"] < amount:
        return await ctx.send("❌ Không đủ linh thạch")

    user["stone"] -= amount

    symbols = [
        "🍒",
        "🍋",
        "💎",
        "⭐",
        "🍀",
        "🔔",
        "7️⃣",
        "🔥"
    ]

    result = [
        random.choice(symbols),
        random.choice(symbols),
        random.choice(symbols)
    ]

    reels = ["❔", "❔", "❔"]

    def render():

        line = f"{reels[0]} │ {reels[1]} │ {reels[2]}"

        return (
            "```fix\n"
            "╔══════════════════════════════╗\n"
            "║      🎰 SLOT MACHINE 🎰      ║\n"
            "╠══════════════════════════════╣\n"
            f"║{line.center(30)}║\n"
            "╠══════════════════════════════╣\n"
            f"║ 💎 Bet: {str(amount).center(18)} ║\n"
            "╚══════════════════════════════╝\n"
            "```"
        )

    msg = await ctx.send(render())

    speed = 0.05

    for _ in range(18):

        reels[0] = random.choice(symbols)

        await msg.edit(content=render())

        await asyncio.sleep(speed)

        speed += 0.003

    reels[0] = result[0]

    await msg.edit(content=render())

    speed = 0.05

    for _ in range(24):

        reels[1] = random.choice(symbols)

        await msg.edit(content=render())

        await asyncio.sleep(speed)

        speed += 0.003

    reels[1] = result[1]

    await msg.edit(content=render())

    speed = 0.05

    for _ in range(30):

        reels[2] = random.choice(symbols)

        await msg.edit(content=render())

        await asyncio.sleep(speed)

        speed += 0.003

    reels[2] = result[2]

    await msg.edit(content=render())

    if result[0] == result[1] == result[2]:

        win = amount * 5

        user["stone"] += win

        embed = discord.Embed(
            title="🎉 JACKPOT !!!",
            description=f"{ctx.author.mention} thắng {win:,} 💎",
            color=0xffd700
        )

        await ctx.send(embed=embed)

    else:

        embed = discord.Embed(
            title="☹️ Thua rồi...",
            description=f"{ctx.author.mention} mất {amount:,} 💎",
            color=0xff4444
        )

        await ctx.send(embed=embed)

    save_data()

# ================= PVP =================

@bot.command()
async def pvp(ctx, opponent: discord.Member):

    if opponent.bot:
        return await ctx.send("❌ Không thể đấu bot")

    if opponent == ctx.author:
        return await ctx.send("❌ Không thể tự đấu")

    hp1 = 100
    hp2 = 100

    msg = await ctx.send("⚔️ Trận chiến bắt đầu!")

    while hp1 > 0 and hp2 > 0:

        dmg1 = random.randint(10, 20)
        dmg2 = random.randint(10, 20)

        hp2 -= dmg1
        hp1 -= dmg2

        hp1 = max(hp1, 0)
        hp2 = max(hp2, 0)

        embed = discord.Embed(
            title="⚔️ PvP",
            color=0xff0000
        )

        embed.add_field(
            name=ctx.author.name,
            value=hp_bar(hp1),
            inline=False
        )

        embed.add_field(
            name=opponent.name,
            value=hp_bar(hp2),
            inline=False
        )

        await msg.edit(embed=embed)

        await asyncio.sleep(1)

    winner = ctx.author if hp1 > hp2 else opponent

    await ctx.send(f"🏆 {winner.mention} chiến thắng!")

# ================= HELP =================

@bot.command()
async def help(ctx):

    embed = discord.Embed(
        title="📖 Hệ Thống Lệnh Tu Tiên",
        description="✨ Tất cả lệnh bắt đầu bằng Q",
        color=0x9b59b6
    )

    embed.add_field(
        name="👤 Tu Luyện",
        value=(
            "• Qprofile (@user)\n"
            "• Qtop"
        ),
        inline=False
    )

    embed.add_field(
        name="💎 Linh Thạch",
        value="• Qdaily",
        inline=False
    )

    embed.add_field(
        name="🎰 Minigame",
        value="• Qslot <tiền>",
        inline=False
    )

    embed.add_field(
        name="⚔️ PvP",
        value="• Qpvp @user",
        inline=False
    )

    embed.add_field(
        name="🏯 Tông Môn",
        value="• Nghịch Hà Tông",
        inline=False
    )

    embed.add_field(
        name="🌸 Hệ Thống",
        value=(
            "• Welcome / Leave\n"
            "• XP cooldown 5s\n"
            "• Rank + Tiểu cảnh giới"
        ),
        inline=False
    )

    await ctx.send(embed=embed)

# ================= RUN =================

bot.run(TOKEN)
