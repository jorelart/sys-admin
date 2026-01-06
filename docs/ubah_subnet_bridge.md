---
layout: default
title: Mengganti Default Network Docker
parent: Linux
nav_order: 4
---




# Cara mengubah segmen/subnet network system bridge Docker

Didalam jaringan biasanya kita membagi segmen jaringan lokal untuk kebutuhan tertentu, termasuk untuk docker. Docker sendiri secara default menggunakan ip RFC-1918 biasanya `172.16.0.0/16` yang berkemungkinan besar juga akan berpotensi konflik dengan jaringan lokal kita.
Jika sudah demikian tentu supaya tidak konflik ip kita harus melakukan perubahan subnet pada salah satu jaringan. 
Kali ini kita akan melakukan perubahan default network pada docker

1. Buat file baru di `/etc/docker/daemon.json`

   ```
   {
      "bip": "10.67.46.1/24",
      "default-address-pools": [
        {
          "base": "10.67.50.0/16",
          "size": 24
        }
      ]
    }
   ```
   

  - bip → subnet docker0
  - Gateway otomatis .1
  - Jangan bentrok dengan LAN/VPN

2. Stop service docker

   ```
   sudo systemctl stop docker
   ```
3. Bersihkan cache network Docker
   ```
   sudo rm -rf /var/lib/docker/network
   ```

4. Verifikasi

   ```
   ip addr show docker0
   docker network inspect bridge
   ```
   
