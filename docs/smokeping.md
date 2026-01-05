---
title: Smokeping
parent: Linux
nav_order: 1
---



# Install dan Konfigurasi Smokeping Master 2.9 dan Slave (Docker) di ubuntu

Saat menggunakan mode Master dan Slave, pengaturan hostname sangat penting, karena komunikasi antara master dan slave menggunakan nama hostname.
Hostname bisa diubah dengan
```
hostnamectl --static set-hostname PING-MASTER
```
`PING-MASTER` adalah nama hostname yang di kehendaki

Sebagai contoh kita ingin membuat 1 server master dan 1 server slave.
- Master
  hostname    : smokeping-master
  ip address  : 10.1.1.1
  lokasi      : Jakarta

- Slave
  hostname    : smokeping-pati
  ip address  : 10.222.1.123
  lokasi      : Pati
