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

1. Passwords: 
log into the server. so its ssh `pi@raspberry.local` should get you in. 
change the password with `passwd`
log out and go passwordless with ssh. So press `Ctrl-D` or type `logout`
`ssh-copy-id pi@raspberrypi.local` will log you back in and setup all the files so next time you can just login no sweat.

[note i am assuming you have setup `ssh-keygen` etc so you have a private key on your laptop. If not then google away till you have setup a key etc etc. Or just login with passwords it makes no difference.]

2. Change hostname: 
I like a different server name so 
`sudo nano /etc/hostname`
change `raspberrypi` your preferred name and save

I named mine `rum` but you can use whatever

Now reboot so it all works. Well done. Secure access achieved.

3. Now update the pi and install all the normal useful software.

`sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get -y autoremove --purge && sudo apt-get -y autoclean`

Nice one. All updated now. Worth doing the above every so often. Note this does NOT update Sonarr or SABnzbd as they are not in the normal sources.

`sudo apt-get update && sudo apt-get install wget python3 tmux `

python as its needed for SABnzbd, wget so you can download things and tmux as i use it. Learn tmux. It stops a lot of problems. Great bit of software.

https://learn.adafruit.com/an-illustrated-guide-to-shell-magic-typing-less-and-doing-more/use-a-terminal-multiplexer is a great basic tutorial.

UPDATING SABnzbd, Sonarr and PI

Now i will install the above to the /opt/ directory but you can install it anywhere. You figure it out. 

There will be:
/opt/sabnzbd
/opt/sonarr
/opt/mono <----this is the software that runs Sonarr
/opt/share <----this will have all the files and will be accessible via Samba as this is useful to move things

