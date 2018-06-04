This guide has been tested with an Ubuntu 16.04 VPS installation, for both the [20GB SSD, 512MB RAM, 500GB Bandwidth] and [25GB SSD, 1024MB RAM, 1000GB Bandwidth] options.

The setup assumes that your local pc, either Windows or Linux will run the wallet.  You will be able to view and enable/disable your masternode from there.  A VPS will also be used to host the SnowGem node with a synced blockchain for reliability and the convenience of not needing to have a dedicated local PC on continuously.

This guide will focus on the Linux VPS part.  For a graphical reference on setting up the local Windows wallet, check out… https://snowgem.org/how-to-setup-a-masternode/

I have added steps that make your node more secure, safer than the default configuration, and a bit more convenient.  Those steps are optional, however, highly recommended.   

THIS GUIDE IS BEING PROVIDED "AS IS", WITHOUT ANY WARRANTY OF ANY TYPE OR NATURE, EITHER EXPRESS OR IMPLIED, ... IF YOU RELY UPON THIS SOFTWARE OR PROGRAM, YOU DO SO AT YOUR OWN RISK, AND YOU ASSUME THE RESPONSIBILITY FOR THE RESULTS.


Log into your VPS, and retrieve your username and password.  Your username is likely “root”, and your password a long alpha-numeric string.  Download and install Putty.  Make sure you configure Putty so that you can copy and paste the commands from this guide, otherwise you’ll likely introduce errors if manually entering all commands.

## Upgrade the VPS OS

```ssh
apt-get update && apt-get upgrade -y
```

Give your VPS a meaningful name. Ex. xsg_node
```ssh
hostnamectl set-hostname xsg_node
```
```ssh
nano /etc/hosts
```
Add line to hosts file, you should use your ip address and the name you used with hostnamectl.

123.456.789.987 xsg_node

Set to local time for convenience.  Use timedatectl list-timezones to find your time zone then add to zone to the following command.
```ssh
timedatectl set-timezone 'America/New_York'
```

## Swap Space
You will need at least 4GB of RAM to build from source, OR you can
enable a swap file. To enable a 4GB swap file on modern Linux distributions:
```ssh
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
Now make the swap work better. Add a line to sysctl.conf
```ssh
sudo nano /etc/sysctl.conf
```


add to bottom:

	vm.swappiness=10

Then make it so the swap gets mounted when the server reboots. Edit the fstab file
```ssh
sudo nano /etc/fstab
```
add to bottom:

	/swapfile   none    swap    sw    0   0
	tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0

You can verify swap is set correctly by seeing it show up.
```ssh
free -h
```

Add user with sudo capabilities, change fr4nk to whatever you want your username to
be.
```ssh
adduser fr4nk && adduser fr4nk sudo
```
Disable root user
```ssh
sudo nano /etc/ssh/sshd_config
```
Change “PermitRootLogin no”

```ssh
sudo systemctl restart sshd.service
```

Reboot VPS and make sure you can log in with your new username and password.

## Add a security login banner (Optional, but highly recommended)

Yes, it's purely psychological, but it's such an easy step, it shouldn't be overlooked.
```ssh
sudo nano /etc/issue.net
```
Add the following message, or something similar to the file.  When you log in via SSH again, it will warn any wood be trespassers as they try to get in.

"WARNING:  This service is restricted to authorized users only. All activities
 on this system are logged.  Unauthorized access will be fully investigated
and reported to the appropriate law enforcement agencies."

Next, we need to disable the banner message from motd.
```ssh
sudo nano /etc/pam.d/sshd
```

Comment out the following two lines (adding a # to the beginning of each line):

```ssh
#session optional pam_motd.so motd=/run/motd.dynamic
#session optional pam_motd.so noupdate
```

Now open the sshd_config file:
```ssh
sudo nano /etc/ssh/sshd_config
```
Comment out the following line (adding a # to the beginning):
```ssh
#Banner /etc/issue.net
```
Save and close that file.

Finally, restart the ssh server with the command:
```ssh
sudo service ssh restart
```

## Basic Intrusion Prevention with Fail2Ban (Optional, but highly recommended)

This will stop various people on the Internet from running non-stop dictionary attacks against your system. Well, it will slow them down. After 10 failed login attempts from a single IP address, it blocks that IP address from trying to login again for 10 minutes. Better than no protection, anyway.

```ssh
sudo apt -y install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

If you are interested in seeing what fail2ban is actually doing, watch your fail2ban log for a little while. That’s the great thing about servers, they write things down when things are going well, and especially when things are going badly, In Linux, all those logs are readable. Here is one way to look at the fail2ban log. Type Ctrl-c to exit the tail application.

```ssh
sudo tail -f /var/log/fail2ban.log
```

## Install a rootkit detector (Optional, but highly recommended)

Wouldn’t it be great, if you were hacked, that your VPS had a chance at figuring that out then telling you? That’s what rkhunter does. It’s a basic application to let you know. That way if you are hacked you can wipe your server image clean and restore from backup.

Install rkhunter and do an initial file scan.

```ssh
sudo apt -y install rkhunter
sudo rkhunter --propupd
```

If you update your VPS you will want to run the rkhunter scan right after so it sees the update files. You could write yourself a handy little upgrade script and run it so you don’t forget these things. Let’s do that right now!

```ssh
cd ~
nano upgrade_script.sh
```

Place the following 5 lines into this file and save.
```
#!/bin/bash
sudo apt update
sudo apt -y dist-upgrade
sudo apt -y autoremove
sudo rkhunter --propupd
```

