import discord
from discord.ext import commands, tasks
import asyncio
import sqlite3
import random
from datetime import datetime
import os

print("Initializing bot...")

# Initialize the bot with intents
intents = discord.Intents.default()
intents.message_content = True  # To allow the bot to read message content

bot = commands.Bot(command_prefix='!', intents=intents)

# Create a database connection
conn = sqlite3.connect('warnings.db')
c = conn.cursor()

# Create a table to store warnings
c.execute('''CREATE TABLE IF NOT EXISTS warnings (
                user_id INTEGER PRIMARY KEY,
                count INTEGER
            )''')
conn.commit()

# List of role IDs allowed to use the warn command
allowed_roles = [
    123456789012345678,  # Replace with actual role IDs
    1267167097587241053
]

# Roles assigned for warnings
warning_roles = {
    1: 1267531389079781376,  # Replace with actual role IDs for first warning
    2: 1267531593564557354,  # Replace with actual role IDs for second warning
    3: 1267531833973805157   # Replace with actual role IDs for third warning
}

# The message you want the bot to send
response_message = ('https://media.discordapp.net/attachments/1223553009850650654/1268251849333674056/25_96CF2C6.gif?ex=66abbf13&is=66aa6d93&hm=fa5cd613a660b251c4d78381ca5cd728cfde30e2d52cddb533f6ea37e94de8e5&')

# List of channel IDs where the bot should respond with the image
allowed_channels = [
    1267422339734310968,
    1267315041368146034,
    1267175072687460463,
    1267115810141245512,
    1267195252532449300,
    1268346783038308362,
    1267568423894909123,
    1267914771915870250,
    1268676315620311133,
    1268676371043975332,
    1268994114196869283
    
]

# Channel ID where the bot should send the duaas
duaa_channel_id = 1267245715286003844

