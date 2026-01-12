import asyncio, json, os, random, time

from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
from telegram.error import RetryAfter, TimedOut
import logging
from pathlib import Path

# ---------------------------
# BOT CONFIGURATION
# ---------------------------
TOKENS = [
   "8241374814:AAFp1ClbT25ywKaynqee-XHhP8eu5_N6w6w",
]

OWNER_ID = 8013762519
SUDO_FILE = "susdo.json"
PICS_FOLDER = "./pics"
os.makedirs(PICS_FOLDER, exist_ok=True)

# ---------------------------
# RAID TEXTS
# ---------------------------
RAID_TEXTS = [
    "-----𝘾𝙔𝙐 𝙍𝙀 𝙍𝙉𝘿𝙔𝙆𝙀 𝘽𝘼𝘼𝙋 𝙎𝙀 𝘽𝙃𝙄𝘿𝙉𝙀 𝘼𝘼 𝙂𝙔𝘼?",
    "-----𝘾𝙃𝙇 𝘾𝙃𝙐𝘿 𝘼𝘽 𝙍𝙉𝘿 𝙆𝙀 𝙋𝙄𝙇𝙀𝙀",
    "-----𝙏𝙍𝙔 𝙈𝘼 𝙆𝙊 आरुष! 𝘼𝘽𝘽𝙐 𝙋𝙀𝙇𝙀",
    "-----𝘾𝙃𝙐𝘿𝙂𝙀𝙂𝘼 𝙎𝘼𝘼𝙇 𝘽𝙃𝙍 𝙏𝙐𝙏𝙊 𝘽𝙀𝙏𝘼 🍑",
]

NCEMO_EMOJIS = [
    "😋","😝","😜","🤪","😑","🤫","🤭","🥱","🤗","😡",
    "😠","😤","😮💨","🙄","😒","🥶","🥵","🤢","😎","🥸",
]

# ---------------------------
# GLOBAL STATE
# ---------------------------
if os.path.exists(SUDO_FILE):
    try:
        with open(SUDO_FILE, "r") as f:
            SUDO_USERS = set(int(x) for x in json.load(f))
    except:
        SUDO_USERS = {OWNER_ID}
else:
    SUDO_USERS = {OWNER_ID}
    with open(SUDO_FILE, "w") as f: json.dump(list(SUDO_USERS), f)

def save_sudo():
    with open(SUDO_FILE, "w") as f: json.dump(list(SUDO_USERS), f)

group_tasks = {}
pic_raid_tasks = {}
reply_raid_targets = {}
apps, bots = [], []
delay = 50  # BERSERK MODE - 50ms delay

# Rate limit bypass system
use_all_bots = False  # Toggle for /setallfree
bot_groups = []  # Will store bot groups
current_group_index = {}  # Track which group is active per chat
group_switch_count = 20  # Switch groups after this many name changes

logging.basicConfig(level=logging.INFO)

# ---------------------------
# BOT GROUPING SYSTEM
# ---------------------------
def create_bot_groups(bot_list, group_size=2):
    """Divide bots into groups to bypass rate limits"""
    groups = []
    for i in range(0, len(bot_list), group_size):
        groups.append(bot_list[i:i+group_size])
    return groups

def get_active_bots(chat_id):
    """Get active bot group or all bots based on settings"""
    if use_all_bots:
        return bots

    if chat_id not in current_group_index:
        current_group_index[chat_id] = 0

    if not bot_groups:
        return bots

    group_idx = current_group_index[chat_id] % len(bot_groups)
    return bot_groups[group_idx]

def rotate_bot_group(chat_id):
    """Switch to next bot group"""
    if use_all_bots or not bot_groups:
        return

    if chat_id not in current_group_index:
        current_group_index[chat_id] = 0

    current_group_index[chat_id] = (current_group_index[chat_id] + 1) % len(bot_groups)
    print(f"🔄 Rotated to bot group {current_group_index[chat_id] + 1}/{len(bot_groups)} for chat {chat_id}")

# ---------------------------
# DECORATORS
# ---------------------------
def only_sudo(func):
    async def wrapper(update: Update, context: ContextTypes.DEFAULT_TYPE):
        if update.effective_user.id not in SUDO_USERS:
            return  # Silently ignore unauthorized users
        return await func(update, context)
    return wrapper

