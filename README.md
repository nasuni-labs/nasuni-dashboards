About
=====

Nasuni Performance Monitoring - A modern monitoring architecture.

Components
===========

*   Telegraf is an agent for collecting, processing, aggregating, and writing metrics.
    
*   InfluxDB is an open-source time series database for saving data.
    
*   Grafana is a multi-platform open-source analytics and interactive visualization web application for displaying data.
    

![TIGStackComponents](/images/TIGStackComponents.png)

Sizing Guidelines
==================

InfluxDB
--------

InfluxDB is designed to run-on solid-state drives (SSDs) and memory-optimized instances.

<table data-layout="default" data-local-id="8e403bb5-545e-4876-8e57-082bc184df20" class="confluenceTable"><colgroup><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"><col style="width: 113.33px;"></colgroup><tbody><tr><th class="confluenceTh"><p style="text-align: center;"><strong>vCPU or CPU</strong></p><p style="text-align: center;">(Cores)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>RAM</strong></p><p style="text-align: center;">(GB)</p></th><th class="confluenceTh"><p style="text-align: center;"><strong>IOPS</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Writes per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Queries per second</strong></p></th><th class="confluenceTh"><p style="text-align: center;"><strong>Unique</strong></p><p style="text-align: center;"><strong>series</strong></p></th></tr><tr><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">2-4</p></td><td class="confluenceTd"><p style="text-align: center;">500</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 5</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 100,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">4-6</p></td><td class="confluenceTd"><p style="text-align: center;">8-32</p></td><td class="confluenceTd"><p style="text-align: center;">500-1000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&lt; 1,000,000</p></td></tr><tr><td class="confluenceTd"><p style="text-align: center;">8+</p></td><td class="confluenceTd"><p style="text-align: center;">32+</p></td><td class="confluenceTd"><p style="text-align: center;">1000+</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 250,000</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 25</p></td><td class="confluenceTd"><p style="text-align: center;">&gt; 1,000,000</p></td></tr></tbody></table>

Grafana
-------

*   Grafana does not use a lot of resources and is very lightweight in use of memory and CPU.
    
*   Minimum recommended memory: 255 MB
    
*   Minimum recommended CPU: 1
    

Single Node Installation
-------------------------

*   OS: Rocky Linux 8.5
    
*   Sizing: 4 vCPU, 16GB RAM, and 100GB boot disk
    

Installation
============

Configure SNMP for Nasuni Appliances
------------------------------------

1.  Log in to the Nasuni Management Console.
    
2.  Click Console Settings in the top bar.
    
3.  Click SNMP Monitoring on the left-hand menu under Console Settings.
    
4.  Make sure that Enable v1,v2c Support is set to On.
    
5.  Set Community Name to public (or whatever you plan to use).
    
6.  Set your preferred System Location & System Contact.
    
7.  Click Save SNMP Settings.
    
8.  Click Filers in the top bar.
    
9.  Click SNMP Settings on the left-hand menu under Filer Settings.
    
10.  For each filer that should be monitored, select the filer(s) (Note: you can choose multiple filers if all values are shared).
    
11.  Click Edit {x} Filer button in the upper right.
    
12.  Make sure that Enable v1, v2c Support is set to On.
    
13.  Set Community Name (default is “public”).
    
14.  Set your preferred System Location & System Contact.
    
15.  Click Save SNMP Settings.
    

Install & Configure Influx DB
-----------------------------

To install InfluxDB, ssh to the VM and run the following commands:

1.  Update all packages:
    
    ```shell
    sudo yum update
    ```
    
2.  Install wget (if not present):
    
    ```shell
    sudo yum install wget
    ```
    
3.  Use wget to download InfluxDB:
    
    ```shell
    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.9.x86_64.rpm
    ```
    
4.  Install InfluxDB:
    
    ```shell
    sudo yum localinstall influxdb-1.7.9.x86_64.rpm
    ```
    
5.  Start and enable the InfluxDB service:
    
    ```shell
    sudo systemctl start influxdb && sudo systemctl enable influxdb
    ```
    
6.  Configure the firewall for InfluxDB (only required if firewall is enabled):
    
    ```shell
    sudo firewall-cmd --add-port=8086/tcp --permanent
    ```
    