# List of duaas to send every half hour
duaas = [
    "رَبَّنَا لاَ تُؤَاخِذْنَا إِن نَّسِينَا أَوْ أَخْطَأْنَا رَبَّنَا وَلاَ تَحْمِلْ عَلَيْنَا إِصْرًا كَمَا حَمَلْتَهُ عَلَى الَّذِينَ مِن قَبْلِنَا رَبَّنَا وَلاَ تُحَمِّلْنَا مَا لاَ طَاقَةَ لَنَا بِهِ وَاعْفُ عَنَّا وَاغْفِرْ لَنَا وَارْحَمْنَا أَنتَ مَوْلاَنَا فَانصُرْنَا عَلَى الْقَوْمِ الْكَافِرِينَ","رَبَّنَا مَا خَلَقْتَ هَذا بَاطِلاً سُبْحَانَكَ فَقِنَا عَذَابَ النَّارِ رَبَّنَا إِنَّكَ مَن تُدْخِلِ النَّارَ فَقَدْ أَخْزَيْتَهُ وَمَا لِلظَّالِمِينَ مِنْ أَنصَارٍ رَّبَّنَا إِنَّنَا سَمِعْنَا مُنَادِيًا يُنَادِي لِلإِيمَانِ أَنْ آمِنُواْ بِرَبِّكُمْ فَآمَنَّا رَبَّنَا فَاغْفِرْ لَنَا ذُنُوبَنَا وَكَفِّرْ عَنَّا سَيِّئَاتِنَا وَتَوَفَّنَا مَعَ الأبْرَارِ رَبَّنَا وَآتِنَا مَا وَعَدتَّنَا عَلَى رُسُلِكَ وَلاَ تُخْزِنَا يَوْمَ الْقِيَامَةِ إِنَّكَ لاَ تُخْلِفُ الْمِيعَادَ","اللَّهُمَّ أنَْتَ رَبيِّ لَا إلِهََ إلَِّا أنَتَ، خَلَقْتنَيِ وَأنََا عَبدُْكَ، وَأنََا عَلَى عَهْدِكَ وَوَعْدِكَ مَا اسْتَطَعْتُ، أَعُوذُ بِكَ مِنْ شَرِّ مَا صَنَعْتُ، أَبُوءُ لَكَ بِنِعْمَتِكَ عَلَيَّ، وَأَبُوءُ بِذَنْبِي فَاغْفِرْ لِي فَإِنَّهُ لَا يَغْفِرُ الذُّنُوبَ إِلَّا أَنْتَ.","اللَّهُمَّ إِنِّي ظَلَمْتُ نَفْسِي ظُلْمًا كَثِيرًا، وَلَا يَغْفِرُ الذُّنُوبَ إِلَّا أَنْتَ، فَاغْفِرْ لِي مَغْفِرَةً مِنْ عِنْدِكَ وَارْحَمْنِي إِنَّك أَنْتَ الْغَفُورُ الرَّحِيمُ","رَبِّ اغْفِرْ لِي خَطِيئَتِي وَجَهْلِي وَإِسْرَافِي فِي أَمْرِي كُلِّهِ وَمَا أَنْتَ أَعْلَمُ بِهِ مِنِّي، اللَّهُمَّ اغْفِرْ لِي خَطَايَايَ وَعَمْدِي وَجَهْلِي وَهَزْلِي، وَكُلُّ ذَلِكَ عِنْدِي، اللَّهُمَّ اغْفِرْ لِي مَا قَدَّمْتُ وَمَا أَخَّرْتُ وَمَا أَسْرَرْتُ وَمَا أَعْلَنْتُ أَنْتَ الْمُقَدِّمُ وَأَنْتَ الْمُؤَخِّرُ وَأَنْتَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ.","اللَّهُمَّ رَبَّ السَّمَوَاتِ وَرَبَّ الْأَرْضِ وَرَبَّ الْعَرْشِ الْعَظِيمِ، رَبَّنَا وَرَبَّ كُلِّ شَيْءٍ، فَالِقَ الْحَبِّ وَالنَّوَى وَمُنْزِلَ التَّوْرَاةِ وَالْإِنْجِيلِ وَالْفُرْقَانِ، أَعُوذُ بِكَ مِنْ شَرِّ كُلِّ شَيْءٍ أَنْتَ آخِذٌ بِنَاصِيَتهِِ، اللَّهُمَّ أَنْتَ الْأوََّلُ فَلَيْسَ قَبْلَكَ شَيْءٌ، وَأَنْتَ الْآخِرُ فَلَيْسَ بَعْدَكَ شَيْءٌ، وَأَنْتَ الظَّاهِرُ فَلَيْسَ فَوْقَكَ شَيْءٌ، وَأَنْتَ الْبَاطِنُ فَلَيْسَ دُونَكَ شَيْءٌ، اقْضِ عَنَّا الدَّيْنَ وَأَغْنِنَا مِنَ الْفَقْرِ.","اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنَ الْكَسَلِ وَالْهَرَمِ وَالْمَأْثَمِ وَالْمَغْرَمِ، وَمِنْ فِتْنَةِ الْقَبْرِ وَعَذَابِ الْقَبْرِ، وَمِنْ فِتْنَةِ النَّارِ وَعَذَابِ النَّارِ، وَمِنْ شَرِّ فِتْنَةِ الْغِنَى، وَأَعُوذُ بِكَ مِنْ فِتْنَةِ الْفَقْرِ، وَأَعُوذُ بِكَ مِنْ فِتْنَةِ الْمَسِيحِ الدَّجَّالِ، اللَّهُمَّ اغْسِلْ عَنِّي خَطَايَايَ بِمَاءِ الثَّلْجِ وَالْبَرَدِ، وَنَقِّ قَلْبِي مِنَ الْخَطَايَا كَمَا نَقَّيْتَ الثَّوْبَ الْأَبْيَضَ مِنَ الدَّنَسِ، وَبَاعِدْ بَيْنِي وَبَيْنَ خَطَايَايَ كَمَا بَاعَدْتَ بَيْنَ الْمَشْرِقِ وَالْمَغْرِبِ.","اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنَ الْهَمِّ وَالْحَزَنِ وَالْعَجْزِ وَالْكَسَلِ وَالْجُبْنِ وَالْبُخْلِ وَضَلَعِ الدَّيْنِ وَغَلَبَةِ الرِّجَالِ","اللَّهُمَّ إنِِّي أَعُوذُ بكَِ مِنَ الْبُخْلِ، وَأَعُوذُ بكَِ مِنَ الْجُبْنِ، وَأَعُوذُ بكَِ أَنْ أُرَدَّ إلَِى أَرْذَلِ الْعُمُرِ، وَأَعُوذُ بِكَ مِنْ فِتْنَةِ الدُّنْيَا، وَأَعُوذُ بِكَ مِنْ عَذَابِ الْقَبْرِ.","رَبِّ اغْفِرْ لِي خَطِيئَتِي وَجَهْلِي وَإِسْرَافِي فِي أَمْرِي كُلِّهِ وَمَا أَنْتَ أَعْلَمُ بِهِ مِنِّي، اللَّهُمَّ اغْفِرْ لِي خَطَايَايَ وَعَمْدِي وَجَهْلِي وَهَزْلِي، وَكُلُّ ذَلِكَ عِنْدِي، اللَّهُمَّ اغْفِرْ لِي مَا قَدَّمْتُ وَمَا أَخَّرْتُ وَمَا أَسْرَرْتُ وَمَا أَعْلَنْتُ أَنْتَ الْمُقَدِّمُ وَأَنْتَ الْمُؤَخِّرُ وَأَنْتَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ."
]