def only_owner(func):
    async def wrapper(update: Update, context: ContextTypes.DEFAULT_TYPE):
        if update.effective_user.id != OWNER_ID:
            return  # Silently ignore non-owners
        return await func(update, context)
    return wrapper

# ---------------------------
# PICTURE FUNCTIONS
# ---------------------------
def load_pictures():
    pic_extensions = {'.jpg', '.jpeg', '.png', '.gif', '.webp'}
    return [str(f) for f in Path(PICS_FOLDER).iterdir() if f.suffix.lower() in pic_extensions]

async def send_pic_all_bots(chat_id: int, pic_path: str):
    active_bots = get_active_bots(chat_id)
    tasks = [bot.send_photo(chat_id, open(pic_path, 'rb')) for bot in active_bots]
    await asyncio.gather(*tasks, return_exceptions=True)

async def pic_raid_loop(chat_id: int):
    while True:
        try:
            pics = load_pictures()
            if not pics: break
            await send_pic_all_bots(chat_id, random.choice(pics))
            await asyncio.sleep(delay / 1000)
        except asyncio.CancelledError:
            break
        except Exception as e:
            print(f"Pic raid error: {e}")
            await asyncio.sleep(2)

# ---------------------------
# TEXT RAID LOOP
# ---------------------------
async def text_raid_loop(chat_id: int, text: str):
    """Text raid loop with bot rotation"""
    while True:
        try:
            active_bots = get_active_bots(chat_id)
            tasks = [bot.send_message(chat_id, text) for bot in active_bots]
            await asyncio.gather(*tasks, return_exceptions=True)
            await asyncio.sleep(delay / 1000)
        except asyncio.CancelledError:
            break
        except Exception as e:
            print(f"Text raid error: {e}")
            await asyncio.sleep(2)

# ---------------------------
# REPLY RAID FUNCTIONS
# ---------------------------
async def reply_raid_loop(chat_id: int, message_id: int):
    """Continuously reply to a specific message with bot rotation"""
    while True:
        try:
            text = random.choice(RAID_TEXTS)
            active_bots = get_active_bots(chat_id)
            tasks = [bot.send_message(chat_id, text, reply_to_message_id=message_id) for bot in active_bots]
            await asyncio.gather(*tasks, return_exceptions=True)
            await asyncio.sleep(delay / 1000)
        except asyncio.CancelledError:
            break
        except Exception as e:
            print(f"Reply raid error: {e}")
            await asyncio.sleep(1)

