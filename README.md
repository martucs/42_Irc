# 42_Irc
Creating our own IRC server in C++

About the project
 -----------------------------
 The goal of this project is to create an IRC server that allows communication between different clients via channels or private chats.
 We did only the mandatory part, but implemented the following commands:

- WELP / HELP
- NICK
- USER
- PASS
- QUIT
- JOIN
- PART
- TOPIC
- INVITE
- KICK
- MODE
- PRIVMSG
- OPER
- DIE
- KILL
- NOTICE
 
We used Hexchat as our reference client, but also did thorough testing with Netcat.

What the program should do
 -----------------------------
1. Create a listening socket in a specific port and address
2. Wait for clients to connect to it
3. Wait for any client to send information (main loop, done with poll())
4. Get the line of data
5. Parse that line to identify the command
6. Execute the command (with the parsed arguments of the command)
7. Keep waiting for any client to send data

Our class arquitecture
 -----------------------------
<img width="2097" height="1098" alt="Screenshot from 2025-07-29 16-00-04" src="https://github.com/user-attachments/assets/2bd60248-36f9-4c6f-81a2-6bdca6d93518" />
With this structure we can access all the information we need from the Clients, as well as from the Channels.

It is very important to keep it in mind when doing the commands and not forget to notify, close and handle all the information from each client and channel accordingly. 

In general, you should always think about who may be be affected by the changes you make when executing a command, not just for the client who sends it, but for everyone else as well.



 


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

