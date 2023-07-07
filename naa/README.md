# About
Nasuni Access Anywhere can be extended in order to provide a rich metrics API that can provide a dashboard of system and application metrics.  The dashboard metrics are gathered and displayed using the opensource technology stack of Telegraf-InfluxDB-Grafana. 

# Components
	•	Naamon - Nasuni Access Anywhere monitoring service. This is deployed on each Access Anywhere server in the environment.
	•	Telegraf - agent for collecting, processing, aggregating, and writing metrics.
	•	InfluxDB -  an open-source time series database for saving data.
	•	Grafana - a multi-platform open-source analytics and interactive visualization web application for displaying data.

# Configuring Nasuni Access Anywhere Monitoring API

The monitoring API will be installed from the CLI via the standard package deployments (yum/rpm). 

As the root user on the machine run the following command:
	
```shell
	yum install naamon
```

Once completed, you will need to update the firewalls to allow the default port for the NAA mon api:

	Add the following line into /etc/sysconfig/iptables before the "-j REJECT" line:
```shell	
		-A RH-Firewall-1-INPUT -p tcp -m state --state NEW -m tcp --dport 8010 -j ACCEPT
```

	After added restart the firewall and docker:
```shell
		systemctl restart iptables docker 
```
		
## Configuration File Changes

Naamon leverages the following configuration file: /etc/naa/naamon.hcl

There are one required change that is required for this setup, and other optional changes as well.

### Allowlist IPs

The default config file has just localhost in the allow list as such:

```shell
	allowlist {
	    ip = [
	        "127.0.0.1"
	        ]
	}
```

You will want to add your TIG server, or any other servers/networks that should be allowed to access the apis like so:

```shell
	allowlist {
	    ip = [
	        "127.0.0.1",
		"172.20.25.0/24",
	        "172.20.1.128"
	        ]
	}
```
	
### HTTPS

The default configuration has all API metrics data transferred in plain text (HTTP). If you would like to ensure all data in transit for the API is encrypted you will update the ssl setting, replacing false with true like this example:

```shell
	ssl "true" {
	    certfile = "/etc/ssl/fullchain.pem"
	    keyfile = "/etc/ssl/privkey.pem"
	}
```

Pointing to your relevant ssl files. 
Please note, that as you renew/update your certs you will need to restart the naamon service for the new certificates to be available. 

### Database Replication

The default naamon doesn't check for database replication status. If you're system is setup with a replicated  database,  and you would like to monitor status of the database replication via naamon you will need to update the config file like so:
```shell
	dbrepl "true" {
	    username = "replicationcheckuser"
	    password = "replicationcheckerpasswordhere"
	}
```
	
That user in the configuration will need to be created in the database, and have permissions to check replication status. Here's an example command that will setup the user in the database with the correct permissions: 

```shell
	mysql -e "CREATE USER 'replicationcheckuser'@'localhost' IDENTIFIED BY 'replicationcheckerpasswordhere';"
	mysql -e "GRANT REPLICATION SLAVE ON *.* TO 'replicationcheckuser'@'localhost';"
```
	
# Starting the naamon service

The naamon is installed as a systemd service. To startup the service and enable it please run the following command, after updating the configuration above
	
```shell
	systemctl enable --now naamon
```
	
You can  check the status of the service like so:
	
```shell
	systemctl status naamon
```
	
	
	
	
# Using Telegraf to retrieve naamon statistics

You can use the example TIG buildout as a template. Or if you have an existing TIG installation you will want to add the following configuration: 

```shell
[[inputs.http]]
  interval = "60s"
  data_format="json"
  urls = [
        "https://files.customer.tld:8010/v1/naa/metrics",
  ]
  tag_keys = [
    "systemName",
  ]
  name_override = "naametrics"
```

# Importing NAA dashboard into Grafana

Once you have your grafana system setup, use the [Grafana Dashboard Json](naa/Nasuni Access Anywhere-1685122347925.json) to load the default dasb

# TIG Stack in Docker

To deploy all components in a TIG stack via docker see the docker-compose.yaml in this directory. You would create a path in your system called /opt/tig with a subdirectory for each component like so:

```shell
mkdir -p /opt/tig/telegraf /opt/tig/influx/data /opt/tig/lok /opt/tig/grafana/data
```
Place the [docker-compose.yaml](naa/docker-compose.yaml) in that /opt/tig directory. 

Edit /opt/tig/telegraf/telegraf.conf as required. Then start up the containers like so:

```shell
docker-compose up -d
```