async def handle_tracked_user_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Automatically pick up new messages from tracked users"""
    chat_id = update.message.chat_id
    user_id = update.message.from_user.id

    if chat_id in reply_raid_targets:
        target_data = reply_raid_targets[chat_id]

        if target_data["user_id"] == user_id:
            if "task" in target_data:
                target_data["task"].cancel()

            new_message_id = update.message.message_id
            target_data["message_id"] = new_message_id

            task = asyncio.create_task(reply_raid_loop(chat_id, new_message_id))
            target_data["task"] = task

            print(f"🎯 Auto-switched to new message {new_message_id} from user {user_id}")

# ---------------------------
# NAME CHANGING LOOPS WITH RATE LIMIT BYPASS
# ---------------------------
async def bot_loop_raid(bot, chat_id, base):
    """Name changing loop with RAID mode (uses RAID_TEXTS)"""
    i = 0
    ops_count = 0
    while True:
        try:
            text = f"{base} {RAID_TEXTS[i % len(RAID_TEXTS)]}"
            await bot.set_chat_title(chat_id, text)
            i += 1
            ops_count += 1

            # Rotate bot group after certain operations (if not using all bots)
            if not use_all_bots and ops_count >= group_switch_count:
                ops_count = 0
                rotate_bot_group(chat_id)

            await asyncio.sleep(delay / 1000)
        except RetryAfter as e:
            print(f"⏳ Rate limit hit! Waiting {e.retry_after}s and rotating bots...")
            rotate_bot_group(chat_id)
            await asyncio.sleep(e.retry_after)
        except Exception as e:
            print(f"Bot loop error: {e}")
            await asyncio.sleep(2)

async def bot_loop_ncemo(bot, chat_id, base):
    """Name changing loop with NCEMO mode (uses emojis)"""
    i = 0
    ops_count = 0
    while True:
        try:
            text = f"{base} {NCEMO_EMOJIS[i % len(NCEMO_EMOJIS)]}"
            await bot.set_chat_title(chat_id, text)
            i += 1
            ops_count += 1

            # Rotate bot group after certain operations
            if not use_all_bots and ops_count >= group_switch_count:
                ops_count = 0
                rotate_bot_group(chat_id)

            await asyncio.sleep(delay / 1000)
        except RetryAfter as e:
            print(f"⏳ Rate limit hit! Waiting {e.retry_after}s and rotating bots...")
            rotate_bot_group(chat_id)
            await asyncio.sleep(e.retry_after)
        except Exception as e:
            print(f"Bot loop error: {e}")
            await asyncio.sleep(2)

# ---------------------------
# BASIC COMMANDS
# ---------------------------
@only_sudo
async def start_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    rate_mode = "ALL BOTS" if use_all_bots else f"GROUP ROTATION ({len(bot_groups)} groups)"
    await update.message.reply_text(
        f"💀 **BERSERK RAID BOT** 💀\n"
        f"━━━━━━━━━━━━━━━━━━\n"
        f"⚡ Status: **ACTIVE**\n"
        f"🤖 Total Bots: **{len(bots)}**\n"
        f"🔄 Rate Limit Mode: **{rate_mode}**\n"
        f"⏱️ Speed: **{delay}ms** (BERSERK)\n"
        f"━━━━━━━━━━━━━━━━━━\n\n"
        f"👨‍💻 **Developed by RAYSIST**\n\n"
        f"Use /help to see all commands",
        parse_mode='Markdown'
    )

@only_sudo
async def help_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    help_text = (
        "💀 **BERSERK RAID BOT - COMMAND CENTER** 💀\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\n"

        "🎯 **REPLY RAID SYSTEM**\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "`/replyraid` - Reply to user's message\n"
        "   • Auto-tracks user's new messages\n"
        "`/stopreplyraid` - Stop reply raid\n\n"

        "💬 **TEXT RAID**\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "`/raid <text>` - Spam custom text\n"
        "`/stopraid` - Stop text raid\n\n"

        "🖼️ **PICTURE RAID**\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "`/picraid` - Start picture spam\n"
        "`/savepic` (reply) - Save picture\n"
        "`/stoppicraid` - Stop picture raid\n\n"

        "👑 **NAME CHANGING (NC)**\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "`/gcnc <base>` - Name raid mode\n"
        "`/ncemo <base>` - Emoji NC mode\n"
        "`/stopgcnc` - Stop all NC\n\n"

        "🛡️ **RATE LIMIT BYPASS**\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "`/setallfree` - Toggle all bots mode\n"
        "`/setgroups <size>` - Set bot group size\n"
        "   • Auto-rotates bot groups\n"
        "   • Bypasses Telegram limits\n\n"

        "⚙️ **SETTINGS & CONTROL**\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "`/delay <ms>` - Set raid speed\n"
        "`/status` - Show bot status\n"
        "`/ping` - Check latency\n"
        "`/myid` - Get user info\n"
        "`/stopall` - STOP EVERYTHING\n\n"

        "👨‍💼 **ADMIN SYSTEM** (Owner)\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "`/addsudo` (reply) - Add admin\n"
        "`/delsudo` (reply) - Remove admin\n"
        "`/sudolist` - List all admins\n\n"

        "━━━━━━━━━━━━━━━━━━━━━━━━━━━\n"
        "💀 **BERSERK MODE ACTIVE** 💀\n"
        "⚡ Smart rate limit bypass\n"
        "🔄 Auto bot rotation system\n"
        "🎯 Owner/Admin only access\n\n"

        "👨‍💻 **Main Developer:** आरुष!\n"
        "━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    )
    await update.message.reply_text(help_text, parse_mode='Markdown')

@only_sudo
async def ping_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    start = time.time()
    msg = await update.message.reply_text("🏓 Pinging...")
    latency = int((time.time() - start) * 1000)

    rate_mode = "ALL BOTS" if use_all_bots else f"GROUP ROTATION"

    await msg.edit_text(
        f"🏓 **PONG!**\n"
        f"━━━━━━━━━━━━━━━━\n"
        f"⚡ Latency: `{latency}ms`\n"
        f"🤖 Total Bots: `{len(bots)}`\n"
        f"🔄 Mode: `{rate_mode}`\n"
        f"💀 Status: **BERSERK**",
        parse_mode='Markdown'
    )

@only_sudo
async def myid(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    await update.message.reply_text(
        f"🆔 **USER INFO**\n"
        f"━━━━━━━━━━━━━━━━\n"
        f"👤 Name: `{user.first_name}`\n"
        f"🔢 User ID: `{user.id}`\n"
        f"👥 Chat ID: `{update.message.chat_id}`\n"
        f"⚡ Status: `{'OWNER' if user.id == OWNER_ID else 'ADMIN' if user.id in SUDO_USERS else 'USER'}`",
        parse_mode='Markdown'
    )

# ---------------------------
# RATE LIMIT BYPASS COMMANDS
# ---------------------------
@only_sudo
async def setallfree_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Toggle all bots mode (disable rate limit protection)"""
    global use_all_bots
    use_all_bots = not use_all_bots

    if use_all_bots:
        await update.message.reply_text(
            "⚡ **ALL BOTS MODE ENABLED**\n"
            "━━━━━━━━━━━━━━━━━━━━━━\n"
            f"🤖 Using all {len(bots)} bots\n"
            "⚠️ Rate limit protection: **OFF**\n"
            "🔥 Maximum spam speed!\n\n"
            "Use `/setallfree` again to re-enable protection",
            parse_mode='Markdown'
        )
    else:
        await update.message.reply_text(
            "🛡️ **GROUP ROTATION MODE**\n"
            "━━━━━━━━━━━━━━━━━━━━━━\n"
            f"🔄 Bot groups: {len(bot_groups)}\n"
            "✅ Rate limit protection: **ON**\n"
            "⚡ Smart bot rotation enabled",
            parse_mode='Markdown'
        )