After saving the file, change its permissions so it can be run
```ssh
chmod +x upgrade_script.sh
```
Now run it with admin permissions. Now you don’t need to remember all the commands to upgrade your system. Just login and run the shell script.
```ssh
sudo ./upgrade_script.sh
```

## Firewall (Optional, but highly recommended)

Check if firewall is already running. It should not be, so disable if it is.
```ssh
sudo ufw status
```
The next steps should be done carefully, and in the correct order.

```ssh
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow http/tcp
sudo ufw allow https/tcp
sudo ufw allow 16113/tcp
sudo ufw logging on
sudo ufw enable
```

## Snowgem Node Setup

```ssh
sudo apt-get update
```
```ssh
sudo apt-get -y install build-essential pkg-config libc6-dev m4 g++-multilib autoconf libtool ncurses-dev git python python-zmq zlib1g-dev wget bsdmainutils automake curl unzip nano
```

```ssh
mkdir ~/.snowgem
```

To add your data into these .conf files you will need to go back to the local pc and copy/paste into these files.  See the visual guide, https://snowgem.org/how-to-setup-a-masternode/, for reference.

Back to local machine, Click on Tools -> Copy snowgem.conf data.
```ssh
touch ~/.snowgem/snowgem.conf
nano ~/.snowgem/snowgem.conf
```
Paste and save information. Then go back to local machine, click on your alias, right click and choose “Copy alias data”
```ssh
touch ~/.snowgem/masternode.conf
nano ~/.snowgem/masternode.conf
```

```ssh
wget https://snowgem.org/downloads/snowgemparams.zip -N
unzip -o snowgemparams.zip -d ~/
```

_Build the binary_
```ssh
git clone https://github.com/Snowgem/Snowgem.git snowgem-wallet
cd snowgem-wallet
chmod +x zcutil/build.sh depends/config.guess depends/config.sub autogen.sh share/genbuild.sh src/leveldb/build_detect_platform
./zcutil/build.sh --disable-rust #could take up to 2 hours, it depends on VPS hardware you selected.
```
Run the following command to start the wallet:
```shh
cd ~/snowgem-wallet/
./src/snowgemd -daemon
```
Verify it’s syncing by making sure block count is increasing.
```ssh
./src/snowgem-cli getinfo
```

Wait for the sync finish. Could take up to 8 hours.  Run the following command to check masternode syncing process:
```ssh
./src/snowgem-cli masternodedebug
```

If the response is: “Not capable masternode: Hot node, waiting for remote activation.” you can move to next step.

Start MasterNode
In your local windows wallet, select your Alias then click on Start MN button.

You will get the success message.  Then click on Start Alias button:
You will get another success message.  Your masternode is now active.

Wait for some minutes, your masternode will be listed.

Go to VPS, run the following command:
```shh
cd ~/snowgem-wallet/
./src/snowgem-cli masternodedebug
```

If the response is: “Masternode successfully started“, you’re finished.

Monitor your snowgemd process and restart

For additional peace of mind you can install Monit on your linux Masternode VPS for unattended monitoring of the snowgemd daemon process.

Start off by installing Monit.

Log in to your VPS using PuTTY or anyone of your favorite SSH clients.
```shh
sudo apt-get install monit​
```


In all of the below examples is it IMPORTANT you change fr4nk to the username you are using.

Create file start_snowgemd.sh
```ssh
cd ~
sudo nano /home/fr4nk/.snowgem/start_snowgemd.sh
```

and add the below two lines to it:

***Be careful with the quotes, make sure they match file***
```
#!/bin/bash
/bin/su fr4nk -c '/home/fr4nk/snowgem-wallet/src/snowgemd 2>&1 >> /home/fr4nk/.snowgem/rc.local.log'
```

Make it executable.
```ssh
sudo chmod 755 /home/fr4nk/.snowgem/start_snowgemd.sh​
```

Edit monitrc file
```ssh
sudo nano /etc/monit/monitrc​
```

Edit the file as follows:

uncomment these 3 lines
```
set httpd port 2812 and
use address localhost # only accept connection from localhost
allow localhost # allow localhost to connect to the server and
```

Add the following lines to the bottom of the monitrc file.

***Be careful with the quotes, make sure they match file***  Also this works with Ubuntu 16.04, if using something else the path may have to be adjusted!

```ssh
check process snowgemd with pidfile /home/fr4nk/.snowgem/snowgemd.pid
  start program = "/home/fr4nk/.snowgem/start_snowgemd.sh" with timeout 60 seconds
  stop program = "/bin/su fr4nk -c /home/fr4nk/snowgem-wallet/src/snowgem-cli stop"​
```

Load the new configuration
```ssh
sudo monit reload
```

Enable the watchdog
```ssh
sudo monit start snowgemd​
```

That’s it. You only have to issue the above command once, monit will auto start in future. You can check monit’s status by typing below:
```ssh
sudo monit status​
```

It will keep your snowgemd running for you (across reboots too, no need for any crons or scripts) and keep you from fighting with your chosen OS. Monit only runs once a minute, so be patient if you’re waiting for it to do something.

If you need proof it works, once you see your snowgemd in the ‘sudo monit status’ output, you can test it by simply stopping your snowgemd. Within 2 minutes it should start back up automatically.
```ssh
cd ~/snowgem-wallet/
./src/snowgem-cli stop
```

Restart your local PC wallet and check under Masternode tab if your MN is listed and running, if not you can “Start Alias” on selected MN again.

## Other useful information

Before rebooting the VPS, always stop snowgemd.  
	cd ~/snowgem-wallet/
	./src/snowgem-cli stop

More to come...
