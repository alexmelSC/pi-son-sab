# pi-son-sab

This is just my notes that others should be able to use to get a new raspberry piOS up and running with SABnzBD and Sonarr

To start you need:
1. Raspberry Pi (3 orr newer)
2. Large SD card (I got a new 256GB card but anything over 32GB should be fine)
3. NZB indexer 
4. Usenet provider

If you are new to this then read 
https://www.reddit.com/r/usenet/comments/4x2mc9/new_to_the_usenet_read_this/

I'm going to go over setting up the pi, installing sonarr and installing sabnzbd. I'm doing this partly to remind myself of howto but also to give back to the community. If you like it let me know. This is the first time I've ever done this.

SETTING UP THE RASPBERRY PI

1. Get your raspberry pi and install Raspberry PiOS by using the imager provided

https://www.raspberrypi.com/software/

Install the normal non desktop one onto your SD card.

2. Remember to got to the card using your mac or linux box (no idea howto on Windows, it can't do unix filesystems) and do touch ssd on the root of the sd card.

so on mac type `touch ssd` when you have go into `Volumes\boot`. Open a command line (hit command and space then type `Terminal`)

`cd \Volumes\boot`
`touch ssd`

this lets you just put it into the Raspberry Pi and it will start the SSH daemon and allow you to log in. Otherwise you have to attach it to the TV or monitor and go into various menus to start SSH daemon.

OK, on with initial server setup. Loosely based on my experience. Feel free to skip if you know better. Do give me tips if you think it will help users.

SERVER SETUP

OK switch on the beast and go and make a coffee or tea or beverage of your choice. It takes time on the first boot. 

1. 



UPDATING SABnzbd, Sonarr and PI