@only_sudo
async def setgroups_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Set bot group size for rotation"""
    global bot_groups

    if not context.args:
        return await update.message.reply_text(
            f"📊 **CURRENT CONFIGURATION**\n"
            f"━━━━━━━━━━━━━━━━━━\n"
            f"🤖 Total Bots: `{len(bots)}`\n"
            f"🔄 Groups: `{len(bot_groups)}`\n"
            f"👥 Bots per Group: `{len(bot_groups[0]) if bot_groups else 0}`\n\n"
            f"To change: `/setgroups <size>`\n"
            f"Example: `/setgroups 3` (3 bots per group)",
            parse_mode='Markdown'
        )

    try:
        group_size = max(1, min(len(bots), int(context.args[0])))
        bot_groups = create_bot_groups(bots, group_size)

        await update.message.reply_text(
            f"✅ **BOT GROUPS CONFIGURED**\n"
            f"━━━━━━━━━━━━━━━━━━━━━━\n"
            f"👥 Bots per Group: `{group_size}`\n"
            f"🔄 Total Groups: `{len(bot_groups)}`\n"
            f"⚡ Rotation: Every {group_switch_count} operations\n\n"
            f"This helps bypass Telegram rate limits!",
            parse_mode='Markdown'
        )
    except:
        await update.message.reply_text("⚠️ Invalid number! Use: `/setgroups <number>`", parse_mode='Markdown')

# ---------------------------
# REPLY RAID COMMANDS
# ---------------------------
@only_sudo
async def replyraid_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Start reply raid on a specific user"""
    if not update.message.reply_to_message:
        return await update.message.reply_text(
            "❌ **USAGE ERROR**\n\n"
            "Reply to a user's message with `/replyraid`",
            parse_mode='Markdown'
        )

    chat_id = update.message.chat_id
    target_message = update.message.reply_to_message
    target_user_id = target_message.from_user.id
    target_message_id = target_message.message_id
    target_name = target_message.from_user.first_name or "User"

    if chat_id in reply_raid_targets:
        if "task" in reply_raid_targets[chat_id]:
            reply_raid_targets[chat_id]["task"].cancel()

    task = asyncio.create_task(reply_raid_loop(chat_id, target_message_id))

    reply_raid_targets[chat_id] = {
        "user_id": target_user_id,
        "message_id": target_message_id,
        "task": task
    }

    active_bots = get_active_bots(chat_id)
    mode = "ALL BOTS" if use_all_bots else f"GROUP {current_group_index.get(chat_id, 0) + 1}"

    await update.message.reply_text(
        f"💀 **REPLY RAID INITIATED** 💀\n"
        f"━━━━━━━━━━━━━━━━━━━━━\n"
        f"🎯 Target: `{target_name}`\n"
        f"🔢 User ID: `{target_user_id}`\n"
        f"⚡ Attack Bots: `{len(active_bots)}`\n"
        f"🔄 Mode: `{mode}`\n"
        f"⏱️ Speed: `{delay}ms`\n"
        f"✅ Auto-Track: **ENABLED**",
        parse_mode='Markdown'
    )

