# homeassistant
help.github.com/articles/basic-writing-and-formatting-syntax/
## Install docker and docker compose
Use built in docker and docker-compose from raspian
```
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker pi
sudo apt install python-pip
sudo pip install --upgrade docker-compose
```
## Create basic structure
```
pi@raspberrypi:/ $ mkdir /var/docker
pi@raspberrypi:/ $ chown pi:pi /var/docker
pi@raspberrypi:/ $ cd /var/docker
pi@raspberrypi:/var/docker $ mkdir config
pi@raspberrypi:/var/docker $ mkdir data
pi@raspberrypi:/var/docker $ mkdir config/grafana
pi@raspberrypi:/var/docker $ mkdir data/grafana
pi@raspberrypi:/var/docker $ mkdir config/mosquitto
pi@raspberrypi:/var/docker $ mkdir data/mosquitto
pi@raspberrypi:/var/docker $ mkdir config/homeassistant
pi@raspberrypi:/var/docker $ mkdir data/homeassistant
pi@raspberrypi:/var/docker $ mkdir config/dockermon
```

### Preload images
```
pi@raspberrypi:/var/docker $ docker-compose pull
```

Create influxdb Config
pi@RPi3:/opt $ docker run --rm influxdb influxd config > /var/docker/config/influxdb/influxdb.conf

Create influxdb (witout passwordprotection"
pi@RPi3:/opt $ docker run --rm -v /var/docker/config/influxdb/influxdb.conf:/etc/influxdb/influxdb.conf -v //var/docker/data/influxdb:/var/lib/influxdb -e INFLUXDB_DB=homeassistant influxdb -config /etc/influxdb/influxdb.conf /init-influxdb.sh


pi@RPi3:/opt $ docker-compose up -d influxdb
Press Control-C after database creation



Mosquitto 

put this to /var/docker/config/mosquitto.conf

persistence true
persistence_location /mosquitto/data/

allow_anonymous true

# Port to use for the default listener.
port 1883

log_dest stdout

#listener 9001
#protocol websockets

pi@RPi3:/opt $ docker-compose up -d mosquitto


homeassistant 

 docker-compose up -d homeassistant #base config creation
 docker-compose down -d homeassistant


add this to /var/docker/config/homassistant
influxdb:
  host: 127.0.0.1
  port: 8086
# uncomment if you used a password
#  username: admin  
#  password: !secret influxdb_password
  database: homeassistant
  default_measurement: state
# exclude stuff that is pointless to log
  exclude: 
    domains:
      - group 
    entities:
      - sensor.other_junk_you_dont_care_about

docker-compose up -d homeassistant


Portainer

docker-compose up -d portainer

2. Tests with docker
   docker run -d \
  --name="home-assistant" --net=host --restart=always \
  -v /home/pi/home-assistant:/config \
  -v /etc/localtime:/etc/localtime:ro \
  homeassistant/raspberrypi2-homeassistant:0.78.0b2



#Autostart docker-compose


```
pi@raspberrypi:/etc/systemd/system $ sudo vi docker-compose-homeassistant.service
```

```
# /etc/systemd/system/docker-compose-homeassistant.service

[Unit]
Description=Docker Compose Opt Service
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/var/docker/config
ExecStart=/usr/local/bin/docker-compose up -d
ExecStop=/usr/local/bin/docker-compose stop
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

```
pi@raspberrypi:/etc/systemd/system $ sudo systemctl enable docker-compose-homeassistant
Created symlink /etc/systemd/system/multi-user.target.wants/docker-compose-homeassistant.service → /etc/systemd/system/docker-compose-homeassistant.service.
pi@raspberrypi:/etc/systemd/system $A
```

A

# MISC
## Differnet IDs for zwave/climate/sensor (Devolo Thermostats)
shutdown HA
edit  .storage/core.entity_registry
## Icons
https://materialdesignicons.com
