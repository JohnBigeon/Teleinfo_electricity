# Teleinfo_electricity

## Concept
Display (remote) electricity consumption of the house with a Raspberry Pi connected to a 'french compteur'.

## Raspberry Pi

### Installation
Format the sd card
#### With graphical interface
With rpi-imager
```
sudo apt install snapd
sudo snap install rpi-imager
rpi-imager
```

#### Terminal
With terminal
````
wget https://downloads.raspberrypi.org/raspios_armhf/images/raspios_armhf-2022-01-28/2022-01-28-raspios-bullseye-armhf.zip
tar -xzvf raspios_armhf-2022-01-28/2022-01-28-raspios-bullseye-armhf.zip.tar.gz -C /Download
````
unmount SD card 
Now you have to unmount the device:
```
sudo umount /dev/mmcblk0
```
Check 
```
sudo fdisk -l
```
You might see:
```
/dev/mmcblk0
```
The name of your card should appear in the latest results. It will probably start with the letters mm.

To install Raspbian on the card, we will use the command dd, which perform a copy of a file at the binary level:
```
sudo dd bs=1M if=2022-01-28-raspios-bullseye-armhf.img of=/dev/mmcblk0 status=progress conv=fsync
```

### Enable remote access
#### SSH
Navigate on your SD card to the /boot/directory and add a empty file with:
````
touch ssh
````

#### Connect the Raspberry pi to the Wifi

In boot directory, create a new file called wpa_supplicant.conf, which will hold the necessary credentials required to connect to the WIFI network. Open a new file (wpa_supplicant.conf) with your text editor and paste the contents below.

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=FR
     
network={
    ssid="#Box_name"
    psk="#Password"
    scan_ssid=1
}
````

#### Connect your laptop to the Raspberry pi via ethernet cable
````
ping raspberrypi.local
````
````
ssh pi@raspberrypi.local
````