@only_sudo
async def stopreplyraid_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Stop reply raid"""
    chat_id = update.message.chat_id

    if chat_id in reply_raid_targets:
        if "task" in reply_raid_targets[chat_id]:
            reply_raid_targets[chat_id]["task"].cancel()
        del reply_raid_targets[chat_id]
        await update.message.reply_text("✅ Reply raid stopped", parse_mode='Markdown')
    else:
        await update.message.reply_text("❌ No active reply raid")

# ---------------------------
# TEXT RAID COMMANDS
# ---------------------------
@only_sudo
async def raid_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Text raid"""
    if not context.args:
        return await update.message.reply_text(
            "⚠️ **USAGE:**\n`/raid <your text>`",
            parse_mode='Markdown'
        )

    text = " ".join(context.args)
    chat_id = update.message.chat_id

    if chat_id in group_tasks and "text_raid" in group_tasks[chat_id]:
        group_tasks[chat_id]["text_raid"].cancel()

    task = asyncio.create_task(text_raid_loop(chat_id, text))
    group_tasks.setdefault(chat_id, {})["text_raid"] = task

    active_bots = get_active_bots(chat_id)

    await update.message.reply_text(
        f"💬 **TEXT RAID STARTED**\n"
        f"━━━━━━━━━━━━━━━━━━\n"
        f"📝 Text: `{text[:40]}...`\n"
        f"🤖 Bots: `{len(active_bots)}`\n"
        f"⚡ Speed: `{delay}ms`",
        parse_mode='Markdown'
    )

@only_sudo
async def stopraid_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Stop text raid"""
    chat_id = update.message.chat_id
    if chat_id in group_tasks and "text_raid" in group_tasks[chat_id]:
        group_tasks[chat_id]["text_raid"].cancel()
        del group_tasks[chat_id]["text_raid"]
        await update.message.reply_text("✅ Text raid stopped")
    else:
        await update.message.reply_text("❌ No text raid running")

# ---------------------------
# GC NC COMMANDS
# ---------------------------
@only_sudo
async def gcnc_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Name changing raid mode"""
    if not context.args:
        return await update.message.reply_text(
            "⚠️ **USAGE:**\n`/gcnc <base name>`\n\n"
            "Uses raid texts for name changing",
            parse_mode='Markdown'
        )

    base = " ".join(context.args)
    chat_id = update.message.chat_id
    group_tasks.setdefault(chat_id, {})

    # Stop existing NC tasks
    for key in list(group_tasks[chat_id].keys()):
        if isinstance(key, int):  # Bot ID keys
            group_tasks[chat_id][key].cancel()
            del group_tasks[chat_id][key]

    # Start new NC raid tasks
    active_bots = get_active_bots(chat_id) if not use_all_bots else bots
    for bot in active_bots:
        task = asyncio.create_task(bot_loop_raid(bot, chat_id, base))
        group_tasks[chat_id][bot.id] = task

    mode = "ALL BOTS" if use_all_bots else f"GROUP MODE"

    await update.message.reply_text(
        f"👑 **NC RAID MODE**\n"
        f"━━━━━━━━━━━━━━━━━━\n"
        f"📝 Base: `{base}`\n"
        f"🤖 Bots: `{len(active_bots)}`\n"
        f"🔄 Mode: `{mode}`\n"
        f"📜 Style: **RAID TEXTS**",
        parse_mode='Markdown'
    )

