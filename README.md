A cool, but yet easy thing to do at home with your old computer is using it as a local server/disk where to place anything you do not want to keep at you diary-use pc or you just want to keep a backup copy.

So, what has to be done? Well, setting up your custom server can include several steps and can be as complicated as desired, but here there are some ideas to start with the process:

**Disclaimer**
>This tutorial has been written using GNU/Linux and specifically Debian based OS. But for most of the cases, shown commands are correct for most of GNU/Linux distributions or OSx.


# Static IP

First step should be binding our old pc to a certain local IP, this way, every coming step would be easier to accomplish.
While connected to our local network, we have to login into our router.
Normally, when our own IP its something like 192.168.1.12 we will find our router at 192.168.1.0 or 192.168.1.1
Once inside our route admin panel, we would look for an option labeled as 'static DHCP' or equivalent and should be placed inside a LAN menu.

Once inside this panel, we need to paste our PC's MAC and assign it an IP.

1. Our 'PC MAC' is in need our network's chip's MAC. When we have two network adapters (such as WiFi and ethernet), we would have two MAC addresses. So, pay attention to which network adapter you are assigning the IP, cause in case of error, you might drive crazy looking for the mistake.

2. Obtaining the IP is as simple as running an ifconfing on your terminal:
	```sh
	ifconfig
	```
and for your desired (i.e. ethX or wlanY) interface, copy the 'HWaddr' value.


3. Once you have the MAC address and the IP, just, fill the values in the new entry of your DHCP table, save the changes  check that everything is working correctly.


# IP alias

A nice way of organizing your network's ips and alias is by choosing and old music group and using album's years as IP values. I.e. Pink Floyd launched 'Meddle' on 1971, so calling your server 'meddle' (take a look to the alias part) and assigning it the value x.y.z.71 would be quick way of deciding names & numbers, and also an easy way of memorizing them.

Futher more, once we have set a static IP to our PC, we can assing that alias in our (client) pc by editing our hosts file.
 ```
  vim /etc/hosts
 ```
 and we just need to add our new static ip and choosen name. I.e.
 ```
echo '192.168.1.71	meddle' | sudo cat - >> /etc/hosts

 ```


# SSH access

ssh software needs to be installed in your server (and client pc), some GNU/Linux distributions include this sofware by default, but, since not all of them do so...just confirm:

Client:
```
sudo apt-get install openssh-client
```

Server
```
sudo apt-get install openssh-server
sudo restart ssh
```


And that's it. As simple as that, now, from our (client) pc, we can check our connection by typing in our terminal:

```
ssh {server-user}@{server-ip}
```

Following our previous examples, if we have a user named 'tony' in our server, we could access by typing:

```
ssh tony@192.168.1.71
```

or even easier:

```
ssh tony@meddle
```


# Password-less (ssh key based) access


Until now we have setup a simple ssh access to our server.

Although the syntax is quite simple to be remembered, we would have to be introducing our password eveytime we want to login into the server, such process can be simplified by usin ssh keys. We just need to type coming commands at specified scopes:


Server:

```sh
cd ~/.ssh/
touch authorized_keys
chmod 600 authorized_keys
```


Client (for each client we have in our network, we want to access from):

```
cat ~/.ssh/id_dsa.pub| ssh tony@meddle 'cat - >> ~/.ssh/authorized_keys'
```

For this step we need to have ssh-keys already generated, in case of not having them, just execute:

```
ssh-keygen
```

And follow promt's instructions.


Just for security reasons, we should disable password access to our machine specially if we are going to access to allow the access from the outside.
In our server, we need to add the following options to our /etc/ssh/sshd_config

```
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
```
And after saving, our ssh service needs to be restarted:

```
sudo service ssh(d?) restart
```

# SSHFS

Okay, until know we have our old pc working, set up with a static IP an ssh access. Yes, is really easy to login into our computer without memorizing IP addresses, but, what's the point in all this process?

Well, from now on, the whole file-system of our remote computer is just a few types away.

I.e.
```sh
nemo ssh://tony@meddle/home/tony/
#or
nemo sftp://tony@meddle/home/tony/

```
(this example works on nautilus as well)

Cool. Now we can access and use the whole file-system on our remote pc as it were part of our own system. There's no need on using USBs or tedious apps to acess your pc. As simple as typing the previous command.

What are the real options in front of us?

  * Music library. Now is remote.
  * Books? Too!
  * In the end, any kind of document can now be managed/used on

