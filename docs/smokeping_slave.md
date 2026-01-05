---
layout: default
title: Smokeping Slave Install
parent: Linux
nav_order: 3
---


# Instalasi Smokepig sebagai Slave (remote)
Selain bisa digunakan sebagai standalone, smokeping juga bisa menggunakan client-server atau lebih tepatnya slave.
Dengan slave atau probe server lain akan memberikan gambaran ping secara realtime dari server yang berbeda.

1. Hostname sangat berpengaruh
   Saat konfigurasi slave, nama hostname akan digunakan untuk komunukasi kepada master, lebih tepatnya saat mencocokan kredensial atau secret. Hostname bisa dirubah dengan cara berikut.
   ```
   hostnamectl --staatic set-hostname monitor-slave
   ```
   sesuaikan `monitor-slave` dengan kebutuhan anda.

2. Install Smokeping 2.9 Baik di master maupun di slave seperti tutorial [Instalasi Smokeping ubuntu 2.9]()
   Setelah instalasi kita asumsikan sebagai berikut :
     | Mode | Master  | Slave |
    | ----------- | ------------- | ------------- |
    | Ip | 10.123.12.4  | 10.123.12.5  |
    | Hostname | monitor-master  | monitor-slave  |
    | Lokasi | Jakarta  | Pati  |

Jika kedua server sudah terinstall, lanjutkan langkah berikutnya

## A. Konfigurasi Master
1. Edit konfigurasi pada master `/opt/smokeping/etc/config` pada bagian `*** Slaves ***` seperti berikut.
   ```
    *** Slaves ***
    secrets=/usr/local/smokeping/slavesecrets.conf
     
    +PING-SLAVE
    display_name=PING-SLAVE
    location=indonesia
    color=00ff00
     
     
    *** Targets ***
    slaves = PING-SLAVE
    ```
2. Buat file baru untuk menyimpan secret, dengan format `hostname:password` seperti berikut.
   ```
   echo "monitor-slave:supersecretpassword" > /opt/smokeping/etc/slavesecrets.conf
   ```
3. Ubah file permission `slavesecrets.conf` agar bisa dibaca oleh user yang manjalankan smokeping
   ```
   chown root:www-data /opt/smokeping/slavesecrets.conf
   chmod 640 /opt/smokeping/slavesecrets.conf
    ```
4. Jalankan smokeping
   ```
   /opt/smokeping/bin/smokeping --config=/opt/smokeping/etc/config
   ```

## B. Konfigurasi Slave
Khusus untuk mode master-slave ini, di slave tidak perlu di ubah konfigurasi apapun kecuali password dan juga format untuk menjalankan smokeping.

1. Buat file untuk menyimpan kredensial atau secret.
   ```
   echo "supersecretpassword" > /opt/smokeping/slavesecrets.conf
   ```
2. Ubah file permission
   ```
   chown root:www-data /usr/local/smokeping/slavesecrets.conf
   chmod 600 /usr/local/smokeping/slavesecrets.conf
    ```

3. Jalankan smokeping sebagai slave
   ```
   /usr/local/smokeping/bin/smokeping --master-url=http://10.123.12.4/smokeping/smokeping.fcgi.dist  --shared-secret=/usr/local/smokeping/slavesecrets.conf --cache-dir=/usr/local/smokeping/cache/
   ```

## Troubleshooting
Beberapa kesalahan yang sering dilakukan sehingga mode slave tidak bisa berjalan.
1. Url master salah atau tidak bisa dijangkau
   pastikan url master adalah yang menggunakan `.cgi`. Bukan url web ui master
   Pastikan url bisa dijangkau, tidak menggunkan htpasswd, cek dari slave dengan `curl -I http://url-master` pastikan status 200OK

   