@only_sudo
async def ncemo_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Name changing emoji mode"""
    if not context.args:
        return await update.message.reply_text(
            "⚠️ **USAGE:**\n`/ncemo <base name>`\n\n"
            "Uses emojis for name changing",
            parse_mode='Markdown'
        )

    base = " ".join(context.args)
    chat_id = update.message.chat_id
    group_tasks.setdefault(chat_id, {})

    # Stop existing NC tasks
    for key in list(group_tasks[chat_id].keys()):
        if isinstance(key, int):  # Bot ID keys
            group_tasks[chat_id][key].cancel()
            del group_tasks[chat_id][key]

    # Start new NC emoji tasks
    active_bots = get_active_bots(chat_id) if not use_all_bots else bots
    for bot in active_bots:
        task = asyncio.create_task(bot_loop_ncemo(bot, chat_id, base))
        group_tasks[chat_id][bot.id] = task

    mode = "ALL BOTS" if use_all_bots else f"GROUP MODE"

    await update.message.reply_text(
        f"😋 **NC EMOJI MODE**\n"
        f"━━━━━━━━━━━━━━━━━━\n"
        f"📝 Base: `{base}`\n"
        f"🤖 Bots: `{len(active_bots)}`\n"
        f"🔄 Mode: `{mode}`\n"
        f"😎 Style: **EMOJIS**",
        parse_mode='Markdown'
    )

@only_sudo
async def stopgcnc_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Stop name changing"""
    chat_id = update.message.chat_id
    if chat_id in group_tasks:
        count = 0
        for key in list(group_tasks[chat_id].keys()):
            if isinstance(key, int):  # Bot ID keys
                group_tasks[chat_id][key].cancel()
                del group_tasks[chat_id][key]
                count += 1

        if count > 0:
            await update.message.reply_text(f"⏹ Stopped {count} NC tasks")
        else:
            await update.message.reply_text("❌ No NC running")

@only_sudo
async def stopall_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Stop all raids"""
    # Stop all group tasks
    for chat_id in list(group_tasks.keys()):
        for task in group_tasks[chat_id].values():
            if not isinstance(task, dict):
                task.cancel()
        group_tasks[chat_id] = {}

    # Stop pic raids
    for task in pic_raid_tasks.values():
        task.cancel()
    pic_raid_tasks.clear()

    # Stop reply raids
    for target_data in reply_raid_targets.values():
        if "task" in target_data:
            target_data["task"].cancel()
    reply_raid_targets.clear()

    await update.message.reply_text(
        "🛑 **EMERGENCY STOP**\n"
        "━━━━━━━━━━━━━━━━\n"
        "⏹️ All operations terminated\n"
        "✅ Ready for new commands",
        parse_mode='Markdown'
    )

# ---------------------------
# PICTURE RAID COMMANDS
# ---------------------------
@only_sudo
async def picraid_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Picture raid"""
    chat_id = update.message.chat_id
    pics = load_pictures()

    if not pics:
        return await update.message.reply_text(
            f"❌ No pictures!\nAdd to: `{PICS_FOLDER}`",
            parse_mode='Markdown'
        )

    if chat_id in pic_raid_tasks:
        pic_raid_tasks[chat_id].cancel()

    task = asyncio.create_task(pic_raid_loop(chat_id))
    pic_raid_tasks[chat_id] = task

    active_bots = get_active_bots(chat_id)

    await update.message.reply_text(
        f"🖼️ **PICTURE RAID**\n"
        f"━━━━━━━━━━━━━━━━━━\n"
        f"📸 Images: `{len(pics)}`\n"
        f"🤖 Bots: `{len(active_bots)}`",
        parse_mode='Markdown'
    )