#### Connect your laptop to the Raspberry pi via router
On your computer:
````
ifconfig
````
````
wlo1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.a.xx  netmask 255.255.255.0  broadcast 192.168.1.255
````
Scan the network
````
sudo nmap 192.168.a.*
````
````
Nmap scan report for raspberrypi.home (192.168.a.bb)
Host is up (0.017s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
````
If the raspberry is connected to the wifi, you should see him with an ip's address: **192.168.a.bb**.
Make a ssh to be connected with the raspberry
````
ssh pi@192.168.a.bb
````
Check the distribution you have
````
uname -a
Linux raspberrypi 5.10.103-v7l+ #1530 SMP Tue Mar 8 13:05:01 GMT 2022 armv7l GNU/Linux
````

#### Enable remote access to GPIO pins
Execute the command:
````
sudo raspi-config
````
Go to "Interfacing Options/Remote GPIO"
Choose "Yes"

Be sure, the remote GPIO feature is already installed
````
sudo apt install pigpio
````

To automate running the daemon at boot time, run:
````
sudo systemctl enable pigpiod
````

To run the daemon once using systemctl, run:
````
sudo systemctl start pigpiod
````
#### Activate serial port
````
sudo raspi-config
````
#### Activate uart
In the folder /boot/cmdline.txt, add at the end: 
````
enable_uart=1
````
delete
````
console=serial0,115200
````
Then install GPIO Zero and the pigpio library for Python 3:
````
$ sudo apt install python3-gpiozero python3-pigpio
````

### Test first script with LED blinking
````
mkdir Scripts
cd Scripts
nano blink_LED_v01.py
````
````
from gpiozero import LED
from time import sleep

red = LED(18)

while True:
    red.on()
    sleep(1)
    red.off()
    sleep(1)
````

Execute script
````
python3 blink_LED_v01.py
````
#### Fritzing
resistance = 220 Ohms

## Teleinfo
### Libraries
````
sudo apt update
sudo apt upgrade
sudo apt install git-core libmysqlclient-dev libcurl4-openssl-dev
````
````
git clone https://github.com/hallard/teleinfo/
cd teleinfo
make
sudo make install
````
#### Errors:
````
fatal error: mysql/mysql.h: No such file or directory
````
````
sudo apt install libmariadb-dev-compat libmariadb-dev mariadb-server
````
````
fatal error: curl/curl.h: No such file or directory
````
````
sudo apt install libcurl4-openssl-dev
````

### First try
````
sudo cp teleinfo.conf /etc/
````
````
teleinfo --help
````
````
teleinfo -m test
````
### Test serial port
````
picocom -b 1200 -d 7 -p e -f n /dev/ttyS0
picocom v3.1

port is        : /dev/ttyS0
flowcontrol    : none
baudrate is    : 1200
parity is      : even
databits are   : 7
stopbits are   : 1
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
hangup is      : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv -E
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,
logfile is     : none
initstring     : none
exit_after is  : not set
exit is        : no

````
 
### Data
Source: https://www.magdiblog.fr/gpio/teleinfo-edf-suivi-conso-de-votre-compteur-electrique/
````
Données de la téléinfo EDF

Comme indiqué dans le document « Sorties de télé-information client des appareils de comptage électroniques utilisés par ERDF« , publié par EDF et disponible ici : ERDF-NOI-CPT_02E.pdf, et selon votre type d’abonnement EDF, vous pouvez récupérer les informations suivantes :

    ADCO : Identifiant du compteur
    OPTARIF : Option tarifaire (type d’abonnement)
    ISOUSC : Intensité souscrite
    BASE : Index si option = base (en Wh)
    HCHC : Index heures creuses si option = heures creuses (en Wh)
    HCHP : Index heures pleines si option = heures creuses (en Wh)
    EJP HN : Index heures normales si option = EJP (en Wh)
    EJP HPM : Index heures de pointe mobile si option = EJP (en Wh)
    BBR HC JB : Index heures creuses jours bleus si option = tempo (en Wh)
    BBR HP JB : Index heures pleines jours bleus si option = tempo (en Wh)
    BBR HC JW : Index heures creuses jours blancs si option = tempo (en Wh)
    BBR HC JW : Index heures pleines jours blancs si option = tempo (en Wh)
    BBR HC JR : Index heures creuses jours rouges si option = tempo  (en Wh)
    BBR HP JR : Index heures pleines jours rouges si option = tempo (en Wh)
    PEJP : Préavis EJP si option = EJP 30mn avant période EJP
    PTEC : Période tarifaire en cours
    DEMAIN : Couleur du lendemain si option = tempo
    IINST : Intensité instantanée (en ampères)
    ADPS : Avertissement de dépassement de puissance souscrite (en ampères)
    IMAX : Intensité maximale (en ampères)
    PAPP : Puissance apparente (en Volt.ampères)
    HHPHC : Groupe horaire si option = heures creuses ou tempo
    MOTDETAT : Mot d’état (autocontrôle)

Une trame commence toujours par l’étiquette ADCO et se termine par le MOTDETAT.

Chaque message, ou ligne, d’une trame est formé de la manière suivante :

    ETIQUETTE espace VALEUR espace CHECKSUM

Seules l’ETIQUETTE et la VALEUR nous seront utiles. La CHEKSUM, ou somme de contrôle sert uniquement à vérifier l’intégrité que la trame.
````
### PHP installation
````
sudo apt install apache2 php7.4 php7.4-cli chromium openbox xinit
````
````
sudo apt install imagemagick php7.4-imagick php7.4-gd xplanet unclutter mingetty x11-xserver-utils
sudo apt install sqlite3
````
##### Check Apache interface
On your computer, test if apache is installed, type in firefox the ip's address of your raspberry
````
http://192.168.a.bb
````

create files in /var/www/html/ teleinfo_puissance.php, ...
````
sudo nano teleinfo_puissance.php 
````
See cron job
````
crontab -l
````
add to cron
````
crontabl -e
````
````
* * * * * /usr/bin/php /var/www/html/teleinfo_puissance.php
* * * * * /usr/bin/php /var/www/html/teleinfo_conso.php
````
check cron excecution
````
grep CRON /var/log/syslog
````
````
sudo apt-get install postfix
````
Check connection
````
picocom -b 1200 -d 7 -p e -f n /dev/ttyS0
````
````
sudo cat /dev/ttyS0
````
## Locate php for cron
whereis php7
Catch error 
tail -2 /var/log/apache2/error.log 

shutdown_button.py · Execute the command: 
sudo shutdown -h now

sudo apt install mariadb-server-10.0

install with curl domoticz/
http://192.168.a.bb:8080/#/Utility







send 1 frame on database to check connection
````
teleinfo -m r -q
````
````
teleinfo 1.0.8 Statistics
==========================
Frames Sent         : 0
Frames checked      : 1
Frames OK           : 1
Checksum errors     : 0
Frame format Errors : 0
Frame size Errors   : 0
MySQL init OK       : 1
MySQL init errors   : 0
MySQL connect OK    : 1
MySQL connect errors: 0
MySQL queries OK    : 1
MySQL queries errors: 0
EmonCMS total post  : 0
EmonCMS post OK     : 0
EmonCMS post errors : 0
EmonCMS timeout     : 0
--------------------------
````

## Create database
````
mysql -u root -p
````
````
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 229
Server version: 10.5.12-MariaDB-0+deb11u1 Raspbian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE data_test;
Query OK, 1 row affected (0.002 sec)

MariaDB [(none)]> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| bdd_teleinfo       |
| data_test          |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
5 rows in set (0.001 sec)

MariaDB [(none)]> USE data_test;
Database changed
MariaDB [data_test]> CREATE TABLE Dbb;
ERROR 1113 (42000): A table must have at least 1 column
MariaDB [data_test]> CREATE TABLE Dbb(
    ->   id int(11) NOT NULL AUTO_INCREMENT,
    ->   DATE datetime DEFAULT NULL,
    ->   ADCO varchar(12) DEFAULT NULL,
    ->   OPTARIF varchar(4) DEFAULT NULL,
    ->   ISOUSC decimal(2,0) DEFAULT NULL,
    ->   BASE decimal(9,0) DEFAULT NULL,
    ->   HCHC decimal(9,0) DEFAULT NULL,
    ->   HCHP decimal(9,0) DEFAULT NULL,
    ->   BBRHCJB decimal(9,0) DEFAULT NULL,
    ->   BBRHPJB decimal(9,0) DEFAULT NULL,
    ->   BBRHCJW decimal(9,0) DEFAULT NULL,
    ->   BBRHPJW decimal(9,0) DEFAULT NULL,
    ->   BBRHCJR decimal(9,0) DEFAULT NULL,
    ->   BBRHPJR decimal(9,0) DEFAULT NULL,
    ->   DEMAIN varchar(4) DEFAULT NULL,
    ->   EJPHN decimal(9,0) DEFAULT NULL,
    ->   EJPHPM decimal(9,0) DEFAULT NULL,
    ->   PEJP varchar(2) DEFAULT NULL,
    ->   PTEC varchar(4) DEFAULT NULL,
    ->   IINST1 decimal(3,0) DEFAULT NULL,
    ->   IINST2 decimal(3,0) DEFAULT NULL,
    ->   IINST3 decimal(3,0) DEFAULT NULL,
    ->   IMAX1 decimal(3,0) DEFAULT NULL,
    ->   IMAX2 decimal(3,0) DEFAULT NULL,
    ->   IMAX3 decimal(3,0) DEFAULT NULL,
    ->   HHPHC varchar(1) DEFAULT NULL,
    ->   PMAX decimal(5,0) DEFAULT NULL,
    ->   PAPP decimal(5,0) DEFAULT NULL,
    ->   ADPS decimal(3,0) DEFAULT NULL,
    ->   MOTDETAT varchar(6) DEFAULT NULL,
    ->   TENSION decimal(3,0) DEFAULT NULL,
    ->   PRIMARY KEY (id),
    ->   KEY SEARCH_INDEX (ADCO,DATE)
    -> ) ENGINE=MyISAM DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.015 sec)

