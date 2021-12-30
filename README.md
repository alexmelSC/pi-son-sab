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

There will be individual service files and instructions on how to update things.

SabnzBD

Mostly from 
https://sabnzbd.org/wiki/installation/install-off-modules


1. install various bits and bobs
`sudo apt-get update; sudo apt-get install python3-pip par2 unzip p7zip-full`

2. OK hassle point. You need non free unrar....sooo
unrar
To do this we have first to add the source repository to apt
`sudo nano /etc/apt/source.list`

and add this entry at the end
deb-src http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi
(it might have a # in front of it, just remove the # and save it)

Update apt
`sudo apt-get update`

Let’s create a new folder for download and compile unrar
mkdir ~/unrar && cd ~/unrar

Tell apt to install the needed dependencies required by unrar
`sudo apt-get build-dep unrar-nonfree`

Download unrar sources and build the package
`sudo apt-get source -b unrar-nonfree`

Grab a coffee and wait the end of process… At the end we can install the package generated (x.x.x-x report the version)
sudo dpkg -i unrar_x.x.x-x_armhf.deb

well done. `unrar` now works! 

3. Sabnzbd installation

Make the /opt dir and own it
`mkdir /opt/sabnzbd /opt/share; sudo chown pi:pi /opt/*; cd /opt/sabnzbd/`

Go to sabnzbd.org Downloads and get the latest version
https://sabnzbd.org/downloads
`wget https://github.com/sabnzbd/sabnzbd/releases/download/3.4.2/SABnzbd-3.4.2-src.tar.gz`
do remember to substitute the latest Linux version (should be a .tar.gz ending)

Now untar it and 

`tar -xvf SABnzbd-3.4.2-src.tar.gz`
`cd SABnzbd-3.4.2; mkdir ../server ; mv * ../server/` 
Don't forget to change to latest version here. The last command moves all the files to a folder called server. 

Now install it using python pip
`python3 -m pip install -r requirements.txt -U`

Its done now. Well done.
`/usr/bin/python3 /opt/sabnzbd/server/SABnzbd.py -s 0.0.0.0:9090 -f /opt/sabnzbd/ini/ -d` 

will run the server. We will do this while setting up the server.

Now login to the server with your browser by going to 
`servername.local:9090`

If you get a weird hostname error, you will need to edit sabnzbd.ini located in /opt/sabnzbd/ini/ and find the "host_whitelist"
https://sabnzbd.org/wiki/extra/hostname-check.html

I'm going to go fairly light touch here but include a checklist.
a. Make "download" and "tmp" directories in share. download will have final downloads and tmp will be for files as they download. We will share both so you can use a network share to see the files. Useful to debug, delete, etc.
b. Test all your server details; saves hassles later. try to use SSL
c. Change the temp and completed folders to /opt/share/tmp and /opt/share/downloads respectively. 
d. you can add a watched folder too
e. In switches i activate "Pause Downloading During Post-Processing" and integrate reporting with my indexer "Enable Indexer Integration". You don't need to.
f. Add in RSS if that is the way you download

Test the server works.

4. Making it automatically restart

I use systemd as its my preference. You don't need to do this but if the server restarts you'll need to manually run the big line above.
`sudo nano /etc/systemd/system/sabnzbd.service`

Add in the below

`[Unit]
Description=SABnzbd
After=multi-user.target

[Service]
Type=simple
User=pi
Group=pi
Restart=always
ExecStart=/usr/bin/python3 /opt/sabnzbd/server/SABnzbd.py -b 0 -f /opt/sabnzbd/ini/
ExecStop=/usr/bin/curl -s "http://localhost:9090/sabnzbd/api?mode=shutdown&apikey=627978c144394af78391398fe82b223f"

[Install]
WantedBy=multi-user.target`

This will install the service. 

Useful systemctl commands
`sudo systemctl stop sabnzbd.service` stops the service
`sudo systemctl start sabnzbd.service` starts the service
`sudo systemctl restart sabnzbd.service` restarts
`sudo journalctl -u sabnzbd -f` shows the logs for this 
`systemctl daemon-reload` reloads the daemon, do this to incorporate any change to the service i.e. after writing the above file.

5. Updating

Updating is as simple as stopping the server then downloading the newer version then mving it to /opt/sabnzbd/server and restarting. 


SONARR

1. Again hassle here is to install mono which Sonarr runs on

`sudo apt -y install apt-transport-https dirmngr gnupg ca-certificates`
`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF`
`sudo sh -c 'echo "deb https://download.mono-project.com/repo/debian stable-buster main" > /etc/apt/sources.list.d/mono-official-stable.list'`
`sudo apt -y update ; sudo apt -y install mono-devel mediainfo sqlite3 libmono-cil-dev libchromaprint-tools`
`sudo apt install mono-complete`

Now create folders and install mediainfo
`sudo mkdir /opt/sonarr ; sudo chown pi:pip /opt/sonarr ; cd /opt/sonarr ; mkdir mediainfo ; cd mediainfo`
`wget https://mediaarea.net/repo/deb/repo-mediaarea_1.0-19_all.deb && sudo dpkg -i repo-mediaarea_1.0-19_all.deb && sudo apt-get update ; rm -rf /opt/sonarr/mediainfo/`

Now get Sonarr and fix perms and move to right place
`curl -sL "https://services.sonarr.tv/v1/download/phantom-develop/latest?version=3&os=linux" -o /tmp/Sonarr.tgz ; sudo tar xvzf /tmp/Sonarr.tgz -C /opt/sonarr/ ; sudo chown -R pi:pi /opt/sonarr/ ; mv /opt/sonarr/Sonarr/* /opt/sonarr/server; mkdir /opt/sonarr/ini ; rmdir /opt/sonarr/Sonarr`

Finally setup the systemd file
`[Unit]
Description=Sonarr Daemon
After=network.target

[Service]
# Change and/or create the required user and group.
User=pi
Group=pi

ExecStart=/usr/bin/mono --debug /opt/sonarr/server/Sonarr.exe -nobrowser -data=/opt/sonarr/ini/

Type=simple
TimeoutStopSec=20
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target`

Now enable the service
`sudo systemctl daemon-reload ; sudo systemctl enable sonarr.service ; sudo systemctl start sonarr.service ; sudo journalctl -u sonarr -f`
this will dump you into the logs while it all starts. Once its working go to the browser and do what you need to. The default port is 8989.

MOUNTING EXTERNAL DRIVES AND AUTOMOUNT

These are more notes for me, i hope they are useful to somone.

//servername/Folder /folder/folder/local cifs credentials=/home/pi/.smbcredentials,vers=2.1,uid=33,gid=33,rw,nounix,iocharset=utf8,file_mode=0777,dir_mode=0777 0 0

.smbcredentials should be

user=
password=

then run `chmod o-rwx,-x ~/.smbcredentials`


