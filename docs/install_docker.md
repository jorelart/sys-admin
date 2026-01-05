


# Install Docker Engine Ubuntu 22.04

1. Hapus Paket yang berkemungkinan konflik

   ```
   sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
   ```

2. Tambahkan Repository resmi docker

   ```
   # Add Docker's official GPG key:
    sudo apt update
    sudo apt install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    
    # Add the repository to Apt sources:
    sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
    Types: deb
    URIs: https://download.docker.com/linux/ubuntu
    Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
    Components: stable
    Signed-By: /etc/apt/keyrings/docker.asc
    EOF
    
    sudo apt update
   ```

3. Install docker terbaru

   ```
   sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

4. Cek instalasi

   ```
   sudo systemctl status docker
   ```

### Troubleshoot
Docker versi 2.91 memiliki masalah pada containerd v2.0.0, jika menemukan serupa lakukan downgrade pada containerd.io ke versi 1.7

- Cek versi containerd.io
  ```
  containerd --version
  ```

- Downgrade ke versi 1.7
  Cari dulu versi yang tersedia
  
  ```
  apt-cache madison containerd.io
  ```

  Install downgrade ke versi sebelumnya

  ```
  sudo apt install --allow-downgrades -y containerd.io=1.7.28-0ubuntu1~22.04.1~jammy
  ```
- Cegah auto update (hold version)
  ini penting supaya tidak diupgrade lagi (akan error)
  ```
  sudo apt-mark hold containerd.io
  ```

- Start ulang service
  ```
  sudo systemctl daemon-reexec
  sudo systemctl daemon-reload
  sudo systemctl start containerd
  sudo systemctl start docker
  ```