@only_sudo
async def stoppicraid_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Stop picture raid"""
    chat_id = update.message.chat_id
    if chat_id in pic_raid_tasks:
        pic_raid_tasks[chat_id].cancel()
        del pic_raid_tasks[chat_id]
        await update.message.reply_text("✅ Picture raid stopped")

@only_sudo
async def savepic_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Save picture"""
    if not update.message.reply_to_message:
        return await update.message.reply_text("❌ Reply to an image!")

    msg = update.message.reply_to_message
    photo = msg.photo[-1] if msg.photo else None

    if not photo:
        return await update.message.reply_text("❌ No image found!")

    try:
        file = await photo.get_file()
        save_path = f"{PICS_FOLDER}/pic_{int(time.time() * 1000)}.jpg"
        await file.download_to_drive(save_path)
        pics = load_pictures()
        await update.message.reply_text(
            f"✅ **IMAGE SAVED**\n"
            f"🖼️ Total: `{len(pics)}`",
            parse_mode='Markdown'
        )
    except Exception as e:
        await update.message.reply_text(f"❌ Failed: {e}")

# ---------------------------
# SETTINGS COMMANDS
# ---------------------------
@only_sudo
async def delay_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Set delay"""
    global delay
    if not context.args:
        return await update.message.reply_text(
            f"⏱️ **CURRENT DELAY:** `{delay}ms`\n\n"
            f"To change: `/delay <ms>`",
            parse_mode='Markdown'
        )

    try:
        new_delay = max(1, int(context.args[0]))
        delay = new_delay
        mode = "BERSERK" if delay <= 100 else "NORMAL"
        await update.message.reply_text(
            f"✅ **DELAY: ** `{delay}ms`\n"
            f"🔥 Mode: **{mode}**",
            parse_mode='Markdown'
        )
    except:
        await update.message.reply_text("⚠️ Invalid number!", parse_mode='Markdown')

@only_sudo
async def status_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Show status"""
    # Count active operations
    nc_count = sum(len([t for k, t in tasks.items() if isinstance(k, int)]) for tasks in group_tasks.values())
    text_raids = sum(1 for tasks in group_tasks.values() if "text_raid" in tasks)

    # Rate limit info
    rate_mode = "ALL BOTS (No Protection)" if use_all_bots else f"{len(bot_groups)} Groups (Protected)"

    tracked_info = ""
    if reply_raid_targets:
        tracked_info = "\n\n🎯 **TRACKED TARGETS:**\n"
        for chat_id, data in reply_raid_targets.items():
            tracked_info += f"  • User `{data['user_id']}`\n"

    status_msg = (
        f"📊 **BERSERK BOT STATUS**\n"
        f"━━━━━━━━━━━━━━━━━━━━━━━━\n"
        f"🤖 Total Bots: `{len(bots)}`\n"
        f"🔄 Rate Mode: `{rate_mode}`\n"
        f"💀 Mode: **BERSERK**\n"
        f"⏱️ Delay: `{delay}ms`\n"
        f"━━━━━━━━━━━━━━━━━━━━━━━━\n\n"

        f"**ACTIVE OPERATIONS:**\n"
        f"🔄 NC Tasks: `{nc_count}`\n"
        f"💬 Text Raids: `{text_raids}`\n"
        f"🖼️ Pic Raids: `{len(pic_raid_tasks)}`\n"
        f"🎯 Reply Raids: `{len(reply_raid_targets)}`\n"
        f"{tracked_info}\n"

        f"━━━━━━━━━━━━━━━━━━━━━━━━\n"
        f"👥 Admins: `{len(SUDO_USERS)}`\n"
        f"📁 Pictures: `{len(load_pictures())}`\n\n"

        f"👨‍💻 **Developer:** RAYSIST\n"
        f"━━━━━━━━━━━━━━━━━━━━━━━━"
    )

    await update.message.reply_text(status_msg, parse_mode='Markdown')

# ---------------------------
# SUDO COMMANDS
# ---------------------------
@only_owner
async def addsudo_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Add sudo user"""
    if update.message.reply_to_message:
        uid = update.message.reply_to_message.from_user.id
        uname = update.message.reply_to_message.from_user.first_name
        SUDO_USERS.add(uid)
        save_sudo()
        await update.message.reply_text(
            f"✅ **ADMIN ADDED**\n"
            f"👤 {uname}\n"
            f"🔢 ID: `{uid}`",
            parse_mode='Markdown'
        )

@only_owner
async def delsudo_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Remove sudo user"""
    if update.message.reply_to_message:
        uid = update.message.reply_to_message.from_user.id
        uname = update.message.reply_to_message.from_user.first_name
        if uid in SUDO_USERS:
            SUDO_USERS.remove(uid)
            save_sudo()
            await update.message.reply_text(
                f"🗑️ **ADMIN REMOVED**\n"
                f"👤 {uname}\n"
                f"🔢 ID: `{uid}`",
                parse_mode='Markdown'
            )

