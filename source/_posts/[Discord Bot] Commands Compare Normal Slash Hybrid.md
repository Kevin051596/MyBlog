---
title:       "[Discord Bot] Commands Compare: Normal、Slash、Hybrid"
subtitle:    "基於 Python 開發的 Discord Bot 各種指令類型使用與比較"
excerpt: "紀錄 Discord Bot 開發過程中會涉略到的指令類型，並了解各自的效用與定位來搭配出人性化指令設計"
date:        2023-07-06
tags:        ["python"]
categories:  ["Develop", "Chatbot"]
cover: /gallery/covers/discord.jpg
toc: true
---
## 前言

本文適合對 OOP 有略知，且已邀請機器人至伺服器的讀者。

若還未完成，這是我推薦的設置教程：[**Discord Bot**](https://www.freecodecamp.org/news/create-a-discord-bot-with-python/)

如果都沒問題就可以開始了~
以下為本次測試的 Discord Bot 初始化設定
```python
import os
import discord
from discord.ext import commands
from discord import app_commands
from dotenv import load_dotenv

def main():
    # 建立一個機器人物件
    client = commands.Bot(
        command_prefix = "$",
        intents = discord.Intents.all()
    )

    # 機器人事件
    @client.event
    async def on_ready():
        await client.change_presence(activity=discord.Game(name = "coding For Fun"))
        logging_channel = await client.fetch_channel(os.getenv("LOGGING_CHANNEL"))
        await logging_channel.send(f"Bot ready")

    # 執行機器人
    client.run(os.getenv('BOT_TOKEN'))

if __name__ == '__main__':
    load_dotenv()
    main()
```
1. 首先用 `discord.py` 的 `commands.Bot` 建立一個機器人實體，並定義指令前綴`（command prefix）`和機器人權限
2. 接著設定機器人事件，如果機器人準備好了，就在機器人所處的頻道打印 Bot ready
3. 最後是執行機器人，我們會需要這台機器人的 `Token`，我是將 `Token` 放在 `.env`，透過 `python_dotenv` 這個套件提取至 `os` 的環境變數內

當設定好後，就可以玩各種指令啦，基本上 `discord.py` 的套件提供三種指令類型：`Normal Command`、`Slash Command`、`Hybrid Command`，接下來將會逐個介紹

## Normal Command

這種指令藉由指令前綴（command prefix）來呼叫，也就是要先打指令前綴讓機器人知道你在呼叫他

### Build A Normal Command
我們只需要使用 `@client.command()` 就能生成這種指令，而在此之下的函式就是這個指令要做的事情。ctx 代表 Context 物件，可以理解成是機器人的對話框。以這個指令為例，當使用者打出 `$hello`，機器人就會傳送 `hello!!`。 
```python
@client.command(name="hello", description="Say Hello")
async def hello(ctx):
    await ctx.send("hello!!")
```
而我們可以在 `@client.command()` 給予一些參數設定，例如：`name`，指令名稱（如果沒有提供，就會以該函數名稱代替）。以下為一些常見的參數設定：
```python
@client.command(
    aliases=["hello"],
    name="greeting",
    description="Hello Everyone",
    brief="This is greeting command",
    enabled=True,
    hidden=True
)
async def greeting(ctx):
    await ctx.send("hello!!")
```
- `aliases`（別名）：提供該指令其他呼叫名稱
- `description`（說明）：描述該指令作用
- `help`：在 `help` 資訊欄，詳述該指令作用
- `brief`（簡述）：在 `help` 資訊欄，簡述該指令作用
- `enabled`：當 True 時，該指令能被呼叫，反之不行
- `hidden`：當 True 時，該指令不會顯示在 `help` 資訊欄

### Commands Group
如果我們有多個功能類似的 Normal Command，我們可以將其歸納成群組。  

藉由 `@client.group` 我們可以決定這個群組名稱，同時這個群組可以單獨被視為一個指令，因此也會有呼叫這個指令時應該執行的函式。接者我們可以根據群組名去衍生更多的子指令，用法與 Normal Command 一樣。
```python
@client.group(name="greeting")
async def greeting(ctx):
    await ctx.send("This is greeting Command")

@greeting.command(name="hello")
async def hello(ctx):
    await ctx.send("HELLO!!")

@greeting.command(name="hi")
async def hi(ctx):
    await ctx.send("HI!!")
```

這種 Commands Group 可以層層相套，也就是群組裡面的子指令也可以是一個群組，用來為指令分類很好用的

## Slash Command

這種指令可以直接用 `/` 呼叫，並且會在使用者介面上提供指令提示。但不能用自定義的指令前綴呼叫

### Build A Slash Command
discord.py 在 版本 2.0 時於機器人底下新增一個 `tree` 屬性，解釋如下：
> The command tree responsible for handling the application commands in this bot.

在 discord.py API 提到 `tree` 是一種 `CommandTree` 類型的物件，裡面主要存放 application commands，而 Slash Command 就是數屬於這種，因此我們要做的是想辦法將自定義的 Slash Command 加入這個 `CommandTree` 物件 

建立方法與 Normal Command 類似，用 `@client.tree.command()`來定義，與 Normal Command 不同的是，下面定義的函式第一個參數會是 `interaction`（互動物件），我們可以透過這樣的寫法來讓機器人發送訊息
```python
@client.tree.command(name = "hello")
async def first_command(interaction):
    await interaction.response.send_message("Hello!")
```

### Slash Commands Group
如果有多個功能類似的 Slash Commands，我們也可以將其歸類成群組，這時會需要用到 `app_commands.Group` 這個類別。  

我們可以定義一個 `app_commands.Group`，並將相關的 `Slash 子指令` 放入，要注意將原本是機器人的部分要改成 `app_commands`，讓這個指令單純只是 Slash Command。
```python
class Math_(app_commands.Group):
    
    @app_commands.command()
    async def add(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} + {two} = {one + two}")
    @app_commands.command()
    async def subtract(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} - {two} = {one - two}")
    @app_commands.command()
    async def multiple(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} x {two} = {one * two}")
    @app_commands.command()
    async def division(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} / {two} = {one / two}")
```
最後實體化這個 `app_commands.Group` 物件，並將其加入機器人的指令庫中
```python
math_ = Math_(name="math_")
client.tree.add_command(math_)
```

完整程式碼
```python
import discord
from discord.ext import commands
from discord import app_commands

class Math_(app_commands.Group):
    
    @app_commands.command()
    async def add(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} + {two} = {one + two}")
    @app_commands.command()
    async def subtract(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} - {two} = {one - two}")
    @app_commands.command()
    async def multiple(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} x {two} = {one * two}")
    @app_commands.command()
    async def division(self, interaction, one: int, two: int):
        await interaction.response.send_message(f"{one} / {two} = {one / two}")

def main():
    intents = discord.Intents.all()
    client = commands.Bot(
        command_prefix="$",
        intents=intents
    )

    @client.event
    async def on_ready():
        client.tree.copy_global_to(guild=settings.GUILD_ID)
        await client.tree.sync(guild=settings.GUILD_ID)

        math_ = Math_(name="math_")
        client.tree.add_command(math_)

        await client.change_presence(activity=discord.Game(name = "coding For Fun"))
        logging_channel = await client.fetch_channel(os.getenv("LOGGING_CHANNEL"))
        await logging_channel.send(f"Bot ready")

    client.run(settings.BOT_TOKEN, root_logger=True)

if __name__ == '__main__':
    main()
```

### CommandTree Sync
當我們建立完成 Slash Command 並加入 `CommandTree` 後，會發現 Discord 上這個指令沒有作用，這是因為 discord 沒有接收 API 的請求將我們的 slash command 複製過去。

所以這時會需要用到 `CommandTree.sync()` 的這個方法，才會將當前的 Slash Command 同步複製一份給 discord。但是這個方法有個瑕疵，就是新增 Slash Command 時有速度限制，導致開發者有時候會誤入這個 Slash Command 沒有即時更新的陷阱。

於是 discord.py 提供了 `CommandTree.copy_global_to()` 先將當前的 CommandTree 複製一份給 discord，藉此優化後續的同步速度
```python
@client.event
async def on_ready():
    client.tree.copy_global_to(guild=settings.GUILD_ID)
    slashTree = await client.tree.sync(guild = discord.Object(id = os.getenv('GUILD_ID')))
```
關於這個問題有一篇完整的 github gist 說明：[discord.py 2.0+ slash command info and examples.](https://gist.github.com/AbstractUmbra/a9c188797ae194e592efe05fa129c57f)

## Hybrid Command

這種指令可以同時被指令前綴和 `/` 呼叫，所以如同字面上意思，即兼具 Slash Command 和 Normal Command 的呼叫特性。注意該命令也需要 `CommandTree.sync()` 同步。

### Build A Hybrid Command
定義 `@client.hybrid_command()` 就能使用這種指令，函式內的第一個參數是 `ctx`（Context 物件）

```python
@client.hybrid_command(
        name = "hello",
        description = "My first application Command",
        guild = discord.Object(id = os.getenv('GUILD_ID'))
    )
async def first_command(ctx):
    await ctx.send("Hello!")
```

### Hybrid Commands Group
與 Normal Commands Group 寫法差不多，如下
```python
@client.hybrid_group()
async def test(ctx):
    await ctx.send("Test!!")

@test.command(name="hi")
async def hi(ctx):
    await ctx.send("HI!!")
@test.command(name="hey")
async def hey(ctx):
    await ctx.send("HEY!!")
```

## 延伸：在 Cog 中使用這些指令

在 `discord.py` 中，對 `Cog` 的定義是這樣：
> A cog is a collection of commands, listeners, and optional state to help group commands together

與 Commands Group 相比之下這個集合具有監聽、事件處理的額外功能，也就是說 `Cog` 可以當成是一個更強大的指令集合，此外藉由這個方式定義的指令在 `help` 資訊欄中會標記其集合名稱，讓其可讀性更好。

與在主程式的寫法很像，只是要把原本機器人的部分（client）改掉就可以，記住參數裡的第一個參數是 `self`（這與類別有關，但非這次討論重點）。最後要補上 `setup` 函式，讓這個 `cog` 類別能被初始化
```python
class cmds(commands.Cog):
    def __init__(self, client):
        self.client = client

    @commands.Cog.listener()
    async def on_ready(self):
        print(f'{__name__} loaded successfully!')

    # normal command
    @commands.command(name="hello", description="Say Hello")
    async def command(self, ctx):
        await ctx.send("HELLO!!")

    # slash command
    @app_commands.command(name="happy", description="Say Happy Happy")
    async def command(self, interaction: discord.Interaction):
        await interaction.response.send_message("Happy! Happy! Happy~")
    
    # hybrid command
    @commands.hybrid_command(name="hi", description="Say Hi")
    async def command(self, ctx):
        await ctx.send("HI!!")

async def setup(client):
    await client.add_cog(cmds(client))
```
至於要怎麼在主程式呼叫這個類別，只要在 `on_ready` 的函式下加入這條就可以了，注意檔案名稱的後綴（`.py`）要拿掉
```python
await client.load_extension("你的 Cog 檔案名稱")
```

## 小結

1. **Normal Command** 是最基礎的指令，藉由指令前綴的方式呼叫，可以將多個 Normal Command 集合成 Commands Group，分類出子指令來使用
2. **Slash Command** 於 discord.py 版本 2 中加入，可以透過 `/` 方式呼叫，讓使用者不需要呼叫 `help` 資訊欄，就能知道哪些指令可以使用，是一種較為新型的寫法，可以透過 `app_command.Group` 類別將 Slash Commands 集合成群組
3. **Hybrid Command** 兼具 Slash 和 Normal 的呼叫方式
4. 與 `Command Group` 相比，我會更偏向使用 `Cog` 作為分類依據，接著再使用 Commands Group 做更精細的分類

## 參考資料

- [discord.py API Reference](https://discordpy.readthedocs.io/en/stable/index.html)  
- [How to create a DISCORD BOT (2023 with discordpy 2 in python)](https://www.youtube.com/playlist?list=PLESMQx4LeD3N0-KKPPDaToZhBsom2E_Ju)