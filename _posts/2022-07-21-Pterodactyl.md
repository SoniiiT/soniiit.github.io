---
title: Game Server Pterodactyl
date: 2022-07-21 12:30 +0200
categories: [gameserver,voiceserver,hosting,linux]
tags: [gameserver,voiceserver,pterodactyl, minecraft server,ts3 server,linux]          #Tags müssen immer klein geschrieben werden.
published: true
# https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet
---

![img-description](https://cdn.pterodactyl.io/logos/new/pterodactyl_logo_transparent.png)

# Was ist Pterodactyl?
Pterodactyl ist ein Service in dem du deine eigenen Game Server / Voice Server erstellen und verwalten kannst. In diesem Guide wirst du schritt für schritt erfahren wie du deine eigenen Game Server / Voice Server erstellen kannst.

# Abhängigkeiten
Hier sind ein paar Befehle um die Abhängigkeiten zu installieren.
```shell
# Add "add-apt-repository" command
sudo apt -y install software-properties-common curl apt-transport-https ca-certificates gnupg

# Add additional repositories for PHP, Redis, and MariaDB
sudo LC_ALL=C.UTF-8 add-apt-repository -y ppa:ondrej/php
sudo add-apt-repository ppa:redislabs/redis -y

# MariaDB repo setup script can be skipped on Ubuntu 22.04
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash

# Update repositories list
sudo apt update

# Add universe repository if you are on Ubuntu 18.04
sudo apt-add-repository universe

# Install Dependencies
sudo apt -y install php8.1 php8.1-{common,cli,gd,mysql,mbstring,bcmath,xml,fpm,curl,zip} mariadb-server nginx tar unzip git redis-server
```
# Installieren von Composer
Composer ist ein Abhängigkeitsmanager für PHP. Du musst den Composer installieren um hier weiter machen zu können.
```shell
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

# Pterodactyl runterladen
Um Pterodactyl runterzuladen müsst du erst ein Verzeichnis dafür erstellen.
```shell
sudo mkdir -p /var/www/pterodactyl
cd /var/www/pterodactyl
```
Nachdem du ein neues Verzeichnis erstellt hast und auch schon drin bist kannst du die dateien runterladen.
```shell
sudo curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
sudo tar -xzvf panel.tar.gz
sudo chmod -R 755 storage/* bootstrap/cache/
```

# MySQL einrichten
Um Pterodactyl nutzen zu können musst du für Pterodactyl eine Datenbank erstellen.

### Einloggen
Zuerst musst du dich in der Datenbank anmelden.
```shell
sudo mysql -u root -p
```

### Ein User erstellen
Als nächstes erstellen wir einen User names `pterodactyl` mit einem `passwort`

**Vergiss nicht 'Passwort' mit einem starken Passwort zu ersetzten.**
```shell
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'Passwort';
```

### Eine Datenbank erstellen
Nun müssen wir eine Datenbank erstellen damit Pterodactyl sich später mit verbinden kann.
```shell
CREATE DATABASE panel;
```

### Rechte vergeben
Zu guter letzt müssen wir dem User Pterodactyl seine Rechte geben damit es sich auf die Datenbank verbinden kann.
```shell
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
```

# Eine Datenbank für die Nodes erstellen
### Ein neuen User erstellen
Nun müssen wir einen neuen User erstellen damit wir über den Panel oder auch der Daemon Server konfigurieren kann. Sollte die Datenbank auf einem anderen Server sein ändere die IP `127.0.0.1` auf die IP des Server wo auch die Datenbank liegt.
**Vergiss hier nicht den 'User' umzubennen und auch das 'Passwort'.**
```shell
CREATE USER 'User'@'127.0.0.1' IDENTIFIED BY 'Passwort';
```

### Rechte Vergeben
Nun vegibst du die Rechte für den neuen User. Auch hier sollte wie oben auch die Datenbank auf einem andern server sein, ändere die IP `127.0.0.1` auf die IP des Servers.
**# Auch hier nicht vergessen 'User' umzubenennen.**
```shell
GRANT ALL PRIVILEGES ON *.* TO 'User'@'127.0.0.1' WITH GRANT OPTION;
```
Gebe danach `exit` ein.

### Zugriff erlauben von externen Datenbasen
Warscheinlich musst du den zugriff auf externen Server erlauben um drauf connecten zu können. Dafür musst du die `my.cnf` finden. Das geht einfach indem du `sudo find /etc -iname my.cnf` eingibst.
Warscheinlich ist es aber unter `sudo nano /etc/mysql/my.cnf` zu finden.
Öffne die `my.cnf` und füge das hinzu:
```
[mysqld]
bind-address=0.0.0.0
```
# Installation
Zuerst musst du eine Einstellungs Datei kopieren und dann ein Core installieren. Dann wir eine eine Application Key generiert.
```shell
sudo cp .env.example .env
sudo composer install --no-dev --optimize-autoloader
sudo php artisan key:generate --force
```

### Umgebungskonfiguration
Pterodactyl's Core zu konfigurieren ist einfach, da es in ein paar CLI Befehlen konfigueriert werden kann.
```shell
sudo php artisan p:environment:setup
```
Danach öffnet sich das CLI:
```shell
vgssoniii@vgameserver:/var/www/pterodactyl$ sudo php artisan p:environment:setup

 // Provide the email address that eggs exported by this Panel should be from. This should be a valid email address.

 Egg Author Email [unknown@unknown.com]:
 > Deine@eMail.de

 // The application URL MUST begin with https:// or http:// depending on if you are using SSL or not. If you do not
 // include the scheme your emails and other content will link to the wrong location.

 Application URL [http://panel.example.com]:
 > http://IP-Adresse

 // The timezone should match one of PHPs supported timezones. If you are unsure, please reference
 // http://php.net/manual/en/timezones.php.

 Application Timezone [America/New_York]:
 > Europe/Berlin

 Cache Driver [Filesystem]:
  [redis    ] Redis (recommended)
  [memcached] Memcached
  [file     ] Filesystem
 > redis

 Session Driver [Filesystem]:
  [redis    ] Redis (recommended)
  [memcached] Memcached
  [database ] MySQL Database
  [file     ] Filesystem
  [cookie   ] Cookie
 > redis

 Queue Driver [Redis (recommended)]:
  [redis   ] Redis (recommended)
  [database] MySQL Database
  [sync    ] Sync
 > redis

 Enable UI based settings editor? (yes/no) [yes]:
 > yes

 ! [NOTE] Youve selected the Redis driver for one or more options, please provide valid connection information below.
 !        In most cases you can use the defaults provided unless you have modified your setup.

 Redis Host [localhost]:
 > localhost

 // By default a Redis server instance has no password as it is running locally and inaccessible to the outside world.
 // If this is the case, simply hit enter without entering a value.

 Redis Password:
 > Dein_Passwort

 Redis Port [6379]:
 > 6379
```
## Redis einrichten
Nun nachdem Redis das erstemal aufgesetzt wurde, musst du es einrichten.

### Passwort einrichten
Du musst ein Passwort für Redis einrichten.
Dafür musst du folgendes eingeben `sudo nano /etc/redis/redis.conf` und dann musst du nach folgender Zeile suchen `# requirepass foobared` und ändere es auf `requirepass Dein_Passwort`.

### Redis einrichtung abschließen
Nachdem das Passwort geändert wurde, muss redis kurz configuriert werden und dann kann die Installation von Pterodactyl weitergehen.
```shell
redis-cli
```
**Alles was nach dem `">"` kommt eingeben und `"Dein_Passwort"` durch das Passwort ersetzten was in der `"redis.conf"` angegeben wurde.
```shell
127.0.0.1:6379> CONFIG SET requirepass "Dein_Passwort"
OK
127.0.0.1:6379> AUTH "Dein_Passwort"
OK
127.0.0.1:6379> exit
```

### Umgebungskonfiguration
Nun da Redis konfiguriert wurde können wir nun weiter machen.
## Datenbank einrichten
Gebe folgendes ein damit die Datenbank eingebunden werden kann.
```shell
sudo php artisan p:environment:database
```
```shell
vgssoniii@vgameserver:/var/www/pterodactyl$ sudo php artisan p:environment:database

 ! [NOTE] It is highly recommended to not use "localhost" as your database host as we have seen frequent socket
 !        connection issues. If you want to use a local connection you should be using "127.0.0.1".

 Database Host [127.0.0.1]:
 > 127.0.0.1

 Database Port [3306]:
 > 3306

 Database Name [panel]:
 > panel

 ! [NOTE] Using the "root" account for MySQL connections is not only highly frowned upon, it is also not allowed by this
 !        application. Youll need to have created a MySQL user for this software.

 Database Username [pterodactyl]:
 > pterodactyl

 Database Password:
 > Passwort
```

## Mail Server einrichten
Hier werde ich einen nicht so guten Mail Server einrichten. Hat aber auch keine große rolle da man es nie brauchen wird.
```shell
sudo php artisan p:environment:mail
```
```shell
vgssoniii@vgameserver:/var/www/pterodactyl$ sudo php artisan p:environment:mail

 Which driver should be used for sending emails? [SMTP Server]:
  [smtp    ] SMTP Server
  [mail    ] PHPs Internal Mail Function
  [mailgun ] Mailgun Transactional Email
  [mandrill] Mandrill Transactional Email
  [postmark] Postmarkapp Transactional Email
 > mail

 Email address emails should originate from [no-reply@example.com]:
 > no-reply@seikreativ.de

 Name that emails should appear from [Pterodactyl Panel]:
 > Pterodactyl Panel

 Encryption method to use [TLS]:
  [tls] TLS
  [ssl] SSL
  [   ] None
 > tls

Updating stored environment configuration file.
```

## Datenbank abschließen
Nun wird die Datenbank die letzten befehle bekommen und dann ist es abgeschlossen.
```shell
sudo php artisan migrate --seed --force
```
**Je nach system kann das etwas länger dauern.**

## Den ersten User hinzufügen
Nun kümmern wir uns darum das wir einen eigenen User erstellen, damit wir im Panel dann auch ein Profil haben um Server errichten zu können.
```shell
sudo php artisan p:user:make
```
```shell
vgssoniii@vgameserver:/var/www/pterodactyl$ sudo php artisan p:user:make

 Is this user an administrator? (yes/no) [no]:
 > yes

 Email Address:
 > Deine@eMail.de

 Username:
 > Dein_Username

 First Name:
 > Dein_Vorname

 Last Name:
 > Dein_Nachname

Passwords must be at least 8 characters in length and contain at least one capital letter and number.
If you would like to create an account with a random password emailed to the user, re-run this command (CTRL+C) and pass the `--no-password` flag.

 Password:
 >

+----------+--------------------------------------+
| Field    | Value                                |
+----------+--------------------------------------+
| UUID     | 00000000-0000-0000-0000-000000000000 |
| Email    | Deine@eMail.de                       |
| Username | Dein_Username                        |
| Name     | Dein_Vor&Nachname                    |
| Admin    | Yes                                  |
+----------+--------------------------------------+
```

## Rechte verteilen
Nun der letzte befehl um deinen Account die rechte zu geben.
```shell
-> # If using NGINX or Apache (not on CentOS):
sudo chown -R www-data:www-data /var/www/pterodactyl/*

# If using NGINX on CentOS:
sudo chown -R nginx:nginx /var/www/pterodactyl/*

# If using Apache on CentOS
sudo chown -R apache:apache /var/www/pterodactyl/*
```

### Warteschlange Daemon
Damit der service schneller laufen kann da er im hintergrund mehrere sachen ausführen kann. Musst dies eingeben `sudo crontab -e` und gebe dann in der letzten Zeile dies ein `* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1`

## Die warteschlangen Arbeiter erstellen
Jetzt musst du die warteschlangen Arbeiter erstellen. Dies geht indem du ein datei erstellen unzwar `sudo nano /etc/systemd/system/pteroq.service` der folgendes beinhaltet.
```
# Pterodactyl Queue Worker File
# ----------------------------------

[Unit]
Description=Pterodactyl Queue Worker
After=redis-server.service

[Service]
# On some systems the user and group might be different.
# Some systems use `apache` or `nginx` as the user and group.
User=www-data
Group=www-data
Restart=always
ExecStart=/usr/bin/php /var/www/pterodactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
Nachdem alles eingerichtet wurde können wir nun den Redis Server und den Service starten.
```shell
sudo systemctl enable --now redis-server
sudo systemctl enable --now pteroq.service
```

# Website Erstellung
Nun wird die Website erstellet indem du deiner Game Server oder auch Voice Server erstellen und verwalten kannst.
## Nginx ohne SSL
Als erstes musst du die NGINX konfiguration entfernen.
```shell
sudo rm /etc/nginx/sites-enabled/default
```
Nun musst du die Konfiguration die unten steht kopieren und ersetzt `127.0.0.1` mit der domain die du angegeben hast (falls die IP-Adresse angegeben wurde nur IP-Adresse angeben, **OHNE** `"http://"`) und fügst sie in `sudo nano /etc/nginx/sites-available/pterodactyl.conf` ein.
```
server {
    # Replace the example <domain> with your domain name or IP address
    listen 80;
    server_name 127.0.0.1;


    root /var/www/pterodactyl/public;
    index index.html index.htm index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
## Konfiguration Aktivieren
Nun musst du die Konfiguration aktivieren und danach neustarten.
```shell
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf
sudo systemctl restart nginx
```

# Wings Installation
## Systemanforderungen
Du bist schon bald fertig. Jetzt muss noch der Daemon installiert werden doch bevor man dies kann, muss man sein System kurz checken.
```shell
systemd-detect-virt
```
**Sollte der nicht funktionieren kann man auch den Befehl benutzen.**
```shell
sudo dmidecode -s system-manufacturer
```
Wenn der provider `Virtuozzo`, `OpenVZ` (oder `OVZ`), oder `LXC` als virtualizierung nutzt, kann man höchstwarscheinlich nicht Wings nutzen. Sollte also als Output `OpenVZ` oder `LXC` kommen kann man höchstwarscheinlich keine Server aufsetzten.

Ungefähr so sollte es aussehen.
```shell
dane@pterodactyl:~$ sudo dmidecode -s system-manufacturer
VMware, Inc.
///
vgssoniii@vgameserver:/var/www/pterodactyl$ sudo dmidecode -s system-manufacturer
Microsoft Corporation
```

## Abhängigkeiten
## Docker Installieren
Um schnell und einfach Docker zu installieren benutz diesen Befehl.
```shell
curl -sSL https://get.docker.com/ | CHANNEL=stable bash
```
**Auch wichtig: Sollte die Kernel verion nicht supportet sein, so kann man kein Docker benutzen. Man kann es ganz einfach testen indem man `uname -r` in sein Terminal eingibt. Sollte als Output `-xxxx-grs-ipv6-64` oder `-xxxx-mod-std-ipv6-64` rauskommen, so untersützt Docker sie nicht. Sollte das bei dir der fall sein, klicke [hier](https://pterodactyl.io/daemon/0.6/kernel_modifications.html) um zu schauen wie du das beheben kannst.**

Sollte diese Meldung auftauchen
```shell
================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================
```

führe bitte diesen Befehl aus.
```ssh
dockerd-rootless-setuptool.sh install
```

Sollte dieser Befehl bei dir nicht funktionieren führe diese Befehle aus.
```shell
sudo sh -eux <<EOF
```
```
# Install newuidmap & newgidmap binaries
```
```
apt-get install -y uidmap
```
```
EOF
```

## Docker beim hochfahren starten
Geben diesen Befehl ein damit Docker immer beim hochfahren des Server startet.
```shell
sudo systemctl enable --now docker
```

## Activiere Swap
Auf den meisten System ist Swap warscheinlich nicht aktiviert. Um herauszufinden ob es bei dir aktiviert ist oder nicht, führe `sudo docker info` im Terminal aus und wenn einer der letzten Zeilen WARNING: No swap limit support beinhaltet so musst du es aktivieren.

Sollte aber Swap bei dir Aktiv sein kannst du diesen punkt überspringen.

Um Swap zu aktivieren musst du `sudo nano /etc/default/grub` und dann musst du nach der Zeile `GRUB_CMDLINE_LINUX_DEFAULT` suchen. Füge dann in den "" `swapaccount=1` so dass es so aussieht.
```
GRUB_CMDLINE_LINUX_DEFAULT="swapaccount=1"
```
Um sicherzustellen das SWAP erfolgreich aktiviert wurde kann man wieder den Befehl eingeben.
```shell
sudo docker info
```
## Wings Installieren
Der erste schritt um Wings zu installieren ist ein neuen ordner zu erstellen und Dateien zu installieren.
```shell
sudo mkdir -p /etc/pterodactyl
sudo curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_$([[ "$(uname -m)" == "x86_64" ]] && echo "amd64" || echo "arm64")"
sudo chmod u+x /usr/local/bin/wings
```
## Konfigurieren
Nachdem alle komponenten installieret sind und laufen kann man sich auf die Website begeben. Die IP ist die, die man als Domain angegeben hat.

### Location & Nodes einstellen
Du musst Links an der Seite `"Location"` auswählen.
Klicke dann auf `"Create New"`.
Gebe bei `"Short Code"` einen belibigen Namen ein.
![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999605351781498920/unknown.png)

Klicke dann Links an der Seite `"Nodes"` und klick auf den knopf `"Create New"`.

Gebe unter `"Name"` einen belibigen Namen ein.
Gebe unter `"FQDN"` die Domain des Server ein.

Wähle bei `"Communicate Over SSL"` -> `"Use HTTP Connection"` und bei `"Behind Proxy"` wähle `"Behind Proxy"`

Gebe Bei `"Total Memory"` beliebig Ram und bei `"Memory Over-Allocation"` gebe `"10%"`.

Gebe bei `"Total Disk Space"` beliebig viel Speicherplatz und bei `"Disk Over-Allocation"` gebe `"10%"`. 

`"Daemon Port"` und `"Daemon SFTP Port"` nicht ändern außer es sollte belegt sein.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999605369636667462/unknown.png)

Nachdem alles eingestellt wurde musst du in den Tab `"Configuration"` rein und die `"Configuration File"` kopieren und in `"sudo nano /etc/pterodactyl/config.yml"` einfügen.
![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999607078710681641/unknown.png)

## Starte Wings
Nun da das meiste fertig ist und du bald deine eigene Server hosten kannst musst du dich aber noch etwas gedulden. Du musst nun Wings starten.
```shell
sudo wings --debug
```
Nachdem Wings gestartet wurde musst du wieder auf die Pterodactyl Seite gehen und klicke dann auf `"Nodes"` sollte das Herz nicht Grün sein wie untem im Bild gezeigt, musst du nach deinem Problem googlen.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999605320781398056/unknown.png)

Danach mit `"STRG + C"` den Service stoppen.
## Daemonizing
Nachdem Wings ohne probleme starten konnte und auch **KEINE** fehlermeldungen kam.
Kannst du nun weiter machen mit dem nächsten schritt.

Füge den Inhalt der unten steht in die folgende datei hinzu `"sudo nano /etc/systemd/system/wings.service"`
```
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service
Requires=docker.service
PartOf=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
LimitNOFILE=4096
PIDFile=/var/run/wings/daemon.pid
ExecStart=/usr/local/bin/wings
Restart=on-failure
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
Nachdem du den Inhalt in die Datei hinzugefügt hast, musst du noch einen Befehl eingeben.
```shell
sudo systemctl enable --now wings
```
Dieser Befehl ist dafür da damit Wings immer bei jedem Server hochfahren auch mit startet.

# Weitere Eier hinzufügen
Um Pterodactyl im vollen umpfang nutzen zu können musst du sogenannte `"Egg"` hinzufügen um mit deinen Freunden andere Spiele oder auch Modpacks zu spielen.

Um weitere Eggs hinzuzufügen muss man diese Repo klonen: https://github.com/parkervcp/eggs

Ich habe es mit WSL gemacht, geht aber auch unter Windows dafür einfach ein Tutorial anschauen wie man Repos unter Windows klont.

Wie klont man eine Repo in WSL:
Zuerst braucht man Git (meistens ist es schon vorinstalliert).
Dann gibt man diesen folgenden Befehl ein `git clone https://github.com/parkervcp/eggs`

Danach hat man diese Repo runtergeladen doch leider ist sie in WSL.
Nun kann man es aber über den Explorer auf seinen PC rüberziehen.
Dafür links an der seite Linux ausgewählt und auf Ubuntu (oder anderes OS) geklickt, dann auf home -> Deiner User name und dann gibt es einen Ordner names `"Eggs"`. Denn kann man kopieren und auf seinen PC ziehen.

Nun da du die Eggs hast kann du ganz einfach über den Panel die Eier hinzufügen.

Klicke hierfür auf `"Nests"`

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999618406653370368/unknown.png)

Dann auf `"Create New"`

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999618423623524372/unknown.png)

Gebe dann den Namen des Spieles oder Service ein den du gerne hinzufügen möchstest und klicke `"Save"`

Gehe dann zurück auf `"Nests"`

Klicke dann auf `"Import Egg"`

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999618436760096838/unknown.png)

Wähle dann im Eggs Ordner den du gerade runtergeladen hast das Spiel aus und die Modi

In meinem fall Terraria und dann Vanilla

In diesem Ordner befindet sich eine .json Datei. Wähle diese Datei aus.

Nachdem du sie ausgewählt klicke aus `"Associated Nest"` und wähle das `"Nest"` aus für das du dich entschieden hast und klicke auf `"Import"`.

Es sollte sich ein neues Fenster geöffnet haben, du kannst runterscrollen und auf save klicken.

Wenn du dann wieder auf Nests klickst sollte nun hinter deinem Spiel ein neuer `"Egg"`erscheinen.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/999618453109477417/unknown.png)

Nun kannst du dies beliebg oft mit anderen Spielen widerholen.

# Server einrichten
Solltest du einen Server einrichten wollen musst du [diese](https://soniiit.github.io/posts/Pterodactyl/#weitere-eier-hinzuf%C3%BCgen) schritte erst machen. Solltest du aber mit den Standart Server weiter machen kannst dies auch machen.

In diesem beispiel nehme ich ein Minecraft Modpack. Solltest du also kein Minecraft Modpack Server erstellen wollen kannst du den schritt `"Curseforge API Key"` und `"Nest einrichten"` überspringen.

## Curseforge API Key
Du musst auf diesen Link hier klicken um dir erstmal einen Account anzulegen, solltest du aber schon einen Account haben kannst du dich einfach anmelden.

[CurseForge](https://console.curseforge.com/#/login)

Nachdem du dich eingelogged hast wirst du nach einer Organizations Namen gefragt. Gibt da einen Namen eine den du möchtest denn so wird der Name des API keys.

Dann klickst du link an der Seite auf API keys.

Du siehst du hast jetzt einen Key den du später brauchst. Also speicher ihn dir irgendwo zwischen.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000105195176464384/unknown.png)

## Nest einrichten
Nachdem du den API Key hast musst du zurück zu deinem Panel um dort den Key einzugeben.

Wenn du wieder auf deinem Panel bist klicks du an der Seite auf `"Nests"` und dann suchst du den Nest wo auch dein Ei mit Cursoforge ist und klickst rein.

Jetzt bist du in deim Nest. Klicke jetzt auf `"Curseforge Generic"` um das Ei einszustellen.

Klicke dann auf `"Variables"` und scroll ganz runter.

Nun findest du ein kleines Fenster das heoßt `"API Key"`, gibt da unter `"Default Value"` dein API Key ein.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000111675464224819/unknown.png)

## Ports einrichten
Nun musst du dich darum kümmern die Ports einzurichten.

Dafür musst du zuerst unter `"Nodes" -> Allocation"` gehen und dann gibst du unter `"IP Adress"` die IP-Adresse des Servers ein und unter `"Ports"` gibst du die Ports `"27000-27099"` ein. Somit haben hast du 100 Ports die du frei belegen kannst.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000111642442485881/unknown.png)

## Server einrichtung
Jetzt da du die Ports hast kannst du dich um die erstellung eines Server kümmern.

Dafür links an der Seite `"Servers"` ausgewählt und dann auf `"Create New"` geklickt. Es sollte sich ein neues Fenster öffnen.

Jetzt kannst du unter `"Server Name"` dem Server einen Namen geben in meinem beispiel: Test Beispliel

Dann musst du unter Server Owner dein Account eingeben.
**(Solltest es dein Account nicht finden gib die E-Mail ein)**

Unter `"Default Allocation"` wählst du den Port aus.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000117255922778212/unknown.png)

Unter dem Fenster `"Resource Management"` kannst du alles einstellen wenn um CPU, Ram und Speicherplatz geht.

Kurze erklärung: CPU wird von 100% - z.b 400% angegeben. 1 CPU -> 100%; 4 CPUs -> 400%. Ram wird in MB angben. 1GB -> 1024MB; 4GB -> 4096MB. Dasselbe gilt auch für Disk Space. Da gibst du den Speicherplatz für den Server in MB an. (Das gleiche beispiel wie bei RAM.)

Solltest du aber die freilassen wird der Server alle Rescourcen nehmen die der Server hat.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000117271269736469/unknown.png)

Nun geht es weiter mit dem nächsten Fenster. Das heißt Nest `"Configuration"`.

**Das hier gesagt bezieht sich nur auf Minecraft und kann auf andere Server anders sein**

Unter `"Nest"` wählst du das Nest aus welches genommen werden soll. In meinem beispiel `"Minecraft Extra"`

Dann unter `"Egg"` wählst du das Ei aus in meinem beispiel ist es `"Curseforge Genetic"` wird bei dir aber auch so heißen.

Rechts vom Fenster hast du ein Fenster das nennt sich `"Docker Configuration"`
Da hast du auch ein dropdown Menü, da wählst du die neueste Java version die für dich verfügbar ist.

**Achtung nun wird etwas für das Minecraft Modpack ausgwählt, solltest du einen anderen Server aufsetzten wollen kannst du denn Punkt überspringen und auf `"Create Server"` klicken**

Nun hast du ein weiteres Fenster das nennt sich `"Service Variables"` da musst du unter `"Modpack Project ID"` die ID des Modpack eingeben. Die findest du wie unten im Bild gezeigt.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000117741719666808/unknown.png)

Nachdem das gemacht wurde kannst du auf `"Create Server"` klicken und starten.

Sollte aber der Server nicht funktionieren, weil in die Konsole irgendwas mit `"not found server.jar"` oder etwas ähnliches sagt. Musst du nur den Server selbstständig runterladen und es auf dem Server hochladen.

Dafür den Server Modpack runterladen wie untern im Bild gezeigt.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000118887045660753/unknown.png)

Dann extrahiere die .zip datei und lade dir das Programm [WinSCP](https://winscp.net/eng/download.php) runter (natürlich gehen auch andere SFTP clients).

Gehe dann wieder zurück auf deinem Panel und wähle `"Servers"` aus.

Klicke dann auf das Symbol rechts an der Seite neben `"Delete"`.

Es sollte dann ein neues Fenster erscheinen indem auch die Konsole ist.

Klicke dann auf `"Settings"` und kicke dann auf `"Launch SFTP"` und bestätige.

![img-description](https://cdn.discordapp.com/attachments/156903785103491073/1000121466651693187/unknown.png)

Es sollte sich dann das Programm öffnen und du wirst aufgefordert einen Passwort einzugeben. Der Passwort ist der normale Passwort den du auch verwendest um dich einzuloggen. Nun solltest du dich schon im Ordner des Servers befinden. Navigiere dich auf der linken hälfte durch deinen PC bis du im die Server Dateien gefunden hast die du gerade runtergeladen hast du zieh sie dann auf den Server (Rechte hälfte).

Versuche nun den Server wieder Starten. Sollte es immernoch nicht funktionieren musst du nach deinem Problem suchen.

# Herzlichen Grückwunsch du hast soeben Pterodactyl installiert
Ja, du hast dich nicht verlesen. Es es nun komplett installiert und du kannst nun auch Server aufsetzten mit ein paar klicks. Solltest du Probleme haben mit dem Interface schlage ich dir vor schau auf YouTube nach einem Guide.

# Quelle
<https://youtu.be/_ypAmCcIlBE>

<https://youtu.be/2mibJeaEq3Q>

<https://pterodactyl.io/panel/1.0/getting_started.html>