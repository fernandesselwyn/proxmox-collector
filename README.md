# proxmox-collector
Proxmox (TIG) export stats with Influx, Grafana and Telegraf 

### Install docker (APT)
```
### Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Install docker (YUM)
```
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
```

### Pull in docker yaml (Need to test if this works with podman)
git pull https://github.com/fernandesselwyn/proxmox-collector.git

### Setup docker network for these containers
docker network create guest_network

### Setup docker volume for presistent storage
docker volume create influxdb-volume
docker volume create grafana-volume

### Docker compose the yaml (make sure both the docker-compose.yml and telegraf.conf are in the folder)
docker-compose up -d

### Configure InfluxDB, create API token

### Configure Grafana, add datasource

### Configure Proxmox

### Import Grafana dashboard
