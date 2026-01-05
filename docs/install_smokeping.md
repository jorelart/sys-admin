---
layout: default
title: Install Smokeping Ubuntu
parent: Linux
nav_order: 1
---



# Install dan Konfigurasi Smokeping 2.9 Ubuntu
Smokeping adalah tool monitoring latency yang banyak digunakan perusahaan jaringan untuk memantau latency jaringan secara realtime.

## 1. Instalasi Smokeping



1. Update & Upgrade system

    ```
    sudo apt update && sudo apt upgrade -y
    ```

2. Install dependencies

   ```
   sudo apt install --no-install-recommends \
        dnsutils curl \
        rrdtool fping \
        dnsutils libwww-perl \
        librrds-perl wget
   ```

3. Cek versi smokeping yang ingin kalian install di `https://oss.oetiker.ch/smokeping/pub/`. Setelah dapat versi yang sesuai lanjutkan download file smokeping.
   ```
   wget https://oss.oetiker.ch/smokeping/pub/smokeping-2.8.2.tar.gz
   ```
4. Ekstrak dan build smokeping
   ```
   sudo tar -xvzf smokeping-2.8.2.tar.gz
    cd smokeping-2.8.2.tar.gz
    LC_ALL=C ./configure --prefix=/opt/smokeping
    sudo make install
   ```
   Proses ini akan memakan beberapa menit, silahkan ditunggu.

5. Verifikasi status instalasi
   ```
   /opt/smokeping/bin/smokeping --version
   ```
6. Konfigurasi Smokeping
   Buka file konfigurasi smokeping, di `/opt/smokeping/etc/config.dist`

   ```
   *** General ***

    owner    = YOUR NAME
    contact  = YOUR_USER@localhost
    mailhost = localhost
    sendmail = /usr/sbin/sendmail
    # NOTE: do not put the Image Cache below cgi-bin
    # since all files under cgi-bin will be executed ... this is not
    # good for images.
    imgcache = /usr/local/smokeping/cache
    imgurl   = cache
    datadir  = /usr/local/smokeping/data
    piddir  = /usr/local/smokeping/var
    # Bellow allows the web gui to be accessed
    cgiurl   = http://0.0.0.0/smokeping.cgi
    smokemail = /usr/local/smokeping/etc/smokemail.dist
    tmail = /usr/local/smokeping/etc/tmail.dist
    # specify this to get syslog logging
    syslogfacility = local0
    # each probe is now run in its own process
    # disable this to revert to the old behaviour
    # concurrentprobes = no
    
    
    *** Alerts ***
    to   = root@localhost
    from = smokealert@localhost
    
    +someloss
    type = loss
    # in percent
    pattern = >0%,*12*,>0%,*12*,>0%
    comment = loss 3 times  in a row
    
    *** Database ***
    
    step     = 300
    pings    = 20
    
    # consfn mrhb steps total
    
    AVERAGE  0.5   1  28800
    AVERAGE  0.5  12   9600
        MIN  0.5  12   9600
        MAX  0.5  12   9600
    AVERAGE  0.5 144   2400
        MAX  0.5 144   2400
        MIN  0.5 144   2400
    
    *** Presentation ***
    
    template = /usr/local/smokeping/etc/basepage.html.dist
    htmltitle = yes
    graphborders = no
    
    + charts
    
    menu = Charts
    title = The most interesting destinations
    
    ++ stddev
    sorter = StdDev(entries=>4)
    title = Top Standard Deviation
    menu = Std Deviation
    format = Standard Deviation %f
    
    ++ max
    sorter = Max(entries=>5)
    title = Top Max Roundtrip Time
    menu = by Max
    format = Max Roundtrip Time %f seconds
    
    ++ loss
    sorter = Loss(entries=>5)
    title = Top Packet Loss
    menu = Loss
    format = Packets Lost %f
    
    ++ median
    sorter = Median(entries=>5)
    title = Top Median Roundtrip Time
    menu = by Median
    format = Median RTT %f seconds
    
    + overview
    
    width = 600
    height = 50
    range = 10h
    
    + detail
    
    width = 600
    height = 200
    unison_tolerance = 2
    
    "Last 3 Hours"    3h
    "Last 30 Hours"   30h
    "Last 10 Days"    10d
    "Last 360 Days"   360d
    
    #+ hierarchies
    #++ owner
    #title = Host Owner
    #++ location
    #title = Location
    
    *** Probes ***
    
    + FPing
    binary = /usr/bin/fping
    
    *** Targets ***
    menu  = Top
    probe = FPing
    title = Network Latency Grapher
    remark = Network Latency Grapher
    
    + SearchEngine
    menu = Search-Engine
    title = Search Engine
    
    ++ Google
    menu = Google
    title = google.com
    host = google.com
    
    ++ Bing
    menu = Bing
    title = Bing.com
    host = bing.com
    
    ++ DuckDuckGo
    menu = DuckDuckGo
    title = DuckDuckGo.com
    host = duckduckgo.com
    
    ++ Yahoo
    menu = Yahoo
    title = Yahoo.com
    host = yahoo.com
    ```

7. Cek apakah konfigurasi ada yang error
   ```
   /opt/smokeping/bin/smokeping --check
   ```

8. Konfigurasi Systemd Service
   Edit file `/lib/systemd/system/smokeping.service`
   ```
   [Unit]
    Description=Smokeping Server: Latency Logging and Graphing System
    Documentation=man:smokeping(1) file:/usr/share/doc/smokeping/examples/systemd/slave_mode.conf
    After=network.target
    
    [Service]
    # It would in theory be simpler to run smokeping with the --nodaemon option and
    # Type=simple, but smokeping does not work properly when in "slave" mode with
    # --nodaemon set.
    Type=Simple
    
    # If you need to run smokeping in slave/master mode, see the example unit
    # override in /usr/share/doc/smokeping/examples/systemd/slave_mode.conf
    ExecStart=/opt/smokeping/bin/smokeping --nodaemon /opt/smokeping/etc/config.dist --logfile=/var/log/smokeping.log
    
    
    [Install]
    WantedBy=multi-user.target
   ```
9. Jalankan service
    Enable :
   ```
   sudo systemctl enable smokeping.service
   ```
   Start:
   ```
   sudo systemctl start smokeping.service
   ```
   Cek status:
   ```
   sudo systemctl status smokeping.service
   ```
   Hentikan
   ```
   sudo systemctl stop smokeping.service
   ```

## 2. Konfigurasi Web Gui 

1. Install apache jika belum ada
   ```
   sudo apt install libapache2-mod-fcgid apache2 -y
   ```
2. Ubah permission direktori smokeping supaya apache bisa membukanya
   ```
   sudo chown www-data:www-data -R /opt/smokeping/
   ```
3. Konfigurasi apache2
   Edit file `/etc/apache2/conf-available/smokeping.conf`

   ```
   Alias /smokeping/cache /opt/smokeping/cache
    Alias /smokeping /opt/smokeping/htdocs/
    
    <Directory "/opt/smokeping/cache">
      AllowOverride all
      Require all granted
    </Directory>
    
    <Directory "/opt/smokeping/htdocs/">
     Options FollowSymLinks ExecCGI
     AllowOverride all
     Require all granted
    </Directory>
   ```

4. Aktifkan Konfigurasi apache
   ```
   sudo aeenconf smokeping
   ```
5. Aktifkan CGI Mod dan restart Apache
   ```
   sudo a2enmod cgi
    sudo systemctl restart apache2.service
   ```

6. Cek di browser
   Buka browser dengan alamat http://<ip-server>/smokeping/smokeping.fcgi.dist
   

   
