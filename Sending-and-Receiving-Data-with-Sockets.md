## Overview

Read about sockets generally on the [Oracle Sockets Tutorial](http://docs.oracle.com/javase/tutorial/networking/sockets/index.html).

(**Needs Attention**)

### Socket Programming (Client)

We need a way to send data to a computer from our android device. Easiest way is given the local ip address of the computer on the network to simply send data via a socket:

* http://android-er.blogspot.com/2011/01/simple-communication-using.html - Guide to sending data via a socket
* http://stackoverflow.com/a/20131985/313399 - stackoverflow snippet which connects to a host, and can send data to it via an asynctask
* http://stackoverflow.com/questions/6309201/sending-tcp-data-from-android-as-client-no-data-being-sent - stackoverflow snippet sending data to an ip address via a socket
* http://thinkandroid.wordpress.com/2010/03/27/incorporating-socket-programming-into-your-applications/ (client piece is phone, server piece needs to be on computer)

## Socket Programming (Server)

To send messages between an Android phone and a computer, the computer needs to be running a program that listens for messages sent on the socket coming from the phone:

* http://tutorials.jenkov.com/java-networking/sockets.html
* http://www.journaldev.com/741/java-socket-server-client-read-write-example
* http://www.java2s.com/Code/Java/Network-Protocol/StringbasedcommunicationbetweenSocket.htm