Many programs can handle remote paths/URIs, but, in the case you find an app that do not handles such scenarios (vlc has some problems with sftp/ssh), we can always mount a remote folder as a local one and then manage it.

```
#i.e.
sshfs tony@meddle:~/music ~/music-remote
```
> Note. While using nemo/nautilus, is faster to open a remote folder on nemo and the bookmark it. Nemo/Nautils will mount the folder inside the '/run/user/1000/gvfs' folder.
This opaque way of managing remote folder simplifies the mount/umount process, although it gives errors from time to time since the process is a little bit different from using sshfs.
In summary. While you wanna manage yourself the remote folder (u)mount process, I suggest you to use sshfs, but in the case you do not really have a need, using nemo/nautilus simplifies your life.

We can make a difference on whether we want to access our PC as a backup place or a 'live use' disk.
In this case we are talking about 'live use', but in coming parts we will see how to synchronize our remote PC with our daily-use one.
In the end, it all depends on what do we need or what kind of daily-use computer do we have (workstations vs ultrabooks, etc, etc)

# Transmission

You may have already think about it. From now on, you can use your local server as a 'download center' (and through sshfs easily access/use/watch your downloaded data).

Also, you may have already try to do
```
local> ssh tony@meddle
tony> transmission-gtk
```
an realize that once you logout (the ssh session is over), transmission gets closed as well.

The easiest way of dealing with this issue is setting our local server and adding transmission as an application that runs on starup and after that, set up the remote access.
In transmission: Edit > Preferences > Remote.
> Note. In the case of following this solution, you will need to add transmission as part of the startup applications.

**But**, a *cleaner* way would be using transmission-daemon. This way, transmission would be independent from the X, ssh sessions since in this case we would be dealing with a (background) service.

> Note. Every command in phase is supposed to be performed against your local server, **NOT** your personal machine.

```sh
sudo apt-get install transmission-daemon
```
Once we have it installed, we need to setup the web access.
> Depending on your GNU/Linux distribution, your settings.json might be a link to /etc/transmission-daemon/settings.json.
In such case, you might have to change your file permissions or even add our own user to the debian-transmission group.

```sh
sudo service transmission-daemon stop #JIC, if not, any change made over the settings JSON, will be ignored.
sudo vim /lib/systemd/system/transmission-daemon.service # Change user to custom
vim ~/.config/transmission-daemon/settings.json
# Change fields mentioned in the next snippet
# Check that download folder belongs to your user
sudo reboot
```


## Fields to change in your settings.json

```JSON
"download-dir": "download-dir"
"incomplete-dir-enabled": true
"incomplete-dir": "incomplete-dir"
"rpc-enabled": true
"rpc-bind-address": "0.0.0.0"
"rpc-username": "transmission"
"rpc-whitelist-enabled": false /*True for limited IP access based on the rcp-whitelist field*/
"rpc-whitelist": "127.0.0.1"
```

And that's it! You can now check on your browser (http://animal/transmission/web).
If fails, some useful commands:
+ `sudo tail -f /var/log/syslog | grep -i transmission`
+ `sudo systemctl daemon-reload && sudo service transmission-daemon restart`

## Transmission daemon bibliography

+ https://askubuntu.com/questions/261252/how-do-i-change-the-user-transmission-runs-under
+ https://askubuntu.com/questions/221081/permission-denied-when-downloading-with-transmission-deamon
+ https://askubuntu.com/questions/261252/how-do-i-change-the-user-transmission-runs-under
+ https://help.ubuntu.com/community/TransmissionHowTo
+ https://wiki.archlinux.org/index.php/transmission#Configuring_the_daemon # new user location


# Kodi (or OSMC)

One of the best things in kodi is that you can use different file-system protocols when adding folders to your library.
This way, we can use sftp and add a folder in our local server as a regular folder and then watch any media item that would be contained in it.
(Yes, somehow, our local server will be *streaming* movies to us)

For adding our remote folders we only need to go to:
> Videos > Files > Add videos

...and manually the path to the desired folder
> I.e. sftp://tony@meddle:22~/movies


A few more steps will be ahead like defining the kind of content of the folder, but basically, it's as simple as that.
In the case you are performing this steps on your raspberry-pi/OSMC and you are planning to keep it plugged all day, you will find that every movie/tvShow you download using transmission, it will *automatically* appear on your library. OR in the worst case scenario, the library will be updated on the startup.

# Backup, (r|c)sync and crontabs

A useful feature our local server can provide is using it as our own private-cloud provider.

There are already some private cloud (like OwnCloud) options that might perfectly fit for your needs.

