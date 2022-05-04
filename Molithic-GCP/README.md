# Monolithic Deployment

## Requirements
- A VM with Debian GNU/Linux 11 (bullseye).
- Public IP.
    - Preferably Static or Elastic (name depends on provider).
- Allow traffic from ports 80 (HTTP) and 443 (HTTPS).
- An Internet conection to that machine (i.e. ssh).
- A user with root privileges (sudo).

## Install Docker and Docker-Compose
```bash
# update system
sudo apt-get update
# install deps
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
# download and add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# add docker's repository to the APT registry
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# install docker engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose
# enable and start docker service
sudo systemctl enable docker --now
# allow user to use docker without root
USER=$(whoami)
sudo usermod -G docker $USER
```
- Log out and back in to reload changes.

## Create docker-compose.yml
- Create a directory for the project in your home dir. `mkdir ~/project`
- CD into it. `cd ~/project`
- Create a `docker-compose.yml` file and paste the following:
    ```yml
    version: '2'
    services:
    mariadb:
        image: docker.io/bitnami/mariadb:10.6
        environment:
        # ALLOW_EMPTY_PASSWORD is recommended only for development.
        - ALLOW_EMPTY_PASSWORD=yes
        - MARIADB_USER=bn_moodle
        - MARIADB_DATABASE=bitnami_moodle
        - MARIADB_CHARACTER_SET=utf8mb4
        - MARIADB_COLLATE=utf8mb4_unicode_ci
        volumes:
        - 'mariadb_data:/bitnami/mariadb'
    moodle:
        image: docker.io/bitnami/moodle:3
        ports:
        - '80:8080'
        - '443:8443'
        environment:
        - MOODLE_DATABASE_HOST=mariadb
        - MOODLE_DATABASE_PORT_NUMBER=3306
        - MOODLE_DATABASE_USER=bn_moodle
        - MOODLE_DATABASE_NAME=bitnami_moodle
        # ALLOW_EMPTY_PASSWORD is recommended only for development.
        - ALLOW_EMPTY_PASSWORD=yes
        volumes:
        - 'moodle_data:/bitnami/moodle'
        - 'moodledata_data:/bitnami/moodledata'
        depends_on:
        - mariadb
    volumes:
    mariadb_data:
        driver: local
    moodle_data:
        driver: local
    moodledata_data:
        driver: local
    ```

## Run
- Run: `docker-compose up -d`
