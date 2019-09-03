# Consul Setup

Consul Setup for 3 node cluster, where each node has Ubuntu 16.04 installed.

## 1. Download consul on all nodes

First, on all nodes export variables to be called:

```
export node1='internal ip of node1'
export node2='internal ip of node2'
export node3='internal ip of node3'
```

Next, run the following commands on all nodes:

```
sudo apt install unzip
export VER="1.5.1"
wget https://releases.hashicorp.com/consul/${VER}/consul_${VER}_linux_amd64.zip
unzip consul_${VER}_linux_amd64.zip
sudo mv consul /usr/local/bin/
sudo groupadd --system consul
sudo useradd -s /sbin/nologin --system -g consul consul
sudo mkdir -p /var/lib/consul
sudo chown -R consul:consul /var/lib/consul
sudo chmod -R 775 /var/lib/consul
sudo mkdir /etc/consul.d
sudo chown -R consul:consul /etc/consul.d
sudo sh -c "echo $node1 'consul-01.example.com consul-01' >> /etc/hosts"
sudo sh -c "echo $node2 'consul-02.example.com consul-02' >> /etc/hosts"
sudo sh -c "echo $node3 'consul-03.example.com consul-03' >> /etc/hosts"
```

Run `consul -v` to verify installation.

## 2. Edit consul service

Next, on node1 edit the consul service file:

`sudo vim /etc/systemd/system/consul.service`

```
[Unit]
Description=Consul Service Discovery Agent
Documentation=https://www.consul.io/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent -server -ui \
	-advertise=<internal ip of node1> \
	-bind=<internal ip of node1> \
	-data-dir=/var/lib/consul \
	-node=consul-01 \
	-config-dir=/etc/consul.d

ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure
SyslogIdentifier=consul

[Install]
WantedBy=multi-user.target
```

Next, on node2 and node3 edit the consul service as follows:

`sudo vim /etc/systemd/system/consul.service`

```
[Unit]
Description=Consul Service Discovery Agent
Documentation=https://www.consul.io/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent \
	-node=consul-01 \
	-config-dir=/etc/consul.d

ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure
SyslogIdentifier=consul

[Install]
WantedBy=multi-user.target
```

## 3. Generate consul keys

On node1 generate the keys:

`consul keygen`

And run the following on each node, replacing the internal ip with the internal ip of the node you are running the commands on:

`sudo vim /etc/consul.d/config.json`

```
{
    "advertise_addr": "<internal ip of node1>",
    "bind_addr": "<internal ip of node1>",
    "bootstrap_expect": 3,
    "client_addr": "0.0.0.0",
    "datacenter": "KE-DC",
    "data_dir": "/var/lib/consul",
    "domain": "consul",
    "enable_script_checks": true,
    "dns_config": {
        "enable_truncate": true,
        "only_passing": true
    },
    "enable_syslog": true,
    "encrypt": "<genereated key>",
    "leave_on_terminate": true,
    "log_level": "INFO",
    "rejoin_after_leave": true,
    "retry_join": [
    	"consul-01",
    	"consul-02",
    	"consul-03"
    ],
    "server": true,
    "start_join": [
        "consul-01",
        "consul-02",
        "consul-03"
    ],
    "ui": true
}
```

On node2 run:

`sudo vim /etc/systemd/system/consul.service`
```
[Unit]
Description=Consul Service Discovery Agent
Documentation=https://www.consul.io/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent \
	-node=consul-02 \
	-config-dir=/etc/consul.d

ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure
SyslogIdentifier=consul

[Install]
WantedBy=multi-user.target
```

Again on node2 run:

`sudo vim /etc/consul.d/config.json`

```
{
    "advertise_addr": "internal ip of node2",
    "bind_addr": "<internal ip of node2",
    "bootstrap_expect": 3,
    "client_addr": "0.0.0.0",
    "datacenter": "KE-DC",
    "data_dir": "/var/lib/consul",
    "domain": "consul",
    "enable_script_checks": true,
    "dns_config": {
        "enable_truncate": true,
        "only_passing": true
    },
    "enable_syslog": true,
    "encrypt": "<generated password>",
    "leave_on_terminate": true,
    "log_level": "INFO",
    "rejoin_after_leave": true,
    "retry_join": [
    	"consul-01",
    	"consul-02",
    	"consul-03"
    ],
    "server": true,
    "start_join": [
        "consul-01",
        "consul-02",
        "consul-03"
    ],
    "ui": true
}
```

On node3 run:

`sudo vim /etc/systemd/system/consul.service`
```
[Unit]
Description=Consul Service Discovery Agent
Documentation=https://www.consul.io/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent \
	-node=consul-03 \
	-config-dir=/etc/consul.d

ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure
SyslogIdentifier=consul

[Install]
WantedBy=multi-user.target
```
Again on node3 run:

`sudo vim /etc/consul.d/config.json`

```
{
    "advertise_addr": "internal ip of node3",
    "bind_addr": "<internal ip of node3",
    "bootstrap_expect": 3,
    "client_addr": "0.0.0.0",
    "datacenter": "KE-DC",
    "data_dir": "/var/lib/consul",
    "domain": "consul",
    "enable_script_checks": true,
    "dns_config": {
        "enable_truncate": true,
        "only_passing": true
    },
    "enable_syslog": true,
    "encrypt": "<generated password>",
    "leave_on_terminate": true,
    "log_level": "INFO",
    "rejoin_after_leave": true,
    "retry_join": [
    	"consul-01",
    	"consul-02",
    	"consul-03"
    ],
    "server": true,
    "start_join": [
        "consul-01",
        "consul-02",
        "consul-03"
    ],
    "ui": true
}
```

## 4. Start consul on all nodes

Run the following commands on all nodes:

```
sudo systemctl start consul
sudo systemctl enable consul
```

Finally, view members with:

```
consul members
```
