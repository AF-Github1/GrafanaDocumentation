# GrafanaDocumentation
Self learning Grafana documentation


# Installing Grafana (yum, RHEL 9.2)


Go the the download site

```https://grafana.com/grafana/download```

Get the latest version for RHEL

At the moment of writing of this document this is the command

```sudo yum install -y https://dl.grafana.com/enterprise/release/grafana-enterprise-10.0.3-1.x86_64.rpm```

Use these commands to enable the service on system boot, and to start up the grafana service

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable grafana-server.service
sudo /bin/systemctl start grafana-server.service
```

Afterwards go to ```http://localhost:3000/```

If you are using a remote instance and can’t connect to the local host through your remote terminal, you can just ssh into your instance through your local machine terminal and retry the localhost link.

Something like this command should work

```ssh -i /PATH/keyfile -L 3000:localhost:3000 user@ipORhostname```

Default credentials are:

```User: admin```

```Password: admin```

It will ask you to change the password after logging in


# Setting up Prometheus for Grafana

```Prometheus installation (RHEL)```

Go to the download site of Prometheus and grab the name for the latest version for linux

```https://prometheus.io/download/#prometheus```

Install wget if not already installed

```yum install wget```

Run this command, change version as needed

```wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz```

Create the prometheus user

```useradd --no-create-home --shell /bin/false prometheus```

Create prometheus directories

```mkdir /etc/prometheus /var/lib/prometheus```



Extract file

```tar -xvzf prometheus-2.46.0.linux-amd64.tar.gz```

Change the name of the file

```mv prometheus-2.46.0.linux-amd64 prometheus```

Send prometheus files to /usr/local/bin/

```cp prometheus/prometheus /usr/local/bin/```

```cp prometheus/promtool /usr/local/bin/```


Copy these files over

```cp -r prometheus/consoles /etc/prometheus```

```cp -r prometheus/console_libraries /etc/prometheus```

Create this file

```nano /etc/prometheus/prometheus.yml```

Add this to the yml file
```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus_master'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

```

Create prometheus service file

```nano /etc/systemd/system/prometheus.service```

Add this. Note that this file points to where it expects the other files to be. If you decide to change the location of your files you will need to update your prometheus.service accordingly
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

```

Change all ownerships to the prometheus user

```
chown prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries
chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

Enable, start the service and check if it has launched properly
```
systemctl enable prometheus
systemctl start prometheus
systemctl status prometheus
```
Then much the same way you accessed the grafana server

```http://YOURSERVERIPHERE:9090/```

So something similar to this

```http://35.174.20.227:9090/```

This lets you test if your Prometheus is in fact fully functional

Now lets handle serving your Grafana instance

Go back to ```/etc/prometheus/prometheus.yml```

Add these lines
```
remote_write:
- url: <https://your-remote-write-endpoint>
  basic_auth:
    username: <your user name>
    password: <Your Grafana.com API Key>
```
url should be replaced with ```http://localhost:3000/``` or whichever url you have the Grafana instance hosted on

username if still left by default is admin

For your grafana API key, if you are in a recent version of Grafana (If you just finished installing Grafana you probably are) you will need to use something called Service account instead

Follow the instructions on this link and use the token you generate as the password

https://grafana.com/docs/grafana/latest/administration/service-accounts/

Remember to restart the grafana service after these changes

Afterwards it’s only a matter of creating a new dashboard and using Prometheus as the data source. Choose your host which should be the localhost:9090 and you should be able to start querying for data and making graphs



![Screenshot from 2023-07-26 11-36-21](https://github.com/AF-Github1/GrafanaDocumentation/assets/133685290/fdca0c12-099b-4b98-a896-487c809a3b9f)


# Adding Node exporter

Prometheus will need Node exporter to get some of the information on your machine. 

Check this link to see latest versions for the system you want

```https://prometheus.io/download/#node_exporter```

wget the tar

```wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz```

Extract and open directory

```node_exporter-1.6.1.linux-amd64.tar.gz```

```cd node_exporter-1.6.1.linux-amd64```

Run node exporter

```./node_exporter```


In another terminal window you can check which metrics are being handled

```curl http://localhost:9100/metrics | grep "node_"```

In /etc/prometheus/prometheus.yml add the localhost 9100 below the localhost 9090 line
```
 - targets: ['localhost:9100']
```
Copy your config file over to your prometheus executable location and run this command

```./prometheus --config.file=./prometheus.yml```

You should now be able to query what you couldn’t access before through the Grafana dashboards




# Relevant commands/files


```systemctl edit grafana-server.service```

Opens override.conf file

```/etc/grafana/grafana.ini```

Location for the grafana configuration file.



# Prometheus metrics explorer

In the Prometheus localhost browser you can see the list of available queries that you can use through the metrics explorer. Handy guidebook to look for commands you need.

![Screenshot from 2023-07-27 10-13-39](https://github.com/AF-Github1/GrafanaDocumentation/assets/133685290/554b7776-baff-429d-a8aa-9137506b2b4e)

```up``` Is a particularly useful query that shows which instances are functional. Lets you track when and which instances are being tracked by Prometheus











































































































































































































































































