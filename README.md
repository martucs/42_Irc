# 42_Irc
Creating our own IRC server in C++

About the project
 -----------------------------
 The goal of this project is to create an IRC server that allows communication between different clients via channels or private chats.
 We did only the mandatory part, but implemented the following commands:

    WELP / HELP          QUIT               PART              KICK               OPER            NOTICE
    NICK                 JOIN               TOPIC             MODE               DIE
    USER                 PASS               INVITE            PRIVMSG            KILL
 
We used Hexchat as our reference client, but also did thorough testing with Netcat.

Compared to what we have done in 42 this is not a hard project, but organization and clean code is going to be important.

We were 3 members and started the division of the project by:

 - server setup
 - parsing of the line received

Both of these are pretty simple steps (be careful with the management of fds for the server though), I would say most of the work comes from the commands, which can be easily distributed between the team.

I believe with this project I have grown to understand and apprecciate C++ because I was able to apply what I have learned in the CPPs and I finally realized the perks of OOP.

Step by step
 -----------------------------
 In its core, the program should:
1. Create a listening socket in a specific port and address
2. Wait for clients to connect to it
3. Wait for any client to send information (main loop, done with poll())
4. Get the line of data
5. Parse that line to identify the command
6. Execute the command (with the parsed arguments of the command)
7. Keep waiting for any client to send data

Our class arquitecture
 -----------------------------
 To help clarify the structure of the classes, here is a simplified representation:
<img width="2097" height="1098" alt="Screenshot from 2025-07-29 16-00-04" src="https://github.com/user-attachments/assets/2bd60248-36f9-4c6f-81a2-6bdca6d93518" />
With this structure we can access all the information we need from the Clients, as well as from the Channels.

It is very important to keep it in mind when doing the commands and not forget to notify, close and handle all the information from each client and channel accordingly. 

In general, you should always think about who may be be affected by the changes you make when executing a command, not just for the client who sends it, but for everyone else as well.


    
## The commands, explained

### ***Mandatory***

#### NICK
  
| Command | Description     |
| :-------- | :------- |
| `/nick <nick>` | Sets or changes your nickname. |

Be careful here! Seems so simple that you can forget to notify all the server members about your change of nickname, if you don't do so, weird things can happen in the channels that you are in.

 #### USER
| Command | Description     |
| :-------- | :------- |
| `/user <username> <hostname> <servername> <realname>` | Sets your user. |

It's only intended to be used when logging into the server, since hexchat does it automatically, you will have to test it in netcat. Usually you will use it like this:
```
USER user 0 * :realname
```
#### PASS
| Command | Description     |
| :-------- | :------- |
| `/pass <password>` | Puts the server password. |

It's only intended to be used when logging into the server, since hexchat does it automatically, you will have to test it in netcat or configure the server in hexchat to put the password wrong.

####  JOIN

| Command | Description     |
| :-------- | :------- |
| `/join <#chann>[,<#chann>]` | Joins the channels |
| `/join <#chann> [<key>]` | Joins a channel that requires a key to join |
| `/join 0` | Leaves all the joined channels |

