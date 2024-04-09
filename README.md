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

### Move Mount for docker/podman to seperate disk
Create/Modify /etc/docker/daemon.conf where path is your new disk
```
cat /etc/docker/daemon.conf<<EOF
{
  "data-root": "/u01/docker"
}
EOF
```

If you already created some containers, after moving them you need to update config.v2.json file 
```
sudo perl -pi -e 's%/var/lib/docker%/u01/var/lib/docker%g' `find /u01/var/lib/docker -name "config.v2.json"`
```

### Pull in docker/podman yaml (Need to test if this works with podman)
```
git pull https://github.com/fernandesselwyn/proxmox-collector.git
```

### Setup docker/podman network for these containers
```
docker network create guest_network
```

### Setup docker/podman volume for presistent storage
```
docker volume create influxdb-volume
docker volume create grafana-volume
```

### docker/podman compose the yaml (make sure both the docker-compose.yml and telegraf.conf are in the folder)
```
docker-compose up -d
```

### Configure InfluxDB, create API token
Log into https://<server_ip>:8086
Set organization name, user, password
Capture API Token to influx_t.txt

Get and configure influxCLI
```
cd /tmp
wget https://dl.influxdata.com/influxdb/releases/influxdb2-client-2.7.3-linux-amd64.tar.gz
sudo tar xvzf ./influxdb2-client-2.7.3-linux-amd64.tar.gz
sudo cp ./influx /usr/local/bin/
export admin_token=(`cat influx_t.txt`)
influx config create --config-name proxmox --host-url http://localhost:8086 --org daycollector --token ${admin_token} --active
```

Create second user 
If you skip getting the local CLI you can pass it via docker exec. (same command if you did it cli version)
```
sudo docker exec -it influxdb_container influx user create \
--org <org-name>
--name readonly \
--password <some_password> \
--t <admin_token>

sudo docker exec -it influxdb_container influx user create \
> --org <org-name>
> --name readonly \
> --password <some_password> \
> -t <admin_token>
```
This can be done via the GUI http://<your-fqdn>:8086

### Configure Proxmox
Techinically with just this we can start sending metric data
Datacenter>Metric Server>Add>InfluxDB
Name: <some_name>
Server: <IP_Address>
Port: 8086 (This is default/per docker config)
Protocol: HTTP
Organization: <org-name>
Bucket: <named_bucket> (I went with proxmox but may want a ## iteration to possibly add a second instance)
Token: <new_token> OR <admin_token> (not recommended)

From influxDB you should see the bucket created
```
influx bucket ls
```

### Configure Grafana, add datasource


### Import Grafana dashboard
