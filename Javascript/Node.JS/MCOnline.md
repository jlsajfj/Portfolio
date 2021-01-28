# MCOnline

This is a bot I created to monitor online status of a server. This was so that we could have automated-live updates, instead of requiring an single user to check manually. This is done with the [minecraft-query](https://www.npmjs.com/package/minecraft-query) package for Node.JS.

## Table of Contents

1. [Overview](#overview)
2. [Setup](#setup)
3. [Status Checker](#status-checker)
4. [Code](#code)
   1. [Embed Code](#embed-code)
   2. [Main Code](#main-code)

## Overview

The bot functions through the following:

1. Check online status.
2. Parse to readable format.
3. Edit pre-existing status message.

## Setup

 Since the bot edits a message, rather than sending one, it requires an initial message to be sent.

This has effects in two spots, the first being in startup. If the message config does not exist, then it will not begin checking. The skeleton is shown below.

```javascript
if(FS.existsSync(mp)){ // if message config exists
    // mark that the setup is valid
    // load in data from config
    bot.channels.fetch(channelLoc).then(channel => {
        // load the channel
        ch.messages.fetch(messageLoc).then(message => {
            // load the message and begin checking
        }).catch(e => {
            // invalid config requires exit
            process.exit();
        });
    }).catch(console.error);
}
```

*Figure 1: Startup check for message*

Regardless of if the file exists, the bot will constantly await a `!!load` command. This command will set up that channel to have the update message.

This is done in a few parts. The first part is to clear the active checker and delete the former message. This is done with `clearInterval(running)` where `running` is the ID of the interval. This is shown below.

```javascript
if(setup){
    clearInterval(running);
    mess.delete().catch(console.error)
}
```

*Figure 2: Clearing timer for checking interval*

Next the bot sends the new message and initiates the timer to check.

```javascript
channelLoc = msg.channel.id;

msg.channel.send('',{
    embed: {
        "title": "Hello!",
        "description": "Bot is currently setting up.\nThis message should disappear soon."
    }
}).then( mmsg => {
    messageLoc = mmsg.id;
    var temp = {
        message: messageLoc,
        channel: channelLoc
    };
    
    var data = JSON.stringify(temp);
    FS.writeFileSync('message.json', data);
    mess = mmsg;
    check();
    running = setInterval(check,60000);
    setup = true;
}).catch(console.error);
msg.delete().catch(console.error);
```

*Figure 3: Sending new message and starting checker*

Initially it sends a pre-load message, just so there is a visible message while the bot does the first check. Once the first check is run, it is replaced. This was originally a lot closer to how the bot works while loading the message, however most of that code is redundant and has been removed.

## Status Checker

The checking script itself is very basic, the bulk of the code deals with formatting and such. Below is a basic skeleton.

```javascript
async function check(){
    q = new Query({host: mcIP, port: mcPort, timeout: 7500});
    time = new Date().toLocaleString();
    time = time.slice(0,-6)+time.slice(-3);
    
    await q.basicStat().then(s => {
        // server is online, set up embed
        mess.edit({ embed });
    }).catch(e => {
        // server is offline, set up embed
        mess.edit({ embed });
    });
}
```

*Figure 4: Checking if the server is online*

Simply enough, the query package I used throws an error when the server is offline. For the future, an `instanceof` would be good to ensure the right error is accepted. The embed itself can be found [in the code section](#embed-code).

## Code

The files can be found on the [repository](https://github.com/jlsajfj/mconline).

### Embed Code

```javascript
embed = {
    "author": {
        "name": serverName + " Server Status",
        "icon_url": serverLogo
    },
    "color": color,
    "fields": [
        {
            "name": "Status:",
            "value": status,
            "inline": true
        },
        {
            "name": "Players Online:",
            "value": players,
            "inline": true
        }
    ],
    "footer": {
        "text": "IP: " + dIP + "\nUpdated at: " + time
    }
};
```

### Main Code

```javascript
const Discord = require('discord.js');
const Query = require("minecraft-query");
const FS = require('fs');
const mp = './message.json';

const bot = new Discord.Client();
const config = require('./config.json');
var messageConfig;
bot.login(config.token);

var setup = false;
var messageLoc;
var channelLoc;

var mcIP = config.ip;
var dIP = config.displayip;
var mcPort = config.port;
var serverName = 'KOTH';
var serverLogo = "https://i.imgur.com/szHlGja.png";

var mess;
var q;
var embed;
var status;
var color;
var players;
var time;
var body;
async function check(){
    q = new Query({host: mcIP, port: mcPort, timeout: 7500});
    time = new Date().toLocaleString();
    time = time.slice(0,-6)+time.slice(-3);
    
    await q.basicStat().then(s => {
        console.log("server is online");
        //console.log(s);
        q.close();
        status = "Online";
        color = 65280;
        players = "**" + s.online_players + "** / **" + s.max_players + "**";
        embed = {
            "author": {
                "name": serverName + " Server Status",
                "icon_url": serverLogo
            },
            "color": color,
            "fields": [
                {
                    "name": "Status:",
                    "value": status,
                    "inline": true
                },
                {
                    "name": "Players Online:",
                    "value": players,
                    "inline": true
                }
            ],
            "footer": {
                "text": "IP: " + dIP + "\nUpdated at: " + time
            }
        };
        mess.edit({ embed });
    }).catch(e => {
        console.log("server is offline");
        //console.log(e);
        status = "Offline"
        color = 16711680
        players = "**0** / **0**";
        embed = {
            "author": {
                "name": serverName + " Server Status",
                "icon_url": serverLogo
            },
            "color": color,
            "fields": [
                {
                    "name": "Status:",
                    "value": status,
                    "inline": true
                },
                {
                    "name": "Players Online:",
                    "value": players,
                    "inline": true
                }
            ],
            "footer": {
                "text": "IP: " + dIP + "\nUpdated at: " + time
            }
        };
        mess.edit({ embed });
    });
    console.log(time);
    console.log();
}

var running;

bot.on('message', msg => {
    if(msg.content === '!!load'){
        console.log('\x1b[32mload initiated\x1b[0m');
        if(setup){
            clearInterval(running);
            mess.delete().catch(console.error)
        }
        channelLoc = msg.channel.id;
        
        msg.channel.send('',{
            embed: {
                "title": "Hello!",
                "description": "Bot is currently setting up.\nThis message should disappear soon."
            }
        }).then( mmsg => {
            messageLoc = mmsg.id;
            var temp = {
                message: messageLoc,
                channel: channelLoc
            };
            
            var data = JSON.stringify(temp);
            FS.writeFileSync('message.json', data);
            mess = mmsg;
            check();
            running = setInterval(check,60000);
            setup = true;
        }).catch(console.error);
        msg.delete().catch(console.error);
    }
});

bot.on('ready', () => {
    console.log("Bot is active");
    try{
        if(FS.existsSync(mp)){
            console.log('\x1b[32mbot is starting up\x1b[0m');
            setup = true;
            messageConfig = require(mp);
            messageLoc = messageConfig.message;
            channelLoc = messageConfig.channel;
            bot.channels.fetch(channelLoc).then(channel => {
                console.log(channel.name);
                
                channel.messages.fetch(messageLoc).then(message => {
                    //console.log(message.author);
                    mess = message;
                    check();
                    running = setInterval(check,60000);
                    console.log('\x1b[32mbot is done loading\x1b[0m');
                }).catch(e => {
                    console.log('\x1b[31myour config is invalid\x1b[0m');
                    console.log('\x1b[31mplease delete your message.json file\x1b[0m');
                    process.exit();
                });
            }).catch(console.error);
        } else {
            console.log('\x1b[31mmessage.json does not exist\x1b[0m');
        }
    } catch(e) {
        console.log(e);
        console.log('\x1b[31mplease delete your message.json file\x1b[0m');
    }
});
```