This command is kind of the base of the project, you need this command to test almost every other command. It's tricky because you need to take a lot into account.
Questions that you need to ask yourself every time you join a channel:
- Did the client reach the _[CHANNEL LIMIT](https://modern.ircdocs.horse/#chanlimit-parameter)_?
- Does the channel exist?

  If it does:
     - Did the channel reach the _[CLIENT LIMIT](https://modern.ircdocs.horse/#client-limit-channel-mode)_?
     - Does the channel have an _[INVITE ONLY MODE](https://modern.ircdocs.horse/#invite-only-channel-mode)_?
     - Does the channel need a _[KEY](https://modern.ircdocs.horse/#key-channel-mode)_ to enter?
   
   If it doesn't:
     - Create it (handle the name if it is > [CHANNELLENNBR](https://modern.ircdocs.horse/#channellen-parameter))
     - Add the client as member & operator

There are many replies and errors to send for this command, but they are very well documented in the [Modern IRC Protocol](https://modern.ircdocs.horse/).

This is an example of a complex line with the /join command that should not give seg fault or equivalent, just send the client the corresponding replies:
```
/join #a,b,#c ,,key
```
#### MODE
| Command | Description     |
| :-------- | :------- |
| `/mode <#chann>` | Returns the modes that the channel has. |
| `/mode <#chann> [<mode>]` | Set or remove the mode to the channel. |

The `/mode` command has several modes, but we only did the following ones:

- [-/+t](https://modern.ircdocs.horse/#protected-topic-mode) : The topic can only be changed or set by a channel operator. We set this on each channel when it is created, but it is optional to do so.
- [-/+i](https://modern.ircdocs.horse/#invite-only-channel-mode): No one can join the channel without an invitation from a channel member.
- [-/+k](https://modern.ircdocs.horse/#key-channel-mode): No one can join the channel without the key set. To set or remove this you need to specify the key in the command like `/mode -/+k key`
- [-/+o](https://modern.ircdocs.horse/#oper-user-mode): Makes a member a channel operator. To set or remove this you need to specify the member in the command like `/mode -/+o member`
- [-/+l](https://modern.ircdocs.horse/#client-limit-channel-mode): Sets a limit of members that can join. To set this you need to specify the number in the command like `/mode -/+l 5`
To set or remove a mode, you need to be channel operator. Also, you have to be able to set or remove multiple modes at the same time. 

This is a complex line with the /mode command:
```
/mode +ikl key 2
```

#### TOPIC
| Command | Description     |
| :-------- | :------- |
| `/topic <#chann>` | Returns the topic that the channel has. |
| `/topic <#chann> <topic>` | Sets a topic to the channel.  | 

The topic command seems simple, but it can be difficult to do. There is one of the answers that is weird where you have to save the time the topic was posted. There are three things you have to do, save all the data of who posted the topic, protect when that person leaves the channel and check if the channel has the `+t` mode.

#### INVITE
| Command | Description     |
| :-------- | :------- |
| `/invite <nick> <#chann>` | Invite someone to the channel. |

This command is used when the channel has the `+i` mode, only channel operators can use it. It's simple, just be careful with the responses the client receives, other than that it is very intuitive.

#### KICK
| Command | Description     |
| :-------- | :------- |
| `/kick <#chann> <nick>[,<nick>] [message]` | Removes someone from the channel. |

This command is self-explanatory, only channel operators can use it. You have to be able to eject several members at the same time and give a reason.

### ***Useful commands***
#### PART
| Command | Description     |
| :-------- | :------- |
| `/part <#chann>[,<#chann>] [message]` | Leaves said channels with a message. |

This command isn't very complex, just make sure that everyone in the channel receives that the client has left. You can call this command inside of the commands `kick` and `/join 0`

#### QUIT
| Command | Description     |
| :-------- | :------- |
| `/quit [message]` | Leaves the server with a message. |

This command should be mandatory as it is the one needed to leave the server. Remember that when a client leaves, not only do you need to close the fd, you also need to remove the client on all channels it is on.

#### PRIVMSG
| Command | Description     |
| :-------- | :------- |
| `/privmsg <target>[,<target>] <text>` | Sends a message to the target |

In hexchat it works like magic, you only need to make sure you send the reply as it should and hexchat will create a direct message window on the target. You can send messages to users or channels.

The reply that the server sends is built like this:
```
:client_nick!~client_user@client_host PRIVMSG <target> :<message>
```

### ***Additional commands***
#### NOTICE
| Command | Description     |
| :-------- | :------- |
| `/notice <target>[,<target>] <text>` | Sends a notice to the target |

It is the same as `/privmsg`, the only difference is that this command will never receive an automatic response. Not even if an error occurs.

#### OPER
| Command | Description     |
| :-------- | :------- |
| `/oper <name> <password>` | Logs in as a IRC operator |

The command `/oper` is required if you want to make the `/die` and `/kill` commands (I will explain them later). Being an IRC operator and a channel operator is not the same, so don't mix them! This command allows you to use the said commands AND the name and passwords are specified in the server, only a few know how to enter as such.

#### KILL
| Command | Description     |
| :-------- | :------- |
| `/kill <nickname> <message>` | Removes the target from the server |

Basically it's a remote `/quit` for the target. It's often used as a warning, as the target can instantly reconnect.

#### DIE
| Command | Description     |
| :-------- | :------- |
| `/die` | Kills the program |

This command kills the program, just make sure to close all fds and if you allocated some memory, free them so you don't have any leaks.

 
 To display the list of commands within the client, use the following command:
#### WELP
| Command | Description     |
| :-------- | :------- |
| `/welp` | List all commands |
| `/welp -l` | List all commands with syntaxis and description |
| `/welp <command>` | Give command syntaxis and description |

> [!IMPORTANT]
> In hexchat if you use /help it returns its own help response, it doesn't reach the server. That's why we made our own /help command. In netcat you can still use HELP.

## Running the program

```bash
  make
  ./ircserv <port> <password>
```
 - Port: The port number on which your IRC server will be listening for incoming IRC connections.

- Password: The connection password. It will be needed by any IRC client that tries to connect to your server.


Resources
 -----------------------------
Essential
- Modern IRC Protocol, AKA your best friend - https://modern.ircdocs.horse/
- guide for sockets, explains the functions in detail - https://beej.us/guide/bgnet/pdf/bgnet_a4_c_1.pdf
- ALL replies and errors with format and description - https://www.alien.net.au/irc/irc2numerics.html

Helpful
- Simplified list of commands - https://en.m.wikipedia.org/wiki/List_of_IRC_commands
- Super simple code very well explained  of a server and client in C -> https://www.linuxhowtos.org/C_C++/socket.htm
- 42 guide of an ft_irc - https://reactive.so/post/42-a-comprehensive-guide-to-ft_irc/
- IRC server article with mini C++ server example - https://medium.com/@mohcin.ghalmi/irc-server-internet-relay-chat-
- IRC project with the netcat commands syntax - https://github.com/malatini42/ft_irc?tab=readme-ov-file
- In depth video about protocols - https://www.youtube.com/watch?v=d-zn-wv4Di8
- In depth video about servers - https://www.youtube.com/watch?v=VXmvM2QtuMU
- The OG IRC Protocol, we didn't look at it much and instead looked at the Modern one - https://datatracker.ietf.org/doc/html/rfc2813

