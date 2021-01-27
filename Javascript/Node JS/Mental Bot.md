# Mental Bot

Mental Bot is a bot I created for a personal server, where we had too many bots. The idea was the condense the most used features of every bot into one. The bot itself can be found [here](https://github.com/jlsajfj/MentalBot); it is still a work in progress.

## Contents

1. [Commands](#commands)
   1. [Clear](#clear)
   2. [Ping](#ping)
2. [Full Code](#full-code)
   1. [Main](#main-bot-code)
   2. [Clear](#clear-code)

## Commands

Mental Bot implements modularity through it's command structure. The commands are loaded in automatically so long as the file exists and the permissions are configured, as shown below.

```javascript
const commandFiles = readdirSync(join(__dirname, "commands")).filter(file => file.endsWith(".js"));
for (const file of commandFiles) {
  const command = require(join(__dirname, "commands", `${file}`));
  client.commands.set(command.name, command);
}
```

*Figure 1: Command import structure*

```javascript
const command = client.commands.get(args[1])
if(!command){
    Send.fail(msg, replies.invalid_command)
    return
}

if(msg.member.roles.cache.find(r => perms[args[1]].includes(r.name))){
    command(msg, args, client)
    return
}
Send.fail(msg, replies.insufficient_permissions)
```

*Figure 2: Permission check structure*

The permissions themselves are in a JSON file, `perms.json`.

To activate the bot, the user would tag it, and this functionality was created and tested in [Mention Bot](./Mention-Bot.md). This is detailed in more depth there but a basic overview is that it checks the first argument of the command, to see if the first argument tagged ID is the same as the bot's ID. This is shown below.

```javascript
if(msg.mentions.users.keyArray().includes(client.user.id)){
    Log.info(`${msg.author.username} sent the message: ${msg.content}`)
    var args = msg.content.split(' ')
    if(args.length == 1){
        Send.success(msg, replies.default_reply)
        return
    }
    if(args[0]!=`<@!${client.user.id}>`&&args[0]!=`<@${client.user.id}>`){
        Send.success(msg, replies.mistake_tag)
        return
    }
    // execute commands
}
```

*Figure 3: Checks for valid tag, along with tag location*

The strings `<@!${client.user.id}>` and `<@${client.user.id}>` are how Discord processes tagged users.

### Clear

The clear command is versatile in usage. The basis of the clear command is the delete function, which can take a variety of arguments. A skeletal overview is shown below.

```javascript
async function deleteMessages(cnl, count, client, user, mm){
    if(user){
        // if a user is passed, clear messages from that user
        return
    }
    
    await cnl.bulkDelete(count).then(messages => {
        // otherwise use bulk delete to clear messages
        return
    }).catch(async (e) => {
        // if there are messages over 2 weeks
        // use custom individual deletion to clear them
    })
}
```

The most basic usage is `@MentalBot clear`, where the bot will clear the latest 100 messages. As shown below, it simply calls the delete function for 100 messages.

```javascript
if(args.length == 2){
    await deleteMessages(cnl, 100)
    return
}
```

## Full Code

This section is just strict code, if there is interest in that. The files themselves can also be found in the [repository](https://github.com/jlsajfj/MentalBot).

### Main Bot Code

```javascript
const { Client, Collection } = require("discord.js");
const { readdirSync, readFile } = require("fs");
const { join } = require("path");

const { config, replies, perms } = require("./config")

const Send = require("./send.js")
const Log = require("./logging.js")

const client = new Client();
client.login(config.token);
client.commands = new Collection();

Log.info("MentalBot is initializing")
client.on('ready', () => {
    client.user.setPresence({
        activity:{
            name: 'some mental game'
        },
        status: 'online'
    })
    Log.success('MentalBot is online')
});

const commandFiles = readdirSync(join(__dirname, "commands")).filter(file => file.endsWith(".js"));
for (const file of commandFiles) {
    const command = require(join(__dirname, "commands", `${file}`));
    client.commands.set(command.name, command);
}

client.on("message", async(msg) => {
    if(msg.author.id === client.user.id) return
    await readFile('./auto_replies.json', (err, data) => {
        if (err) throw err;
        var auto_replies = JSON.parse(data);
        if(msg.content in auto_replies){
            Log.info(`${msg.author.username} sent the message: ${msg.content}`)
            Send.success(msg, auto_replies[msg.content])
            return
        }
    });
    if(msg.mentions.users){
        if(msg.mentions.users.keyArray().includes(client.user.id)){
        Log.info(`${msg.author.username} sent the message: ${msg.content}`)
            var args = msg.content.split(' ')
            if(args.length == 1){
                Send.success(msg, replies.default_reply)
                return
            }
            if(args[0]!=`<@!${client.user.id}>`&&args[0]!=`<@${client.user.id}>`){
                Send.success(msg, replies.mistake_tag)
                return
            }
            const command = client.commands.get(args[1])
            if(!command){
                Send.fail(msg, replies.invalid_command)
                return
            }

            if(msg.member.roles.cache.find(r => perms[args[1]].includes(r.name))){
                command(msg, args, client)
                return
            }
            Send.fail(msg, replies.insufficient_permissions)
        }
    }
});

client.on('rateLimit', info => {
    Log.info(`Rate limit hit ${info.timeDifference ? info.timeDifference : info.timeout ? info.timeout: 'Unknown timeout '}`)
})
```

### Clear Code

```javascript
const Send = require("../send.js")
const Log = require("../logging.js")
const { DiscordAPIError } = require("discord.js")

async function clear(msg, args, client){
	var cnl = msg.channel
	if(args.length == 2){
		await deleteMessages(cnl, 100)
		return
	}
	if(isNaN(args[2])){
		if(args[2].match(/<@!{0,1}\d+>/)){
			// console.log(args[2])
			// console.log(args[2].substring(args[2].indexOf('<@')+2+(args[2].indexOf('!')==-1?0:1),args[2].indexOf('>')))
			var count = 100
			if(args[3] && !isNaN(args[3])){
				count = parseInt(args[3])
			}
			var user = args[2].substring(args[2].indexOf('<@')+2+(args[2].indexOf('!')==-1?0:1),args[2].indexOf('>'))
			await deleteMessages(cnl, count, client, user, msg)
		}
		return
	}
	if(args.length == 3){
		var cnt = parseInt(args[2])
		if(cnt < 100){
			deleteMessages(cnl, cnt)
			return
		}
		while(cnt > 0){
			if(cnt > 100){
				await deleteMessages(cnl, 100)
				await new Promise(r => setTimeout(r, 1500));
				cnt -= 100
				continue
			}
			await deleteMessages(cnl, cnt)
			cnt = 0
		}
		return
	}
}

async function deleteMessages(cnl, count, client, user, mm){
	if(user){
		var m;
		var username;
		await client.users.fetch(user).then(u => username = u.username).catch(console.error)
		await Send.info(cnl, `Clearing messages from ${username} in the last ${count} messages`).then( message => m = message).catch(console.error)
		var messages
		await cnl.messages.fetch({limit: count}).then( mmm => messages = mmm ).catch(console.error)
		// console.log(messages)
		var msgs = messages.filter( msg => {
			// console.log(msg)
			return msg.author.id === user
		})
		// console.log(msgs)
		await cnl.bulkDelete(msgs).then(messages => {
			Log.success(`Bulk deleted ${messages.size} message(s)`)
		}).catch(console.error)
		if(user != client.user.id){
			m.delete()
		}
		if(mm.author.id != user){
			mm.delete()
		}
		return
	}
	var m;
	await Send.info(cnl, `Clearing ${count} messages`).then( message => {
		m = message
	})
	await cnl.bulkDelete(count).then(messages => {
		Log.success(`Bulk deleted ${messages.size} message(s)`)
		return
	}).catch(async (e) => {
		if(e instanceof DiscordAPIError){
			Log.fail("Discord API Error: Deleting over two weeks\nStarting workaround\nTHIS IS VERY SLOW")
		}
		// console.error(e)
		var msgs = cnl.messages
		var messages
		await msgs.fetch({limit: count}).then(mmm => messages = mmm).catch(console.error);
		Log.warn(`Received ${messages.size} messages`)
		//console.log(messages)
		var cnt = messages.size
		for (message of messages.keys()) {
			// console.log(`deleting ${message}`)
			await msgs.fetch(message).then( mess => mess.delete()).catch(console.error)
			cnt -= 1
			await new Promise(r => setTimeout(r, 1000))
		}
		Log.Success(`Cleared ${count} messages`)
	})
}

module.exports = clear
```