MariaDB [data_test]> exit
Bye
````
## Must have forum
https://community.ch2i.eu/topic/44/configuration-mysql/6

Pour tuer le daemon il suffit de faire la commande 
````
killall teleinfo
````
````
nano teleinfo.conf
````
````
mysql = 0
server = localhost
user = root
password = bb
database = bdd_teleinfo
table = DbiTeleinfo
mysql_port = 3306
````

### Check the database
````
crontab -l
````
````
*/ * * * * sudo teleinfo -m r -q
````
````
crontab -e
````
````
cp -r Teleinfo/  /var/www/html/
sudo chown -R pi:www-data /var/www/html/
sudo chmod -R 770 /var/www/html/
cp -r Teleinfo/  /var/www/html/
````
http://192.168.a.bb/Teleinfo/teleinfo.php


````
perl: warning: Please check that your locale settings:
        LANGUAGE = (unset),
        LC_ALL = (unset),
        LC_TIME = "fr_FR.UTF-8",
        LC_MONETARY = "fr_FR.UTF-8",
        LC_ADDRESS = "fr_FR.UTF-8",
        LC_TELEPHONE = "fr_FR.UTF-8",
        LC_NAME = "fr_FR.UTF-8",
        LC_MEASUREMENT = "fr_FR.UTF-8",
        LC_IDENTIFICATION = "fr_FR.UTF-8",
        LC_NUMERIC = "fr_FR.UTF-8",
        LC_PAPER = "fr_FR.UTF-8",
        LANG = "en_GB.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_GB.UTF-8").
````
````
sudo locale-gen
````




Install Grafana
Source: https://grafana.com/tutorials/install-grafana-on-raspberry-pi/
Add the APT key used to authenticate packages:
````
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
````
Add the Grafana APT repository:
````
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
````
````
sudo apt-get update
sudo apt-get install -y grafana
````
To make sure Grafana starts up even if the Raspberry Pi is restarted, we need to enable and start the Grafana Systemctl service.
````
sudo /bin/systemctl enable grafana-server
````
Start the Grafana server:
````
sudo /bin/systemctl start grafana-server
````



Log on graphana @ your computer
http://192.168.a.bb:3000

add the source of information
Data sources. The data sources page opens showing a list of previously configured data sources for the Grafana instance.

Click Add data source 'InfluxDB'

````
FileNotFoundError: [Errno 2] No such file or directory: ‘/var/log/teleinfo/releve.log’
````
````
mkdir /var/log/teleinfo/
````
 
 
