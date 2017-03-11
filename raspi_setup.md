# Common settings

```
$ sudo raspi-config
```

```
2 -> change pwd
7 -> A4 -> enable ssh

4 -> I1 -> change to de_DE.utf8
4 -> I2 -> change to Europe -> Berlin
```

```
$ sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
```

```
network={
    ssid="cp-pro_wlan_og"
    psk="cpWLAN07pro%"
}

network={
    ssid="JustAnotherWlan"
    psk="madwlan2go"
}
```

# Update raspian
```
sudo apt-get update && sudo apt-get upgrade -y
```

```
sudo apt-get install python3 python3-venv python3-pip -y
```

```
sudo apt-get install bluetooth libbluetooth-dev -y
```

```
sudo apt-get install libxslt-dev libxml2-dev python3-lxml -y
```

# Home-assistant
```
sudo useradd -rm homeassistant
cd /srv
sudo mkdir homeassistant
sudo chown homeassistant:homeassistant homeassistant
sudo su -s /bin/bash homeassistant 
cd /srv/homeassistant
python3 -m venv homeassistant_venv
```

```
source /srv/homeassistant/homeassistant_venv/bin/activate
```

```
pip3 install homeassistant
```

## Autostart
```
$ sudo vi /etc/systemd/system/homeassistant@homeassistant.service
```

```
[Unit]
Description=Home Assistant
After=network.target

[Service]
Type=simple
User=%i
ExecStartPre=source /srv/homeassistant/homeassistant_venv/bin/activate
ExecStart=/srv/homeassistant/homeassistant_venv/bin/hass -c "/home/%i/.homeassistant"

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl --system daemon-reload
sudo systemctl enable homeassistant@homeassistant
sudo systemctl start homeassistant@homeassistant
```

# influxdb
```
wget https://dl.influxdata.com/influxdb/releases/influxdb_1.1.1_armhf.deb
sudo dpkg -i influxdb_1.1.1_armhf.deb
```

```
$ sudo vi /etc/influxdb/influxdb.conf
```

* comment out `reporting-disabled = true`

```
sudo systemctl --system daemon-reload
sudo systemctl enable influxdb
sudo systemctl start influxdb

sudo systemctl status influxdb
```

```
$ influx
```

```
CREATE DATABASE homeassistant
CREATE USER "homeassistant" WITH PASSWORD '16Smiler'
```


# grafana
```
$ sudo apt-get install libfontconfig -y
```

```
wget https://github.com/fg2it/grafana-on-raspberry/releases/download/v4.0.2/grafana_4.0.2-1481228559_armhf.deb
sudo dpkg -i grafana_4.0.2-1481228559_armhf.deb
```

```
sudo systemctl --system daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

sudo systemctl status grafana-server
```

# homebridge
```
wget https://nodejs.org/dist/v4.6.0/node-v4.6.0-linux-armv7l.tar.gz 
tar -xvf node-v4.6.0-linux-armv7l.tar.gz 
cd node-v4.6.0-linux-armv7l
sudo cp -R * /usr/local/
cd
````

```
$ sudo apt-get install libavahi-compat-libdnssd-dev -y
````

```
sudo npm install -g --unsafe-perm homebridge
sudo npm install -g homebridge-homeassistant
```

```
sudo vi /etc/default/homebridge
```

```
# Defaults / Configuration options for homebridge
# The following settings tells homebridge where to find the config.json file and where to persist the data (i.e. pairing and others)
HOMEBRIDGE_OPTS=-U /var/homebridge

# If you uncomment the following line, homebridge will log more 
# You can display this via systemd's journalctl: journalctl -f -u homebridge
# DEBUG=*
```

```
sudo vi /etc/systemd/system/homebridge.service
```

```
[Unit]
Description=Node.js HomeKit Server 
After=syslog.target network-online.target

[Service]
Type=simple
User=homebridge
EnvironmentFile=/etc/default/homebridge
ExecStart=/usr/local/bin/homebridge $HOMEBRIDGE_OPTS
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```


```
sudo useradd --system homebridge
sudo mkdir /var/homebridge
sudo vi /var/homebridge/config.json
```

```
{
    "bridge": {
        "name": "Homebridge",
        "username": "CC:22:3D:E3:CE:30",
        "port": 51826,
        "pin": "031-45-154"
    },

    "accessories": [
    ],

    "platforms": [
        {
            "platform": "HomeAssistant",
            "name": "HomeAssistant",
            "host": "http://127.0.0.1:8123",
            "password": "16Smiler",
            "supported_types": ["binary_sensor", "cover", "fan", "input_boolean", "lock", "media_player", "scene", "sensor", "switch"]
        }
    ]
}
``` 

```
sudo chown -R homebridge:homebridge /var/homebridge
```

```
sudo su -s /bin/bash homebridge
homebridge -U /var/homebridge
```

```
sudo systemctl daemon-reload
sudo systemctl enable homebridge
sudo systemctl start homebridge

sudo systemctl status homebridge
```



## Snippets
```
sudo systemctl restart homeassistant@homeassistant && sudo journalctl -f -u homeassistant@homeassistant

sudo systemctl status homeassistant@homeassistant -l
sudo journalctl -f -u homeassistant@homeassistant
sudo journalctl -f -u homeassistant@homeassistant -n 2000 | grep -i 'error'

```

* change ssh key

  `ssh-keygen -R 10.0.1.5`


# update ha
sudo su -s /bin/bash homeassistant
source /srv/homeassistant/homeassistant_venv/bin/activate
pip3 install --upgrade homeassistant