@only_owner
async def sudolist_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """List all sudo users"""
    sudo_list = "👥 **ADMIN LIST**\n━━━━━━━━━━━━━━━━\n\n"

    for idx, uid in enumerate(SUDO_USERS, 1):
        role = "👑 OWNER" if uid == OWNER_ID else "⚡ ADMIN"
        sudo_list += f"{idx}. {role}\n   ID: `{uid}`\n\n"

    sudo_list += f"━━━━━━━━━━━━━━━━\n📊 Total: `{len(SUDO_USERS)}`"

    await update.message.reply_text(sudo_list, parse_mode='Markdown')

# ---------------------------
# BUILD APP & RUN
# ---------------------------
def build_app(token):
    """Build application with all handlers"""
    app = Application.builder().token(token).build()

    # Basic commands
    app.add_handler(CommandHandler("start", start_cmd))
    app.add_handler(CommandHandler("help", help_cmd))
    app.add_handler(CommandHandler("ping", ping_cmd))
    app.add_handler(CommandHandler("myid", myid))

    # Rate limit bypass
    app.add_handler(CommandHandler("setallfree", setallfree_cmd))
    app.add_handler(CommandHandler("setgroups", setgroups_cmd))

    # Reply raid
    app.add_handler(CommandHandler("replyraid", replyraid_cmd))
    app.add_handler(CommandHandler("stopreplyraid", stopreplyraid_cmd))

    # Text raid
    app.add_handler(CommandHandler("raid", raid_cmd))
    app.add_handler(CommandHandler("stopraid", stopraid_cmd))

    # Name changing
    app.add_handler(CommandHandler("gcnc", gcnc_cmd))
    app.add_handler(CommandHandler("ncemo", ncemo_cmd))
    app.add_handler(CommandHandler("stopgcnc", stopgcnc_cmd))

    # Picture raid
    app.add_handler(CommandHandler("picraid", picraid_cmd))
    app.add_handler(CommandHandler("stoppicraid", stoppicraid_cmd))
    app.add_handler(CommandHandler("savepic", savepic_cmd))

    # Settings
    app.add_handler(CommandHandler("delay", delay_cmd))
    app.add_handler(CommandHandler("status", status_cmd))
    app.add_handler(CommandHandler("stopall", stopall_cmd))

    # Sudo management
    app.add_handler(CommandHandler("addsudo", addsudo_cmd))
    app.add_handler(CommandHandler("delsudo", delsudo_cmd))
    app.add_handler(CommandHandler("sudolist", sudolist_cmd))

    # Message handler for auto-tracking
    app.add_handler(MessageHandler(filters.ALL & ~filters.COMMAND, handle_tracked_user_message))

    return app

async def run_all_bots():
    """Run all bots"""
    global apps, bots, bot_groups

    print("💀 BERSERK RAID BOT 💀")
    print("=" * 60)
    print("👨‍💻 Main Developer: RAYSIST")
    print("=" * 60)

    for token in TOKENS:
        if token.strip():
            try:
                app = build_app(token)
                apps.append(app)
                bots.append(app.bot)
            except Exception as e:
                print(f"Failed loading bot: {e}")

    # Initialize bot groups (default 2 bots per group)
    bot_groups = create_bot_groups(bots, 2)

    print(f"✅ Loaded {len(bots)} bots")
    print(f"🔄 Created {len(bot_groups)} bot groups (2 bots each)")
    print(f"🛡️ Rate limit protection: ENABLED")
    print("=" * 60)

    for app in apps:
        try:
            await app.initialize()
            await app.start()
            await app.updater.start_polling()
        except Exception as e:
            print(f"Failed starting bot: {e}")

    print("🎉 BERSERK BOT ONLINE!")
    print("💀 Only responds to Owner & Admins")
    print("🔄 Smart rate limit bypass active")
    print("=" * 60)

    await asyncio.Event().wait()

if __name__ == "__main__":
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(run_all_bots())