7.  Reload the firewall configuration (only required if firewall is enabled:
    
    ```shell
    sudo firewall-cmd --reload
    ```
    
8.  Create a new InfluxDB database:
    
    1.  Open the InfluxDB shell:
        
        ```shell
        influx
        ```
        
    2.  Create the database for use by Grafana.
        
        ```shell
        create database nasuni
        ```
        
    3.  Confirm that the database was created:
        
        ```shell
        show databases
        ```
        
    4.  Confirm that “nasuni” is listed.

    5.  Exit InfluxDB shell:

        ```shell
        exit
        ```
        

Install & Configure Telegraf
----------------------------

1.  Use wget to download Telegraf:
    
    ```shell
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.13.0-1.x86_64.rpm
    ```
    
2.  Install Telegraf:
    
    ```shell
    sudo yum localinstall telegraf-1.13.0-1.x86_64.rpm
    ```
    
3.  Install SNMP dependencies for Telegraf:
    
    ```shell
    sudo yum -y install net-snmp net-snmp-utils
    ```
    

### Add Nasuni MIB

1.  Go to NMC > Filers > SNMP Settings.
    
2.  Click the NASUNI-FILER-MIB link to open the file.
    
3.  Select all & copy text.
    
4.  Return to TIG server.
    
5.  Navigate to the mibs directory:
    
    ```python
    cd /usr/share/snmp/mibs
    ```
    
6.  Create a new file for the Nasuni MIB:
    
    ```shell
    sudo vi NASUNI-FILER-MIB.txt
    ```
    
7.  Paste the text copied in step 3.
    
8.  Save and close the file.
    
    1.  Press Esc key, type “:x” and hit Enter
        
9.  Repeat steps 7-13 for the remaining MIBs listed in the NMC (the other MIBs may already be present. No need to recopy them if this is the case).
    

### Setup Telegraf Configuration

1.  Backup the default telegraf configuration file
    ```shell
    sudo mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.bak
    ```

2.  Download [telegraf.conf](telgegraf.conf) from Nasuni Labs and copy it to /etc/telegraf/telegraf.conf
    
3.  Customize telegraf.conf for your environment.
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
    1.  Update OUTPUTS section with the InfluxDB IP address & confirm the port.
        
    2.  Update INPUTS agents with the FQDN of all filers to be monitored.
        
        1.  Found under: “Retrieves SNMP values from remote agents”.
            
    3.  Update INPUTS community with the community name defined in NMC SNMP settings.
        
        1.  Found under: “SNMP version; can be 1, 2, or 3”.
            
    4.  Update INPUTS country with the desired company name and region.
        
4.  Save and close the file.
    
    1.  Press Esc key, type “:x” and hit Enter.
        
5.  Start/Restart the service:
    
    ```shell
    sudo systemctl restart telegraf
    ```
    
6.  Confirm the service is running. Look for “Active: active (running)”:
    
    ```shell
    systemctl status telegraf
    ```
    

Install & Configure Grafana
---------------------------

1.  Use wget to download Grafana:
    
    ```shell
    wget wget https://dl.grafana.com/oss/release/grafana-8.4.4-1.x86_64.rpm
    ```
    
2.  Install Grafana:
    
    ```shell
    sudo yum install grafana-8.4.4-1.x86_64.rpm
    ```
    
3.  Start the Grafana service:
    
    ```shell
    sudo service grafana-server start
    ```
    
4.  Configure the firewall for Grafana (only required if firewall is enabled):
    
    ```shell
    firewall-cmd --permanent --zone=public --add-port=3000/tcp
    firewall-cmd --reload
    ```
    
5.  Install the following Grafana plugin:
    
    ```shell
    sudo grafana-cli plugins install grafana-clock-panel
    ```
    
6.  Restart the Grafana service:
    
    ```shell
    sudo service grafana-server restart
    ```
    
7.  Set Grafana to run on startup:
    
    ```shell
    sudo systemctl enable grafana-server
    ```
    

### Configure Grafana Data Source

1.  Use a web browser to connect to Grafana (http://fqdn:3000).
    
2.  The default username is “admin”, and the default password is “admin”. Grafana will prompt you to change the default password after logging in.
    
3.  Under the Configuration (gear) icon, add InfluxDB as a data source using the details from your install and test the connection. For the default single node installation use the following information:

    URL: http://localhost:8086 <br/>
    Database: nasuni
    
![Add Data Source](/images/AddGrafanaDataSource.png)

4.  Click "Save & Test" to complete add the InfluxDB data source.

### Configure Grafana Dashboard

1.  Download the [PerformanceDashboard.json](PerformanceDashboard.json) dashboard template from Nasuni Labs

2.  From the Grafana left navigation, click the Dashboard icon (four squares), then click the "Browse" link. On the Browse page, click "Import" and upload the JSON file downloaded in step 1.

3.  Navigate the newly-imported dashboard to view the performance information for your Edge Appliances. The dashboard should display the following information:
![Performance Dashboard Screenshot](/images/PerformanceDashboardScreenshot.png)



Maintenance Tasks
=================

**Removing Stale Nasuni Edge Appliances**
-----------------------------------------

In order to save disk space it is good to cleanup the InfluxDB database when you discard a Nasuni Edge Appliance.

1.  Remove Nasuni Edge Appliance IP or FQDN from Telegraf:
    
    ```shell
    sudo vi /etc/telegraf/telegraf.conf
    ```
    
2.  Start/Restart the service:
    
    ```shell
    systemctl restart telegraf
    ```
    
3.  Confirm the service is running. Look for “Active: active (running)”:
    
    ```shell
    systemctl status telegraf
    ```
    
4.  Open the InfluxDB shell:
    
    ```shell
    influx
    ```
    
5.  Identify the correct database for Nasuni:
    
    ```shell
    show databases
    ```
    
6.  Access the database:
    
    ```shell
    use <database_name>
    ```
    
7.  Display
    
    ```shell
    show series
    ```
    
8.  Identify the agent\_host to delete.
    
9.  Delete all series for the agent\_host:
    
    ```shell
    drop series where "agent_host" = <IP or FQDN>
    ```
    
10. Verify if the command was successful:
    
    ```shell
    show series
    ```

11. Exit
    
    ```shell
    exit
    ```
    
    
    

Support Statement
=================

*   This package stack has only been validated with the component versions documented in the README file.
    
*   Nasuni Support is limited to Nasuni APIs and Protocols (SMB, NFS, SNMP).
    
*   Nasuni API and Protocol bugs or feature requests should be communicated to Nasuni Customer Success.
    
*   GitHub project to-do's, bugs, and feature requests, should be submitted as “Issues” in GitHub under its repositories.
    