# Helper function to add a warning
async def add_warning(user, ctx):
    c.execute('SELECT count FROM warnings WHERE user_id = ?', (user.id,))
    row = c.fetchone()
    if row is None:
        c.execute('INSERT INTO warnings (user_id, count) VALUES (?, ?)', (user.id, 1))
        warning_count = 1
    else:
        warning_count = row[0] + 1
        c.execute('UPDATE warnings SET count = ? WHERE user_id = ?', (warning_count, user.id))
    conn.commit()

    # Assign warning role
    if warning_count in warning_roles:
        role = ctx.guild.get_role(warning_roles[warning_count])
        await user.add_roles(role)
        await ctx.send(f"{user.mention} تم تحذيرك! لديك الآن {warning_count} تحذيرات وتم إضافة دور {role.name}.")

    if warning_count > 3:
        await ctx.send(f"{user.mention} تم تجاوز الحد الأقصى للتحذيرات! سيتم اتخاذ إجراءات إضافية.")
    else:
        warning_names = ["تحذير اول", "تحذير ثاني", "تحذير ثالث"]
        await ctx.send(f"{user.mention} تم تحذيرك! لديك الآن {warning_names[warning_count - 1]}.")

# Helper function to get warning count
def get_warning_count(user_id):
    c.execute('SELECT count FROM warnings WHERE user_id = ?', (user_id,))
    row = c.fetchone()
    return row[0] if row else 0

# Helper function to remove a warning
async def remove_warning(user, ctx):
    c.execute('SELECT count FROM warnings WHERE user_id = ?', (user.id,))
    row = c.fetchone()
    if row and row[0] > 0:
        warning_count = row[0] - 1
        c.execute('UPDATE warnings SET count = ? WHERE user_id = ?', (warning_count, user.id))
        conn.commit()

        # Remove the warning role
        if warning_count + 1 in warning_roles:
            role = ctx.guild.get_role(warning_roles[warning_count + 1])
            await user.remove_roles(role)
            await ctx.send(f"{user.mention} تم إزالة دور {role.name}. لديك الآن {warning_count} تحذيرات.")
        else:
            await ctx.send(f"{user.mention} لديك الآن {warning_count} تحذيرات.")
    else:
        await ctx.send(f"{user.mention} ليس لديك أي تحذيرات.")

@bot.event
async def on_ready():
    print(f'We have logged in as {bot.user}')
    send_duaa.start()  # Start the duaa sending task
    print('Bot is running for 12 hours...')
    # Run the bot for 12 hours
    await asyncio.sleep(12 * 3600)  # 12 hours in seconds
    print('12 hours passed. Shutting down...')
    await bot.close()

