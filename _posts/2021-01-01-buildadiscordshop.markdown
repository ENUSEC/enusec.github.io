---
layout: post
title:  "Build your own Shop (Bot) on Discord with Discord Payments"
description: Learn how you can create your own Shop Bot on Discord using Discord Payments to accept crypto currency payments.
date:   2021-01-01 13:00:00 +0000
categories: Discord Bot Shop E-commerce Crypto Programming
---

![](https://discordpayments.com/logo.png)

Discord Payments is a discord bot that allows for secure crypto currency payments to be made by the users on your discord server. 

Simply add the bot to your server and people can start making payments.

## How it works
Discord Payments works by providing a cryptographically secure receipt token when a payment is complete.

This token can then be used as proof of payment and you can issue product to the particular user.

The bot has one main command the `$$buy` command where a user will specify the type of currency, amount in USD, address to send the funds and finally a unique identifier which specifies the particular product.

## Creating a store
#### Stage 1 (Add the bot to your server)
In order to create your own Discord Shop the first thing you need to do is add the discord payments bot to your server. To do that you simply use the below link to add the bot.

[Add the Official Discord Payments Bot to your Server.](https://discord.com/api/oauth2/authorize?client_id=784856689337827359&permissions=67584&scope=bot)

It is as simple as that. Once the bot has been added to your server you can now accept crypto currency payments in a crypto currency of your choice (provided you have a wallet and recieving address).

#### Stage 2 (Specify Products)
The next stage is to set up your products.

For this example I will be implementing a Donations payment store where roles are assigned based on the amount that is donated, however the same proccess can be slightly changed to suite any type of product.

Firstly you will want to create two channels in a seperate category which will only be used for your Discord Shop. In this example I have a category called `DONATIONS` and two channels `#details` to specify the products and `#donate` where users can make payments and pay for the products.

The `#donate` channel is empty whilst in the `#details` channel the bellow message exists, detailing the addresses to make payments too and the products. It is important to specify the identifier for each of the different products as this is stated when making a payment.

```
Make a Donation to of any amount in USD! 

Donations of certain amounts will automatically reward you with a discord role. These can be seen bellow:
+-----------+--------+-----------------+
| Unique ID | Role   | Donation Amount |
+-----------+--------+-----------------+
| 1         | Bronze | 10 USD          |
+-----------+--------+-----------------+
| 2         | Silver | 20 USD          |
+-----------+--------+-----------------+
| 3         | Gold   | 30 USD          |
+-----------+--------+-----------------+

Donations can be made in Litecoin or Bitcoin and you can find our addresses bellow.
BTC Address: 3QT3MUnfJQqj3wzAFtHL4GVKPZYk3tM4K1
LTC Address: MJmfxteBdHUiN8AU47k27vw4bQvQvePkg3
```

#### Stage 3 (Create your Bot)
The next stage is to create your Bot. Creating a Discord Bot is relatively simple. You first have to register it using the Developer Portal and then you can begin to build the bot and commands that can be used.

If you already have a Bot and you are adding a shop to it the majority of this stage can be skipped.

First log in to the [Discord Developer Portal here](https://discord.com/developers/applications).

You will be greeted with a page listing `My Applications`. Now you will need to add an application. To do this click the top right button `New Application`.

![](https://discordpayments.com/images/dashboard.png)

Now choose a name for the application something like `Donations Bot` or whatever you want your Shop to be called.

![](https://discordpayments.com/images/newapplication.png)

Once you have decided a name click create and you will be redirected to the configuration page for your new application.

![](https://discordpayments.com/images/donationsbot.png)

Now navigate to the `Bot` option from the sidebar and click `Add Bot`. 

![](https://discordpayments.com/images/addbot.png)

This will add a Bot to your application which you can customize the description of, the App Icon and other settings as you wish.

At this point it is important to click the `Copy` button underneath where it says `TOKEN` and save this to a file called `secret.txt` we will use this secret later.

Your Secret may look like below:
```
Nzk0MTg0MzI4NzI4NTQzMjMy.X-3Hyw.37khSGXja4fA61aMEOOv2YVz_Yc
```

It is important you keep this secret safe as this is what allows you to control the Bot. If this secret is public or stolen it would allow someone else to control the Bot and gain any permisions that the Bot has. Think of it as the Bots password. For example don't publish the bot secret in a blog post or on github.

Once you have saved your secret you will want to go to the `OAuth2` option on the sidebar. This can be used to actually add the bot to your server.

Now scroll down to the `Scopes` section and select `Bot`. Underneath this you can specify the Bot permisions. These permisions will change regarding what capability you want your Bot to have. For this example the bot needs to be able to control roles, read messages and also send messages. (All bots will need to be able to read messages and send messages).

For simplicity select `Administrator` this will give the bot administrator privileges however I suggest you decide what permisions you want the bot to have and select them individually when pushing this into production.

![](https://discordpayments.com/images/oauth2.png)

Finally copy the URL it creates within the `Scopes` options and visit the URL. Then add the Bot to the Server you are running your Shop from.

At this point the Bot is now present in the server and it is time to start coding.

You can use this bellow template to create a simple Bot which we will be using for this example.

```python
from discord.ext import commands
import discord

DISCORD_TOKEN = "... Secret from secret.txt here! ..."
bot = commands.Bot(command_prefix='!')
bot.remove_command('help')

@bot.command(name='help')
async def help(ctx):
    response = """Bot Help!
    ```
    !command1 -> Description and Usage 
    !command2 -> Description and Usage
    ```
    """
    await ctx.send(response)

@bot.command(name='command1')
async def command1(ctx):
    response = "This is a command"
    await ctx.send(response)

@bot.event
async def on_command_error(ctx, error):
    response = "There was an error!\nUse `!help` for usage information."
    await ctx.send(response)

bot.run(DISCORD_TOKEN)
```
Take this code snippet and add your secret to the `DISCORD_TOKEN` variable.

Now save it to a file called `bot.py` and run with the bellow command.
```
python3 bot.py
```
You should see that your bot comes online and you can begin to interact with it.

A full tutorial on creating a Discord bot can be seen [here](https://realpython.com/how-to-make-a-discord-bot-python/).

#### Stage 4 (Recieve and Validate Payments)
Now we have a bot set up it is time we add the ability to recieve and validate payments.

Since Discord Payments handles the actual payment proccess you don't need to worry about any of that, you simply need to add a command which accepts a token and responds depending on whether the validation checks succeed and unique Identifier exists.

Discord payments actually provides the code for verifying the token and it is verified using the Discord Payments public key.

The first thing you need to do is add a global variable `public_key` to your script.
```python
bot.remove_command('help')

public_key = """-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAroDGdpPsGdP1DKz6OCSC
8uDEIXiaZwA04CFkH6XzyB1wClFbGUx1B/k+a87/7sOhRCgcMyjOmiV3iWUody3D
iJcpbbx+PGekVMf8qMDUJpYTQWOqau5hKcYr0WXw35JKPhqBQ7Kvlc4Q3RUlbtrL
mIjWfr6Vs04fXEY/J8Yqv8QXOOnAJSRG/WLsojBRTsHox7sXGtSHqVGVX7JPRBE2
mtj9NLTFscX3yo8C7Fx7GlKrSW6cDsv8BIsB7SiXBJF1pA5zoOroQEjJSP5Ugspi
Pccdxpv/Ea5DnR1GVe2Dekx844WH22ptWK55vQnroqiQDUj8/9C4n5I2EKJpgAUE
XQIDAQAB
-----END PUBLIC KEY-----"""
```

Now underneath this you need to add the verification function provided by Discord Payments.

```python
def verifyToken(encoded):
    try:
            decoded = jwt.decode(encoded, public_key, algorithms='RS256')
            return decoded
    except jwt.exceptions.InvalidSignatureError:
            # print("Invalid Token")
            return False
    except jwt.exceptions.DecodeError:
            # print("Invalid Format")
            return False
    except Exception as e:
            # print(e)
            # print("Something went wrong")
            return False
```
The `print` statements can be either uncommented, deleted or turned into a more comprehensive logging system.

The next stage is to add the appropriate imports at the top of the python script. In order to verify a Discord Payments receipt token all you need to import is the `jwt` module.
```python
import jwt
```

Now everything is set up we can add a command that will allow a user to actually buy the products. In this example I am using the `!donate` command however this can be changed to suite the context of your Shop.

First I want to add a command `!donate` which takes one parameter being the token issued by the Discord Payments bot.

```python
@bot.command(name='donate')
async def donate(ctx, receipt):
    # CONTENT HERE
    await ctx.send(response)
```
Now we need to validate that the token is legitimate.

So we add the line:
```python
token = verifyToken(receipt)
if token:
    pass
else:
    response = "Invalid Token"
```
This now checks if the token is an actual Discord Payments token or not and whether it has been signed by Discord Payments. If not then the payment is rejected.

Then we want to check whether the user that is supplying the token is the same user that made the payment. This is important so that tokens cannot be shared or stolen. Therefore, inside the current condition we add another condition to check for the author and compare this against the author of the message.

```python
if token['username'] == str(ctx.message.author):
    pass
else:
    response = "That is not your token to use!"
```

The next thing we check is that the payment was made to the correct address. There are multiple ways of doing this, you could use an individual hardcoded address, likewise it would also be possible to specify a list of addresses and check that the provided address exists within the list. The list could contain addresses for different currencies.

Therefore you now check the address that exists within the token against your address. This is nested within the previous condition.
```python
if token["address"] == "Your Address here":
    pass
else:
    response = "Payment made to a different address!"
```
Alternatively using a list of a BTC and LTC addresses would look like below.
```python
ADDRESSES = ["3QT3MUnfJQqj3wzAFtHL4GVKPZYk3tM4K1", "MJmfxteBdHUiN8AU47k27vw4bQvQvePkg3"]
if token["address"] in ADDRESSES:
```
At this point you might want to do a check against the `token['server']` which would check that the payment was made in your server. However, this is not necersary and by not implementing it you also allow users to make payments privately in a DM to the Discord Payments bot.

Once you have checked that the user is the owner of the token and that the payment was made to the correct address you can now begin to check which product the user is buying. This again is completed using a conditional statement (Strings are used as the Unique Identifier can be alpha numeric)
```python
if token['identifier'] == "1":
    pass
elif token['identifier'] == '2':
    pass
elif token['identifier'] == '3':
    pass
else:
    response = "There is not a product for this Identifier!"
```
Now the product has been selected the only thing left to do is check that the right amount has been paid for that particular product. To do this you simply add a condition within each of the conditions for the identifier.

For the below code example I am using the prices described in `Stage 2` however this will be entirely dependant on the prices of the products you are selling.
```python
if token['identifier'] == "1":
    if int(token['amount']) >= 10:
        pass
    else:
        response = "That amount is not enough!"
elif token['identifier'] == '2':
    if int(token['amount']) >= 20:
        pass
    else:
        response = "That amount is not enough!"
elif token['identifier'] == '3':
    if int(token['amount']) >= 30:
        pass
    else:
        response = "That amount is not enough!"
else:
    response = "There is not a product for this Identifier!"
```
At this stage you have successfully validated an individual payment and can program your bot to react to the different products depending on what that gives a particular user.

The full payment validation described in this post is seen bellow.
```python
token = verifyToken(receipt)
if token:
    if token['username'] == str(ctx.message.author):
        if token["address"] == "Your Address here":
            if token['identifier'] == "1":
                if int(token['amount']) >= 10:
                    pass
                else:
                    response = "That amount is not enough!"
            elif token['identifier'] == '2':
                if int(token['amount']) >= 20:
                    pass
                else:
                    response = "That amount is not enough!"
            elif token['identifier'] == '3':
                if int(token['amount']) >= 30:
                    pass
                else:
                    response = "That amount is not enough!"
            else:
                response = "There is not a product for this Identifier!"
        else:
            response = "Payment made to a different address!"
    else:
        response = "That is not your token to use!"
else:
    response = "Invalid Token"
```
#### Stage 5 (Issue Product)
Now our Bot is validating payments you can begin to serve products.

In this example it is relatively simple since I am just controlling discord roles.

In order to assign a discord role you can simply use the bellow function and call it appropriately.
```python
async def assignRole(ctx, roleName):
    user = ctx.message.author
    role = get(ctx.guild.roles, name='... role name goes here ...')
    await user.add_roles(role)

await assignRole(ctx, '... role name here ...')
```
You then add this function to each of the conditions for the different products and set the response showing a user they have successfully bought the product.  
```python
if token['identifier'] == "1":
    if int(token['amount']) >= 10:
        await assignRole(ctx, 'Bronze')
        response = "You have been given the Bronze Role.\n\nThanks for donating!"
    else:
        response = "That amount is not enough!"
elif token['identifier'] == '2':
    if int(token['amount']) >= 20:
        await assignRole(ctx, 'Silver')
        response = "You have been given the Silver Role.\n\nThanks for donating!"
    else:
        response = "That amount is not enough!"
elif token['identifier'] == '3':
    if int(token['amount']) >= 30:
        await assignRole(ctx, 'Gold')
        response = "You have been given the Gold Role.\n\nThanks for donating!"
    else:
        response = "That amount is not enough!"
else:
    response = "There is not a product for this Identifier!"
```

To assign roles you will need to import the following module.
```python
from discord.utils import get
```
#### Stage 6 (Track Receipt tokens)
For shops which are implementing subscription based payments and one time use tokens it is important that you record tokens that have previously been used and when you validate the receipt token in `Stage 4`. When you validate the token you check the token aginst previously used tokens and record if they are expired or not.

I suggest using a DBMS in order to do this. MYSQL is easy enough to implement and you can also pool connectors for efficiency if you expect to recieve a large amount of transactions. [A MYSQL Tutorial can be found Here](https://www.mysqltutorial.org/python-mysql/)

This is to ensure token reuse is not possible. By desgin we have tried to make our tokens as secure as possible and offer all possible variables within the token to proove its legitimacy. However, since the tokens are being used as a proof of payment there is nothing stopping a user using the token twice to prove that they have made two payments when they have only paid once.

In our example here this is not a problem since a user can only be given a discord role once. However for other services this should be a consideration when implemetning your shop.

#### Entire Code
Once you have completed the bot it is up to you to beautify the messages and modify the help message. However, a basic Donation platform implemented with DIscord Payments will look similar to the bellow:
```python
from discord.utils import get
from discord.ext import commands
import discord
import jwt

DISCORD_TOKEN = "... Secret from secret.txt here! ..."
bot = commands.Bot(command_prefix='!')
bot.remove_command('help')

public_key = """-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAroDGdpPsGdP1DKz6OCSC
8uDEIXiaZwA04CFkH6XzyB1wClFbGUx1B/k+a87/7sOhRCgcMyjOmiV3iWUody3D
iJcpbbx+PGekVMf8qMDUJpYTQWOqau5hKcYr0WXw35JKPhqBQ7Kvlc4Q3RUlbtrL
mIjWfr6Vs04fXEY/J8Yqv8QXOOnAJSRG/WLsojBRTsHox7sXGtSHqVGVX7JPRBE2
mtj9NLTFscX3yo8C7Fx7GlKrSW6cDsv8BIsB7SiXBJF1pA5zoOroQEjJSP5Ugspi
Pccdxpv/Ea5DnR1GVe2Dekx844WH22ptWK55vQnroqiQDUj8/9C4n5I2EKJpgAUE
XQIDAQAB
-----END PUBLIC KEY-----"""

def verifyToken(encoded):
    try:
            decoded = jwt.decode(encoded, public_key, algorithms='RS256')
            return decoded
    except jwt.exceptions.InvalidSignatureError:
            # print("Invalid Token")
            return False
    except jwt.exceptions.DecodeError:
            # print("Invalid Format")
            return False
    except Exception as e:
            # print(e)
            # print("Something went wrong")
            return False

async def assignRole(ctx, roleName):
    user = ctx.message.author
    role = get(ctx.guild.roles, name=roleName)
    await user.add_roles(role)

@bot.command(name='help')
async def help(ctx):
    response = """Bot Help!
    ```
    !donate (token) -> Use this to make a donation using a Discord Payment receipt token.  
    ```
    """
    await ctx.send(response)

@bot.command(name='donate')
async def donate(ctx, receipt):
    token = verifyToken(receipt)
    if token:
        if token['username'] == str(ctx.message.author):
            if token["address"] == "Your Address here":
                if token['identifier'] == "1":
                    if int(token['amount']) >= 10:
                        await assignRole(ctx, 'Bronze')
                        response = "You have been given the Bronze Role.\n\nThanks for donating!"
                    else:
                        response = "That amount is not enough!"
                elif token['identifier'] == '2':
                    if int(token['amount']) >= 20:
                        await assignRole(ctx, 'Silver')
                        response = "You have been given the Silver Role.\n\nThanks for donating!"
                    else:
                        response = "That amount is not enough!"
                elif token['identifier'] == '3':
                    if int(token['amount']) >= 30:
                        await assignRole(ctx, 'Gold')
                        response = "You have been given the Gold Role.\n\nThanks for donating!"
                    else:
                        response = "That amount is not enough!"
                else:
                    response = "There is not a product for this Identifier!"
            else:
                response = "Payment made to a different address!"
        else:
            response = "That is not your token to use!"
    else:
        response = "Invalid Token"
    await ctx.send(response)
```

#### Making a payment
Now for a user to make a payment they first use the Discord Payment Bot to get a receipt token.

To make a payment the user decides the amount they want to donate, the crypto currency they will use, the address to send the payment too and finally the identifier of the product they are essentially buying.

In this example we will Donate $10 in order to get the Bronze role. Since $10 is not enough to make a BTC payment we will use Litecoin to make the payment instead.

Therefore the user would issue the following command in order to get a payment link from Discord Payments.
```
$$buy 10 LTC MJmfxteBdHUiN8AU47k27vw4bQvQvePkg3 1
```
Command | Amount in USD | Crypto Currency | Address | Product Identifier
--- | --- | --- | --- | ---
$$buy | 10 | LTC | MJmfxteBdHUiN8AU47k27vw4bQvQvePkg3 | 1

The discord payments Bot would reply with a link for the user to make the payment which would redirect them to [CoinPayments](https://www.coinpayment.net/).

![](https://discordpayments.com/images/coinpayments.png)

Once the payment is completed the user will be shown that the amount has been paid and they will have the option to `Visit Sellers Store` by clicking this the user will be redirected to a page displaying their receipt token.

![](https://discordpayments.com/images/token.png)

This token can then be copied and passed to the Donations Bot you have just created using the bellow command.

```
!donate eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhbW91bnQiOiIxMCIsImlkZW50aWZpZXIiOiIxIiwic2VydmVyIjo3OTE0MjQ0NTU0OTM1NTAxMDAsInVzZXJuYW1lIjoic3VfX2NoYXJsaWUjODI0MCIsImFkZHJlc3MiOiJNSm1meHRlQmRIVWlOOEFVNDdrMjd2dzRiUXZRdmVQa2czIn0.K1KWMrB8HBxehSrsjktmL9byfSFjv7uT4WlDcPP0ZDlVtRf0HIi2Lct-q5bkn4F0Ee_3vmw2X3BuzLfcZXodxGSIwdq1MM8WTITbJUWvHmQrY8F1TacDoLfIbaIjZ5rURQs6dPoPoGVRx93hqn777A-h61VrfGIWT6-DOZk-QqENAvySpJmA9zlKdH7cirys9-Pss6bFtCM82TOwJgNXnGdxPDSKo4pUFg_lQFTWcLuw4_IgL2AW6auIllU_UB3kgupt632l_LFXD7e-clz_29xAAp-beyDSQG13_hOf8p59g6PvNvGKgQ3mSqnl-OR7Dplk23qWVYnwTj4v4doFkA
```

Which will be validated by the Bot and the user will be given their role.


### Find out More
Visit https://discordpayments.com to find out more ... 

You can also join the [Official Discord Payments Community Here](https://discord.gg/EuYnVnCma7).

Thank you for reading!

`Author: @su__charlie`
