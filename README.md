# Useful Linux Commands

A collection of frequently used Linux commands for developers and newcomers.
Focused on Ubuntu/Debian: daily use, troubleshooting, and setting up a web
development environment.

> Convention: lines starting with `$` run as your user, `sudo` where noted.
> Replace `username`, `example.com`, etc. with your own values.

## Table of contents

- [Packages and system updates](#packages-and-system-updates)
- [Cleanup](#cleanup)
- [Services (systemd)](#services-systemd)
- [Logs](#logs)
- [Users, groups and permissions](#users-groups-and-permissions)
- [Files, disk and searching](#files-disk-and-searching)
- [Processes and resources](#processes-and-resources)
- [Networking](#networking)
- [Firewall (UFW)](#firewall-ufw)
- [SSH: keys and passwordless login](#ssh-keys-and-passwordless-login)
- [Copying files to a server](#copying-files-to-a-server)
- [Git basics](#git-basics)
- [Node.js, Sass and build tools](#nodejs-sass-and-build-tools)
- [LAMP (Apache, MySQL, PHP)](#lamp-apache-mysql-php)
- [Apache: rewrite, virtual host, document root](#apache-rewrite-virtual-host-document-root)
- [WordPress / web directory permissions](#wordpress--web-directory-permissions)
- [Desktop apps](#desktop-apps)
- [Audio: loop back microphone to speakers](#audio-loop-back-microphone-to-speakers)
- [Editor note](#editor-note)

## Packages and system updates

```bash
sudo apt update
sudo apt upgrade
sudo apt full-upgrade        # modern replacement for dist-upgrade
sudo apt install git curl wget nano htop
```

Install a package, search for one, show info, remove one:

```bash
sudo apt install filezilla vlc gimp
apt search <keyword>
apt show <package>
sudo apt remove <package>
```

## Cleanup

```bash
sudo apt -f install
sudo apt autoremove
sudo apt autoclean
sudo apt clean
```

## Services (systemd)

Modern replacement for `/etc/init.d/... restart` and `service ... restart`:

```bash
sudo systemctl status apache2
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2
sudo systemctl enable apache2    # start on boot
sudo systemctl disable apache2
```

Works for any service: `mysql`, `mariadb`, `ssh`, `nginx`, etc.

## Logs

```bash
journalctl -xe                       # recent system errors
journalctl -u apache2 --since today  # one service
journalctl -f                        # follow live
sudo tail -n 100 /var/log/syslog
sudo tail -n 100 /var/log/apache2/error.log
```

## Users, groups and permissions

```bash
sudo adduser username
sudo usermod -aG sudo username          # give sudo
sudo usermod -aG www-data username       # add to web group
groups username
id username
```

Ownership and modes (`r=4 w=2 x=1`):

```bash
sudo chown -R www-data:www-data /var/www
sudo chmod -R 755 /var/www
sudo find /var/www -type d -exec chmod 755 {} +  # directories
sudo find /var/www -type f -exec chmod 644 {} +  # files
```

## Files, disk and searching

```bash
ls -lah
df -h                 # disk free
du -sh * | sort -h    # folder sizes
free -h               # memory
du -sh ~/.cache
```

Find, grep, archives, sync:

```bash
find . -name '*.swp' -type f -delete   # delete matching files recursively
find /var/www -name '*.log' -mtime +7  # files older than 7 days
grep -rn "TODO" --include='*.js' .
tar -czvf backup.tar.gz folder/        # create
tar -xzvf backup.tar.gz                # extract
zip -r backup.zip folder/
unzip backup.zip
rsync -avh --delete src/ user@host:/dest/
```

Useful tools (install if missing):

```bash
sudo apt install ripgrep fd-find unzip p7zip-full rar unrar
```

(`rg` is a faster `grep`, `fdfind`/`fd` a friendlier `find` on Debian/Ubuntu.)

## Processes and resources

```bash
ps aux | grep <name>
top        # or: htop
kill <pid>
kill -9 <pid>      # force
pkill -f <pattern>
```

## Networking

```bash
ip addr                          # addresses (modern replacement for ifconfig)
ip route
ss -tulpn                        # listening ports (modern replacement for netstat)
ping -c 4 example.com
curl -I https://example.com
wget https://example.com/file.zip
nmcli device status              # NetworkManager status
nmcli device wifi list
nmcli device wifi connect "SSID" password "secret"
```

Quick Wi-Fi hotspot (replaces the old manual `system-connections` edit):

```bash
nmcli device wifi hotspot ssid MyHotspot password 'strongpassword123'
```

## Firewall (UFW)

```bash
sudo ufw status
sudo ufw enable
sudo ufw allow OpenSSH
sudo ufw allow 80,443/tcp
sudo ufw deny <port>
```

## SSH: keys and passwordless login

Use Ed25519 (modern default). RSA-4096 still works where Ed25519 is unsupported.

```bash
ssh-keygen -t ed25519 -C "you@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-copy-id user@example.com    # installs your public key on the server
ssh user@example.com
```

Handy SSH options:

```bash
ssh -p 2222 user@example.com
ssh -i ~/.ssh/id_ed25519 user@example.com
```

## Copying files to a server

```bash
scp file.txt user@example.com:/home/user/
scp -r folder/ user@example.com:/home/user/
rsync -avh -e ssh folder/ user@example.com:/home/user/folder/
```

## Git basics

```bash
sudo apt install git
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git clone <url>
git status
git pull
git add -A && git commit -m "message" && git push
```

## Node.js, Sass and build tools

Install Node via `nvm` (preferred over the stale distro `npm` package):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
nvm install --lts
node -v && npm -v
```

Sass (Dart Sass — the old `gem install sass` / Ruby Sass is end-of-life):

```bash
npm install -g sass
sass --version
```

Gulp (project-local these days, `gulp-cli` only global):

```bash
npm install -g gulp-cli
npm install --save-dev gulp
npx gulp --version
```

## LAMP (Apache, MySQL, PHP)

```bash
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql php-mbstring php-xml php-curl php-zip
php -v
sudo systemctl status apache2
sudo systemctl status mysql
```

Secure MySQL, then test PHP:

```bash
sudo mysql_secure_installation
echo "<?php phpinfo();" | sudo tee /var/www/html/info.php
# visit http://localhost/info.php, delete it afterwards:
sudo rm /var/www/html/info.php
```

phpMyAdmin (optional — many people use Adminer or DBeaver instead):

```bash
sudo apt install phpmyadmin
```

## Apache: rewrite, virtual host, document root

Enable rewrite and restart:

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

Point a local domain at your project. Edit the site (use `nano`, or
`gnome-text-editor` / `gedit` if you prefer a GUI):

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

```apache
<VirtualHost *:80>
    DocumentRoot /home/username/Sites/public_html
    <Directory "/home/username/Sites/public_html">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

```bash
sudo nano /etc/apache2/apache2.conf
```

```apache
<Directory /home/username/Sites/public_html/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

```bash
sudo nano /etc/hosts
```

```
127.0.0.1  localhost
127.0.0.1  mysite.test
```

```bash
sudo systemctl restart apache2
```

> Prefer group membership (`usermod -aG www-data`) and directory ownership for
> upload/permission issues over changing `APACHE_RUN_USER` in `envvars` —
> that breaks package upgrades and is hard to undo.

## WordPress / web directory permissions

```bash
sudo chown -R www-data:www-data /var/www
sudo find /var/www -type d -exec chmod 755 {} +
sudo find /var/www -type f -exec chmod 644 {} +
sudo usermod -aG www-data username   # log out and back in afterwards
```

## Desktop apps

Google Chrome (official repo, stays updated — instead of a one-off `.deb`):

```bash
wget -qO- https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor | sudo tee /usr/share/keyrings/google-chrome.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
sudo apt update && sudo apt install google-chrome-stable
```

Sublime Text (official repo — the old `webupd8team` PPA is dead):

```bash
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/sublimehq-archive.gpg > /dev/null
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt update && sudo apt install sublime-text
```

Others (still valid):

```bash
sudo apt install filezilla vlc gimp synaptic bleachbit
```

Dropbox (official `.deb` is simplest today):

```bash
cd /tmp && wget -O dropbox.deb "https://www.dropbox.com/download?dl=packages/ubuntu/dropbox_2020.03.04_amd64.deb"
sudo apt install ./dropbox.deb
```

## Audio: loop back microphone to speakers

```bash
pactl unload-module module-loopback
pactl load-module module-loopback
```

## Editor note

Older guides (including earlier versions of this file) used `sudo gedit`.
`gedit` is no longer preinstalled on Ubuntu — use `nano` in the terminal or
`gnome-text-editor` as a GUI replacement. Never `sudo` a GUI editor on Wayland;
prefer `nano` or `sudo -H` for system files.

## Removed / retired (why they're gone)

- **Skype (`skype skype-bin`, `sni-qt:i386`)** — Skype was retired in 2025.
- **Flash (`flashplugin-installer`), Java 7 / `oracle-java8-installer`** —
  Flash is EOL; those PPAs are dead. Use a current Temurin/OpenJDK if you need Java.
- **Unity tweaks (`indicator-netspeed-unity`, `ppa:fixnix/netspeed`, webupd8 PPA)** —
  Unity-era PPAs, unmaintained for years.
- **Ruby Sass (`gem install sass`)** — end-of-life; Dart Sass (`npm i -g sass`) replaced it.
- **PHP 7.0 / `tasksel`-era LAMP** — PHP 7.0 is EOL; the LAMP section above targets PHP 8.x.
- **`pppoeconf`, manual `wifi-hotspot` file edits, Chromebook Crouton,
  ElementaryOS 0.4 Loki, one-off ALSA mic hacks** — superseded by
  NetworkManager/`nmcli`, ChromeOS Linux, current releases, and PipeWire.

## Good luck!