But in my case, i haven't found a solution that fits my needs. The main problem any type of that real-time sync resolves in a poor performance experience. Since I'm a developer, many files are changing at every moment and any program used will be performing sync task almost all the time.

There is also a second issue. I want to keep redundant copies of my data in third-party clouds and making two sync programs work with the same directory while both programs have no clue on the existence of the other...well, we are facing the risk of a disaster.

My first approach to this solution was using rsync. The main gap here was that rsync it's unidirectional and keeping two folders synchronized it's messy.
This is what the first solution look liked:
```sh
remote_host="tony@meddle"

# 3 steps sync.
#   1. local -> remote only newest files (<24 hours)
#     24 hours is the crontab term this script is supposed to be ran.
#   2. local -> remote every file, removing at remote those that are not present anymore.
#     This way
#   3. remote -> local
rsync_folder(){
    local_folder=$1
    remote_folder=$2

    echo -e "\e[32mSyncing $local_folder onto $remote_folder...\e[0m"
    find $local_folder -type f -mtime -1 >> /tmp/rsyncfiles
    rsync -ruvt --files-from=/tmp/rsyncfiles $local_folder  -e ssh ${remote_host}:${remote_folder}
    rsync -ruvt ${local_folder} -e ssh ${remote_host}:${remote_folder} --delete
    rsync -ruvt -e ssh ${remote_host}:${remote_folder} ${local_folder}
    rm /tmp/rsyncfiles
}
```

I mean, it works correctly **as long as** your daily-use computer is powered on at the exact moment the cron event for this task is meant to be triggered.
If the computer is not connected in such precise moment, a really big load of new failure cases appear and you might be quickly running into a disaster.

### So, what then?

csync is the answer. In this case csync **is** bidirectional and solves every conflict cases we could face. Also, the code becomes as simple as:
```sh
csync_folder(){
  local_folder=$1
  remote_folder=$2
  echo -e "\e[32mSyncing $local_folder onto $remote_folder...\e[0m"
  csync ${local_folder} sftp://${remote_host}:${remote_folder}
}

rsync_folder '~/dev/code' '~/Dropbox/development/code'
```

No more 'newest' first, no more 'delete' from local to remote. csync will take of those cases and many other that we might haven't think about yet.

## Crontabs

There is only one thing left, we need to sync our machines periodically in an automated way. The period depends on our need, in my case, once a day is enough.

The main/default software for running jobs periodically is crontab, but it won't be our choice. The reason is that our jobs won't be triggered if the computer is not powered on at the exact moment the job is mean to be executed.
Anacron is a program that solves such issue an manages those cases where a jobs hasn't been executed because the machine was suspended or powered-off.

We only need to add
> 1	10	cron.daily	/home/tony/sync-script.sh --report /etc/cron.daily

to our /etc/anacron file and anacron will do the rest for us. There is only one restriction while using anacron instead cron, out minimum period is 24hous while in cron the minimum is 1 minute. Depending on our needs, one day could not being enough, but that's a restriction we have to assume.



# Accessing from outside

Two things need to be done in order to get access from the outside.
The router DMZ needs to be setup in order to point to the IP of the computer you want to expose to internet (including its services). This setup may depend on your router, but is quite easy to achieve. And since we already implemented some security rules, this step can be done without major risk.

The second thing that needs to be done is knowing your public IP, depending on your situation you could be able to use a static public IP so you won't need to worry about it changing ever.
But, since this services usually cost money, other free options could be using any cloud provider and a script to privately publish your public IP into your personal account, but since this it's quite tricky, at the end, the best solution would be using a dynamic DNS provider such as NO-IP (free). Its website hosts a tutorial for installing the client that would be responsible for publishing any new public IP that your ISP would assign to you.

http://www.noip.com/download?page=linux

Once you end up setting up your no-ip client you would be able to access to your personal server using a simple URL.

# Sources:
* http://en-la-mina.blogspot.com.es/2014/02/raspberry-pi.html
* http://en-la-mina.blogspot.com.es/2014/03/raspberry-pi-montar-clientes-torrent.html
* https://help.ubuntu.com/community/SSH/OpenSSH/Configuring
* https://help.ubuntu.com/lts/serverguide/openssh-server.html
* https://help.github.com/articles/generating-ssh-keys/
* http://blockdeubuntu.blogspot.com.es/2010/07/como-montar-carpetas-remotamente-con.html
* http://www.rickylford.com/transmission-on-ubuntu-server-12-04-lts/
