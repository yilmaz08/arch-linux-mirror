# mirror.timtal.com.tr
A Tier 2 Arch Linux mirror supported by [Teknopark Istanbul Vocational and Technical Anatolian High School](https://teknoparkistanbul.meb.k12.tr/) and its students.

Syncs from: rsync://mirrors.xtom.ee/archlinux/ (a tier 1 mirror) \
Sync interval: Every 6 hours \
Geoghraphical location: Turkiye (Istanbul)

## Maintainers
ayilmaz - Abdurrahim YILMAZ \<ayilmaz08@proton.me\> \
alpoproo - Alperen GOKDENIZ \<alperengokdeniz@gmail.com\> \
omertheroot - Omer Faruk SONMEZ \<faruksonmez1453@gmail.com\>

### Special Thanks
dybdeskarphet - Ahmet Arda KAVAKCI \<ahmetardakavakci@gmail.com\>

# How to setup
Refer to: [DeveloperWiki:NewMirrors](https://wiki.archlinux.org/title/DeveloperWiki:NewMirrors)
## Requirements
- Storage: 120+ GiB suggested
- Required packages: rsync and nginx

## Syncing
### Before syncing
You should first decide on which directory to use for the repository.
```
/srv/http/mirror/archlinux/  # suggested and used by us
```
After creating the directories you should own them.
```
sudo chown -R (yourusername):(yourgroupname) /path/to/mirror/directory
```
### Sync executable
After deciding [what mirror to sync from](https://archlinux.org/mirrors/) and target directory. You can create a syncing executable. You can use **syncrepo** (in the repository here) and put it in **/usr/bin/**.

Then make it executable with:
```
sudo chmod a+x /usr/bin/syncrepo
```

**You have to edit the bash command based on your needs**

Now you can sync for the first time! Execute the command and wait. It may take long based on your bandwith.

### Services and Timers
If you don't want to manually perform a sync everytime you need to, a service with a timer might be your saviour.

#### Service
Service file must be placed at /etc/systemd/system/ directory. You can use **syncrepo.service** (in the repository here) as a base.
**You have to edit this service based on your needs**
#### Timer
Timer file must be placed at /etc/systemd/system/ directory. You can use **syncrepo.timer** (in the repository here) as a base.
**You have to edit this timer based on your needs**

Then you need to enable timer so it starts on boot:
```
sudo systemctl enable syncrepo.timer

sudo systemctl start syncrepo.timer # if you are not going to reboot
```

And finally you are done! You have a directory that is kept up-to-date. It should be published now!

## Nginx File Server
Nginx (an HTTP and reverse proxy server) or an alternative must be installed on the server. This guide is only for nginx.

### Installation
On Debian/Ubuntu:
```
sudo apt install nginx
```
On Arch Linux:
```
sudo pacman -S nginx
```
On CentOS:
```
sudo yum install nginx
```

### Configuration
**nginx_fileserver.conf** (in the repository here) will be placed at **/etc/ngnix/sites-available/** directory.

Download it with curl:
```
sudo curl "https://raw.githubusercontent.com/yilmaz08/arch-linux-mirror/main/nginx_fileserver.conf" -o /etc/nginx/sites-available/nginx_fileserver.conf
```
**You have to edit the configuration based on your needs**

Then the configuration file should be linked to /etc/nginx/sites-enabled/

```
sudo ln -s /etc/nginx/sites-available/nginx_fileserver.conf /etc/nginx/sites-enabled/
```

Finally you can start/restart the service.
```
sudo systemctl start nginx # if not started before

sudo systemctl restart nginx # so it can use the latest configs
```
And do not forget to enable it:
```
sudo systemctl enable nginx
```

### Testing
You can test your mirror even with your browser by visiting its URL or IP.

With curl:
```
curl https://yourdomainhere/archlinux/lastsync
```
### Firewall
[HTTP](https://en.wikipedia.org/wiki/HTTP) and [HTTPS](https://en.wikipedia.org/wiki/HTTPS) protocols needs the 80 and 443 ports. So if you use a firewall, you should allow 80 and 443 ports.

For [UFW](https://en.wikipedia.org/wiki/Uncomplicated_Firewall):
```
sudo ufw allow 80 # for http
sudo ufw allow 443 # for https
```

## Rsync Service
If you are planning to become a Tier 1 Arch Linux mirror you will need to support rsync so other Tier 2 mirrors can sync from you.

### Configuration
The service configuration should be placed at **/etc/rsyncd.conf**. If needed **rsync.conf** (in the repository here) can be used too. To make the service use this new config:
```
sudo systemctl restart rsync.service
```
Then the rsync service must be enabled so it can start on boot.
```
sudo systemctl enable rsync.service
```
### Testing
To test if the rsync service is working:
```
rsync -rlptH --safe-links --delete-delay --delay-updates rsync://yourdomainhere/lastsync ~/
```
**The lastsync file will be downloaded into your home directory.**
### Firewall
[Rsync protocol](https://en.wikipedia.org/wiki/Rsync) needs the port 873. So if you use a firewall, you should allow 873 port.

For [UFW](https://en.wikipedia.org/wiki/Uncomplicated_Firewall):
```
sudo ufw allow 873
```
