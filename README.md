# cosmos-tx-bot

Download . After that, you should unzip it and you are ready to go:


wget https://github.com/alfset/cosmos-tx-bot/blob/main/cosmos-transactions-bot_1.0.0_Linux_i386.tar.gz or https://github.com/alfset/cosmos-tx-bot/blob/main/cosmos-transactions-bot_1.0.0_Linux_x86_64.tar.gz
tar xvfz cosmos-transactions-bot_1.0.0_Linux_x86_64.tar.gz
```

That's not really interesting, what you probably want to do is to have it running in the background. For that, first of all, we have to copy the file to the system apps folder:

```sh
sudo cp ./cosmos-transactions-bot /usr/bin
```

## How does it work?

It subscribes to Tendermint JSON-RPC endpoint through Websockets (see [this](https://docs.tendermint.com/master/rpc/#/Websocket/subscribe) for more details). After that, once the new transaction with the specified filter is detected, the full node sends a Websocket message, and this program catches it and sends a message to a specified channel (or channels).

## How can I configure it?

You can pass the artuments to the executable file to configure it. Here is the parameters list:

- `--node` - the gRPC node URL. Defaults to `localhost:9090`.
- `--log-devel` - logger level. Defaults to `info`. You can set it to `debug` or even `trace` to make it more verbose.
- `--telegram-token` - Telegram bot token
- `--telegram-chat` - Telegram user or chat ID
- `--mintscan-prefix` - This bot generates links to Mintscan for validators, using this prefix. Links have the following format: `https://mintscan.io/<mintscan-prefix>/validator/<validator ID>`.
- `--query` - See below.


Additionally, you can pass a `--config` flag with a path to your config file (we use `.toml`, but anything supported by [viper](https://github.com/spf13/viper) should work).

### Query

You can specify a `--query` that serves as a filter. If the transaction does not match this filter, this program won't send a notification on that. The default filter is `tx.height > 1`, which matches all transactions. You would probably want to use your own filter.

For example, we're using this tool to monitor new delegations for our validator and this is what we have in our `.toml` configuration file:
query = [
    # claiming rewards from validator's wallet
    "withdraw_rewards.validator = 'plqvaloper1qyyzwm55snqrwuchmem7yzxmxkg3lsevxmdzel'",
    # incoming delegations from validator
    "delegate.validator = 'plqvaloper1qyyzwm55snqrwuchmem7yzxmxkg3lsevxmdzel'",
    # redelegations from and to validator
    "redelegate.source_validator = 'plqvaloper1qyyzwm55snqrwuchmem7yzxmxkg3lsevxmdzel'",
    "redelegate.destination_validator = 'plqvaloper1qyyzwm55snqrwuchmem7yzxmxkg3lsevxmdzel'",
    # unbonding from validator
    "unbond.validator= 'plqvaloper1qyyzwm55snqrwuchmem7yzxmxkg3lsevxmdzel'",
    #tokens sent from validator's wallet
    "transfer.sender = 'plq1qyyzwm55snqrwuchmem7yzxmxkg3lsevc9qclw'",
    #tokens sent to validator's wallet
    "transfer.recipient = 'plqvaloper1qyyzwm55snqrwuchmem7yzxmxkg3lsevxmdzel'",
    #IBC token transferred from validator's wallet
    "ibc_transfer.sender = 'plqvaloper1qyyzwm55snqrwuchmem7yzxmxkg3lsevxmdzel'",
    #IBC token received at validator's wallet
    "fungible_token_packet.receiver = 'sent1sazxkmhym0zcg9tmzvc4qxesqegs3q4u9l5v5q'",
]


Unfortunately there is no OR operator support. See [this](https://stackoverflow.com/questions/65709248/how-to-use-an-or-condition-with-the-tendermint-websocket-subscribe-method) and [this](https://github.com/tendermint/tendermint/issues/5206) for context. You can add a few filters in the config though.

See [the documentation](https://docs.tendermint.com/master/rpc/#/Websocket/subscribe) for more information.

One important thing to keep in mind: by default, Tendermint RPC now only allows 5 connections per client, so if you have more than 5 filters specified, this will fail when subscribing to 6th one. To fix this, change this parameter to something that suits your needs in `<fullnode folder>/config/config.toml`:

```
max_subscriptions_per_client = 5
```

## Notifications channels

Currently this program supports the following notifications channels:
1) Telegram

Go to [@BotFather](https://t.me/BotFather) in Telegram and create a bot. After that, there are two options:
- you want to send messages to a user. This user should write a message to [@getmyid_bot](https://t.me/getmyid_bot), then copy the `Your user ID` number. Also keep in mind that the bot won't be able to send messages unless you contact it first, so write a message to a bot before proceeding.
- you want to send messages to a channel. Write something to a channel, then forward it to [@getmyid_bot](https://t.me/getmyid_bot) and copy the `Forwarded from chat` number. Then add the bot as an admin.


Then run a program with `--telegram-token <token> --telegram-chat <chat ID>`.

2) Slack

Go to the Slack web interface -> Manage apps and create a new app.
Give the app the `chat:write` scope and add the integration to a channel by typing `/invite <bot username>` there.
After that, run the program with `--slack-token <token> --slack-chat <channel name>`.

## Labels

You can add a label to specific wallets, so when a tx is done where the wallet is participating at, there'll be a label in the notification sent by this app. Check the Slack image at the beginning of this README to see how it looks like.

To set it up, you'll need a few things:

1. In the app config, set `labels-config` - a path to the `.toml` file where all the labels are stored and persistent.
2. Then you'll need to configure an app to handle commands.


##example command to runing your own 
cosmos-transactions-bot --base-denom aplanq --denom aplanq --tendermint-rpc tcp://localhost:33657 --node 127.0.0.1:33090 --config ~/val.toml --telegram-token <telegram Token from botfather> --telegram-chat <Chat-Id>