@tasks.loop(minutes=30)
async def send_duaa():
    channel = bot.get_channel(duaa_channel_id)
    if channel:
        duaa = random.choice(duaas)
        embed = discord.Embed(description=duaa, color=0x0000FF)
        embed.set_footer(text=f"{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        await channel.send(embed=embed)

@bot.event
async def on_message(message):
    # Avoid the bot replying to itself
    if message.author == bot.user:
        return

    # Check if the message is in one of the allowed channels for the response message
    if message.channel.id in allowed_channels and not message.content.startswith('!'):
        await message.channel.send(response_message)

    # Check if the message is just a period "."
    if message.content.strip() == ".":
        await message.channel.send("منور 💫")
    # Check if the message is "السلام عليكم"
    elif message.content.strip() == "السلام عليكم":
        await message.channel.send("وعليكم السلام ورحمة الله وبركاته")
    # Check if the message is "تسجيل دخول"
    elif message.content.strip() == "تسجيل دخول":
        await message.channel.send(f"تم تسجيل دخول {message.author.mention} ✅")
    # Check if the message is "تسجيل خروج"
    elif message.content.strip() == "تسجيل خروج":
        await message.channel.send(f"تم تسجيل خروج {message.author.mention} ✅")
    # Check if the message is "اين الحريه"
    elif message.content.strip() == "اين الحريه":
        await message.channel.send("هنا في سيرفر 𝐅𝐌〢𝐟𝐫𝐞𝐞𝐝𝐨𝐦")

    elif message.content.strip() == "<@1204452111908208641>":
        await message.channel.send("تمنشنه تاني هتندم ._.")

    elif message.content.strip() == "<@1197576409783210037>":
        await message.channel.send("غير متاح حاليا يرجي عدم المنشنه لاحقا")

    elif message.content.strip() == "<@1187481492146884688>":
        await message.channel.send("انتظر الرد")
    await bot.process_commands(message)

@bot.command(name='تحذير')
async def warn(ctx, member: discord.Member):
    print(f"Warn command invoked by {ctx.author} for {member}")
    if any(role.id in allowed_roles for role in ctx.author.roles):
        await add_warning(member, ctx)
    else:
        await ctx.send("ليس لديك صلاحية لاستخدام هذا الأمر.")

@bot.command(name='تحذيراتي')
async def my_warnings(ctx, member: discord.Member = None):
    print(f"My Warnings command invoked by {ctx.author}")
    if member:
        count = get_warning_count(member.id)
        await ctx.send(f"{member.mention} لديه {count} تحذيرات.")
    else:
        count = get_warning_count(ctx.author.id)
        await ctx.send(f"لديك {count} تحذيرات.")

@bot.command(name='حذفتحذير')
async def remove_warn(ctx, member: discord.Member):
    print(f"Remove Warn command invoked by {ctx.author} for {member}")
    if any(role.id in allowed_roles for role in ctx.author.roles):
        await remove_warning(member, ctx)
    else:
        await ctx.send("ليس لديك صلاحية لاستخدام هذا الأمر.")

@bot.command(name='إضافةدور')
async def add_role(ctx, role: discord.Role):
    print(f"Add Role command invoked by {ctx.author} for {role}")
    if ctx.author.guild_permissions.administrator:  # Only administrators can use this command
        if role.id not in allowed_roles:
            allowed_roles.append(role.id)
            await ctx.send(f"تمت إضافة دور {role.name} إلى قائمة الأدوار المسموح بها للتحذيرات.")
        else:
            await ctx.send(f"الدور {role.name} موجود بالفعل في قائمة الأدوار المسموح بها.")
    else:
        await ctx.send("ليس لديك صلاحية لاستخدام هذا الأمر.")

@bot.command(name='إزالةدور')
async def remove_role(ctx, role: discord.Role):
    print(f"Remove Role command invoked by {ctx.author} for {role}")
    if ctx.author.guild_permissions.administrator:  # Only administrators can use this command
        if role.id in allowed_roles:
            allowed_roles.remove(role.id)
            await ctx.send(f"تم إزالة دور {role.name} من قائمة الأدوار المسموح بها للتحذيرات.")
        else:
            await ctx.send(f"الدور {role.name} غير موجود في قائمة الأدوار المسموح بها.")
    else:
        await ctx.send("ليس لديك صلاحية لاستخدام هذا الأمر.")

# This function is to run the bot and handle login failures
async def run_bot():
    while True:
        try:
            print("Starting bot...")  # رسالة تصحيح
            await bot.start('')
        except discord.LoginFailure:
            print("Failed to login. Check your token.")  # رسالة تصحيح
            await asyncio.sleep(60)  # Wait before retrying
        except Exception as e:
            print(f"An error occurred: {e}")  # رسالة تصحيح
            await asyncio.sleep(60)  # Wait before retrying

if __name__ == "__main__":
    asyncio.run(run_bot())
