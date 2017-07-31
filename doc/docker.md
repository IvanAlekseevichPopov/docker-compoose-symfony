## Docker installation

1. Install docker and docker-compose packages for your OS.
    1. Ubuntu/Debian:

    ```bash
       sudo apt-get install docker docker-compose
    ```
    2. Fedora/Centos:

    ```bash
       sudo dnf install docker docker-compose
    ```

2. For Ubuntu/Debian you need to fill /etc/resolv.conf with actual DNS servers
(it needs for correct work of docker internal network)

    ```bash
       sudo bash -c 'echo "nameserver 8.8.8.8" >> /etc/resolv.conf'
       #Just example. Replace 8.8.8.8 with your dns server
    ```
    
3. Add your user to docker group for access to docker commands without sudo:

    ```bash
       sudo groupadd docker
       sudo usermod -aG docker $USER
    ```

4. Enable docker service:

    ```bash
       sudo systemctl enable docker
       sudo systemctl start docker
    ```
    
5. Log out and log back in so that your group membership is re-evaluated.