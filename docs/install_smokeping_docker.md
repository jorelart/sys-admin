---
layout: default
title: Install Smokeping Docker
parent: Linux
nav_order: 4
---

# Install smokeping di Docker
1. Menggunakan CLI
   ```
   docker run -d \
    --name=smokeping \
    --hostname=smokeping `#optional` \
    -e PUID=1000 \
    -e PGID=1000 \
    -e TZ=Etc/UTC \
    -e MASTER_URL=http://<master-host-ip>:80/smokeping/ `#optional` \
    -e SHARED_SECRET=password `#optional` \
    -e CACHE_DIR=/tmp `#optional` \
    -p 80:80 \
    -v /path/to/smokeping/config:/config \
    -v /path/to/smokeping/data:/data \
    --restart unless-stopped \
    lscr.io/linuxserver/smokeping:latest
    ```

2. Menggunakan docker compose (rekomended)
   ```
    services:
      smokeping:
        image: lscr.io/linuxserver/smokeping:latest
        container_name: smokeping
        hostname: smokeping #optional
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=Asia/Jakarta
          - MASTER_URL=http://<master-host-ip>:80/smokeping/ #optional
          - SHARED_SECRET=password #optional
          - CACHE_DIR=/tmp #optional
        volumes:
          - /path/to/smokeping/config:/config
          - /path/to/smokeping/data:/data
        ports:
          - 80:80 #optional jika sebagai slave
        restart: unless-stopped
   ```

## Troubleshoot

- Cek logs
  ```
  docker logs -f smokeping
  ```

- Binary fping di docker ada di `/usr/sbin/fping`. Jika digunakan sebagai slave kemungkinan akan muncul error karena binary fping pada smokeping biasa berbeda dengan versi docker
  cara mengatasinya cukup dengan membuat symlink pada binary fping
  ```
  ln -s /usr/sbin/fping /usr/bin/fping
  ```

- Tes kembali kedua binary
  ```
  /usr/sbin/fping www.google.com
  ```

  ```
  /usr/bin/fping www.google.com
  ```


  
